# System Design Document: Conceptly – Autonomous Learning Agent

## AI for Bharat Hackathon | Student Track: AI for Learning & Developer Productivity

---

## 1. System Overview

### 1.1 High-Level Description
Conceptly is a **Structured AI Learning Engine** that orchestrates personalized learning experiences through intelligent workflows. Unlike traditional AI chatbots that simply generate responses, Conceptly implements a rigorous, state-controlled learning pipeline that:

1. **Validates content quality** before teaching (Context Quality ≥ 85%)
2. **Evaluates understanding semantically** through targeted assessments
3. **Adapts teaching dynamically** using Feynman Technique when concepts aren't grasped
4. **Orchestrates workflows reliably** with LangGraph state machines
5. **Gamifies learning** to maintain engagement and motivation

### 1.2 Key Innovations

**Innovation 1: Validation-First Learning**
- AI-generated content is validated for quality before being presented to learners
- Ensures reliability and prevents misinformation/hallucinations
- Automatic retry mechanism if validation fails

**Innovation 2: State Machine Orchestration**
- Learning workflow implemented as deterministic state machine using LangGraph
- Predictable progression through: context gathering → validation → teaching → assessment
- Traceable execution for debugging and optimization

**Innovation 3: Semantic Understanding Evaluation**
- Goes beyond keyword matching to assess true conceptual understanding
- Identifies specific weak areas for targeted improvement
- Understanding Score threshold (70%) ensures mastery before progression

**Innovation 4: Adaptive Feynman Re-Teaching Loop**
- When learners struggle, system applies Feynman Technique automatically
- Explains prerequisite concepts before re-teaching weak areas
- Rotates through 5 different teaching approaches for varied learning


**Innovation 5: Gamification for Sustained Engagement**
- XP, levels, badges, and streaks drive motivation
- Progress analytics provide transparency and goal-setting
- Daily challenges maintain consistent learning habits

### 1.3 System Goals
- **Reliability**: Validated AI content, deterministic workflows, error handling
- **Adaptability**: Personalized explanations, targeted re-teaching, tutor personalities
- **Engagement**: Gamification, progress tracking, achievement recognition
- **Scalability**: Modular architecture, horizontal scaling, efficient resource usage
- **Observability**: LangSmith tracing, structured logging, performance monitoring

---

## 2. System Architecture

### 2.1 Architecture Layers

```
┌─────────────────────────────────────────────────────────────────┐
│                     PRESENTATION LAYER                          │
│  React + Vite Frontend (Dashboard, Session, Quiz, Analytics)    │
└────────────────────────┬────────────────────────────────────────┘
                         │ HTTPS/REST API
┌────────────────────────┴────────────────────────────────────────┐
│                        API LAYER                                │
│  FastAPI (Auth, Sessions, Checkpoints, Analytics, Gamification) │
└────────────────────────┬────────────────────────────────────────┘
                         │
┌────────────────────────┴────────────────────────────────────────┐
│                   ORCHESTRATION LAYER                           │
│  LangGraph Workflows (State Machine for Learning Pipeline)      │
└────────────┬──────────────────────────────┬─────────────────────┘
             │                              │
┌────────────┴────────────────┐  ┌──────────┴─────────────────────┐
│   AI INTERACTION LAYER      │  │   OBSERVABILITY LAYER          │
│  LangChain + Groq LLM       │  │  LangSmith Tracing             │
│  (Llama 3.3 70B Versatile)  │  │  (Workflow Debugging)          │
└─────────────────────────────┘  └────────────────────────────────┘
                         │
┌────────────────────────┴────────────────────────────────────────┐
│                        DATA LAYER                               │
│  PostgreSQL (Users, Sessions, Checkpoints, Analytics, Badges)   │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Component Descriptions

**Presentation Layer (React + Vite)**
- User interface for all learning interactions
- Pages: Landing, Login, Register, Dashboard, Session, Quiz, Feynman, Completion, History, Analytics
- Components: Navbar, SessionCard, ProgressBar, BadgeCard, Sidebar
- State management via Context API (AuthContext)
- Responsive design for mobile, tablet, desktop

**API Layer (FastAPI)**
- RESTful API endpoints for all operations
- Routers: auth, sessions, checkpoints, analytics, gamification
- JWT-based authentication and authorization
- Request validation with Pydantic schemas
- CORS middleware for frontend communication
- Error handling and logging

**Orchestration Layer (LangGraph)**
- State machine workflow for checkpoint learning
- Nodes: gather_context, validate_context, explain, generate_questions
- Conditional edges for validation retry logic
- State management across workflow execution
- Traceable via LangSmith for debugging

**AI Interaction Layer (LangChain + Groq)**
- LangChain for LLM interaction abstraction
- Groq API for fast inference (Llama 3.3 70B Versatile model)
- Structured prompts for consistent output
- Temperature tuning for creativity vs. consistency
- Fallback mechanisms for API failures

**Observability Layer (LangSmith)**
- Traces all LangGraph workflow executions
- Captures: state transitions, node execution times, LLM calls, errors
- Enables debugging and performance optimization
- Retention: 30 days of trace data

**Data Layer (PostgreSQL)**
- Relational database for all persistent data
- SQLAlchemy ORM for database interactions
- Models: User, LearningSession, Checkpoint, QuizAttempt, UserAnalytics, UserBadge, WeakTopic, DailyChallenge, UserNote
- Foreign key relationships for data integrity
- Indexes for query performance

---

## 3. Workflow Description

### 3.1 Complete Learning Workflow

```
User Creates Session
        ↓
System Generates Checkpoints
        ↓
User Starts Checkpoint
        ↓
┌───────────────────────────────────────────────────────────────┐
│              LANGGRAPH WORKFLOW EXECUTION                     │
│                                                               │
│  [1] gather_context                                           │
│      ↓                                                        │
│  [2] validate_context                                         │
│      ├─ Score ≥ 85% → proceed                                 │
│      └─ Score < 85% → retry gather_context (max 2 attempts)   │
│      ↓                                                        │
│  [3] explain (generate personalized explanation)              │
│      ↓                                                        │
│  [4] generate_questions (create assessment)                   │
│                                                               │
└───────────────────────────────────────────────────────────────┘
        ↓
User Reads Explanation
        ↓
User Takes Quiz
        ↓
System Evaluates Answers
        ↓
┌─────────────────────────────────────────────────────────────┐
│                    EVALUATION LOGIC                         │
│                                                             │
│  Understanding Score = (Correct / Total) × 100%             │
│                                                             │
│  IF Score ≥ 70%:                                            │
│    → Checkpoint PASSED                                      │
│    → Award XP (proportional to score)                       │
│    → Update analytics                                       │
│    → Check for badges                                       │
│    → Move to next checkpoint                                │
│                                                             │
│  IF Score < 70%:                                            │
│    → Checkpoint FAILED                                      │
│    → Identify weak areas                                    │
│    → Trigger Feynman re-teaching                            │
│    → Generate new questions targeting weak areas            │
│    → Allow retry (max 3 attempts)                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
        ↓
All Checkpoints Completed
        ↓
Session Marked Complete
        ↓
Award Session XP & Badges
```


### 3.2 Context Validation Threshold Logic

```python
def should_retry_context(state: LearningState) -> str:
    """
    Determines whether to retry context gathering based on validation score
    
    Threshold: Context Quality Score ≥ 85%
    Max Retries: 2 attempts
    """
    if state.get('context_validated', False):
        return "proceed"  # Already validated, move forward
    
    validation_score = state.get('validation_score', 0)
    
    if validation_score >= 85:
        state['context_validated'] = True
        return "proceed"  # Passed validation
    
    # If score < 85 and retries available, retry
    # Otherwise, proceed with current context (fallback)
    return "retry" if can_retry(state) else "proceed"
```

**Why 85% Threshold?**
- Ensures high-quality, accurate content for learners
- Balances quality with system efficiency (not too strict)
- Prevents misinformation and AI hallucinations
- Based on empirical testing for optimal learning outcomes

### 3.3 Understanding Score Evaluation

```python
def evaluate_answers(questions: List[Dict], answers: List[str]) -> Dict:
    """
    Evaluates user answers semantically
    
    Returns:
    - understanding_score: 0.0 to 1.0 (70% = 0.7 required to pass)
    - weak_areas: List of concepts where user struggled
    - detailed_results: Per-question feedback
    """
    correct_count = 0
    weak_areas = []
    
    for i, question in enumerate(questions):
        user_answer = normalize_answer(answers[i])
        correct_answer = normalize_answer(question['correct_answer'])
        
        if user_answer == correct_answer:
            correct_count += 1
        else:
            # Identify weak concept
            weak_areas.append(question['tested_concept'])
    
    understanding_score = correct_count / len(questions)
    passed = understanding_score >= 0.70
    
    return {
        'understanding_score': understanding_score,
        'correct_count': correct_count,
        'total_questions': len(questions),
        'passed': passed,
        'weak_areas': list(set(weak_areas))  # Unique weak areas
    }
```

**Why 70% Threshold?**
- Industry standard for demonstrating competency
- Ensures learner has grasped majority of concepts
- Allows for minor mistakes while maintaining rigor
- Aligns with academic passing standards

### 3.4 Feynman Re-Teaching Process

```
Failed Assessment (Score < 70%)
        ↓
Identify Weak Areas (concepts from incorrect answers)
        ↓
┌────────────────────────────────────────────────────────────┐
│              FEYNMAN TECHNIQUE APPLICATION                 │
│                                                            │
│  Step 1: Identify Prerequisites                            │
│    - What concepts must be understood first?               │
│    - What foundational knowledge is missing?               │
│                                                            │
│  Step 2: Explain Prerequisites Simply                      │
│    - Use analogies and everyday examples                   │
│    - Break down into smallest components                   │
│                                                            │
│  Step 3: Re-Teach Weak Areas                               │
│    - Use one of 5 teaching approaches (rotated):           │
│      1. Everyday analogies & real-world examples           │
│      2. Step-by-step breakdown with visual descriptions    │
│      3. Storytelling and narrative explanation             │
│      4. Question-answer format with guided reasoning       │
│      5. Comparison with familiar concepts & metaphors      │
│                                                            │
│  Step 4: Connect Prerequisites to Weak Areas               │
│    - Show how foundational concepts relate                 │
│    - Build understanding incrementally                     │
│                                                            │
│  Output: 500-800 word re-explanation focused on weak areas │
│                                                            │
└────────────────────────────────────────────────────────────┘
        ↓
User Reads Feynman Explanation
        ↓
Generate New Questions Targeting Weak Areas
        ↓
User Retries Assessment
        ↓
Evaluate Again (repeat if needed, max 3 attempts)
```

**Teaching Approach Rotation Example:**
- Attempt 1 fails → Use "everyday analogies"
- Attempt 2 fails → Use "step-by-step breakdown"
- Attempt 3 fails → Use "storytelling"
- This ensures varied learning styles are addressed

---

## 4. Data Model

### 4.1 Entity Relationship Diagram

```
┌──────────────┐
│    User      │
├──────────────┤
│ id (PK)      │
│ email        │
│ password_hash│
│ name         │
│ tutor_mode   │
│ xp           │
│ level        │
│ created_at   │
└──────┬───────┘
       │ 1
       │
       │ N
┌──────┴──────────────┐
│  LearningSession    │
├─────────────────────┤
│ id (PK)             │
│ user_id (FK)        │
│ topic               │
│ user_notes          │
│ status              │
│ created_at          │
│ completed_at        │
│ xp_earned           │
└──────┬──────────────┘
       │ 1
       │
       │ N
┌──────┴──────────────┐
│    Checkpoint       │
├─────────────────────┤
│ id (PK)             │
│ session_id (FK)     │
│ checkpoint_index    │
│ topic               │
│ objectives (JSON)   │
│ key_concepts (JSON) │
│ level               │
│ status              │
│ understanding_score │
│ attempts            │
│ context (Text)      │
│ explanation (Text)  │
│ questions_cache JSON│
│ validation_score    │
│ content_generated   │
│ completed_at        │
│ xp_earned           │
└──────┬──────────────┘
       │ 1
       │
       │ N
┌──────┴──────────────┐
│    QuizAttempt      │
├─────────────────────┤
│ id (PK)             │
│ checkpoint_id (FK)  │
│ attempt_number      │
│ score               │
│ correct_count       │
│ total_questions     │
│ answers (JSON)      │
│ questions_used JSON │
│ attempted_at        │
└─────────────────────┘

┌─────────────┐
│    User     │
└──────┬──────┘
       │ 1
       │
       │ 1
┌──────┴──────────────┐
│   UserAnalytics     │
├─────────────────────┤
│ id (PK)             │
│ user_id (FK)        │
│ total_sessions      │
│ completed_sessions  │
│ total_checkpoints   │
│ avg_score           │
│ total_time_minutes  │
│ current_streak      │
│ longest_streak      │
│ last_study_date     │
│ updated_at          │
└─────────────────────┘

┌─────────────┐
│    User     │
└──────┬──────┘
       │ 1
       │
       │ N
┌──────┴──────────────┐
│    UserBadge        │
├─────────────────────┤
│ id (PK)             │
│ user_id (FK)        │
│ badge_name          │
│ badge_type          │
│ description         │
│ earned_at           │
└─────────────────────┘

┌─────────────┐
│    User     │
└──────┬──────┘
       │ 1
       │
       │ N
┌──────┴──────────────┐
│    WeakTopic        │
├─────────────────────┤
│ id (PK)             │
│ user_id (FK)        │
│ topic               │
│ concept             │
│ strength_score      │
│ last_practiced      │
└─────────────────────┘

┌─────────────┐
│    User     │
└──────┬──────┘
       │ 1
       │
       │ N
┌──────┴──────────────┐
│  DailyChallenge     │
├─────────────────────┤
│ id (PK)             │
│ user_id (FK)        │
│ task                │
│ bonus_xp            │
│ completed           │
│ date                │
└─────────────────────┘
```


### 4.2 Key Data Models

**User Model**
```python
class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True, nullable=False)
    password_hash = Column(String, nullable=False)  # bcrypt hashed
    name = Column(String, nullable=False)
    tutor_mode = Column(String, default="supportive_buddy")
    xp = Column(Integer, default=0)
    level = Column(Integer, default=1)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    # Relationships
    sessions = relationship("LearningSession", back_populates="user")
    badges = relationship("UserBadge", back_populates="user")
    analytics = relationship("UserAnalytics", back_populates="user", uselist=False)
    weak_topics = relationship("WeakTopic", back_populates="user")
    challenges = relationship("DailyChallenge", back_populates="user")
```

**Checkpoint Model (Core Learning Unit)**
```python
class Checkpoint(Base):
    __tablename__ = "checkpoints"
    
    id = Column(Integer, primary_key=True, index=True)
    session_id = Column(Integer, ForeignKey("learning_sessions.id"))
    checkpoint_index = Column(Integer)  # Order within session
    topic = Column(String)
    objectives = Column(JSON)  # List of learning objectives
    key_concepts = Column(JSON)  # List of key concepts to master
    level = Column(String)  # beginner, intermediate, advanced
    status = Column(String, default="pending")  # pending, in_progress, passed, failed
    understanding_score = Column(Float)  # 0.0 to 1.0
    attempts = Column(Integer, default=0)
    completed_at = Column(DateTime)
    xp_earned = Column(Integer, default=0)
    
    # AI-generated content (cached)
    context = Column(Text)  # Gathered learning context
    explanation = Column(Text)  # Personalized explanation
    content_generated = Column(Boolean, default=False)
    questions_cache = Column(JSON)  # Generated questions
    validation_score = Column(Float)  # Context quality score (0-100)
    
    # Relationships
    session = relationship("LearningSession", back_populates="checkpoints")
    quiz_attempts = relationship("QuizAttempt", back_populates="checkpoint")
```

**QuizAttempt Model (Tracks Each Assessment)**
```python
class QuizAttempt(Base):
    __tablename__ = "quiz_attempts"
    
    id = Column(Integer, primary_key=True, index=True)
    checkpoint_id = Column(Integer, ForeignKey("checkpoints.id"))
    attempt_number = Column(Integer)  # 1, 2, 3
    score = Column(Float)  # Understanding score (0.0 to 1.0)
    correct_count = Column(Integer)
    total_questions = Column(Integer)
    answers = Column(JSON)  # User's selected answers
    questions_used = Column(JSON)  # Questions presented in this attempt
    attempted_at = Column(DateTime, default=datetime.utcnow)
    
    checkpoint = relationship("Checkpoint", back_populates="quiz_attempts")
```

---

## 5. API Contracts

### 5.1 Authentication Endpoints

**POST /auth/register**
```json
Request:
{
  "email": "user@example.com",
  "password": "securepassword123",
  "name": "John Doe"
}

Response (201 Created):
{
  "id": 1,
  "email": "user@example.com",
  "name": "John Doe",
  "tutor_mode": "supportive_buddy",
  "xp": 0,
  "level": 1,
  "created_at": "2026-02-14T10:30:00Z"
}
```

**POST /auth/login**
```json
Request:
{
  "email": "user@example.com",
  "password": "securepassword123"
}

Response (200 OK):
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer"
}
```

### 5.2 Session Endpoints

**POST /sessions**
```json
Request:
{
  "topic": "Operating System Scheduling Algorithms",
  "user_notes": "Preparing for GATE exam"
}

Response (201 Created):
{
  "id": 42,
  "topic": "Operating System Scheduling Algorithms",
  "status": "in_progress",
  "created_at": "2026-02-14T10:35:00Z",
  "completed_at": null,
  "xp_earned": 0
}
```

**GET /sessions/{id}**
```json
Response (200 OK):
{
  "id": 42,
  "topic": "Operating System Scheduling Algorithms",
  "status": "in_progress",
  "created_at": "2026-02-14T10:35:00Z",
  "completed_at": null,
  "xp_earned": 0,
  "checkpoints": [
    {
      "id": 101,
      "checkpoint_index": 0,
      "topic": "FCFS and SJF Scheduling",
      "status": "passed",
      "understanding_score": 0.75,
      "attempts": 1,
      "xp_earned": 40
    },
    {
      "id": 102,
      "checkpoint_index": 1,
      "topic": "Round Robin and Priority Scheduling",
      "status": "in_progress",
      "understanding_score": null,
      "attempts": 0,
      "xp_earned": 0
    }
  ]
}
```

### 5.3 Checkpoint Endpoints

**POST /checkpoints/{id}/generate**
```json
Request: (empty body, uses checkpoint data from DB)

Response (200 OK):
{
  "context": "Round Robin scheduling is a preemptive algorithm...",
  "explanation": "Imagine you're at a restaurant with friends...",
  "questions": [
    {
      "type": "mcq",
      "question": "What is the main advantage of Round Robin scheduling?",
      "options": [
        "Fair CPU time distribution",
        "Shortest execution time",
        "Highest priority first",
        "No context switching"
      ],
      "correct_answer": "Fair CPU time distribution",
      "explanation": "Round Robin ensures each process gets equal CPU time...",
      "difficulty": "intermediate",
      "tested_concept": "Round Robin fairness"
    }
  ],
  "validation_score": 92.5,
  "workflow_complete": true
}
```

**POST /checkpoints/{id}/submit**
```json
Request:
{
  "answers": [
    "Fair CPU time distribution",
    "Time quantum",
    "Preemptive",
    "Context switching overhead"
  ]
}

Response (200 OK):
{
  "score": 0.75,
  "correct_count": 3,
  "total_questions": 4,
  "passed": true,
  "detailed_results": [
    {
      "question": "What is the main advantage of Round Robin scheduling?",
      "user_answer": "Fair CPU time distribution",
      "correct_answer": "Fair CPU time distribution",
      "is_correct": true,
      "explanation": "Round Robin ensures each process gets equal CPU time...",
      "tested_concept": "Round Robin fairness"
    }
  ],
  "weak_areas": ["Context switching overhead"],
  "xp_earned": 40
}
```

### 5.4 Analytics Endpoints

**GET /analytics**
```json
Response (200 OK):
{
  "total_sessions": 15,
  "completed_sessions": 12,
  "total_checkpoints": 48,
  "avg_score": 0.78,
  "current_streak": 5,
  "longest_streak": 12,
  "last_study_date": "2026-02-14T10:00:00Z"
}
```

**GET /analytics/weak-topics**
```json
Response (200 OK):
[
  {
    "topic": "Operating Systems",
    "concept": "Context switching overhead",
    "strength_score": 0.45,
    "last_practiced": "2026-02-14T10:00:00Z"
  },
  {
    "topic": "Data Structures",
    "concept": "AVL tree rotations",
    "strength_score": 0.52,
    "last_practiced": "2026-02-13T15:30:00Z"
  }
]
```

### 5.5 Gamification Endpoints

**GET /badges**
```json
Response (200 OK):
[
  {
    "id": 1,
    "badge_name": "First Steps",
    "badge_type": "milestone",
    "description": "Completed your first learning session",
    "earned_at": "2026-02-10T12:00:00Z"
  },
  {
    "id": 2,
    "badge_name": "Week Warrior",
    "badge_type": "streak",
    "description": "Maintained a 7-day learning streak",
    "earned_at": "2026-02-14T09:00:00Z"
  }
]
```

**PATCH /users/tutor-mode**
```json
Request:
{
  "tutor_mode": "exam_mode"
}

Response (200 OK):
{
  "id": 1,
  "email": "user@example.com",
  "name": "John Doe",
  "tutor_mode": "exam_mode",
  "xp": 450,
  "level": 3
}
```

---

## 6. Module Responsibilities

### 6.1 Backend Modules

**app/main.py**
- FastAPI application initialization
- Middleware configuration (CORS, authentication)
- Router registration
- Database initialization
- Health check endpoints

**app/auth.py**
- Password hashing (bcrypt)
- JWT token generation and validation
- User authentication logic
- Token refresh mechanism

**app/routes/**
- `auth.py`: Registration, login, token refresh, user profile
- `sessions.py`: Create session, list sessions, get session details
- `checkpoints.py`: Generate content, submit quiz, get checkpoint data
- `analytics.py`: User analytics, weak topics, performance trends
- `gamification.py`: Badges, daily challenges, XP/level updates

**app/services/workflow.py**
- LangGraph workflow definition
- State machine nodes: gather_context, validate_context, explain, generate_questions
- Conditional edge logic for validation retry
- Workflow execution orchestration
- LangSmith tracing integration

**app/services/context_gatherer.py**
- AI-powered context gathering for topics
- Context validation logic (quality scoring)
- Retry mechanism for failed validation
- Structured prompts for consistent output

**app/services/explainer.py**
- Personalized explanation generation
- Tutor mode personality implementation
- Example and analogy generation
- Difficulty level adaptation

**app/services/question_generator.py**
- MCQ question generation
- Question deduplication logic
- Weak area targeting for retries
- Answer validation (single correct answer)
- Question caching

**app/services/feynman.py**
- Feynman Technique implementation
- Prerequisite concept identification
- Teaching approach rotation (5 approaches)
- Weak area re-teaching
- Tutor mode integration

**app/services/evaluator.py**
- Answer normalization
- Semantic evaluation logic
- Weak area identification
- Detailed feedback generation
- Understanding score calculation

**app/models.py**
- SQLAlchemy ORM models
- Database schema definitions
- Relationships between entities

**app/schemas.py**
- Pydantic request/response schemas
- Data validation
- API contract definitions

**app/database.py**
- Database connection management
- Session factory
- Database initialization

### 6.2 Frontend Modules

**src/pages/**
- `Landing.jsx`: Marketing page with feature highlights
- `Login.jsx`: User authentication
- `Register.jsx`: New user registration
- `Dashboard.jsx`: User home with sessions, analytics, badges
- `Session.jsx`: Checkpoint list and progress for a session
- `Quiz.jsx`: Assessment interface with MCQ questions
- `Feynman.jsx`: Re-teaching explanation display
- `Completion.jsx`: Session completion celebration
- `History.jsx`: Past sessions and performance
- `Analytics.jsx`: Detailed analytics and weak topics

**src/components/**
- `Navbar.jsx`: Navigation bar with auth state
- `SessionCard.jsx`: Session summary card
- `ProgressBar.jsx`: Visual progress indicator
- `BadgeCard.jsx`: Badge display component
- `Sidebar.jsx`: Navigation sidebar

**src/context/**
- `AuthContext.jsx`: Global authentication state management

**src/services/**
- `api.js`: Axios HTTP client with interceptors for auth tokens

---

## 7. Sequence Diagrams

### 7.1 Checkpoint Learning Flow

```
User          Frontend       API          LangGraph      LangChain/LLM      Database
 |               |            |               |                |               |
 |--Start Checkpoint--------->|               |                |               |
 |               |            |--Get Checkpoint Data---------->|               |
 |               |            |<--Checkpoint Data--------------|               |
 |               |            |               |                |               |
 |               |            |--Invoke Workflow-------------->|               |
 |               |            |               |                |               |
 |               |            |               |--gather_context--------------->|
 |               |            |               |                |<--LLM Call----|
 |               |            |               |<--Context------|               |
 |               |            |               |                |               |
 |               |            |               |--validate_context------------->|
 |               |            |               |                |<--LLM Call----|
 |               |            |               |<--Score 92%----|               |
 |               |            |               |                |               |
 |               |            |               |--explain---------------------->|
 |               |            |               |                |<--LLM Call----|
 |               |            |               |<--Explanation--|               |
 |               |            |               |                |               |
 |               |            |               |--generate_questions----------->|
 |               |            |               |                |<--LLM Call----|
 |               |            |               |<--Questions----|               |
 |               |            |               |                |               |
 |               |            |<--Workflow Result--------------|               |
 |               |            |                                                |
 |               |            |--Save Content--------------------------------->|
 |               |            |<--Saved-----------------------------------------|
 |               |            |                                                |
 |               |<--Content--|                                                |
 |<--Display Explanation------|                                                |
 |               |            |                                                |
 |--Take Quiz--->|            |                                                |
 |               |--Submit Answers-------->|                                   |
 |               |            |--Evaluate Answers----------------------------->|
 |               |            |<--Results---------------------------------------|
 |               |            |                                                |
 |               |            |--Update Checkpoint, Analytics----------------->|
 |               |            |<--Updated---------------------------------------|
 |               |            |                                                |
 |               |<--Results--|                                                |
 |<--Display Results----------|                                                |
```


### 7.2 Feynman Re-Teaching Flow

```
User          Frontend       API          Feynman Service    LangChain/LLM      Database
 |               |            |                  |                  |               |
 |--Failed Quiz (Score 50%)-->|                  |                  |               |
 |               |            |--Identify Weak Areas-------------->|               |
 |               |            |<--Weak: ["Priority Scheduling"]-----|               |
 |               |            |                  |                  |               |
 |               |            |--Apply Feynman-->|                  |               |
 |               |            |                  |--Generate Re-Teaching---------->|
 |               |            |                  |                  |<--LLM Call----|
 |               |            |                  |<--Explanation----|               |
 |               |            |<--Feynman Text---|                  |               |
 |               |            |                                                     |
 |               |            |--Save Feynman Explanation-------------------------->|
 |               |            |<--Saved---------------------------------------------|
 |               |            |                                                     |
 |               |<--Feynman--|                                                     |
 |<--Display Re-Teaching------|                                                     |
 |               |            |                                                     |
 |--Ready to Retry----------->|                                                     |
 |               |            |--Generate New Questions (targeting weak areas)----->|
 |               |            |<--New Questions-------------------------------------|
 |               |            |                                                     |
 |               |<--Quiz-----|                                                     |
 |<--Take Retry Quiz----------|                                                     |
```

---

## 8. Deployment Architecture

### 8.1 Deployment Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLOUD HOSTING                           │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    FRONTEND DEPLOYMENT                    │  │
│  │  Platform: Vercel / Netlify / AWS Amplify                 │  │
│  │  Build: npm run build (Vite)                              │  │
│  │  CDN: Global edge caching                                 │  │
│  │  Domain: conceptly.app                                    │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              ↓ HTTPS                            │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    BACKEND DEPLOYMENT                     │  │
│  │  Platform: Render / AWS Elastic Beanstalk / Railway       │  │
│  │  Runtime: Python 3.11+                                    │  │
│  │  Framework: FastAPI with Uvicorn                          │  │
│  │  Environment Variables:                                   │  │
│  │    - DATABASE_URL                                         │  │
│  │    - GROQ_API_KEY                                         │  │
│  │    - JWT_SECRET_KEY                                       │  │
│  │    - LANGCHAIN_API_KEY (LangSmith)                        │  │
│  │  Auto-scaling: Based on CPU/memory                        │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              ↓                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    DATABASE DEPLOYMENT                    │  │
│  │  Platform: Render PostgreSQL / AWS RDS / Supabase         │  │
│  │  Version: PostgreSQL 15+                                  │  │
│  │  Backup: Daily automated backups                          │  │
│  │  Connection Pooling: Enabled                              │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    EXTERNAL SERVICES                      │  │
│  │  - Groq API (LLM inference)                               │  │
│  │  - LangSmith (tracing & observability)                    │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 8.2 Environment Configuration

**Backend Environment Variables**
```bash
# Database
DATABASE_URL=postgresql://user:password@host:5432/conceptly

# AI Services
GROQ_API_KEY=gsk_xxxxxxxxxxxxxxxxxxxxx
LANGCHAIN_API_KEY=ls__xxxxxxxxxxxxxxxxxxxxx
LANGCHAIN_TRACING_V2=true
LANGCHAIN_PROJECT=conceptly-production

# Authentication
JWT_SECRET_KEY=your-secret-key-here
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=60
REFRESH_TOKEN_EXPIRE_DAYS=7

# CORS
FRONTEND_URL=https://conceptly.app

# Application
ENVIRONMENT=production
LOG_LEVEL=INFO
```

**Frontend Environment Variables**
```bash
VITE_API_URL=https://api.conceptly.app
VITE_APP_NAME=Conceptly
```

### 8.3 Build & Deployment Steps

**Backend Deployment**
```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Run database migrations
alembic upgrade head

# 3. Start application
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

**Frontend Deployment**
```bash
# 1. Install dependencies
npm install

# 2. Build production bundle
npm run build

# 3. Deploy to hosting platform
# (Automatic via Git push for Vercel/Netlify)
```

### 8.4 CI/CD Pipeline

```yaml
# .github/workflows/deploy.yml
name: Deploy Conceptly

on:
  push:
    branches: [main]

jobs:
  test-backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run backend tests
        run: |
          cd backend
          pip install -r requirements.txt
          pytest

  test-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run frontend tests
        run: |
          cd frontend
          npm install
          npm run test

  deploy-backend:
    needs: [test-backend]
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Render
        run: |
          curl -X POST ${{ secrets.RENDER_DEPLOY_HOOK }}

  deploy-frontend:
    needs: [test-frontend]
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Vercel
        run: |
          vercel --prod --token=${{ secrets.VERCEL_TOKEN }}
```

---

## 9. Security Considerations

### 9.1 Authentication Security

**Password Security**
- Passwords hashed using bcrypt with 12 rounds
- Minimum password length: 8 characters
- Password complexity requirements enforced
- No password stored in plain text

**Token Security**
- JWT tokens signed with HS256 algorithm
- Access tokens expire after 1 hour
- Refresh tokens expire after 7 days
- Tokens include user ID and email claims
- Token validati


### 11.2 Application Logging

**Structured Logging**
- JSON format for easy parsing
- Log levels: DEBUG, INFO, WARNING, ERROR, CRITICAL
- Context included: user_id, session_id, checkpoint_id, timestamp
- Centralized logging to cloud provider

**Key Metrics Logged**
- API request/response times
- LLM generation times
- Database query times
- Error rates and stack traces
- User actions (login, session creation, quiz submission)

### 11.3 Performance Monitoring

**Metrics to Track**
- API endpoint latency (p50, p95, p99)
- LLM API call success rate
- Database connection pool usage
- Memory and CPU utilization
- Active user count

**Alerting**
- High error rate (> 5%)
- Slow API responses (p95 > 5s)
- Database connection failures
- LLM API failures
- System downtime

---

## 12. Testing Strategy

### 12.1 Backend Testing

**Unit Tests**
- Test individual functions (evaluator, question_generator)
- Mock LLM API calls for deterministic tests
- Test database models and relationships
- Coverage target: 80%+

**Integration Tests**
- Test API endpoints end-to-end
- Test LangGraph workflow execution
- Test database transactions
- Use test database for isolation

**Example Test**
```python
def test_evaluate_answers():
    questions = [
        {"correct_answer": "Option A", "tested_concept": "Concept 1"},
        {"correct_answer": "Option B", "tested_concept": "Concept 2"}
    ]
    answers = ["Option A", "Option C"]
    
    result = evaluate_answers(questions, answers)
    
    assert result['understanding_score'] == 0.5
    assert result['correct_count'] == 1
    assert result['passed'] == False
    assert "Concept 2" in result['weak_areas']
```

### 12.2 Frontend Testing

**Component Tests**
- Test React components in isolation
- Test user interactions (button clicks, form submissions)
- Test conditional rendering
- Use React Testing Library

**E2E Tests**
- Test complete user flows (registration → login → session → quiz)
- Use Playwright or Cypress
- Test on multiple browsers

---

## 13. Future Enhancements

### 13.1 Advanced AI Features

**Retrieval Augmented Generation (RAG)**
- Integrate external knowledge sources (documentation, videos, articles)
- Vector database for semantic search (Pinecone, Weaviate)
- Improves context quality and accuracy

**Personalized Difficulty Scaling**
- Analyze user performance trends
- Automatically adjust checkpoint difficulty
- Adaptive learning paths based on mastery

**Multi-Modal Learning**
- Support images in questions (diagrams, charts)
- Code snippet evaluation for programming topics
- Mathematical formula rendering (LaTeX)
- Video explanations for complex concepts

**Voice Interface**
- Speech-to-text for quiz answers
- Text-to-speech for explanations
- Conversational learning experience

### 13.2 Collaborative Features

**Study Groups**
- Users form groups to learn together
- Shared sessions and progress tracking
- Group leaderboards

**Peer Review**
- Users review each other's explanations
- Collaborative knowledge building
- Community-driven content improvement

### 13.3 Enterprise Features

**Organization Management**
- Schools/companies manage learner cohorts
- Bulk user creation and management
- Custom branding

**Instructor Dashboard**
- Teachers track student progress
- Identify struggling students
- Assign custom learning paths

**Custom Content Creation**
- Instructors create custom checkpoints
- Upload course materials
- Define custom assessment criteria

### 13.4 Accessibility & Reach

**Regional Language Support**
- Hindi, Tamil, Telugu, Bengali, Marathi, etc.
- Localized UI and AI-generated content
- Expands reach across India

**Offline Mode**
- Cache content for offline access
- Sync progress when online
- Critical for low-bandwidth users

**Mobile Native Apps**
- iOS and Android applications
- Push notifications for streaks and challenges
- Better mobile experience

---

## 14. Why Conceptly Wins the Hackathon

### 14.1 Technical Excellence

✅ **Structured AI Orchestration**: LangGraph state machines ensure reliable, deterministic workflows  
✅ **Validation-First Approach**: 85% context quality threshold prevents misinformation  
✅ **Semantic Evaluation**: Deep understanding assessment beyond keyword matching  
✅ **Observable Systems**: LangSmith tracing enables debugging and optimization  
✅ **Production-Ready**: Security, error handling, monitoring, scalability built-in

### 14.2 Real Learning Impact

✅ **Faster Learning**: Adaptive explanations accelerate comprehension  
✅ **Deeper Understanding**: Semantic evaluation ensures true mastery  
✅ **Higher Engagement**: Gamification maintains motivation (XP, badges, streaks)  
✅ **Personalized Experience**: 4 tutor modes match individual learning styles  
✅ **Adaptive Re-Teaching**: Feynman Technique fills knowledge gaps automatically

### 14.3 Developer Productivity

✅ **Validated Technical Content**: Developers learn accurate information  
✅ **Efficient Learning Paths**: Checkpoint-based progression focuses effort  
✅ **Progress Tracking**: Analytics show learning velocity and trends  
✅ **Weak Area Identification**: Targeted practice on struggling concepts

### 14.4 Innovation & Vision

✅ **Not Just a Chatbot**: Structured learning engine with quality control  
✅ **Measurable Thresholds**: Clear metrics (85%, 70%) show rigor  
✅ **Extensible Architecture**: Ready for RAG, multi-modal, voice, regional languages  
✅ **Scalable Design**: Horizontal scaling, caching, performance optimization

### 14.5 Indian Context Relevance

✅ **Exam Preparation**: Supports GATE, UPSC, JEE, and competitive exams  
✅ **Self-Paced Learning**: Ideal for students with limited tutor access  
✅ **Cost-Effective**: AI-powered tutoring at scale  
✅ **Future-Ready**: Extensible to regional languages and offline modes

---

## 15. Conclusion

Conceptly represents a **paradigm shift in AI-powered learning**. By combining:
- **Validation-first content generation** (85% quality threshold)
- **Semantic understanding evaluation** (70% mastery threshold)
- **Adaptive Feynman re-teaching** (5 teaching approaches)
- **State machine orchestration** (LangGraph workflows)
- **Gamification for engagement** (XP, badges, streaks)

...we create a **reliable, adaptive, and engaging learning experience** that meaningfully improves how people learn and build with technology.

Conceptly is not just another AI tutor—it's a **Structured AI Learning Engine** that validates quality, measures understanding deeply, and adapts intelligently. This is the future of personalized education.

---

**Document Version**: 1.0 (Hackathon Submission)  
**Last Updated**: 2026-02-14  
**Track**: AI for Bharat - Student Track: AI for Learning & Developer Productivity  
**Team**: Conceptly  
**Status**: Ready for Submission

---

## Appendix: Quick Reference

### Key Thresholds
- Context Quality Score: ≥ 85% (validation passes)
- Understanding Score: ≥ 70% (checkpoint passes)
- Max Retry Attempts: 3 per checkpoint
- Max Context Retries: 2 per workflow

### Tech Stack Summary
- **Frontend**: React + Vite
- **Backend**: FastAPI (Python)
- **Database**: PostgreSQL + SQLAlchemy
- **AI Framework**: LangChain + LangGraph
- **LLM**: Groq (Llama 3.3 70B Versatile)
- **Observability**: LangSmith
- **Auth**: JWT with bcrypt

### API Base URLs
- Production: `https://api.conceptly.app`
- Development: `http://localhost:8000`

### Contact
- GitHub: [github.com/conceptly/conceptly](https://github.com/conceptly/conceptly)
- Demo: [conceptly.app](https://conceptly.app)
- Email: team@conceptly.app
