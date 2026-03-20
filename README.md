# GitHelp — AI Powered Code Review System
## Updated Roadmap (All Fixes Applied)

---

## Important: Your File Structure Convention

You use **singular naming** throughout. Every module folder, import, and reference
in this roadmap follows that convention.

```
backend/
├── app/
│   ├── api/
│   │   └── health.py
│   ├── common/
│   │   ├── base.py
│   │   └── enums.py
│   ├── core/
│   │   ├── cache.py
│   │   ├── config.py
│   │   ├── database.py
│   │   ├── dependencies.py
│   │   ├── logger.py
│   │   ├── middleware.py
│   │   ├── metrics.py          ← Day 27
│   │   └── security.py
│   ├── modules/
│   │   ├── auth/
│   │   │   ├── router.py
│   │   │   ├── schemas.py
│   │   │   └── service.py
│   │   ├── user/               ← singular
│   │   │   ├── model.py
│   │   │   ├── router.py
│   │   │   ├── schemas.py
│   │   │   └── service.py
│   │   ├── repository/         ← singular
│   │   │   ├── model.py
│   │   │   ├── router.py
│   │   │   ├── schemas.py
│   │   │   └── service.py
│   │   ├── review/             ← singular
│   │   │   ├── model.py
│   │   │   ├── router.py
│   │   │   ├── schemas.py
│   │   │   └── service.py
│   │   ├── finding/            ← Day 10, singular
│   │   │   ├── model.py
│   │   │   ├── router.py
│   │   │   ├── schemas.py
│   │   │   └── service.py
│   │   ├── code_chunk/         ← Day 11, singular
│   │   │   ├── model.py
│   │   │   ├── schemas.py
│   │   │   └── service.py
│   │   ├── finding_feedback/   ← Day 13
│   │   │   ├── model.py
│   │   │   ├── router.py
│   │   │   └── schemas.py
│   │   ├── commit_history/     ← Day 14
│   │   │   ├── model.py
│   │   │   ├── schemas.py
│   │   │   └── service.py
│   │   ├── knowledge_base/     ← Day 15
│   │   │   ├── model.py
│   │   │   ├── router.py
│   │   │   └── schemas.py
│   │   ├── api_key/            ← Day 20, singular
│   │   │   ├── model.py
│   │   │   ├── router.py
│   │   │   └── schemas.py
│   │   ├── audit_log/          ← Day 21, singular
│   │   │   ├── model.py
│   │   │   ├── schemas.py
│   │   │   └── service.py
│   │   ├── ingestion/          ← Day 8, no model
│   │   │   ├── git_cloner.py
│   │   │   └── file_extractor.py
│   │   ├── analysis/           ← Days 9-11, no model
│   │   │   ├── static_analyzer.py
│   │   │   ├── ast_analyzer.py
│   │   │   ├── dependency_mapper.py
│   │   │   └── chunker.py
│   │   ├── ai/                 ← Days 12-16
│   │   │   ├── prompt_builder.py
│   │   │   ├── llm_client.py
│   │   │   ├── hybrid_engine.py
│   │   │   ├── embeddings.py
│   │   │   └── rag_retriever.py
│   │   ├── graph/              ← Days 18-19
│   │   │   ├── state.py
│   │   │   ├── nodes.py
│   │   │   ├── analysis_graph.py
│   │   │   └── agent.py
│   │   ├── reporting/          ← Day 14
│   │   │   ├── scorer.py
│   │   │   └── report_builder.py
│   │   ├── integration/        ← Day 14, singular
│   │   │   └── github_client.py
│   │   ├── task/               ← Day 5, singular, no model
│   │   │   ├── celery_app.py
│   │   │   ├── analysis_task.py
│   │   │   ├── embedding_task.py
│   │   │   └── webhook_task.py
│   │   └── admin/              ← Day 27
│   │       ├── router.py
│   │       └── schemas.py
│   ├── prompts/
│   │   ├── system_prompt_v1.j2
│   │   ├── system_prompt_v2.j2
│   │   ├── user_prompt.j2
│   │   ├── few_shot_examples.j2
│   │   └── rag_prompt.j2
│   └── main.py
│
├── migrations/                 ← Alembic (you already have this)
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── unit/
│   │   ├── __init__.py
│   │   └── test_security.py
│   └── integration/
│       ├── __init__.py
│       └── test_auth_flow.py
├── scripts/
│   └── seed_kb.py              ← Day 15
├── .env
├── .env.example
├── alembic.ini
├── Dockerfile
├── pytest.ini
└── requirements.txt

frontend/                       ← Day 20
docker-compose.yml
docker-compose.prod.yml         ← Day 23
.github/workflows/ci.yml        ← Day 24
```

---

## Import Convention — Roadmap vs Your Code

The original roadmap uses plural naming. You use singular.
Every time you copy from the roadmap, apply this translation:

```
Roadmap writes:                     You write:
────────────────────────────────    ────────────────────────────────
from app.modules.repository...      from app.modules.repository...  (same)
from app.modules.users...           from app.modules.user...
from app.modules.reviews...         from app.modules.review...
from app.modules.findings...        from app.modules.finding...
from app.modules.code_chunks...     from app.modules.code_chunk...
from app.modules.tasks...           from app.modules.task...
from app.modules.api_keys...        from app.modules.api_key...
from app.modules.audit_logs...      from app.modules.audit_log...
from app.modules.integrations...    from app.modules.integration...
```

---

## Updated Tech Stack

| Layer | Technology | Change From Original | Day |
|-------|-----------|---------------------|-----|
| API | FastAPI + Uvicorn | no change | Day 1 |
| Database | PostgreSQL + pgvector | no change | Day 1 |
| ORM | SQLAlchemy 2.0 async | no change | Day 1 |
| Migrations | Alembic | replaces init_db.py (you already did this) | Day 1 |
| Validation | Pydantic v2 | no change | Day 3 |
| Auth | **PyJWT** + **pwdlib[argon2]** | replaces python-jose + passlib | Day 4 |
| Task Queue | Celery 5.x + Redis | no change | Day 5 |
| Linting | **ruff** | replaces pylint + flake8 | Day 9 |
| Complexity | radon | no change | Day 9 |
| Security scan | bandit | no change | Day 9 |
| Dep audit | pip-audit | no change | Day 9 |
| AST Analysis | Python ast + networkx | no change | Day 10 |
| Tokenization | tiktoken | no change | Day 11 |
| LLM | **LiteLLM** + Anthropic/OpenAI | replaces raw SDK calls | Day 12 |
| Prompt Templates | Jinja2 | no change | Day 12 |
| Embeddings | text-embedding-3-small via LiteLLM | no change | Day 15 |
| Vector DB | pgvector (HNSW) | no change | Day 15 |
| RAG | Custom retriever + re-ranking | no change | Day 16 |
| AI Pipeline | LangGraph StateGraph (pinned) | pin version | Day 18 |
| Observability | **LangSmith** | new addition | Day 18 |
| AI Agents | LangGraph ReAct | no change | Day 19 |
| Frontend | **Next.js 15** + TypeScript + Tailwind | was Next.js 14 | Day 20 |
| Token Storage | **httpOnly cookies** | replaces localStorage | Day 20 |
| HTTP Client | Axios + interceptors | no change | Day 20 |
| Charts | recharts | no change | Day 22 |
| Syntax Highlight | react-syntax-highlighter | no change | Day 21 |
| Reverse Proxy | Nginx | no change | Day 23 |
| Containers | Docker multi-stage | no change | Day 23 |
| CI/CD | GitHub Actions (**v4 actions**) | updated action versions | Day 24 |
| Rate Limiting | slowapi | no change | Day 25 |
| Monitoring | prometheus-fastapi-instrumentator | no change | Day 27 |
| Logging | python-json-logger | no change | Day 7 |
| Cloud | Railway / Render | no change | Day 28 |

---

## Updated requirements.txt

```
# Core
fastapi>=0.111.0
uvicorn[standard]>=0.29.0
pydantic>=2.0.0
pydantic-settings>=2.0.0

# Database
sqlalchemy>=2.0.0
asyncpg>=0.29.0
alembic>=1.13.0
pgvector>=0.2.5

# Auth — UPDATED
PyJWT>=2.8.0
pwdlib[argon2]>=0.2.0
python-multipart>=0.0.9

# Cache / Queue
redis>=4.6.0
celery>=5.3.0
flower>=2.0.0

# HTTP
httpx>=0.27.0

# Analysis — UPDATED (ruff replaces pylint + flake8)
ruff>=0.4.0
radon>=6.0.1
bandit>=1.7.8
pip-audit>=2.7.3

# AST
networkx>=3.3

# Tokenization
tiktoken>=0.7.0

# LLM — UPDATED
litellm>=1.40.0
anthropic>=0.28.0

# Prompts
jinja2>=3.1.4

# AI Pipeline — UPDATED (pinned)
langgraph==0.1.5
langsmith>=0.1.0

# Logging
python-json-logger>=2.0.7

# Testing
pytest>=8.0.0
pytest-asyncio>=0.23.0
httpx>=0.27.0
pytest-cov>=4.1.0

# Monitoring
prometheus-fastapi-instrumentator>=6.1.0

# Utilities
python-dotenv>=1.0.0
tenacity>=8.3.0
```

---

# WEEK 1 — Infrastructure, Auth & Core API

---

## Day 1 — Docker + Skeleton ✅ DONE

**What you built:**
- `docker-compose.yml` with app, postgres (pgvector), redis
- `app/common/base.py` — DeclarativeBase
- `app/common/enums.py` — all enums
- `app/core/config.py`, `database.py`, `security.py`, `dependencies.py`
- `app/main.py` with health endpoint
- Alembic migrations setup (better than init_db.py)

---

## Day 2 — Users Module ✅ DONE

**What you built:**
- `app/modules/user/model.py`
- `app/modules/user/schemas.py`
- `app/modules/user/router.py`

---

## Day 3 — Auth Module Schemas + Routers ✅ DONE

**What you built:**
- `app/modules/auth/schemas.py`
- `app/modules/auth/router.py`
- All enums in `app/common/enums.py`

---

## Day 4 — Repository Module + Real JWT Auth ✅ DONE

### Security Changes From Original Roadmap

**Use PyJWT instead of python-jose:**
```python
# WRONG (original roadmap):
from jose import jwt
pip install python-jose

# CORRECT (updated):
import jwt
pip install PyJWT
# WHY: python-jose is barely maintained since 2022
# PyJWT is maintained by the JWT working group themselves
# more secure, actively updated
```

**Use pwdlib instead of passlib:**
```python
# WRONG (original roadmap):
from passlib.context import CryptContext
pwd_context = CryptContext(schemes=["bcrypt"])

# CORRECT (updated):
from pwdlib import PasswordHash
from pwdlib.hashers.argon2 import Argon2Hasher
password_hash = PasswordHash((Argon2Hasher(),))

def get_password_hash(password: str) -> str:
    return password_hash.hash(password)

def verify_password(plain: str, hashed: str) -> bool:
    return password_hash.verify(plain, hashed)

# WHY argon2 over bcrypt:
# argon2 won the Password Hashing Competition (2015)
# resistant to GPU attacks — uses memory not just CPU
# bcrypt still works but argon2 is the current standard
```

**Repository model — what you wrote is correct:**
```python
# Your model.py has:
metadata_ = Column("metadata", JSONB, nullable=True)
# Good — avoids SQLAlchemy reserved name conflict

updated_at = Column(DateTime(timezone=True),
    server_default=func.now(), onupdate=func.now())
# Good — both server_default AND onupdate needed
```

**Repository schemas — fixes applied:**
```python
# Every Optional field needs = None default:
celery_task_id: str | None = None   # not: str | None
error_message: str | None = None    # not: str | None
name: str | None = None             # not: str | None
status: RepoStatus                  # ADD THIS — was missing
```

**Repository service — architecture note:**
```python
# Move analyze_repository.delay() OUT of service INTO router
# WHY: service should be testable without Celery running
# service = DB operations only
# router = orchestrate: call service → schedule task → update task_id
```

**Repository router — fix:**
```python
# WRONG:
router = APIRouter(prefix="/repo", tags=["Repositories"])
# Creates double prefix: /api/v1/repository/repo/

# CORRECT:
router = APIRouter(tags=["Repositories"])
# main.py owns the prefix: prefix="/api/v1/repository"
```

---

## Day 5 — Celery + Redis + Task Queue ✅ DONE

### Celery Architecture Clarification

```
Celery uses TRUE parallelism via multiprocessing:
  Each worker process = separate Python interpreter
  Each process = separate GIL
  4 workers = 4 repos analyzed simultaneously

celery worker --concurrency=4
  = 1 worker command, 4 OS processes
  = 4 truly parallel analyses

100 users submit at once:
  FastAPI async handles all 100 in ~2 seconds (async)
  Redis queue holds all 100 tasks
  4 workers process them in parallel
  Total time = 100 tasks × task_duration ÷ 4 workers
```

### Redis — One Package, Two Submodules

```python
# requirements.txt — only ONE redis package needed:
redis>=4.6.0
# redis>=4.2 includes async support built-in
# NO separate aioredis package needed (deprecated)

# In FastAPI code (async context):
import redis.asyncio as aioredis
client = aioredis.from_url(settings.REDIS_URL)
await client.get(key)   # async, non-blocking

# In Celery tasks (sync context):
import redis
client = redis.Redis.from_url(settings.REDIS_URL)
client.get(key)         # sync, fine in Celery worker
```

### Celery Task Pattern — asyncio.run() Bridge

```python
# app/modules/task/analysis_task.py

import asyncio
import uuid
import logging
from app.modules.task.celery_app import celery_app
from app.core.database import AsyncSessionLocal

logger = logging.getLogger(__name__)


async def _run_analysis_async(repository_id: str, celery_task) -> dict:
    """
    Async inner function — does all the real work.
    Uses AsyncSessionLocal (async session) for DB operations.
    Receives celery_task (self) to call update_state() for Flower UI.
    """
    loop = asyncio.get_event_loop()
    repo_path = None

    async with AsyncSessionLocal() as db:
        from app.modules.repository.model import Repository
        from app.modules.review.model import Review
        from app.common.enums import RepoStatus, ReviewStatus

        repo = await db.get(Repository, uuid.UUID(repository_id))
        if not repo:
            logger.error("Repository not found",
                extra={"repository_id": repository_id})
            return {"status": "error", "message": "Repository not found"}

        repo.status = RepoStatus.analyzing
        await db.commit()

        try:
            celery_task.update_state(
                state="STARTED",
                meta={"progress": 10, "stage": "cloning"}
            )

            # Day 8: run_in_executor for sync subprocess calls
            from app.modules.ingestion.git_cloner import (
                clone_repository, cleanup_repository, get_file_list
            )
            from app.modules.ingestion.file_extractor import get_files_by_language

            repo_path = await loop.run_in_executor(
                None, clone_repository, repo.url
            )
            # WHY run_in_executor: clone_repository calls subprocess.run()
            # subprocess is synchronous — blocks the event loop if called directly
            # run_in_executor runs it in a thread pool — event loop stays free

            celery_task.update_state(
                state="STARTED",
                meta={"progress": 30, "stage": "scanning files"}
            )

            files = await loop.run_in_executor(None, get_file_list, repo_path)
            files_by_language = get_files_by_language(files)

            logger.info("Files scanned", extra={
                "repository_id": repository_id,
                "total_files": len(files),
                "languages": {lang: len(f) for lang, f in files_by_language.items()}
            })

            review = Review(
                repository_id=repo.id,
                status=ReviewStatus.in_progress
            )
            db.add(review)
            await db.commit()
            await db.refresh(review)

            # Days 9-12 replace these stubs
            celery_task.update_state(
                state="STARTED",
                meta={"progress": 90, "stage": "analyzing"}
            )

            review.status = ReviewStatus.completed
            review.quality_score = 75
            repo.status = RepoStatus.completed
            await db.commit()

            return {
                "status": "completed",
                "repository_id": repository_id,
                "review_id": str(review.id),
                "file_count": len(files),
                "languages": list(files_by_language.keys())
            }

        except Exception as e:
            logger.error("Analysis failed", extra={
                "repository_id": repository_id,
                "error": str(e)
            })
            repo.status = RepoStatus.failed
            repo.error_message = str(e)
            await db.commit()
            raise
            # WHY bare raise: re-raises original exception with original traceback
            # Celery sees the exception → marks task FAILURE → triggers retry

        finally:
            # ALWAYS runs — success, exception, even sys.exit()
            # Guarantees temp dir cleanup
            if repo_path:
                await loop.run_in_executor(None, cleanup_repository, repo_path)


@celery_app.task(
    bind=True,
    max_retries=3,
    name="analysis_task.analyze_repository"
    # WHY explicit name: stable across refactors
    # without name: Celery uses Python path — rename file = queued tasks break
)
def analyze_repository(self, repository_id: str):
    logger.info("Task started", extra={"repository_id": repository_id})
    return asyncio.run(_run_analysis_async(repository_id, self))
    # asyncio.run() = creates NEW event loop, runs async function, destroys loop
    # WHY this works: we create OUR OWN loop — not using FastAPI's loop
    # Celery worker has no event loop — asyncio.run() provides one
```

---

## Day 6 — Wire API → Queue → DB + Review Module ✅ DONE

Review model and schemas are complete. See Day 5 task file above for the
wired-up Celery task that creates Review rows.

---

## Day 7 — Logging, Middleware, Health Checks & Tests ✅ DONE

### Key Fixes Applied

**logger.py — setup_logging() must be called FIRST in main.py:**
```python
# app/main.py — ORDER MATTERS:
from app.core.logger import setup_logging
setup_logging()   # ← before ANY other imports that might log

# Then import everything else:
from app.core.middleware import CorrelationIDMiddleware
# etc.
```

**No manual event_loop fixture:**
```python
# pytest.ini:
asyncio_mode = auto
# pytest-asyncio >= 0.21 manages the event loop automatically
# DO NOT add a manual event_loop fixture — it causes:
# "Task attached to a different event loop" errors
```

**No nested transaction + rollback in tests:**
```python
# conftest.py — WRONG (causes InvalidRequestError):
async with session.begin():
    yield session
    await session.rollback()  # crashes if service called db.commit()

# CORRECT — let service commit freely:
async with TestSessionLocal() as session:
    yield session
    # session closes naturally — no rollback fight
```

**OAuth2 login = FORM DATA not JSON:**
```python
# Tests and frontend both:
await client.post("/api/v1/auth/login",
    data={"username": "email@test.com", "password": "Password123!"}
)
# data= not json=
# field is "username" not "email" (OAuth2 spec)
```

**Register returns user data, NOT tokens:**
```python
# register endpoint → returns user {id, email, ...}
# login endpoint → returns {access_token, refresh_token, token_type}
# always get tokens from /auth/login not /auth/register
```

**async Redis everywhere:**
```python
# app/core/cache.py — all async:
import redis.asyncio as aioredis

async def cache_get(key: str) -> str | None: ...
async def cache_set(key: str, value: str, ttl: int) -> None: ...
async def cache_delete(key: str) -> None: ...
async def ping_redis() -> bool: ...

# All callers must await:
cached = await cache_get(key)
await cache_set(key, value, ttl=60)
await cache_delete(key)
```

**Health router — prefix in main.py:**
```python
# app/api/health.py:
router = APIRouter(tags=["Health"])

@router.get("/live")    # not /health/live
@router.get("/ready")   # not /health/ready

# app/main.py:
app.include_router(health_router, prefix="/health")
# Final URLs: /health/live and /health/ready
```

**Logging rules:**
```
File                    Log?    Rule
──────────────────────────────────────────────────────────
core/config.py          NO      just reads env vars
core/database.py        NO      get_db is a factory
core/security.py        SOME    only on token rejection
core/cache.py           SOME    only on Redis failures
core/middleware.py      YES     this IS the HTTP request logger
core/dependencies.py    SOME    only on auth failures
modules/*/router.py     NO      middleware covers HTTP logging
modules/*/model.py      NO      just data definitions
modules/*/schemas.py    NO      just validation
modules/*/service.py    YES     business operations
modules/task/*.py       YES     only visibility into background work
modules/ingestion/*.py  YES     external process + disk operations
modules/analysis/*.py   YES     subprocess calls + tool results
modules/ai/*.py         YES     LLM calls + token usage + retries
```

**python -m pytest always:**
```bash
python -m pytest              # always use this
# NOT: pytest
# WHY: python -m uses your venv's pytest
# bare pytest might use system pytest that can't see your packages
```

---

# WEEK 2 — Code Intelligence & AI Engine

---

## Day 8 — Git Ingestion Module ✅ DONE

### Files Created
```
app/modules/ingestion/
├── __init__.py
├── git_cloner.py
└── file_extractor.py
```

### Key Patterns

**validate_git_url — pure function, no logging:**
```python
def validate_git_url(url: str) -> str:
    # pure function — input → output OR raises ValueError
    # no side effects → no logging
    # WHY ValueError not HTTPException:
    # pure Python function, no FastAPI context
    # caller converts to HTTPException
```

**clone_repository — logs because it changes the world:**
```python
def clone_repository(url: str, token: str | None = None) -> str:
    # creates /tmp directory, runs subprocess, downloads files
    # side effects → log start, log success, log failure
    logger.info("Starting repository clone", extra={"url": url})
```

**Always use shell=False:**
```python
subprocess.run(["git", "clone", "--depth", "1", url, tmp_dir],
    shell=False)  # explicit — prevents shell injection
# never: subprocess.run(f"git clone {url}", shell=True)
```

**always finally for cleanup:**
```python
try:
    repo_path = clone_repository(url)
    # ... do work ...
finally:
    if repo_path:
        cleanup_repository(repo_path)
# finally ALWAYS runs — success, exception, sys.exit()
# /tmp never fills up
```

**run_in_executor for sync calls in async context:**
```python
# In analysis_task.py async function:
loop = asyncio.get_event_loop()
repo_path = await loop.run_in_executor(
    None, clone_repository, repo.url
)
# subprocess.run() is sync — blocks event loop if called directly
# run_in_executor runs in thread pool — loop stays free
```

---

## Day 9 — Static Analysis Module

### UPDATED: ruff replaces pylint + flake8

```
Original plan:          Updated plan:
──────────────────────  ──────────────────────────────────
pylint                  ruff (replaces pylint + flake8)
flake8                  ↑ same tool
radon                   radon (keep — richer complexity)
bandit                  bandit (keep — deeper security)
pip-audit               pip-audit (keep — scans dependencies)
custom secrets scan     custom secrets scan (keep)

Speed: pylint+flake8 = ~53s for 500 files
       ruff alone    = ~0.3s for 500 files
```

### Files to Create
```
app/modules/analysis/
├── __init__.py
└── static_analyzer.py
```

### ruff replaces two tools in one call:
```python
def run_ruff(file_path: str) -> list[dict]:
    result = subprocess.run([
        "ruff", "check",
        "--output-format", "json",
        "--select", "E,W,F,C,B,S",
        # E,W = PEP8 (was flake8)
        # F   = pyflakes: unused imports, undefined names (was flake8)
        # C   = complexity (was pylint C)
        # B   = bugbear: real bugs like mutable defaults (was pylint W)
        # S   = security basics (bonus — partial bandit coverage)
        "--line-length", "120",
        file_path
    ], capture_output=True, text=True, timeout=30, shell=False)
```

### All findings normalized to same structure:
```python
{
    "file_path": "src/main.py",
    "line_start": 45,
    "line_end": 45,
    "severity": "major",          # critical/major/minor/info
    "category": "security",       # security/maintainability/performance
    "source": "static_only",
    "rule_ids": ["B324"],
    "title": "MD5 is cryptographically broken",
    "explanation": "...",
    "suggestion": "Use SHA256 or better"
}
```

### Wire into analysis_task.py:
```python
from app.modules.analysis.static_analyzer import analyze_repository_files

static_findings = await loop.run_in_executor(
    None,
    analyze_repository_files,
    files,
    repo_path
)
# run_in_executor: analyze_repository_files calls subprocess multiple times
# must run in thread pool — not directly in async function
```

---

## Day 10 — AST Analysis + Finding Module

### Files to Create
```
app/modules/analysis/
└── ast_analyzer.py          ← new

app/modules/finding/         ← new module
├── __init__.py
├── model.py
├── router.py
├── schemas.py
└── service.py
```

### Finding model — important details:
```python
class Finding(Base):
    __tablename__ = "findings"

    severity_rank = Column(Integer)
    # WHY: PostgreSQL ENUMs sort alphabetically
    # critical, info, major, minor — alphabetical order
    # severity_rank = 1,2,3,4 gives correct sort
    # always ORDER BY severity_rank not severity

    rule_ids = Column(JSONB, server_default="'[]'::jsonb", default=list)
    # WHY server_default AND default:
    # server_default = fires at DB level (always)
    # default = fires at Python level (when creating objects)
    # both needed for all code paths

    embedding_id = Column(UUID, ForeignKey("code_chunk.id",
        ondelete="SET NULL"), nullable=True)
    # ondelete SET NULL: if code_chunk deleted, finding stays
    # just loses the embedding reference
```

### FindingCreate is internal only:
```python
class FindingCreate(BaseModel):
    # NEVER expose via API — internal schema only
    # analysis engine uses this to create findings
    # API exposure would let anyone inject findings
    # no status field, no suppression fields
    # only the analysis engine sets those
```

---

## Day 11 — Code Chunks Module + Tokenization

### Files to Create
```
app/modules/code_chunk/      ← singular
├── __init__.py
├── model.py
├── schemas.py
└── service.py

app/modules/analysis/
└── chunker.py               ← new
```

### HNSW index — raw SQL migration (not in model.py):
```sql
-- Create Alembic migration for this:
-- alembic revision --autogenerate -m "add_hnsw_index"
-- Then add manually to the migration file:
CREATE INDEX ix_code_chunk_embedding ON code_chunk
USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);
-- WHY raw SQL: SQLAlchemy declarative cannot express HNSW parameters
-- WHY Alembic not raw psql: versioned, reproducible, part of migration history
```

### Token limit — 3000 per chunk:
```python
def count_tokens(text: str) -> int:
    import tiktoken
    enc = tiktoken.get_encoding("cl100k_base")
    return len(enc.encode(text))

# WHY 3000 limit:
# system prompt (~500) + static findings (~1000) +
# KB context (~1500, Day 16) + chunk (3000) + response (1000)
# = ~7000 tokens total per LLM call — fits in context window
```

---

## Day 12 — LLM Module + Prompt Engineering I

### UPDATED: Use LiteLLM instead of raw SDK

```python
# app/modules/ai/llm_client.py

from litellm import acompletion
# WHY litellm:
# one interface for 100+ LLM providers
# switch Claude → GPT-4 → Gemini by changing model string only
# built-in retry logic and rate limiting
# no code changes when switching providers

async def call_llm(messages: list[dict], model: str = None) -> str:
    response = await acompletion(
        model=model or settings.LLM_MODEL,
        # settings.LLM_MODEL = "claude-sonnet-4-6" or "gpt-4o"
        # change in .env — no code changes needed
        messages=messages,
        max_tokens=1000,
    )
    return response.choices[0].message.content
```

### Parallel LLM calls with semaphore:
```python
async def review_chunks_parallel(
    chunks: list,
    static_findings: list[dict]
) -> list:
    semaphore = asyncio.Semaphore(5)
    # WHY Semaphore(5):
    # without limit: 20 chunks → 20 simultaneous API calls
    # Anthropic rate limit: ~5-10 concurrent requests
    # Semaphore(5) = max 5 in flight at once
    # 20 chunks ÷ 5 parallel = 4 batches × 7.5s = 30s
    # vs sequential: 20 × 7.5s = 150s

    async def review_one(chunk):
        async with semaphore:
            return await review_chunk(chunk, static_findings)

    results = await asyncio.gather(
        *[review_one(chunk) for chunk in chunks],
        return_exceptions=True
        # WHY return_exceptions=True:
        # one failed chunk doesn't crash all 20
        # failed chunk returns Exception object
        # we handle it: skip failed, keep successful
    )
    return results
```

### Call parallel LLM from Celery task:
```python
# In _run_analysis_async:
llm_findings = await review_chunks_parallel(chunks, static_findings)
# await works because we're in an async function inside asyncio.run()
```

---

## Day 13 — Hybrid Engine + Finding Feedback + Suppression

### Hybrid engine selective LLM:
```python
# Only send chunks with findings to LLM (~70% cost reduction):
chunks_for_llm = [
    chunk for chunk in chunks
    if any(f["file_path"] == chunk.file_path for f in static_findings)
]
# WHY: clean chunks have no static findings — LLM review unlikely to add value
# sending all chunks: 20 LLM calls
# sending only flagged: ~6 LLM calls
# 70% cost reduction, minimal quality loss
```

### Finding source tags:
```python
# Tag each finding with its origin:
"source": "static_only"   # only static/AST tools found this
"source": "llm_only"      # only LLM found this
"source": "hybrid"        # multiple sources agree — boost confidence
# hybrid findings = highest confidence — multiple independent sources agree
```

---

## Day 14 — Scoring + Commit History + GitHub Integration

### Scorer:
```python
def calculate_quality_score(findings: list[dict]) -> int:
    score = 100
    score -= sum(15 for f in findings if f["severity"] == "critical")
    score -= sum(7  for f in findings if f["severity"] == "major")
    score -= sum(3  for f in findings if f["severity"] == "minor")
    score -= sum(1  for f in findings if f["severity"] == "info")
    return max(0, score)  # floor at 0
```

### CommitHistory — score_delta pre-computed:
```python
class CommitHistory(Base):
    score_delta = Column(Integer, nullable=True)
    # WHY pre-compute: current_quality - previous_quality
    # frontend reads score_delta directly — no recalculation
    # "↑ +12 from last commit" — instant, no query needed

    __table_args__ = (
        UniqueConstraint("repository_id", "commit_sha"),
        # WHY unique: prevents duplicate rows when same commit re-analyzed
        # use ON CONFLICT DO NOTHING in insert logic
    )
```

### GitHub Checks API:
```python
def verify_webhook_signature(
    payload_bytes: bytes,
    signature_header: str,
    secret: str
) -> bool:
    import hmac
    expected = hmac.new(
        secret.encode(), payload_bytes, "sha256"
    ).hexdigest()
    received = signature_header.replace("sha256=", "")
    return hmac.compare_digest(expected, received)
    # WHY compare_digest not ==:
    # == leaks timing info — longer common prefix = longer comparison
    # attacker can brute-force the secret with timing attacks
    # compare_digest always takes the same time — timing-safe
```

---

## Day 15 — Knowledge Base + Embeddings + pgvector

### Embeddings via LiteLLM:
```python
from litellm import aembedding

async def generate_embedding(text: str) -> list[float]:
    response = await aembedding(
        model="text-embedding-3-small",
        input=text
    )
    return response.data[0]["embedding"]
# WHY litellm for embeddings too:
# same benefit — swap to Cohere or local models by changing model string
```

### HNSW index — Alembic migration:
```python
# After creating code_chunk table, create new migration:
# alembic revision -m "add_embedding_hnsw_index"

# In the migration file:
def upgrade():
    op.execute("""
        CREATE INDEX ix_code_chunk_embedding
        ON code_chunk USING hnsw (embedding vector_cosine_ops)
        WITH (m = 16, ef_construction = 64)
    """)
    # WHY Alembic not raw psql:
    # part of version history
    # reproducible on any environment
    # rollback: op.execute("DROP INDEX ix_code_chunk_embedding")
```

---

## Day 16 — RAG Pipeline

### Re-ranking formula:
```python
def rerank(results: list[dict]) -> list[dict]:
    for r in results:
        usefulness_ratio = r["times_useful"] / max(r["times_retrieved"], 1)
        r["final_score"] = r["similarity"] * (0.7 + 0.3 * usefulness_ratio)
        # WHY this formula:
        # pure similarity = ignores whether this KB entry was actually helpful
        # a rule retrieved 100 times but useful only 5% = noisy
        # usefulness_ratio downweights it
        # a rule useful 90% of the time gets boosted
    return sorted(results, key=lambda x: x["final_score"], reverse=True)
```

---

## Day 17 — Prompt Engineering II + A/B Testing

### Chain-of-thought v2:
```python
# system_prompt_v2.j2:
# Before producing findings, reason step by step:
# 1. What is this function trying to accomplish?
# 2. What could go wrong?
# 3. Are there security implications?
# 4. Is this maintainable?
# 5. Only now — produce JSON findings.
```

### A/B testing measurement:
```sql
SELECT prompt_version,
       AVG(critical_count + major_count) as avg_findings,
       AVG(quality_score) as avg_score,
       COUNT(*) as sample_size
FROM review
GROUP BY prompt_version
HAVING COUNT(*) >= 10;
-- 10 reviews minimum per version before drawing conclusions
```

---

## Day 18 — LangGraph Pipeline

### UPDATED: Pin LangGraph version + Add LangSmith

```python
# requirements.txt:
langgraph==0.1.5       # pin exact version
langsmith>=0.1.0       # new addition for tracing

# .env:
LANGCHAIN_TRACING_V2=true
LANGCHAIN_API_KEY=your_langsmith_key
LANGCHAIN_PROJECT=githelp

# WHY LangSmith:
# LangGraph auto-sends traces to LangSmith
# see exactly what each node received and returned
# timing per node, state diffs, error locations
# invaluable for debugging graph issues
# free tier is enough for development
```

### State definition:
```python
# app/modules/graph/state.py
from typing import TypedDict

class AnalysisState(TypedDict):
    repository_id: str
    repo_path: str | None
    file_list: list[dict]
    static_findings: list[dict]
    ast_findings: list[dict]
    chunks: list[dict]
    rag_context: dict
    llm_findings: list[dict]
    all_findings: list[dict]
    scores: dict
    error: str | None
    current_stage: str
```

### Error routing:
```python
# Each node sets state["error"] on failure:
def ingest_node(state: AnalysisState) -> AnalysisState:
    try:
        # ... do work ...
        return {**state, "repo_path": path}
    except Exception as e:
        return {**state, "error": str(e)}

# Graph routes to error_node when error is set:
builder.add_conditional_edges(
    "ingest",
    lambda s: "error" if s.get("error") else "static_analysis",
    {"error": "error_node", "static_analysis": "static_analysis_node"}
)
# WHY: instead of try/except swallowing errors
# graph routes to dedicated error handler
# error_node updates DB status = failed, logs cleanly
```

---

## Day 19 — LangGraph Agents (ReAct + Tools)

### check_function_callers — why it works:
```python
def check_function_callers(function_name: str, repository_id: str) -> list[dict]:
    # Uses JSONB containment operator:
    # SELECT * FROM code_chunk
    # WHERE calls @> '["function_name"]'::jsonb
    # WHY fast: JSONB is indexed — this query is O(log n)
    # WHY we stored calls during chunking (Day 11):
    # this tool is exactly why
    # agent can ask "who calls authenticate_user?"
    # finds cross-file security issues that fixed pipelines miss
```

---

# WEEK 3 — Embeddings, RAG, LangGraph & Frontend

---

## Day 20 — API Keys + Next.js Setup + Auth

### UPDATED: Next.js 15 (not 14)

```bash
npx create-next-app@latest frontend --typescript --tailwind --app
# installs Next.js 15 (latest)

# Next.js 15 breaking change vs 14:
# params are now async in page components:

# Next.js 14 (old):
export default function Page({ params }) {
    const { id } = params   // sync access

# Next.js 15 (current):
export default async function Page({ params }) {
    const { id } = await params   // must await
```

### UPDATED: httpOnly cookies instead of localStorage

```typescript
// WRONG (original roadmap — security vulnerability):
localStorage.setItem("access_token", token)
localStorage.getItem("access_token")
// WHY wrong: XSS attack → script reads localStorage → steals token

// CORRECT (updated):
// Backend sets httpOnly cookie on login:
response.set_cookie(
    key="refresh_token",
    value=refresh_token,
    httponly=True,      # JavaScript CANNOT read this
    secure=True,        # HTTPS only
    samesite="lax",     # CSRF protection
    max_age=60 * 60 * 24 * 7  # 7 days
)

// Frontend stores access token in MEMORY only:
// lib/auth.ts
let accessToken: string | null = null

export function setAccessToken(token: string) {
    accessToken = token
    // stored in JS memory — lost on page refresh (intentional)
}

export function getAccessToken(): string | null {
    return accessToken
}

// On page load/refresh: call /auth/refresh
// browser automatically sends httpOnly cookie
// server returns new access token
// store in memory → ready to use

// WHY memory for access token:
// short-lived (15 min) — losing it on refresh is fine
// refresh cookie handles re-authentication silently
// XSS cannot steal memory variables as easily as localStorage
```

### Axios interceptors — updated for cookie auth:
```typescript
// lib/axios.ts
const api = axios.create({
    baseURL: process.env.NEXT_PUBLIC_API_URL,
    withCredentials: true,
    // WHY withCredentials: sends httpOnly cookies automatically
    // without this: browser won't include cookies in requests
})

api.interceptors.request.use(config => {
    const token = getAccessToken()  // from memory, not localStorage
    if (token) config.headers.Authorization = `Bearer ${token}`
    return config
})

api.interceptors.response.use(
    res => res,
    async err => {
        if (err.response?.status === 401) {
            // Try refresh — cookie sent automatically
            const refresh = await axios.post("/api/v1/auth/refresh",
                {}, { withCredentials: true })
            if (refresh.data.access_token) {
                setAccessToken(refresh.data.access_token)
                err.config.headers.Authorization =
                    `Bearer ${refresh.data.access_token}`
                return api(err.config)
            }
            window.location.href = "/login"
        }
        return Promise.reject(err)
    }
)
```

---

## Day 21 — Audit Logs + Next.js Dashboard

### AuditLog — append only rule:
```python
class AuditLog(Base):
    __tablename__ = "audit_log"  # singular (your convention)
    # APPEND ONLY — never UPDATE or DELETE rows in this table
    # WHY: immutable event log for security investigation
    # if you need to "undo" something: write a new compensating event
    # "user.dismissed_finding" then "user.restored_finding"
    # never delete the dismissed_finding row
```

### useEffect cleanup in progress poller:
```typescript
// components/ProgressPoller.tsx
useEffect(() => {
    const id = setInterval(async () => {
        const status = await getTaskStatus(taskId)
        setProgress(status)
        if (status.state === "SUCCESS") {
            clearInterval(id)
            router.push(`/repository/${repoId}`)
        }
    }, 2000)

    return () => clearInterval(id)
    // WHY return cleanup: without this, interval keeps running
    // after component unmounts → memory leak + stale state updates
    // React calls this cleanup when component unmounts
}, [taskId])
```

---

# WEEK 4 — DevOps, Scale, Security & Production

---

## Day 22 — Multi-Language Support + Trend Charts

### Strategy pattern for analyzers:
```python
# Each language gets its own analyzer class:
class PythonAnalyzer:
    def analyze(self, files: list[dict]) -> list[dict]:
        return run_ruff(...) + run_bandit(...) + run_radon(...)

class JavaScriptAnalyzer:
    def analyze(self, files: list[dict]) -> list[dict]:
        return run_eslint(...)

class TypeScriptAnalyzer(JavaScriptAnalyzer):
    def analyze(self, files: list[dict]) -> list[dict]:
        return super().analyze(files) + run_tsc(...)

class AnalyzerRegistry:
    _registry = {
        "python": PythonAnalyzer,
        "javascript": JavaScriptAnalyzer,
        "typescript": TypeScriptAnalyzer,
    }

    @classmethod
    def get(cls, language: str):
        return cls._registry.get(language, NullAnalyzer)()
        # NullAnalyzer = returns [] — safe default for unknown languages
```

---

## Day 23 — Docker Production + Nginx

### Multi-stage Dockerfile for backend:
```dockerfile
# Stage 1: build dependencies
FROM python:3.12-slim as builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Stage 2: runtime only (no build tools)
FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /usr/local/lib/python3.12 /usr/local/lib/python3.12
COPY . .
# WHY multi-stage: build stage includes compilers, headers, pip cache
# runtime stage is ~5-10x smaller (no build tools)
# smaller image = faster deployment, smaller attack surface
```

### Nginx config:
```nginx
# WHY Nginx in front of FastAPI:
# serves Next.js static files directly (no Node.js server needed)
# proxies /api/ to FastAPI (eliminates CORS entirely)
# handles SSL termination
# gzip compression
# /api/ → FastAPI:  no CORS headers needed (same domain)
# /    → Next.js:   static files served directly

location /api/ {
    proxy_pass http://app:8000;
}
location / {
    try_files $uri $uri/ /index.html;
    # try_files: for Next.js routing (SPA fallback)
}
```

---

## Day 24 — CI/CD (GitHub Actions)

### UPDATED: Use v4 actions (not v3)

```yaml
# .github/workflows/ci.yml
jobs:
  backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4      # was v3
      - uses: actions/setup-python@v5  # was v4
      - uses: actions/cache@v4         # was v3
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}

      - run: pip install -r requirements.txt
      - run: ruff check app/          # replaces flake8
      - run: mypy app/ --strict
      - run: python -m pytest tests/ --cov=app --cov-fail-under=70
      - run: bandit -r app/

  frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4    # was v3
      - run: npm ci
      - run: npm run lint
      - run: npx tsc --noEmit
      - run: npm run build

  docker:
    needs: [backend, frontend]
    steps:
      - uses: actions/checkout@v4
      - name: Build backend
        run: docker build -t githelp-backend ./backend
      - name: Build frontend
        run: docker build -t githelp-frontend ./frontend
```

---

## Day 25 — Performance: Caching + Rate Limiting + N+1

### Rate limiting by user ID (not IP):
```python
from slowapi import Limiter
from slowapi.util import get_remote_address

def get_user_id(request: Request) -> str:
    # rate limit by user ID not IP
    # WHY: multiple users behind same IP (office, university)
    # would share rate limit — unfair
    # user ID = per-account limit — correct behavior
    user = getattr(request.state, "user", None)
    if user:
        return str(user.id)
    return get_remote_address(request)  # fallback for unauth

limiter = Limiter(key_func=get_user_id)

@router.post("/")
@limiter.limit("5/hour")
async def submit_repository(...):
    # 5 repo submissions per user per hour
    # 6th request: 429 Too Many Requests
```

### N+1 fix with selectinload:
```python
# WRONG — causes N+1 queries:
reviews = await db.execute(select(Review))
for review in reviews:
    print(review.findings)  # each access = separate SELECT

# CORRECT — 2 queries total:
from sqlalchemy.orm import selectinload
result = await db.execute(
    select(Review)
    .options(selectinload(Review.findings))
    # loads all findings in ONE additional query
    # not one query per review
)
```

---

## Day 26 — Security Hardening

### Security checklist:
```python
# 1. Refresh token rotation (already in Day 4)
# 2. Path traversal guard:
safe_base = Path(tmp_dir).resolve()
resolved = Path(file_path).resolve()
if not str(resolved).startswith(str(safe_base)):
    raise ValueError("Path traversal detected")

# 3. Security headers middleware:
response.headers["X-Content-Type-Options"] = "nosniff"
response.headers["X-Frame-Options"] = "DENY"
response.headers["X-XSS-Protection"] = "1; mode=block"

# 4. Run bandit on your OWN code:
bandit -r app/ -ll  # -ll = only MEDIUM and HIGH severity
# fix all findings before submission
```

---

## Day 27 — Monitoring: Prometheus + Admin Analytics

### Custom metrics:
```python
# app/core/metrics.py
from prometheus_client import Counter, Histogram, Gauge

analysis_duration = Histogram(
    "githelp_analysis_duration_seconds",
    "Time spent on full repository analysis",
    ["mode"]  # "full" or "pr_only"
)

llm_tokens_used = Counter(
    "githelp_llm_tokens_total",
    "Total LLM tokens consumed"
)

findings_per_review = Histogram(
    "githelp_findings_per_review",
    "Distribution of findings count per review"
)

active_analyses = Gauge(
    "githelp_active_analyses",
    "Currently running analyses"
)

# Use in code:
with analysis_duration.labels(mode="full").time():
    result = run_analysis(...)
# .time() = context manager that records duration automatically
```

---

## Day 28 — Cloud Deployment + Self-Review + Final Docs

### Deploy to Railway or Render:
```bash
# Railway:
railway login
railway init
railway up
# connects GitHub repo → auto-deploys on push to main
# HTTPS via Let's Encrypt — automatic
# PostgreSQL + Redis as Railway services

# Environment variables to set:
DATABASE_URL=postgresql+asyncpg://...
REDIS_URL=redis://...
SECRET_KEY=your-secret-key-here
LLM_MODEL=claude-sonnet-4-6
ANTHROPIC_API_KEY=sk-ant-...
LANGCHAIN_TRACING_V2=true
LANGCHAIN_API_KEY=...
```

### BONUS: MCP Server wrapper (after everything works):
```python
# githelp_mcp_server.py
# Exposes GitHelp as a tool for Claude Desktop, Cursor, etc.

from mcp import FastMCP
mcp = FastMCP("GitHelp")

@mcp.tool()
async def analyze_repository(url: str) -> dict:
    """Analyze a GitHub repository and return findings with scores."""
    # calls your existing FastAPI endpoint
    response = await http_client.post(
        f"{settings.API_URL}/api/v1/repository",
        json={"url": url},
        headers={"Authorization": f"Bearer {api_key}"}
    )
    task_id = response.json()["task_id"]
    # poll until complete, return findings
    return findings

# WHY do this LAST (Day 28+):
# MCP wrapper is only valuable if the engine underneath works perfectly
# build the engine first — 50 lines of MCP on top at the end
```

---

## DB Model Rollout Timeline

| Day | Module Folder | Model Class | Why Introduced |
|-----|--------------|-------------|----------------|
| 2 | user/ | User | auth foundation |
| 4 | repository/ | Repository | POST /repository |
| 6 | review/ | Review | analysis task creates review |
| 10 | finding/ | Finding | store analysis results |
| 11 | code_chunk/ | CodeChunk | chunking + embeddings |
| 13 | finding_feedback/ | FindingFeedback | feedback loop |
| 14 | commit_history/ | CommitHistory | trend tracking |
| 15 | knowledge_base/ | KnowledgeBase | RAG retrieval |
| 20 | api_key/ | ApiKey | programmatic access |
| 21 | audit_log/ | AuditLog | compliance + security |

---

## Known Issues & Lessons Learned

| # | Issue | Correct Approach |
|---|-------|-----------------|
| 1 | python-jose outdated | Use PyJWT |
| 2 | passlib in maintenance mode | Use pwdlib[argon2] |
| 3 | pylint + flake8 slow | Use ruff (100x faster) |
| 4 | localStorage for tokens | httpOnly cookies for refresh, memory for access |
| 5 | Next.js 14 | Use Next.js 15 (async params) |
| 6 | GitHub Actions v3 | Use v4 (checkout, cache, setup-python) |
| 7 | LangGraph unpinned | Pin exact version (breaking changes between releases) |
| 8 | No LLM provider flexibility | Use LiteLLM (swap providers by changing model string) |
| 9 | sync Redis in async code | redis.asyncio submodule (same package) |
| 10 | Manual event_loop fixture | Remove — asyncio_mode=auto handles it |
| 11 | Nested transaction + rollback | Let service commit freely in tests |
| 12 | Login with json= | Use data= (OAuth2PasswordRequestForm) |
| 13 | Login field "email" | Field is "username" (OAuth2 spec) |
| 14 | Register returns tokens | Register returns user data, login returns tokens |
| 15 | pytest not python -m pytest | Always python -m pytest (uses venv) |
| 16 | prefix in APIRouter | prefix belongs in main.py only |
| 17 | Logging everywhere | Log based on side effects, not in every function |
| 18 | init_db.py create_all | Alembic for all migrations (you already did this) |
| 19 | Sync Redis in Celery | Use import redis (sync submodule) for Celery tasks |
| 20 | clone called directly in async | Use run_in_executor for sync subprocess calls |

---

## Performance Expectations By Day

```
Day 8  (clone only):              ~15s per repo
Day 9  (+ static analysis):       ~30s per repo
Day 11 (+ chunking):              ~35s per repo
Day 12 (+ LLM sequential):        ~3 minutes per repo
Day 12 (+ LLM parallel semaphore=5): ~45s per repo
Day 13 (+ selective LLM 30%):     ~25s per repo
Day 14 (PR mode, changed files):  ~10s per PR review
Day 28 (all optimizations):
  Full repo first time:   45-90s
  PR analysis:            10-20s   ← main use case
  Re-analysis (cached):   20-30s
```
