# GitHelp — AI Powered Code Review System
## Updated 4-Week Day-by-Day Build Plan
**Student:** Nilesh Kumar | **Enrollment:** 06116403222 | **Supervisor:** Prof. Anurag Jain
**University:** USICT, Sector-14 Dwarka, Delhi-110075

---

## Your Architecture: Module-Based Structure

Every feature lives in its own folder under `app/modules/`. Each module owns its model, schemas, router, and service. New DB tables are introduced only when the feature that needs them is being built — not all at once.

```
backend/
├── app/
│   ├── common/
│   │   ├── base.py          ← DeclarativeBase (shared by all models)
│   │   └── enums.py         ← All enums (shared by all modules)
│   │
│   ├── core/
│   │   ├── config.py        ← Pydantic Settings (env vars)
│   │   ├── database.py      ← engine, AsyncSession, get_db()
│   │   ├── dependencies.py  ← get_current_user, require_role()
│   │   └── security.py      ← hash_password, create_token, verify_token
│   │
│   ├── modules/
│   │   ├── users/           ← Day 2 ✅ (you're here)
│   │   │   ├── model.py
│   │   │   ├── schemas.py
│   │   │   ├── router.py
│   │   │   └── service.py
│   │   │
│   │   ├── auth/            ← Day 3 (current) / Day 4
│   │   │   ├── schemas.py
│   │   │   ├── router.py
│   │   │   └── service.py   ← no model.py — auth uses User model
│   │   │
│   │   ├── repository/      ← Day 4 (empty folder now)
│   │   │   ├── model.py
│   │   │   ├── schemas.py
│   │   │   ├── router.py
│   │   │   └── service.py
│   │   │
│   │   ├── reviews/         ← Day 6
│   │   ├── findings/        ← Day 10
│   │   ├── code_chunks/     ← Day 11
│   │   ├── finding_feedback/ ← Day 13
│   │   ├── commit_history/  ← Day 14
│   │   ├── knowledge_base/  ← Day 15
│   │   ├── api_keys/        ← Day 20
│   │   ├── audit_logs/      ← Day 21
│   │   └── tasks/           ← Day 5 (no model — Celery tasks only)
│   │
│   └── main.py
│
├── init_db.py               ← runs Base.metadata.create_all()
├── Dockerfile
├── requirements.txt
├── .env
└── .env.example

frontend/                    ← Next.js (Day 20)
docker-compose.yml
```

### Module Model Rollout Order
Models appear exactly when their feature is built:

```
Day 2  → users/model.py          ✅ done
Day 4  → repository/model.py
Day 6  → reviews/model.py
Day 10 → findings/model.py
Day 11 → code_chunks/model.py
Day 13 → finding_feedback/model.py
Day 14 → commit_history/model.py
Day 15 → knowledge_base/model.py
Day 20 → api_keys/model.py
Day 21 → audit_logs/model.py
```

---

# WEEK 1 — Infrastructure, Auth & Core API
*Goal: Docker running, User + Auth + Repository modules complete with real JWT auth and wired task queue.*

---

## Day 1 — Docker + Skeleton ✅ DONE

You already have this. Docker running, FastAPI up, `common/` and `core/` files created, `/docs` accessible.

**What you built:**
- `docker-compose.yml` with app, postgres (pgvector), redis
- `app/common/base.py` — DeclarativeBase
- `app/common/enums.py` — all enums
- `app/core/config.py`, `database.py`, `security.py`, `dependencies.py`
- `app/main.py` with health endpoint
- `init_db.py` for creating tables

---

## Day 2 — Users Module ✅ DONE

You already have `modules/users/` with `model.py`, `schemas.py`, `router.py`.

**What you built:**
- `users/model.py` — User SQLAlchemy model (UUID pk, email, hashed_password, role, OAuth fields, refresh token)
- `users/schemas.py` — `UserRegisterRequest`, `UserLoginRequest`, `UserUpdateRequest`, `UserRoleUpdateRequest`, `UserResponse`
- `users/router.py` — stub endpoints: GET /me, PATCH /me, POST /logout

**Today complete when:** `GET /api/v1/users/me` returns 501 (stub), shows in `/docs`.

---

## Day 3 — Auth Module Schemas + Routers (Current Day)

### Goal
Finish all auth + user schemas and routers returning stub data. `/docs` shows every endpoint grouped and typed. No 500 errors anywhere.

### What You Build

**`app/modules/auth/schemas.py`** — already partially done:
```python
# Complete these if not already done:
class UserRegisterRequest     # email, password, full_name?
class UserLoginRequest        # email, password (no min_length — existing users)
class TokenResponse           # access_token, refresh_token, token_type, expires_in
class RefreshTokenRequest     # refresh_token
class OAuthCallbackResponse   # access_token, refresh_token, is_new_user
```

**`app/modules/auth/router.py`** — stub all endpoints:
```
POST /api/v1/auth/register    → return dummy TokenResponse
POST /api/v1/auth/login       → return dummy TokenResponse
POST /api/v1/auth/refresh     → return dummy TokenResponse
GET  /api/v1/auth/github/callback → return dummy OAuthCallbackResponse
```

**`app/modules/users/router.py`** — complete stubs:
```
GET    /api/v1/users/me       → return dummy UserResponse
PATCH  /api/v1/users/me       → return dummy UserResponse
POST   /api/v1/users/logout   → return {"message": "logged out"}
```

**`app/main.py`** — register both routers:
```python
app.include_router(auth_router,  prefix="/api/v1/auth",  tags=["Auth"])
app.include_router(users_router, prefix="/api/v1/users", tags=["Users"])
```

**`app/common/enums.py`** — make sure all enums used by ALL future modules are here now:
`UserRole`, `RepoStatus`, `RepoProvider`, `AnalysisMode`, `ReviewStatus`,
`FindingSeverity`, `FindingCategory`, `FindingSource`, `FindingStatus`,
`ChunkType`, `KBEntryType`, `AuditAction`

**Why all enums now?** Enums are shared across modules. `findings/model.py` imports `FindingSeverity` from `common/enums.py`. If you add enums later, you'll be editing this file anyway — define the complete set today so future modules just import.

### Tech & Concepts
- **Separate schemas per operation**: `UserRegisterRequest` has `password` required. `UserUpdateRequest` has only `full_name`. They look similar but serve completely different contracts — don't merge them
- **`from_attributes=True`** on response schemas: `ConfigDict(from_attributes=True)` lets Pydantic read SQLAlchemy ORM objects directly — `UserResponse.model_validate(db_user)`
- **Stub pattern**: `raise HTTPException(status_code=501, detail="Not implemented yet. Wait for Day 4.")` — better than 500 because it's intentional and shows in docs
- **Router prefix vs tag**: `prefix="/api/v1/auth"` controls the URL. `tags=["Auth"]` controls the Swagger group. Both needed

### Milestone
`/docs` shows all auth + user endpoints grouped correctly. Every endpoint returns 200 with valid schema (dummy data) or 501 (stub). No 500 errors. `PATCH /api/v1/users/me` no longer crashes.

---

## Day 4 — Repository Module + Real JWT Auth

### Goal
`repository` module fully scaffolded with its model and real JWT authentication implemented. Register → Login → get real JWT → use it on protected routes.

### What You Build

**`app/modules/repository/model.py`** ← first new model this week:
```python
class Repository(Base):
    __tablename__ = "repositories"
    id             = Column(UUID, pk)
    owner_id       = Column(UUID, FK → users.id, CASCADE)
    url            = Column(String(2048))
    name           = Column(String(255), nullable)
    provider       = Column(Enum(RepoProvider))
    is_private     = Column(Boolean)
    default_branch = Column(String(100))
    celery_task_id = Column(String(255), nullable, index)
    status         = Column(Enum(RepoStatus), index)
    error_message  = Column(Text, nullable)
    metadata_      = Column("metadata", JSONB, nullable)  # metadata_ avoids SQLAlchemy reserved name
    analysis_config= Column(JSONB, nullable)
    created_at     = Column(TIMESTAMPTZ, server_default)
    updated_at     = Column(TIMESTAMPTZ, server_default, onupdate)  # BOTH server_default AND onupdate
```

**`app/modules/repository/schemas.py`**:
```python
class RepoSubmitRequest    # url (validated), is_private, analysis_config
class RepoResponse         # all fields, from_attributes=True
class RepoSummaryResponse  # lightweight list view
class TaskStatusResponse   # state, progress, stage, result, error
```

**`app/modules/repository/router.py`** — stubs:
```
POST /api/v1/repos           → 501
GET  /api/v1/repos           → []
GET  /api/v1/repos/{id}      → 501
POST /api/v1/repos/{id}/reanalyze → 501
GET  /api/v1/repos/{id}/history   → 501
```

**`app/core/security.py`** — real implementations:
```python
hash_password(plain: str) -> str          # passlib bcrypt
verify_password(plain, hashed) -> bool
create_access_token(data, expires=15min)  # python-jose
create_refresh_token(data, expires=7days)
decode_token(token) -> dict               # raises 401 if invalid/expired
```

**`app/core/dependencies.py`** — real implementation:
```python
async def get_current_user(token: str = Depends(oauth2_scheme), db = Depends(get_db)) -> User:
    payload = decode_token(token)           # raises 401 if bad
    user = await db.get(User, payload["sub"])
    if not user or not user.is_active:
        raise HTTPException(401)
    return user

def require_role(*roles: UserRole):         # factory function
    async def check(user = Depends(get_current_user)):
        if user.role not in roles:
            raise HTTPException(403)
        return user
    return check
```

**`app/modules/auth/service.py`** — real implementations:
```python
register_user(request, db) -> TokenResponse
login_user(request, db) -> TokenResponse
refresh_tokens(refresh_token, db) -> TokenResponse  # rotation: invalidate old hash, issue new
```

**`app/modules/auth/router.py`** — replace stubs with real calls to service

**`init_db.py`** — add Repository import so table is created:
```python
from app.modules.users.model import User
from app.modules.repository.model import Repository
Base.metadata.create_all(bind=engine)
```

### Tech & Concepts
- **`metadata_` column name**: `Column("metadata", JSONB)` — store as `metadata_` in Python but `metadata` in DB. `metadata` is a reserved attribute on SQLAlchemy's `DeclarativeBase` — using it directly causes silent bugs
- **`server_default=func.now()` on `updated_at`**: without `server_default`, `updated_at` is NULL on first INSERT. `onupdate` only fires on UPDATE. Both are needed
- **`updated_at` pattern**: every table with both needs:
  ```python
  updated_at = Column(DateTime(timezone=True), server_default=func.now(), onupdate=func.now())
  ```
- **python-jose**: `jose.jwt.encode({"sub": str(user.id), "role": user.role, "exp": ...}, SECRET_KEY, algorithm="HS256")`
- **Refresh token rotation**: store `bcrypt(refresh_token)` as `refresh_token_hash`. On refresh: verify hash → issue new access + refresh → update hash. If old token used again after rotation → both invalidated (compromise detected)
- **`oauth2_scheme`**: `OAuth2PasswordBearer(tokenUrl="/api/v1/auth/login")` — tells Swagger to show the Authorize button

### Milestone
`POST /api/v1/auth/register` → creates User in DB → returns real JWT. `GET /api/v1/users/me` with Bearer token → returns real user data. Same call without token → 401. `GET /api/v1/repos` with valid token → returns `[]`. Without token → 401.

---

## Day 5 — Celery + Redis + Task Queue

### Goal
Background task queue wired up. Submit a Celery task from a FastAPI endpoint, watch it in Flower, see result come back.

### What You Build

**`app/modules/tasks/`** — no model (tasks don't need a DB table):
```
app/modules/tasks/
├── celery_app.py      ← Celery instance
├── analysis_tasks.py  ← analyze_repository task (stub)
├── embedding_tasks.py ← embed_chunks task (stub)
└── webhook_tasks.py   ← process_github_webhook task (stub)
```

**`app/modules/tasks/celery_app.py`**:
```python
from celery import Celery
celery_app = Celery(
    "githelp",
    broker=settings.REDIS_URL,
    backend=settings.REDIS_URL,
    include=["app.modules.tasks.analysis_tasks", ...]
)
celery_app.conf.task_acks_late = True           # re-queue if worker crashes
celery_app.conf.task_reject_on_worker_lost = True
celery_app.conf.task_routes = {
    "app.modules.tasks.analysis_tasks.*": {"queue": "analysis"},
    "app.modules.tasks.embedding_tasks.*": {"queue": "embeddings"},
}
```

**`app/modules/tasks/analysis_tasks.py`**:
```python
@celery_app.task(bind=True, max_retries=3)
def analyze_repository(self, repository_id: str):
    self.update_state(state="STARTED", meta={"progress": 0, "stage": "starting"})
    import time; time.sleep(3)   # stub — real analysis replaces this on Day 8
    self.update_state(state="STARTED", meta={"progress": 100, "stage": "complete"})
    return {"status": "completed", "repository_id": repository_id}
```

**`app/core/cache.py`**:
```python
redis_client = redis.Redis.from_url(settings.REDIS_URL)

def cache_get(key: str) -> str | None
def cache_set(key: str, value: str, ttl: int = 300)
def cache_delete(key: str)
```

**`docker-compose.yml`** — add two new services:
```yaml
celery_worker:
  build: ./backend
  command: celery -A app.modules.tasks.celery_app worker --loglevel=info -Q analysis,embeddings
  depends_on: [redis, postgres]
  env_file: backend/.env

flower:
  build: ./backend
  command: celery -A app.modules.tasks.celery_app flower --port=5555
  ports: ["5555:5555"]
  depends_on: [redis]
```

**`app/modules/tasks/router.py`** — task status endpoint:
```
GET /api/v1/tasks/{task_id}/status
→ query AsyncResult(task_id)
→ return {state, progress, stage, result, error}
```

### Tech & Concepts
- **Celery broker vs backend**: broker (Redis) = where task messages queue. Backend (Redis) = where task results are stored. Both are Redis here but serve different purposes
- **`task_acks_late=True`**: by default, Celery removes the task from the queue when a worker picks it up — if the worker crashes mid-execution, the task is lost. `acks_late=True` removes the task only after successful completion
- **`bind=True`**: gives the task function access to `self` — needed for `self.update_state()` (progress reporting) and `self.retry()`
- **`self.update_state(state="STARTED", meta={...})`**: stores arbitrary progress data in Redis backend. The `GET /tasks/{id}/status` endpoint reads this via `AsyncResult(task_id).info`
- **Exponential backoff**: `self.retry(countdown=2**self.request.retries)` — waits 1s, 2s, 4s between retries. Prevents hammering a failing dependency

### Milestone
`docker-compose up` → 4 healthy containers (app, postgres, redis, celery_worker, flower). Call any endpoint → Flower at `:5555` shows task. Task transitions PENDING → STARTED → SUCCESS. `GET /api/v1/tasks/{id}/status` returns progress metadata.

---

## Day 6 — Wire API → Queue → DB + Reviews Module Skeleton

### Goal
`POST /api/v1/repos` creates a DB row, schedules a Celery task, and returns a task ID. Polling that task ID shows real progress. Reviews module created with its model.

### What You Build

**`app/modules/repository/service.py`** — real DB operations:
```python
create_repository(owner_id, request, db) -> Repository
get_repository(repo_id, owner_id, db) -> Repository
list_repositories(owner_id, db) -> list[Repository]
update_repository_status(repo_id, status, task_id, db)
```

**`app/modules/repository/router.py`** — replace stubs:
```python
@router.post("/", status_code=202)
async def submit_repository(
    request: RepoSubmitRequest,
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    # 1. Check not already analyzing
    # 2. Create Repository row (status=pending)
    # 3. Schedule Celery task
    # 4. Update Repository with celery_task_id
    # 5. Return {repo_id, task_id}
    task = analyze_repository.delay(str(repo.id))
    await repo_service.update_repository_status(repo.id, status="pending", task_id=task.id, db=db)
    return {"repository_id": str(repo.id), "task_id": task.id}
```

**`app/modules/reviews/model.py`** ← second new model:
```python
class Review(Base):
    __tablename__ = "reviews"
    id                    = Column(UUID, pk)
    repository_id         = Column(UUID, FK → repositories.id, CASCADE)
    commit_sha            = Column(String(40), nullable)
    analysis_mode         = Column(Enum(AnalysisMode), default="full")
    status                = Column(Enum(ReviewStatus))
    quality_score         = Column(Integer, nullable)
    security_score        = Column(Integer, nullable)
    performance_score     = Column(Integer, nullable)
    maintainability_score = Column(Integer, nullable)
    critical_count        = Column(Integer, default=0)   # denormalized
    major_count           = Column(Integer, default=0)   # denormalized
    minor_count           = Column(Integer, default=0)   # denormalized
    info_count            = Column(Integer, default=0)   # denormalized
    summary               = Column(Text, nullable)
    top_issues            = Column(JSONB, nullable)
    total_tokens_used     = Column(Integer, default=0)
    llm_calls_made        = Column(Integer, default=0)
    analysis_duration_ms  = Column(Integer, nullable)
    prompt_version        = Column(String(50), nullable)  # for A/B testing Day 17
    check_run_id          = Column(String(100), nullable) # GitHub Checks API Day 14
    checks_posted         = Column(Boolean, default=False)
    pr_number             = Column(Integer, nullable)
    pr_url                = Column(String(2048), nullable)
    created_at            = Column(TIMESTAMPTZ, server_default)
    completed_at          = Column(TIMESTAMPTZ, nullable)
```

**`app/modules/reviews/schemas.py`**:
```python
class ReviewResponse        # full with findings list
class ReviewSummaryResponse # lightweight list view
class ReanalysisRequest     # mode, config override
```

**`app/modules/reviews/router.py`** — stubs:
```
GET  /api/v1/reviews/{id}          → 501
GET  /api/v1/reviews/{id}/findings → []
```

Update `analysis_tasks.py` to create a Review row:
```python
def analyze_repository(self, repository_id: str):
    with SessionLocal() as db:   # Celery tasks manage their own sessions
        repo = db.get(Repository, repository_id)
        repo.status = RepoStatus.analyzing
        db.commit()
        # create stub Review
        review = Review(repository_id=repository_id, status=ReviewStatus.in_progress)
        db.add(review); db.commit()
        time.sleep(3)  # stub
        review.status = ReviewStatus.completed
        review.quality_score = 75
        repo.status = RepoStatus.completed
        db.commit()
```

**Cache-aside on GET /repos/{id}**:
```python
async def get_repository(repo_id, owner_id, db):
    cached = cache_get(f"repo:{repo_id}")
    if cached: return json.loads(cached)
    repo = db.query(...).first()
    cache_set(f"repo:{repo_id}", repo.json(), ttl=60)
    return repo
```

### Tech & Concepts
- **Celery tasks use `SessionLocal()` directly**: tasks run in a separate process — they cannot use FastAPI's `get_db()` dependency injection. Create sessions with a plain `with SessionLocal() as db:` context manager
- **202 Accepted vs 200 OK**: `POST /repos` returns 202 because processing happens asynchronously. 200 = done, 202 = accepted and processing
- **Idempotency guard**: before creating a new task, check `repo.status IN (pending, cloning, analyzing)` — don't submit twice
- **Denormalized counts on Review**: `critical_count`, `major_count` etc. are copied from findings at completion time. Dashboard reads these directly instead of `COUNT(*) GROUP BY severity` — much faster

### Milestone
POST /api/v1/repos → 202 with `{repository_id, task_id}` → poll GET /tasks/{task_id}/status → watch `progress` go 0→100 → GET /api/v1/repos/{id} shows `status=completed` → GET /api/v1/reviews/{id} returns stub review with `quality_score=75`.

---

## Day 7 — Tests + Structured Logging + Health Checks

### Goal
pytest suite passing. Every log line is structured JSON with correlation ID. Two health endpoints. README runnable by the professor.

### What You Build

**`app/core/middleware.py`**:
```python
class CorrelationIDMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        correlation_id = request.headers.get("X-Correlation-ID", str(uuid4()))
        request.state.correlation_id = correlation_id
        # inject into logging context
        response = await call_next(request)
        response.headers["X-Correlation-ID"] = correlation_id
        return response
```

**Structured JSON logging** (`python-json-logger`):
Every log line: `{"timestamp", "level", "correlation_id", "path", "method", "duration_ms", "user_id", "message"}`

**Health endpoints** in `main.py`:
```python
GET /health/live   → {"status": "ok"}                         # just alive
GET /health/ready  → check DB + Redis + return all statuses   # ready for traffic
```

**Test structure**:
```
tests/
├── conftest.py                   ← fixtures: test DB, test client, mock user
├── unit/
│   ├── test_security.py          ← hash_password, verify_password, create_token, expiry
│   └── test_schemas.py           ← Pydantic validation edge cases
└── integration/
    ├── test_auth_flow.py         ← register → login → JWT → refresh → rotate
    └── test_repo_flow.py         ← submit → poll → completed
```

**`README.md`** — runnable from scratch:
```
Prerequisites, clone, cp .env.example .env, docker-compose up, init_db.py, /docs URL, run tests
```

### Tech & Concepts
- **Correlation IDs**: every request generates a UUID that appears in every log line for that request — essential for debugging ("show me all logs for request abc-123")
- **`python-json-logger`**: `logging.Formatter` subclass that outputs `{"message": "...", "level": "INFO", ...}` — parseable by Elasticsearch, Loki, CloudWatch
- **Liveness vs Readiness**: liveness = is the process running (K8s restarts on fail). Readiness = is it ready to accept traffic (K8s stops routing on fail). A service can be alive but not ready (DB connection lost)
- **`conftest.py` fixtures**: `@pytest.fixture` with `scope="session"` creates test DB once per test run. `scope="function"` (default) creates fresh data per test

### Milestone
`pytest tests/ -v` → all tests green. `/health/ready` returns `{"database": true, "redis": true}`. Docker logs show JSON per line. Push to GitHub → no broken tests.

---

# WEEK 2 — Code Intelligence & AI Engine
*Goal: Git ingestion → static analysis → AST → chunking → LLM integration → hybrid engine → scoring → GitHub integration. A complete analysis pipeline.*

---

## Day 8 — Git Ingestion Module

### Goal
Clone a real GitHub repo, list its files, get its diff, confirm cleanup happens even on failure.

### What You Build

New folder (no model needed — this is a service, not stored data):
```
app/modules/ingestion/
├── git_cloner.py    ← clone, diff, cleanup
└── file_extractor.py ← detect language, read safely
```

**`git_cloner.py`**:
```python
validate_url(url: str) -> str             # regex for HTTPS GitHub/GitLab, no path traversal
clone_repository(url, token=None) -> str  # returns tmp path, depth=1 shallow clone
get_file_list(repo_path) -> list[dict]    # [{path, language, size_kb}], skip .git/
get_latest_diff(repo_path) -> list[str]   # changed file paths for incremental mode
cleanup_repository(repo_path)            # shutil.rmtree — ALWAYS in finally block
```

**`file_extractor.py`**:
```python
detect_language(filepath) -> str         # extension map: .py→python, .js→javascript
get_files_by_language(files) -> dict     # {"python": [...], "javascript": [...]}
read_file_safely(path) -> str | None     # UTF-8 then latin-1 fallback, None if binary
```

**`.githelpignore` parsing**: read from `analysis_config.exclude_patterns` → `fnmatch` filter on file list.

**Path traversal defense**:
```python
safe_base = Path(tmp_dir).resolve()
for extracted_path in file_list:
    full = (safe_base / extracted_path).resolve()
    if not str(full).startswith(str(safe_base)):
        raise ValueError("Path traversal detected")
```

Update `analysis_tasks.analyze_repository` to call `clone_repository()` with a real repo URL, log the file count, then call `cleanup_repository()` in `finally`.

### Tech & Concepts
- **`depth=1` shallow clone**: only downloads the latest commit snapshot. `git clone --depth 1 https://...` — 10-100x faster for large repos (no history)
- **`finally` block**: cleanup MUST happen even if analysis raises an exception — `/tmp` fills up fast and crashes the server
- **Path traversal**: `git archive` from a malicious repo could contain paths like `../../app/core/security.py` — overwriting your own files. `Path.resolve()` normalizes then checks prefix
- **`shell=False` always**: `subprocess.run(["git", "clone", url, path])` not `subprocess.run(f"git clone {url} {path}", shell=True)` — shell=True enables injection if url contains `;rm -rf /`

### Milestone
`clone_repository("https://github.com/tiangolo/fastapi")` → temp dir exists → files listed by language → cleanup → temp dir gone. Introduce deliberate exception → cleanup still runs → confirmed in logs.

---

## Day 9 — Static Analysis Module

### Goal
`analyze_file(path)` returns normalized findings from 6 tools. Every finding has the same structure regardless of which tool produced it.

### What You Build

```
app/modules/analysis/
└── static_analyzer.py
```

```python
run_pylint(filepath) -> list[dict]        # --output-format=json, parse, normalize
run_flake8(filepath) -> list[dict]        # parse "file:line:col: code message"
run_radon(filepath) -> list[dict]         # cc -j, complexity>10=major, >20=critical
run_bandit(filepath) -> list[dict]        # -f json, map bandit severity to ours
run_pip_audit(requirements_path) -> list[dict]  # -f json, CVE IDs in rule_ids
run_secrets_scan(repo_path) -> list[dict] # regex: AWS keys, GitHub tokens, private keys

normalize_finding(tool, raw) -> dict:     # consistent output regardless of tool:
    {file_path, line_start, severity, category,
     source="static_only", rule_ids, title, explanation, suggestion}

analyze_file(filepath) -> list[dict]      # calls all tools, deduplicates by (file, line, rule_id)
```

All subprocess calls — never `shell=True`:
```python
result = subprocess.run(
    ["pylint", "--output-format=json", filepath],
    capture_output=True, timeout=60, text=True
)
```

Test fixtures needed:
```
tests/fixtures/
├── sample_bad_code.py      ← bare except, mutable default, long function
├── sample_good_code.py     ← clean, passes all tools
└── sample_security_issues.py ← hardcoded API_KEY = "sk-...", eval(), shell=True
```

### Tech & Concepts
- **Why 6 tools**: each catches different things. pylint = style + unused vars. flake8 = PEP8. radon = complexity. bandit = security patterns. pip-audit = known CVEs. secrets scan = hardcoded credentials
- **Normalization**: every tool outputs differently. pylint outputs JSON. flake8 outputs `file.py:12:4: E302 message`. bandit outputs JSON with different severity names. Our job is one consistent schema coming out regardless of tool
- **Deduplication**: `SELECT DISTINCT (file, line, rule_id)` logic — pylint and flake8 both flag E501 line too long. Keep one, mark source = "static_only"
- **`subprocess.run` with `timeout=60`**: some tools hang on certain files (malformed code). 60 second timeout kills the process and continues analysis

### Milestone
`analyze_file("tests/fixtures/sample_bad_code.py")` → at least 5 findings from at least 3 tools. `run_secrets_scan("tests/fixtures/sample_security_issues.py")` → flags hardcoded API key as critical. `run_pip_audit` with a known-vulnerable requirements.txt → CVE finding appears.

---

## Day 10 — AST Analysis + Findings Module

### Goal
AST analysis detects structural issues invisible to linters. `findings` module and its model created — findings now actually stored in DB.

### What You Build

**`app/modules/analysis/ast_analyzer.py`**:
```python
class ASTVisitor(ast.NodeVisitor):
    # Detects:
    # Functions > 50 lines          → maintainability
    # Nesting depth > 4             → maintainability (track with depth counter)
    # Classes with > 10 methods     → architecture ("god class")
    # Bare except: clauses          → reliability
    # Mutable default arguments     → bug (def f(x=[]) — list created once, shared)
    # Magic numbers                 → style
    # Functions with > 5 parameters → maintainability

analyze_file(filepath) -> list[dict]
```

**`app/modules/analysis/dependency_mapper.py`**:
```python
build_import_graph(file_list) -> networkx.DiGraph   # node=module, edge=imports
detect_circular_imports(graph) -> list[list[str]]   # networkx.simple_cycles()
```

**`app/modules/findings/model.py`** ← third new model:
```python
class Finding(Base):
    __tablename__ = "findings"
    id               = Column(UUID, pk)
    review_id        = Column(UUID, FK → reviews.id, CASCADE)
    file_path        = Column(String(1024))
    line_start       = Column(Integer, nullable)
    line_end         = Column(Integer, nullable)
    function_name    = Column(String(255), nullable)
    class_name       = Column(String(255), nullable)
    severity         = Column(Enum(FindingSeverity), index)
    severity_rank    = Column(Integer)  # critical=1,major=2,minor=3,info=4 — reliable ORDER BY
    category         = Column(Enum(FindingCategory))
    source           = Column(Enum(FindingSource))
    rule_ids         = Column(JSONB, server_default="'[]'::jsonb", default=list)
    title            = Column(String(500))
    explanation      = Column(Text)
    suggestion       = Column(Text)
    code_example     = Column(Text, nullable)
    ai_confidence    = Column(Float, nullable)
    status           = Column(Enum(FindingStatus), default="open", index)
    dismissed_reason = Column(Text, nullable)
    is_suppressed    = Column(Boolean, default=False)
    suppressed_by    = Column(String(100), nullable)
    embedding_id     = Column(UUID, FK → code_chunks.id, SET NULL, nullable)
    created_at       = Column(TIMESTAMPTZ, server_default)
    resolved_at      = Column(TIMESTAMPTZ, nullable)
```

**`app/modules/findings/schemas.py`**:
```python
class FindingCreate            # internal only — never expose via API
class FindingStatusUpdateRequest  # with model_validator: dismissed requires dismissed_reason
class FindingFilterParams      # severity, category, status, file_path, page, page_size
class FindingResponse          # from_attributes=True
class FindingSummary           # compact for embedding inside ReviewResponse
```

**`app/modules/findings/service.py`**:
```python
SEVERITY_RANK = {"critical": 1, "major": 2, "minor": 3, "info": 4}

bulk_create_findings(review_id, findings: list[FindingCreate], db)
    # auto-set severity_rank from SEVERITY_RANK dict
    # apply .githelpignore suppression patterns

update_finding_status(finding_id, status, dismissed_reason, user_id, db)
```

**`app/modules/findings/router.py`**:
```
PATCH /api/v1/findings/{id}/status   → update status, write audit log
POST  /api/v1/findings/{id}/feedback → 501 (Day 13)
```

### Tech & Concepts
- **`severity_rank` int alongside severity enum**: PostgreSQL ENUMs sort alphabetically — `critical, info, major, minor`. severity_rank gives correct sort: 1,2,3,4. Always ORDER BY severity_rank, not severity
- **`server_default="'[]'::jsonb"`**: DB-level default on `rule_ids`. `default=list` is Python-only — if someone inserts directly via SQL, `default=list` doesn't fire. `server_default` fires at the DB level always
- **`FindingCreate` is internal**: this schema is used by the hybrid engine to create findings. Never expose it via API — that would let anyone inject findings. `FindingCreate` has no status, no suppression fields — only the analysis engine sets those
- **Nesting depth tracking in AST**: maintain a `depth` counter, `generic_visit` increments it for If/For/While/Try nodes, decrements on exit. Max observed depth = cognitive complexity

### Milestone
`ast_analyzer.analyze_file("tests/fixtures/sample_bad_code.py")` → detects bare except, mutable default, long function. Findings stored in DB via `bulk_create_findings`. `GET /reviews/{id}/findings` returns real findings from DB.

---

## Day 11 — Code Chunks Module + Tokenization

### Goal
`chunk_repository()` splits every file at AST boundaries, counts tokens, stores chunks in DB under 3000 tokens each.

### What You Build

**`app/modules/code_chunks/model.py`** ← fourth new model:
```python
class CodeChunk(Base):
    __tablename__ = "code_chunks"
    id           = Column(UUID, pk)
    repository_id= Column(UUID, FK → repositories.id, CASCADE)
    file_path    = Column(String(1024))
    chunk_type   = Column(Enum(ChunkType))  # function/classdef/module/method
    name         = Column(String(255), nullable)
    start_line   = Column(Integer)
    end_line     = Column(Integer)
    language     = Column(String(50))
    source_code  = Column(Text)
    token_count  = Column(Integer)
    embedding    = Column(Vector(1536), nullable)  # pgvector — NULL until Day 15
    imports_used = Column(JSONB, server_default="'[]'::jsonb", default=list)
    calls        = Column(JSONB, server_default="'[]'::jsonb", default=list)
    complexity   = Column(Integer, nullable)
    created_at   = Column(TIMESTAMPTZ, server_default)
```

Note: HNSW index on `embedding` goes in a manual migration (raw SQL — SQLAlchemy declarative can't express HNSW parameters):
```sql
CREATE INDEX ix_code_chunks_embedding ON code_chunks
USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);
```

**`app/modules/analysis/chunker.py`**:
```python
count_tokens(text, model="gpt-4o") -> int           # tiktoken cl100k_base encoding
chunk_file_by_ast(filepath) -> list[dict]            # one chunk per FunctionDef/ClassDef
    # each chunk: {file_path, chunk_type, name, start_line, end_line,
    #              source_code, token_count, language, imports_used, calls}
    # imports_used: modules imported in scope
    # calls: function names this chunk calls (from ast.Call nodes)
    # if chunk > 3000 tokens: split at line boundaries, mark as module type
chunk_repository(repo_path, repository_id, db) -> list[CodeChunk]  # bulk insert
```

Update `analysis_tasks.analyze_repository`:
```
clone → static_analysis → ast_analysis → chunking → store chunks → cleanup
```

### Tech & Concepts
- **tiktoken**: OpenAI's tokenizer library. `tiktoken.encoding_for_model("gpt-4o").encode(text)` returns a list of integers (token IDs). Length = token count. Same counting as the actual model
- **Why 3000 token limit**: Total LLM call budget per chunk = ~6000 tokens. Breakdown: system prompt (~500) + static findings (~1000) + KB context (~1500, Day 16) + chunk (3000) + response space (1000). Must fit in one call
- **`calls` extraction**: `ast.walk(function_node)` → find `ast.Call` nodes → `node.func.id` for direct calls, `node.func.attr` for method calls. Stored in JSONB — used by the agent tool `check_function_callers` on Day 19
- **Bulk insert**: `db.bulk_save_objects(chunk_objects)` then single `db.commit()` — 100x faster than individual inserts for 500+ chunks

### Milestone
`chunk_repository("/tmp/fastapi_clone")` stores chunks in DB. `SELECT MAX(token_count) FROM code_chunks` → no value exceeds 3000. `SELECT chunk_type, COUNT(*) FROM code_chunks GROUP BY chunk_type` shows distribution across function/classdef/module/method.

---

## Day 12 — LLM Module + Prompt Engineering I

### Goal
`llm_client.review_chunk(chunk, static_findings)` calls a real LLM and returns validated structured JSON findings.

### What You Build

```
app/modules/ai/
├── prompt_builder.py   ← Jinja2 template loading
└── llm_client.py       ← API calls + retry + parse

app/prompts/
├── system_prompt_v1.j2
├── user_prompt.j2
└── few_shot_examples.j2
```

**`app/prompts/system_prompt_v1.j2`**:
```
You are a senior software engineer conducting a formal code review.
Identify real bugs, security vulnerabilities, and maintainability issues.
Do NOT flag style preferences or TODO comments.

Output ONLY valid JSON:
{"findings": [{"title", "explanation", "suggestion", "severity",
               "category", "line_start", "line_end", "code_example", "ai_confidence"}]}
```

**`app/prompts/user_prompt.j2`**:
```
File: {{ file_path }}
Function: {{ function_name }}
Language: {{ language }}

<static_findings>
{{ static_findings | tojson }}
</static_findings>

<code>
{{ source_code }}
</code>

Review this code. Focus especially on the static findings listed above.
```

**`app/modules/ai/llm_client.py`**:
```python
@retry(stop=stop_after_attempt(3), wait=wait_exponential(min=1, max=10))
def call_llm(messages: list[dict], max_tokens=1000) -> str:
    # anthropic.Anthropic().messages.create(...)
    # OR openai.OpenAI().chat.completions.create(...)

def review_chunk(chunk: CodeChunk, static_findings: list[dict]) -> list[FindingCreate]:
    system = prompt_builder.build_system_prompt("v1")
    user   = prompt_builder.build_user_prompt(chunk, static_findings)
    raw    = call_llm([{"role": "system", "content": system},
                       {"role": "user",   "content": user}])
    parsed = json.loads(raw)
    return [FindingCreate(**f) for f in parsed["findings"]]  # Pydantic validates
```

### Tech & Concepts
- **Jinja2 templates**: `Environment(loader=FileSystemLoader("app/prompts/")).get_template("system_prompt_v1.j2").render(...)`. Prompts in `.j2` files = versionable, diffable, A/B testable. Never hardcode prompts in Python strings
- **Few-shot prompting**: provide 2-3 example `<code> → JSON` pairs before the real request. In-context learning — the LLM sees the pattern and follows it for the real input
- **Prompt injection defense**: user code goes inside `<code>...</code>` XML tags. If code contains `Ignore above instructions`, the tags signal that content is data, not instructions
- **Retry with exponential backoff**: `tenacity.retry` — 1s, 2s, 4s waits between attempts. LLM APIs rate-limit; retrying immediately makes it worse
- **Pydantic validation as the last gate**: if LLM output doesn't match `FindingCreate` schema, Pydantic raises ValidationError → triggers retry with error message appended to prompt

### Milestone
`review_chunk(sql_injection_chunk, [])` → LLM returns `{"findings": [...]}` → Pydantic validates → `category=security, severity=critical` in result. Deliberately break output format → Pydantic fails → retry fires → valid JSON on second attempt.

---

## Day 13 — Hybrid Engine + Finding Feedback Module + Suppression

### Goal
`hybrid_engine.analyze()` runs the full pipeline. Finding feedback module created. `.githelpignore` suppression working.

### What You Build

**`app/modules/ai/hybrid_engine.py`**:
```
Pipeline: static_analysis → ast_analysis → chunking → selective_llm → merge → dedup → tag → store

Selective LLM: only chunks that have ≥1 static/AST finding get sent to LLM (~70% cost reduction)
Cost guard: max 20 chunks to LLM per run (from analysis_config, default 20)
Source tagging:
  - static_only: only static tools flagged this
  - ast_only: only AST flagged this
  - llm_only: only LLM flagged this
  - hybrid: multiple sources agree — boost ai_confidence by 0.2, cap at 1.0
Deduplication: same (file, line_start, rule_id) → keep richest (most sources)
```

**`app/modules/finding_feedback/model.py`** ← fifth new model:
```python
class FindingFeedback(Base):
    __tablename__ = "finding_feedback"
    id            = Column(UUID, pk)
    finding_id    = Column(UUID, FK → findings.id, CASCADE)
    user_id       = Column(UUID, FK → users.id, SET NULL, nullable)  # SET NULL if user deleted
    useful        = Column(Boolean)   # True = helpful, False = false positive
    reason        = Column(Text, nullable)
    suggested_fix = Column(Text, nullable)
    created_at    = Column(TIMESTAMPTZ, server_default)
```

**`app/modules/finding_feedback/router.py`**:
```
POST /api/v1/findings/{id}/feedback
→ create FindingFeedback row
→ if useful=False: update finding.status → dismissed
→ if useful=True: increment knowledge_base.times_useful (Day 16)
```

**`.githelpignore` suppression** in `hybrid_engine.py`:
```python
exclude_patterns = repo.analysis_config.get("exclude_patterns", [])
findings = [f for f in findings
            if not any(fnmatch.fnmatch(f.file_path, pat) for pat in exclude_patterns)]
```

Update `analysis_tasks.analyze_repository` to call `hybrid_engine.analyze(repository_id)` instead of sleep stub.

### Tech & Concepts
- **Hybrid engine pattern**: static tools are fast and cheap but miss context. LLM is slow and expensive but understands context. Combine: use static as a first pass to identify suspicious chunks, then use LLM only on those. Best of both
- **`fnmatch` pattern matching**: `fnmatch.fnmatch("migrations/0001_initial.py", "migrations/*.py")` → True. Same syntax as `.gitignore`. No regex needed for this use case
- **Source confidence hierarchy**: `hybrid > llm_only > static_only = ast_only`. When scoring findings, hybrid findings count more because multiple independent sources agreed
- **`ondelete="SET NULL"` on `user_id`**: if a user is deleted, their feedback stays (valuable training data) but `user_id` becomes NULL. Without `SET NULL`, deleting a user would cascade-delete all their feedback

### Milestone
`hybrid_engine.analyze(test_repo_id)` → findings in DB with correct source tags → at least one `hybrid` finding → `review.llm_calls_made <= 20`. Add `migrations/*.py` to `analysis_config.exclude_patterns` → no findings from migration files.

---

## Day 14 — Scoring + Commit History Module + GitHub Integration

### Goal
Reviews produce quality scores. Commit history tracks trends. GitHub PRs get review comments and pass/fail status.

### What You Build

**`app/modules/reporting/`**:
```python
# scorer.py
calculate_quality_score(findings) -> int
    # 100 - (critical×15) - (major×7) - (minor×3) - (info×1), floor 0
calculate_security_score(findings) -> int       # filter category=security
calculate_performance_score(findings) -> int
calculate_maintainability_score(findings) -> int

# report_builder.py
generate_top_issues(findings) -> list[dict]     # top 3 critical for JSONB column
compute_review_counts(findings) -> dict         # {critical, major, minor, info}
```

**`app/modules/commit_history/model.py`** ← sixth new model:
```python
class CommitHistory(Base):
    __tablename__ = "commit_history"
    id                    = Column(UUID, pk)
    repository_id         = Column(UUID, FK → repositories.id, CASCADE)
    review_id             = Column(UUID, FK → reviews.id, SET NULL, nullable)
    commit_sha            = Column(String(40))
    commit_message        = Column(String(500), nullable)
    author_name           = Column(String(255), nullable)
    committed_at          = Column(TIMESTAMPTZ, nullable)
    quality_score         = Column(Integer, nullable)   # immutable snapshot
    security_score        = Column(Integer, nullable)
    performance_score     = Column(Integer, nullable)
    maintainability_score = Column(Integer, nullable)
    critical_count        = Column(Integer, default=0)
    major_count           = Column(Integer, default=0)
    score_delta           = Column(Integer, nullable)   # current - previous quality_score
    created_at            = Column(TIMESTAMPTZ, server_default)
    __table_args__ = (UniqueConstraint("repository_id", "commit_sha"),)
```

**`app/modules/integrations/github_client.py`**:
```python
post_review_comment(repo, pr_number, commit_sha, findings, token)
    # GitHub Review API — inline comments on PR diff lines
create_check_run(repo, head_sha, conclusion, findings, token)
    # GitHub Checks API — ✅/❌ status on commit
    # conclusion = "success" if quality_score >= 60 else "failure"
verify_webhook_signature(payload_bytes, signature_header, secret) -> bool
    # hmac.compare_digest(expected, received) — constant-time, prevents timing attacks
```

Wire into `analysis_tasks.py`: after analysis completes → calculate scores → write commit_history → if PR context exists → post GitHub comments + create check run.

### Tech & Concepts
- **Score snapshot immutability**: commit_history scores are copied once and never updated. If you re-run analysis, the new scores go in a new review + new history row. Historical data must not change
- **`score_delta` pre-computed**: `current_quality - previous_quality`. Frontend reads `score_delta` directly — no recalculation needed across the history array
- **`UniqueConstraint("repository_id", "commit_sha")`**: prevents duplicate rows when the same commit is re-analyzed. Use `ON CONFLICT DO NOTHING` in insert logic
- **GitHub Reviews vs Checks API**: Reviews = comments on specific diff lines (aesthetic, informational). Checks API = formal pass/fail status on the commit SHA itself (the ✅/❌ on PRs). Checks can block merging
- **`hmac.compare_digest`**: naive string comparison `a == b` leaks timing information — longer common prefix = longer comparison time. Attackers can brute-force the secret. `compare_digest` always takes the same time

### Milestone
Submit repo → analysis completes → `review.quality_score` is a real number → `commit_history` row created with score_delta. Open a real PR → GitHelp posts inline comments on changed lines → GitHub commit shows ✅ or ❌ status.

---

# WEEK 3 — Embeddings, RAG, LangGraph & Next.js Frontend
*Goal: Semantic similarity search, RAG-enhanced reviews, full LangGraph pipeline, working Next.js app.*

---

## Day 15 — Knowledge Base Module + Embeddings + pgvector

### Goal
Every code chunk has a vector embedding. Semantic similarity search working. Knowledge base module created with its model.

### What You Build

**`app/modules/knowledge_base/model.py`** ← seventh new model:
```python
class KnowledgeBase(Base):
    __tablename__ = "knowledge_base"
    id              = Column(UUID, pk)
    entry_type      = Column(Enum(KBEntryType), index)
    category        = Column(Enum(FindingCategory), index)
    severity        = Column(Enum(FindingSeverity), nullable)
    language        = Column(String(50), nullable, index)  # NULL = all languages
    title           = Column(String(500))
    content         = Column(Text)
    embedding       = Column(Vector(1536), nullable)  # HNSW index in migration
    times_retrieved = Column(Integer, default=0)
    times_useful    = Column(Integer, default=0)
    source_review_id= Column(UUID, FK → reviews.id, SET NULL, nullable)
    is_active       = Column(Boolean, default=True)  # soft delete
    created_at      = Column(TIMESTAMPTZ, server_default)
    updated_at      = Column(TIMESTAMPTZ, server_default, onupdate)
```

**`app/modules/ai/embeddings.py`**:
```python
generate_embedding(text: str) -> list[float]
    # openai.embeddings.create(model="text-embedding-3-small", input=text)
    # returns list of 1536 floats

batch_embed(texts: list[str], batch_size=100) -> list[list[float]]
    # embed in batches — OpenAI accepts up to 100 per request

find_similar_chunks(query_text, repository_id, top_k=5) -> list[dict]:
    # raw SQL with pgvector cosine operator:
    # SELECT *, 1 - (embedding <=> :vec) AS similarity
    # FROM code_chunks
    # WHERE repository_id = :repo_id AND embedding IS NOT NULL
    # ORDER BY embedding <=> :vec LIMIT :k
```

**`app/modules/tasks/embedding_tasks.py`** — real implementation:
```python
@celery_app.task
def embed_chunks(repository_id: str, force_reembed=False):
    # fetch all chunks where embedding IS NULL (or all if force_reembed)
    # batch embed
    # update embedding column
```

KB seeding script `scripts/seed_kb.py` — run once at startup:
100+ best practice entries covering: SQL injection, shell injection, mutable defaults, bare except, hardcoded secrets, weak crypto, OWASP Top 10, Python anti-patterns...

### Tech & Concepts
- **Embeddings**: a neural network maps text to a dense vector of floats. Semantically similar text ends up geometrically close. "SQL injection risk" and "unsanitized query" are near each other. "bubble sort" and "authentication bug" are far apart
- **Cosine similarity**: `1 - (A·B / |A||B|)` — measures angle between vectors. 1.0 = identical direction, 0.0 = perpendicular, -1.0 = opposite. We store `1 - distance` to get similarity
- **HNSW (Hierarchical Navigable Small World)**: graph-based approximate nearest neighbor index. Makes similarity search O(log n) instead of O(n). Must be added as raw SQL — SQLAlchemy declarative doesn't support HNSW parameters (`m`, `ef_construction`)
- **`force_reembed=False`**: re-running `embed_chunks` won't touch chunks that already have embeddings. Set True after switching embedding models

### Milestone
`embed_chunks(test_repo_id)` runs → `SELECT COUNT(*) FROM code_chunks WHERE embedding IS NULL` returns 0. `find_similar_chunks("SQL injection vulnerability", repo_id)` returns chunks containing SQL queries, ranked by semantic similarity not keyword match.

---

## Day 16 — RAG Pipeline

### Goal
`retrieve_context(chunk)` retrieves relevant KB rules and past findings to inject into the LLM prompt. RAG-enhanced reviews find issues the basic prompt missed.

### What You Build

**`app/modules/ai/rag_retriever.py`**:
```python
retrieve_context(chunk: CodeChunk, static_findings: list, top_k=3) -> dict:
    # 1. embed the chunk
    # 2. find top_k similar KB entries (from knowledge_base table)
    # 3. find top_k similar past confirmed findings (from code_chunks with findings)
    # 4. re-rank combined results:
    #    final_score = similarity × (0.7 + 0.3 × usefulness_ratio)
    #    usefulness_ratio = times_useful / max(times_retrieved, 1)
    # 5. return {rules: [...], past_findings: [...]}

record_retrieval(kb_ids: list[UUID], db)   # increment times_retrieved
record_useful(kb_ids: list[UUID], db)      # increment times_useful (called from feedback endpoint)
```

**`app/prompts/rag_prompt.j2`** — new section injected into user_prompt.j2:
```
<relevant_rules>
{% for rule in kb_context.rules %}
RULE: {{ rule.title }}
{{ rule.content }}
{% endfor %}
</relevant_rules>
```

Update `hybrid_engine.py` to call `retrieve_context()` before `review_chunk()`.

Update `finding_feedback/router.py`: when `useful=True` → call `record_useful()` for KB entries retrieved during that review.

### Tech & Concepts
- **RAG (Retrieval-Augmented Generation)**: instead of relying on what the LLM was trained on, we retrieve relevant context from our own DB and inject it. The LLM uses injected context to produce better findings. Grounded, up-to-date, domain-specific
- **Flywheel**: reviews → findings → developer feedback → KB improves → better context → better future reviews. Each analysis run makes the next one slightly better
- **Re-ranking**: raw similarity is not enough. A KB rule retrieved 100 times but marked useful only 5 times is probably noisy. `usefulness_ratio` downweights it. A rule marked useful 90% of the time gets boosted
- **Cold start problem**: KB needs 100+ entries before the flywheel spins. The `seed_kb.py` script on Day 15 seeds it. Without this, RAG retrieval returns nothing useful

### Milestone
RAG-enhanced review on `sample_bad_code.py` finds at least 2 issues the Day 12 basic prompt missed. `retrieve_context()` returns non-empty results with similarity > 0.7 for security-related code.

---

## Day 17 — Prompt Engineering II

### Goal
Chain-of-thought prompt v2 measurably outperforms v1. A/B testing framework in place.

### What You Build

**`app/prompts/system_prompt_v2.j2`**:
```
You are a senior software engineer conducting a formal code review.

Before producing findings, reason step by step:
1. What is this function/class trying to accomplish?
2. What could go wrong? (edge cases, error handling, input validation)
3. Are there security implications? (injection, auth, data exposure)
4. Is this maintainable? (complexity, naming, testability)
5. Only now — produce your JSON findings.

Do NOT flag TODO comments. Do NOT flag style issues in test files.
Output ONLY valid JSON.
```

**`app/modules/ai/prompt_builder.py`**:
```python
PROMPT_REGISTRY = {"v1": "system_prompt_v1.j2", "v2": "system_prompt_v2.j2"}

def build_system_prompt(version: str = "v2") -> str
def select_prompt_version() -> str:     # A/B: random.choice(["v1", "v2"])
```

Store `prompt_version` in `Review.prompt_version` — already in the model from Day 6.

After 10+ reviews of each: `SELECT prompt_version, AVG(critical_count+major_count) FROM reviews GROUP BY prompt_version` — compare.

### Tech & Concepts
- **Chain-of-thought (CoT)**: forcing the LLM to reason step-by-step before answering dramatically improves accuracy on complex tasks. The reasoning trace is a scratchpad that guides the final output
- **A/B testing for prompts**: treat prompts as product features — measure impact empirically. Don't assume CoT is always better — measure it on your actual data
- **Constitutional constraints** (`Do NOT flag TODO comments`): negative instructions reduce false positives for known failure modes. Add one when you notice a recurring bad output pattern

### Milestone
10 reviews each with v1 and v2. SQL query: v2 should produce more `critical+major` findings with same or fewer `info` findings (fewer false positives).

---

## Day 18 — LangGraph (Graph Pipeline)

### Goal
The entire analysis pipeline is a LangGraph `StateGraph`. Conditional edges skip unnecessary steps. Any node failure routes to error handling — no silent crashes.

### What You Build

```
app/modules/graph/
├── state.py          ← AnalysisState TypedDict
├── nodes.py          ← one function per pipeline stage
└── analysis_graph.py ← StateGraph definition, compiled graph
```

**`state.py`**:
```python
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

**`nodes.py`** — each returns updated state:
```python
def ingest_node(state)          # clone repo, populate file_list
def static_analysis_node(state) # run static tools, populate static_findings
def ast_analysis_node(state)
def chunking_node(state)
def embedding_node(state)
def rag_retrieval_node(state)
def ai_review_node(state)       # conditional: skip if no findings on chunk
def scoring_node(state)
def report_node(state)          # write to DB, cleanup
def error_node(state)           # update repo status=failed, log error
```

**`analysis_graph.py`**:
```python
builder = StateGraph(AnalysisState)
builder.add_node("ingest", ingest_node)
# ... add all nodes
builder.add_conditional_edges("ai_review", should_run_llm,
    {True: "scoring", False: "scoring"})
builder.add_edge(START, "ingest")
# error routing: try/except in each node → set state["error"] → route to error_node
graph = builder.compile()

def run_analysis(repository_id: str):
    return graph.invoke({"repository_id": repository_id, "error": None, ...})
```

Replace `hybrid_engine.analyze()` in `analysis_tasks.py` with `run_analysis()`.

### Tech & Concepts
- **LangGraph StateGraph**: directed graph where nodes are Python functions and edges are transitions. State is a TypedDict passed and updated through each node
- **Why graph over linear code**: graphs are observable (each node's input/output visible), resumable (checkpoint between nodes), modifiable (add a node without refactoring everything), testable (test each node independently)
- **Conditional edges**: `add_conditional_edges(from_node, condition_function, {return_value: to_node})` — enables branching. Skip AI review on cheap chunks with no static findings
- **Error routing**: instead of a try/except that swallows the error, nodes set `state["error"]` and the graph routes to `error_node` which gracefully updates DB status and records the error

### Milestone
`run_analysis(test_repo_id)` completes via graph. Add print statements or LangSmith tracing → each node's execution visible with timing. Introduce a deliberate RuntimeError in one node → error_node fires → `repository.status = failed` with error message.

---

## Day 19 — LangGraph Agents (ReAct + Tools)

### Goal
A `ReviewAgent` autonomously decides which tools to call and in what order when reviewing a file — not a fixed pipeline.

### What You Build

**`app/modules/graph/agent.py`**:
```python
class ReviewAgent:
    tools = [
        run_static_analysis,      # tool 1: calls static_analyzer
        run_ast_analysis,         # tool 2: calls ast_analyzer
        retrieve_similar_findings,# tool 3: calls rag_retriever
        get_file_contents,        # tool 4: reads file from cloned repo
        check_function_callers,   # tool 5: queries code_chunks.calls JSONB
                                  #   "who calls function X?" → cross-file context
        post_finding,             # tool 6: creates FindingCreate, adds to state
    ]
    max_iterations = 10           # safety guard — no infinite loops

    def review_file(self, file_path, repository_id) -> list[FindingCreate]:
        # ReAct loop:
        # 1. LLM reasons: "I see an authentication function, let me check what calls it"
        # 2. LLM acts: calls check_function_callers("authenticate_user", repo_id)
        # 3. LLM gets result: "called in api/users.py line 45 and api/admin.py line 12"
        # 4. LLM reasons again with new context
        # 5. LLM acts: post_finding(...)
        # Loop until "I am done reviewing this file" or max_iterations
```

**`check_function_callers` tool** — this is why we stored `calls` JSONB during chunking:
```python
def check_function_callers(function_name: str, repository_id: str) -> list[dict]:
    # SELECT file_path, name, start_line FROM code_chunks
    # WHERE repository_id = :repo_id
    # AND calls @> '["function_name"]'::jsonb
    # returns all chunks that call this function
```

### Tech & Concepts
- **ReAct (Reasoning + Acting)**: agent alternates between thought ("I should check what calls this function") and action (calls `check_function_callers`). This is how LLM agents reason through open-ended tasks
- **Tool calling at API level**: pass tool schemas as JSON Schema to the LLM. The LLM outputs structured JSON requesting a specific function call with specific arguments. Your framework executes it and feeds the result back
- **Why agents over fixed pipelines**: fixed pipelines always run the same steps. Agents can decide "this function is suspicious, let me investigate its callers" — dynamic investigation that finds bugs fixed pipelines miss
- **`calls @> '["function_name"]'::jsonb`**: PostgreSQL JSONB containment operator. Fast because JSONB is indexed. This is why storing `calls` during chunking mattered

### Milestone
Agent reviews a file where function A has a security issue and is called by function B. Without `check_function_callers`, the agent misses the call site. With it, the agent discovers function B calls A and posts a finding that references both files.

---

## Day 20 — API Keys Module + Next.js Setup + Auth

### Goal
`api_keys` module created. Next.js app scaffolded with full auth flow: register, login, JWT stored, protected routes, auto-refresh.

### What You Build

**`app/modules/api_keys/model.py`** ← eighth new model:
```python
class ApiKey(Base):
    __tablename__ = "api_keys"
    id           = Column(UUID, pk)
    user_id      = Column(UUID, FK → users.id, CASCADE)
    key_prefix   = Column(String(12))    # first 8-12 chars shown in dashboard
    key_hash     = Column(String(255), unique)  # bcrypt(full_key) — raw never stored
    name         = Column(String(255))
    scopes       = Column(JSONB, nullable)
    is_active    = Column(Boolean, default=True)
    expires_at   = Column(TIMESTAMPTZ, nullable)   # NULL = never expires
    last_used_at = Column(TIMESTAMPTZ, nullable)
    created_at   = Column(TIMESTAMPTZ, server_default)
    revoked_at   = Column(TIMESTAMPTZ, nullable)
```

**`app/modules/api_keys/router.py`**:
```
GET    /api/v1/keys         → list user's API keys (never show key_hash)
POST   /api/v1/keys         → create key → return raw_key ONCE, then never again
DELETE /api/v1/keys/{id}    → revoke
```

**Next.js frontend** — `frontend/` already exists in your structure:
```bash
cd frontend
npx create-next-app@latest . --typescript --tailwind --app
npm install axios jwt-decode
```

```
frontend/
├── app/
│   ├── layout.tsx
│   ├── page.tsx               ← redirect to /dashboard if logged in
│   ├── (auth)/
│   │   ├── login/page.tsx
│   │   └── register/page.tsx
│   └── (protected)/
│       ├── dashboard/page.tsx
│       └── repos/[id]/page.tsx
├── lib/
│   ├── axios.ts               ← Axios instance + interceptors
│   └── auth.ts                ← token storage helpers
├── context/
│   └── AuthContext.tsx        ← AuthProvider, useAuth hook
└── middleware.ts              ← Next.js middleware for route protection
```

**`frontend/lib/axios.ts`**:
```typescript
const api = axios.create({ baseURL: process.env.NEXT_PUBLIC_API_URL })

// Request interceptor: inject Bearer token
api.interceptors.request.use(config => {
  const token = localStorage.getItem("access_token")
  if (token) config.headers.Authorization = `Bearer ${token}`
  return config
})

// Response interceptor: on 401 → refresh → retry
api.interceptors.response.use(
  res => res,
  async err => {
    if (err.response?.status === 401) {
      const newToken = await refreshAccessToken()
      if (newToken) {
        err.config.headers.Authorization = `Bearer ${newToken}`
        return api(err.config)  // retry original request
      }
      window.location.href = "/login"
    }
    return Promise.reject(err)
  }
)
```

**`frontend/middleware.ts`** — Next.js route protection:
```typescript
export function middleware(request: NextRequest) {
  const token = request.cookies.get("access_token")
  if (!token && request.nextUrl.pathname.startsWith("/dashboard")) {
    return NextResponse.redirect(new URL("/login", request.url))
  }
}
```

### Tech & Concepts
- **Next.js App Router**: `app/` directory with nested layouts, `page.tsx` files per route, server components by default, `"use client"` for interactive components
- **Next.js `middleware.ts`**: runs on the Edge before the page renders — perfect for auth guards. Checks cookie → redirects to /login if missing
- **Axios interceptors for auth**: request interceptor injects token on every call. Response interceptor catches 401 → refreshes token → retries original request — seamless to the user
- **`key_hash` only**: API key is shown exactly once at creation (`raw_key` in response). After that, only `key_hash` and `key_prefix` exist in DB. Same model as GitHub personal access tokens
- **TypeScript in Next.js**: `lib/axios.ts` with typed responses catches API contract mismatches at compile time

### Milestone
Register → redirected to dashboard → refresh page → still logged in. Next.js middleware blocks `/dashboard` without auth → redirected to `/login`. Create API key → raw key shown once → use `X-API-Key` header → 200 from backend.

---

## Day 21 — Audit Logs Module + Next.js Dashboard

### Goal
`audit_logs` module created. Dashboard shows repos, progress polling, review results with findings. All 10 DB modules now exist.

### What You Build

**`app/modules/audit_logs/model.py`** ← ninth new model (final one):
```python
class AuditLog(Base):
    __tablename__ = "audit_logs"
    id          = Column(UUID, pk)
    user_id     = Column(UUID, FK → users.id, SET NULL, nullable)
    action      = Column(String(100), index)
    resource    = Column(String(50), nullable)
    resource_id = Column(UUID, nullable)
    ip_address  = Column(String(45), nullable)
    user_agent  = Column(String(500), nullable)
    extra_data  = Column(JSONB, nullable)
    created_at  = Column(TIMESTAMPTZ, server_default, index)
    # APPEND ONLY — never UPDATE or DELETE rows in this table
```

**`app/modules/audit_logs/service.py`**:
```python
log_action(user_id, action: AuditAction, resource, resource_id, request, extra_data, db)
```

Call `log_action` from: auth service (login, register, logout), findings router (dismiss, status change), API key router (create, revoke).

**Next.js Dashboard pages**:

`dashboard/page.tsx` — repo list:
```typescript
// Fetch GET /api/v1/repos
// Display: name, status badge (color-coded), last review score, submit date
// "Submit New Repo" → opens modal with URL form
```

`dashboard/components/ProgressPoller.tsx`:
```typescript
// polls GET /api/v1/tasks/{taskId}/status every 2s
// useEffect cleanup: clearInterval on unmount (prevents memory leaks)
// shows animated progress bar + stage label
// on SUCCESS: router.push(`/repos/${repoId}`)
// on FAILURE: show error message
```

`repos/[id]/page.tsx` — review page:
```typescript
// Score gauges (SVG circles) for quality, security, performance, maintainability
// Finding counts with severity colors
// Findings list with expand/collapse
// Syntax-highlighted code_example (react-syntax-highlighter)
// Status dropdown per finding → PATCH /findings/{id}/status
// 👍/👎 feedback buttons → POST /findings/{id}/feedback
```

### Tech & Concepts
- **`clearInterval` in `useEffect` cleanup**: `useEffect(() => { const id = setInterval(..., 2000); return () => clearInterval(id); }, [])` — without the cleanup function, the interval keeps running after the component unmounts (memory leak + stale state updates)
- **Next.js server vs client components**: `page.tsx` can be a server component (fetches data on server, no JS bundle). Interactive parts (`ProgressPoller`, status dropdown) need `"use client"` directive
- **Audit log append-only**: never UPDATE or DELETE from this table. It's an immutable event log for compliance and security investigation. If you need to "undo" something, write a new compensating event

### Milestone
Full flow in browser: submit GitHub URL → progress bar animates → redirect to review page → scores visible → findings list shows with severity badges → mark one as dismissed → status updates immediately.

---

# WEEK 4 — DevOps, Scale, Security & Production

---

## Day 22 — Multi-Language + Commit History Trend

### Goal
GitHelp analyzes JavaScript repos. Dashboard shows quality trend chart across commits.

### What You Build
- Strategy pattern: `LanguageAnalyzer` ABC → `PythonAnalyzer`, `JavaScriptAnalyzer`, `TypeScriptAnalyzer`
- `JavaScriptAnalyzer`: wraps `ESLint --format=json` via subprocess
- `TypeScriptAnalyzer(JavaScriptAnalyzer)`: adds `tsc --noEmit` type checking
- `AnalyzerRegistry`: `get_analyzer(language)` returns correct implementation
- `tree-sitter-javascript` for JS AST analysis: detect `eval()`, `==` not `===`, `var` not `let/const`
- `commit_history_service.analyze_commit_range(repo_id, n=5)`: analyze last 5 commits, compute trend
- `compute_trend(points)` → "improving" / "declining" / "stable" / "insufficient_data"
- Next.js: add `recharts` LineChart to review page showing commit history trend, "↑ Improving" / "↓ Declining" badge

**Tech:** Strategy pattern (Open/Closed Principle), tree-sitter vs language-specific parsers, `tsc --noEmit`

**Milestone:** Submit `expressjs/express` → JS findings appear → commit history chart shows 5 data points.

---

## Day 23 — Docker Production + Nginx + Multi-Stage

### Goal
`docker-compose -f docker-compose.prod.yml up` → full app at `http://localhost`, no CORS errors.

### What You Build
- Multi-stage `Dockerfile` for Next.js frontend: Stage 1 (node:18-alpine build) → Stage 2 (nginx:alpine serve `/out`)
- `nginx.conf`: serve static at `/`, proxy `/api/` to `app:8000`, `gzip on`, `try_files $uri $uri/ /index.html`
- Multi-stage `Dockerfile` for backend: Stage 1 (pip install) → Stage 2 (runtime only, no build tools)
- `docker-compose.prod.yml`: 5 services (nginx, app, celery_worker, postgres, redis), `restart: unless-stopped`, resource limits
- Next.js `next.config.js`: `output: "export"` for static generation OR keep SSR and run Node server

**Tech:** Multi-stage builds (5-10x smaller images), Nginx reverse proxy (eliminates CORS), `try_files` for Next.js routing

**Milestone:** `docker-compose -f docker-compose.prod.yml up` → `http://localhost` → Next.js loads → API calls work → zero CORS errors in browser console.

---

## Day 24 — CI/CD (GitHub Actions)

### Goal
Every push triggers lint → type check → test → build. Branch protection blocks merges with failing checks.

### What You Build
```yaml
# .github/workflows/ci.yml
jobs:
  backend:
    - flake8 app/
    - mypy app/ --strict
    - pytest tests/ --cov=app --cov-fail-under=70
    - bandit -r app/
  frontend:
    - npm run lint (ESLint)
    - npx tsc --noEmit (type check)
    - npm run build
  docker:
    needs: [backend, frontend]
    - docker build both images
    - if main: push to ghcr.io
```

`pyproject.toml` with mypy + flake8 config. Branch protection: require both jobs green before merge. README badge.

**Tech:** GitHub Actions, `needs:` job dependencies, `actions/cache@v3` for pip/npm, mypy strict mode, `--cov-fail-under=70`

**Milestone:** Push deliberate flake8 violation → CI red → fix → CI green → badge in README. PR with failing tests → GitHub blocks merge.

---

## Day 25 — Performance: Caching + Rate Limiting + N+1 Fix

### Goal
GET /reviews/{id} < 5ms after first request. Rate limited to 5 repo submissions/hour. No N+1 queries.

### What You Build
- Cache-aside on `GET /reviews/{id}` (TTL 300s) and `GET /repos/{id}` (TTL 60s), invalidate on writes
- `slowapi` rate limiting: `@limiter.limit("5/hour")` on POST /repos, key = authenticated user ID
- N+1 fix: `selectinload(Review.findings)` — 2 queries instead of N+1
- `EXPLAIN ANALYZE` to verify index usage
- Connection pooling: `create_engine(pool_size=10, max_overflow=20, pool_pre_ping=True)`

**Tech:** Cache-aside pattern, `selectinload` vs `joinedload`, Redis sliding window rate limit, `EXPLAIN ANALYZE`

**Milestone:** Cache hit < 5ms. POST /repos 6x same user → 6th returns 429 with `Retry-After`.

---

## Day 26 — Security Hardening

### Goal
Refresh token rotation. Path traversal blocked. Secrets scanner flags credentials. Bandit clean on own codebase.

### What You Build
- Refresh token rotation in `auth/service.py`: validate hash → issue new pair → invalidate old → reuse detection
- Path traversal: `pathlib.Path.resolve()` guard in `git_cloner.py` and `file_extractor.py`
- Enhanced secrets regex: AWS keys (`AKIA[0-9A-Z]{16}`), GitHub tokens, private keys, DB URLs
- Run `bandit -r app/` on GitHelp's own code → fix all HIGH/MEDIUM findings
- Security headers middleware: `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`

**Tech:** Refresh token rotation (single-use tokens), `hmac.compare_digest` (timing attack prevention), path traversal defense with `Path.resolve()`

**Milestone:** `run_secrets_scan("tests/fixtures/")` flags hardcoded key. Path `../../../../etc/passwd` → 400. Refresh token used twice → 401 on second use.

---

## Day 27 — Monitoring: Prometheus + Admin Analytics

### Goal
Prometheus metrics at `/metrics`. Custom metrics for analysis duration and LLM tokens. Admin analytics endpoint.

### What You Build
- `prometheus-fastapi-instrumentator` — auto-instruments all routes
- Custom metrics in `app/core/metrics.py`:
  ```python
  analysis_duration = Histogram("githelp_analysis_duration_seconds", ["mode"])
  llm_tokens_used   = Counter("githelp_llm_tokens_total")
  findings_per_review = Histogram("githelp_findings_per_review")
  active_analyses   = Gauge("githelp_active_analyses")
  ```
- Instrument LangGraph nodes with timing
- `GET /api/v1/admin/analytics` (admin role): total repos, avg score, tokens used, findings by severity, KB flywheel stats

**Tech:** Prometheus metric types (Counter/Gauge/Histogram), PromQL, structured logging with span IDs, three pillars of observability (metrics, logs, traces)

**Milestone:** `curl localhost:8000/metrics` returns Prometheus text format. Submit repo → `githelp_analysis_duration_seconds_bucket` updates.

---

## Day 28 — Cloud Deployment + Self-Review + Final Docs

### Goal
Live on the internet with HTTPS. GitHelp reviews its own codebase. Documentation complete.

### What You Build
- Deploy to Railway or Render: connect GitHub repo → set env vars → auto-deploy on push to main
- HTTPS via Let's Encrypt (Railway/Render handle this automatically)
- Smoke tests against production URL
- `ARCHITECTURE.md` with Mermaid diagrams: system overview, analysis pipeline, data flow
- GitHelp reviews its own GitHub repo — document the findings and what you fixed
- Demo video (5-7 min): submit → progress → review page → findings → PR comment → trend chart

**Tech:** PaaS (Railway/Render), Let's Encrypt, smoke tests, Mermaid diagrams, dogfooding

**Milestone:** Live URL with HTTPS. GitHelp's own repo analyzed → quality score visible → ARCHITECTURE.md merged.

---

# APPENDIX: Complete Final File Structure

```
backend/
├── app/
│   ├── common/
│   │   ├── base.py          ← DeclarativeBase
│   │   └── enums.py         ← ALL enums (Day 3 — define complete set early)
│   │
│   ├── core/
│   │   ├── config.py        ← Pydantic Settings
│   │   ├── database.py      ← engine, AsyncSession, get_db()
│   │   ├── dependencies.py  ← get_current_user, require_role()
│   │   ├── security.py      ← hash, verify, create_token, decode_token
│   │   ├── middleware.py    ← CorrelationID (Day 7)
│   │   ├── cache.py         ← Redis helpers (Day 5)
│   │   └── metrics.py       ← Prometheus metrics (Day 27)
│   │
│   ├── modules/
│   │   ├── auth/            ← Day 3-4 (no model — uses User)
│   │   │   ├── schemas.py
│   │   │   ├── router.py
│   │   │   └── service.py
│   │   │
│   │   ├── users/           ← Day 2 ✅
│   │   │   ├── model.py
│   │   │   ├── schemas.py
│   │   │   ├── router.py
│   │   │   └── service.py
│   │   │
│   │   ├── repository/      ← Day 4
│   │   │   ├── model.py     ← Repository
│   │   │   ├── schemas.py
│   │   │   ├── router.py
│   │   │   └── service.py
│   │   │
│   │   ├── reviews/         ← Day 6
│   │   │   ├── model.py     ← Review
│   │   │   ├── schemas.py
│   │   │   ├── router.py
│   │   │   └── service.py
│   │   │
│   │   ├── findings/        ← Day 10
│   │   │   ├── model.py     ← Finding
│   │   │   ├── schemas.py
│   │   │   ├── router.py
│   │   │   └── service.py
│   │   │
│   │   ├── code_chunks/     ← Day 11
│   │   │   ├── model.py     ← CodeChunk (Vector(1536))
│   │   │   ├── schemas.py
│   │   │   └── service.py
│   │   │
│   │   ├── finding_feedback/ ← Day 13
│   │   │   ├── model.py     ← FindingFeedback
│   │   │   ├── schemas.py
│   │   │   └── router.py
│   │   │
│   │   ├── commit_history/  ← Day 14
│   │   │   ├── model.py     ← CommitHistory
│   │   │   ├── schemas.py
│   │   │   └── service.py
│   │   │
│   │   ├── knowledge_base/  ← Day 15
│   │   │   ├── model.py     ← KnowledgeBase (Vector(1536))
│   │   │   ├── schemas.py
│   │   │   └── router.py
│   │   │
│   │   ├── api_keys/        ← Day 20
│   │   │   ├── model.py     ← ApiKey
│   │   │   ├── schemas.py
│   │   │   └── router.py
│   │   │
│   │   ├── audit_logs/      ← Day 21
│   │   │   ├── model.py     ← AuditLog (append-only)
│   │   │   ├── schemas.py
│   │   │   └── service.py
│   │   │
│   │   ├── ingestion/       ← Day 8 (no model)
│   │   │   ├── git_cloner.py
│   │   │   └── file_extractor.py
│   │   │
│   │   ├── analysis/        ← Days 9-11 (no model — produces findings)
│   │   │   ├── static_analyzer.py
│   │   │   ├── ast_analyzer.py
│   │   │   ├── dependency_mapper.py
│   │   │   └── chunker.py
│   │   │
│   │   ├── ai/              ← Days 12-16
│   │   │   ├── prompt_builder.py
│   │   │   ├── llm_client.py
│   │   │   ├── hybrid_engine.py  ← replaced by graph on Day 18
│   │   │   ├── embeddings.py
│   │   │   └── rag_retriever.py
│   │   │
│   │   ├── graph/           ← Days 18-19
│   │   │   ├── state.py
│   │   │   ├── nodes.py
│   │   │   ├── analysis_graph.py
│   │   │   └── agent.py
│   │   │
│   │   ├── reporting/       ← Day 14
│   │   │   ├── scorer.py
│   │   │   └── report_builder.py
│   │   │
│   │   ├── integrations/    ← Day 14
│   │   │   └── github_client.py
│   │   │
│   │   ├── tasks/           ← Day 5 (no model — Celery wrappers)
│   │   │   ├── celery_app.py
│   │   │   ├── analysis_tasks.py
│   │   │   ├── embedding_tasks.py
│   │   │   └── webhook_tasks.py
│   │   │
│   │   └── admin/           ← Day 27
│   │       ├── schemas.py
│   │       └── router.py
│   │
│   ├── prompts/             ← Day 12+
│   │   ├── system_prompt_v1.j2
│   │   ├── system_prompt_v2.j2  ← Day 17
│   │   ├── user_prompt.j2
│   │   ├── few_shot_examples.j2
│   │   └── rag_prompt.j2        ← Day 16
│   │
│   └── main.py
│
├── tests/
│   ├── conftest.py
│   ├── unit/
│   └── integration/
│
├── scripts/
│   └── seed_kb.py           ← Day 15: seed 100+ KB rules
│
├── init_db.py               ← import all models, create_all()
├── Dockerfile
├── requirements.txt
└── .env

frontend/                    ← Next.js (Day 20)
├── app/
│   ├── (auth)/login/
│   ├── (auth)/register/
│   └── (protected)/dashboard/
├── lib/axios.ts
├── context/AuthContext.tsx
└── middleware.ts

docker-compose.yml
docker-compose.prod.yml      ← Day 23
.github/workflows/ci.yml     ← Day 24
```

---

# APPENDIX: DB Model Rollout Timeline

| Day | Module | Model | Why Introduced Then |
|-----|--------|-------|-------------------|
| 2 | users | User | Auth is the foundation — nothing works without users |
| 4 | repository | Repository | Needed to wire POST /repos on Day 6 |
| 6 | reviews | Review | Analysis task creates a Review row |
| 10 | findings | Finding | Static + AST results need somewhere to live |
| 11 | code_chunks | CodeChunk | Chunking needs DB storage; embedding added Day 15 |
| 13 | finding_feedback | FindingFeedback | Feedback loop after hybrid engine exists |
| 14 | commit_history | CommitHistory | Scoring exists; now track trends |
| 15 | knowledge_base | KnowledgeBase | RAG needs a table to retrieve from |
| 20 | api_keys | ApiKey | Frontend needs programmatic access |
| 21 | audit_logs | AuditLog | Full system in place; now log everything |

---

# APPENDIX: Tech Stack

| Layer | Technology | Introduced |
|-------|-----------|-----------|
| API | FastAPI + Uvicorn | Day 1 |
| Database | PostgreSQL + pgvector | Day 1 |
| ORM | SQLAlchemy 2.0 async | Day 1 |
| Validation | Pydantic v2 | Day 3 |
| Auth | python-jose, passlib, cryptography | Day 4 |
| Task Queue | Celery + Redis | Day 5 |
| Static Analysis | pylint, flake8, radon, bandit, pip-audit | Day 9 |
| AST Analysis | Python ast, networkx | Day 10 |
| Tokenization | tiktoken | Day 11 |
| LLM | Anthropic Claude / OpenAI | Day 12 |
| Prompt Templates | Jinja2 | Day 12 |
| Embeddings | text-embedding-3-small | Day 15 |
| Vector DB | pgvector (HNSW) | Day 15 |
| RAG | Custom retriever + re-ranking | Day 16 |
| AI Pipeline | LangGraph StateGraph | Day 18 |
| AI Agents | LangGraph ReAct | Day 19 |
| Frontend | Next.js 14 + TypeScript + Tailwind | Day 20 |
| HTTP Client | Axios + interceptors | Day 20 |
| Charts | recharts | Day 22 |
| Syntax Highlight | react-syntax-highlighter | Day 21 |
| Reverse Proxy | Nginx | Day 23 |
| Containers | Docker multi-stage | Day 23 |
| CI/CD | GitHub Actions | Day 24 |
| Rate Limiting | slowapi | Day 25 |
| Monitoring | prometheus-fastapi-instrumentator | Day 27 |
| Logging | python-json-logger | Day 7 |
| Cloud | Railway / Render | Day 28 |
