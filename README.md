# GitHelp — AI Powered Code Review System
## Complete 4-Week Day-by-Day Build Plan
**Student:** Nilesh Kumar | **Enrollment:** 06116403222 | **Supervisor:** Prof. Anurag Jain
**University:** USICT, Sector-14 Dwarka, Delhi-110075

---

# WEEK 1 — Infrastructure, Database & Backend Foundation
*Goal: A running FastAPI server with all 11 tables, JWT auth, task queue, and a wired API→Queue→DB pipeline.*

---

## Day 1 — Docker + Project Skeleton

### Goal
Three Docker containers running and healthy: app, postgres (with pgvector), redis.

### What You Build
- `docker-compose.yml` with three services: `app`, `postgres`, `redis`
- `Dockerfile` for the FastAPI app (python:3.11-slim base)
- `.env.example` with all environment variables listed
- `app/config.py` — Pydantic `Settings` class reading from `.env` (single source of truth for all env vars)
- Bare `app/main.py` — FastAPI app instance, one health endpoint GET /health/live returning `{"status": "ok"}`
- `requirements.txt` with pinned versions

### Tech & Concepts
- **Docker**: containerization, image layers, multi-service orchestration
- **Docker Compose**: `depends_on`, `healthcheck`, volume mounts, environment variables
- **pgvector**: postgres image `ankane/pgvector:latest` which ships with the extension pre-installed
- **Pydantic Settings**: `BaseSettings` reads from environment automatically — `DATABASE_URL`, `REDIS_URL`, `SECRET_KEY`, `FERNET_KEY`, `OPENAI_API_KEY`
- **Environment separation**: never hardcode secrets, `.env` in `.gitignore`

### Concepts Learned
Docker networking (containers talk via service name, not localhost), why we pin image versions, what pgvector is and why it lives in Postgres.

### Milestone
`docker-compose up` → all three containers show healthy → `curl localhost:8000/health/live` returns 200.

---

## Day 2 — All 11 Database Tables + pgvector

### Goal
All 11 SQLAlchemy models created, pgvector extension enabled, all tables exist in PostgreSQL.

### What You Build
- `app/models/base.py` — single `DeclarativeBase`
- `app/models/enums.py` — all enums: `UserRole`, `RepoStatus`, `RepoProvider`, `AnalysisMode`, `ReviewStatus`, `FindingSeverity`, `FindingCategory`, `FindingSource`, `FindingStatus`, `ChunkType`, `KBEntryType`, `AuditAction`
- All 11 model files: `user.py`, `repository.py`, `review.py`, `finding.py`, `code_chunk.py`, `knowledge_base.py`, `commit_history.py`, `finding_feedback.py`, `api_key.py`, `audit_log.py`
- `app/models/__init__.py` — imports every model (critical for Alembic)
- `app/core/database.py` — SQLAlchemy engine, `SessionLocal`, `get_db()` dependency
- First Alembic migration: `alembic revision --autogenerate -m "initial_tables"`
- Second migration (manual): enable pgvector extension + HNSW indexes on `code_chunks.embedding` and `knowledge_base.embedding`

### Tech & Concepts
- **SQLAlchemy ORM**: `DeclarativeBase`, `Column`, `relationship`, `ForeignKey`, `cascade`
- **pgvector**: `Vector(1536)` column type, HNSW index vs IVFFlat (HNSW is better for most cases)
- **Alembic**: `env.py` setup, `--autogenerate`, why `models/__init__.py` must import everything
- **HNSW parameters**: `m=16` (connections per node), `ef_construction=64` (build-time accuracy) — raw SQL in migration
- **Key fixes applied**: `server_default=func.now()` on `updated_at`, `ondelete="SET NULL"` on nullable FKs, `server_default='[]'::jsonb` on JSONB arrays, `severity_rank` integer alongside severity enum

### Concepts Learned
Why HNSW index must be raw SQL (SQLAlchemy declarative doesn't support HNSW params), why `ondelete` must be set on every FK, what `server_default` vs `default` means (DB-level vs Python-level).

### Milestone
`docker exec -it postgres psql -U postgres -d githelp -c "\dt"` shows all 11 tables. `\d code_chunks` shows the `embedding vector(1536)` column and HNSW index.

---

## Day 3 — FastAPI Routers + Pydantic Schemas + Swagger

### Goal
All API endpoints defined with correct Pydantic schemas, returning dummy data, fully documented in Swagger UI at /docs.

### What You Build
- All 12 schema files in `app/schemas/`: `common.py`, `user.py`, `repository.py`, `review.py`, `finding.py`, `code_chunk.py`, `knowledge_base.py`, `commit_history.py`, `finding_feedback.py`, `api_key.py`, `audit_log.py`, `__init__.py`
- All router files in `app/routers/`: `auth.py`, `repositories.py`, `reviews.py`, `findings.py`, `tasks.py`, `webhooks.py`, `history.py`, `knowledge_base.py`, `admin.py`
- Register all routers in `main.py` with prefix `/api/v1`
- API versioning from day one: all routes under `/api/v1/`

### Endpoints (returning dummy data today)
```
POST   /api/v1/auth/register
POST   /api/v1/auth/login
POST   /api/v1/auth/refresh
GET    /api/v1/auth/github/callback
POST   /api/v1/repos
GET    /api/v1/repos
GET    /api/v1/repos/{id}
POST   /api/v1/repos/{id}/reanalyze
GET    /api/v1/repos/{id}/history
GET    /api/v1/reviews/{id}
GET    /api/v1/reviews/{id}/findings
PATCH  /api/v1/findings/{id}/status
POST   /api/v1/findings/{id}/feedback
GET    /api/v1/tasks/{task_id}/status
POST   /api/v1/webhooks/github
GET    /api/v1/kb
POST   /api/v1/kb
GET    /api/v1/admin/analytics
GET    /api/v1/keys
POST   /api/v1/keys
DELETE /api/v1/keys/{id}
GET    /api/v1/admin/audit-logs
GET    /health/live
GET    /health/ready
```

### Tech & Concepts
- **Pydantic v2**: `BaseModel`, `ConfigDict(from_attributes=True)`, `field_validator`, `model_validator`, `EmailStr`, `Field` with constraints
- **FastAPI routers**: `APIRouter`, `include_router`, `prefix`, `tags`
- **API versioning**: why `/api/v1/` prefix from day one — once external tools use your API you can't change routes
- **Swagger/OpenAPI**: auto-generated from type hints and docstrings — response_model parameter
- **`from_attributes=True`**: lets Pydantic read SQLAlchemy ORM objects directly (`UserResponse.model_validate(db_user)`)

### Concepts Learned
How Pydantic v2 differs from v1 (model_config vs class Config), why request schemas and response schemas are separate (never expose `FindingCreate` via API), what `model_validate` does vs `from_orm`.

### Milestone
`/docs` shows all endpoints grouped by tag. Every endpoint returns a valid response matching its schema (dummy data). No 500 errors on any route.

---

## Day 4 — JWT Auth + OAuth + RBAC + API Keys

### Goal
Full authentication system: register → login → JWT → protected endpoint → 401 without token. GitHub OAuth flow. API key authentication. Role-based access control.

### What You Build
- `app/core/security.py`: `hash_password`, `verify_password` (passlib bcrypt), `create_access_token`, `create_refresh_token` (python-jose), `get_current_user` dependency, `require_role(UserRole.admin)` factory function
- `app/services/auth_service.py`: `register_user`, `login_user`, `refresh_tokens` (rotation), `github_oauth_callback`, `encrypt_token` / `decrypt_token` (Fernet), `hash_api_key`, `verify_api_key`
- Auth router: real implementations for register, login, refresh, GitHub callback
- `app/models/api_key.py` service: `create_api_key`, `list_api_keys`, `revoke_api_key` — raw key shown once at creation, only hash stored
- Middleware: extract Bearer token OR `X-API-Key` header, set `request.state.user`
- `app/services/audit_service.py`: `log_action(user_id, action, resource, resource_id, ip, extra_data)` — called after every auth event

### Tech & Concepts
- **JWT (JSON Web Tokens)**: `python-jose`, access token (15 min expiry), refresh token (7 days), token payload structure `{sub, exp, role}`
- **Refresh token rotation**: each use invalidates old hash, issues new token — prevents replay attacks
- **bcrypt**: password hashing with salt rounds. Never store plain text passwords
- **Fernet symmetric encryption**: `cryptography` library, used for OAuth tokens stored in DB (`github_token_enc`, `gitlab_token_enc`)
- **OAuth 2.0 flow**: authorization code flow with GitHub — redirect → code → exchange for token → get user info → create/find user
- **RBAC (Role-Based Access Control)**: `require_role` dependency factory — `Depends(require_role(UserRole.admin))` on admin routes
- **API Key security**: `key_prefix` (first 8 chars shown in dashboard), `key_hash` (bcrypt of full key), raw key shown exactly once
- **Audit logging**: every auth event writes to `audit_logs` — who logged in, from what IP, when

### Concepts Learned
Why JWTs are stateless (no DB lookup needed to verify), why refresh token rotation matters, what Fernet is and when to use symmetric vs asymmetric encryption, how OAuth authorization code flow works step by step.

### Milestone
Register → Login → get JWT → call GET /api/v1/repos with Bearer token → 200. Same call without token → 401. Admin-only endpoint with non-admin user → 403. Create API key → use `X-API-Key` header → 200.

---

## Day 5 — Redis + Celery + Flower

### Goal
Background task queue running. Submit a task, see it execute in the worker, result visible in Flower dashboard.

### What You Build
- `app/celery_app.py`: Celery instance configured with Redis broker and Redis result backend
- `app/tasks/analysis_tasks.py`: `analyze_repository` task (dummy — just sleeps 3 seconds and returns success)
- `app/tasks/embedding_tasks.py`: `embed_chunks` task (stub)
- `app/tasks/webhook_tasks.py`: `process_github_webhook` task (stub)
- Worker entry in `docker-compose.yml` as a separate service running `celery -A app.celery_app worker`
- Flower service in `docker-compose.yml`: `celery -A app.celery_app flower`
- `app/core/cache.py`: Redis client, `cache_get`, `cache_set`, `cache_delete` helper functions
- Dead letter queue configuration: `task_reject_on_worker_lost=True`, `task_acks_late=True`, failed tasks route to `dead_letter` queue

### Tech & Concepts
- **Celery**: distributed task queue, `@app.task(bind=True, max_retries=3)`, `self.retry(countdown=2**self.request.retries)` exponential backoff
- **Redis as broker**: task messages stored in Redis lists — reliable delivery
- **Redis as result backend**: task results stored with TTL — `AsyncResult(task_id).state`
- **Celery task states**: PENDING → STARTED → SUCCESS / FAILURE / RETRY
- **Dead letter queue**: tasks that permanently fail go to a separate queue for manual inspection — prevents silent data loss
- **Flower**: web UI for monitoring Celery tasks, workers, queues at port 5555
- **`task_acks_late=True`**: task only marked as done after successful execution — if worker crashes mid-task, task is re-queued

### Concepts Learned
Why background tasks are needed (HTTP requests can't wait 60 seconds), difference between broker (message bus) and result backend (result storage), what exponential backoff prevents (thundering herd on retries).

### Milestone
POST any endpoint → Celery task appears in Flower as STARTED → transitions to SUCCESS → Flower shows task result. Worker logs show task execution.

---

## Day 6 — Wire API → Queue → DB

### Goal
Full pipeline: POST /api/v1/repos → creates DB row → schedules Celery task → GET /api/v1/tasks/{id}/status polls Celery → SUCCESS → DB row updated.

### What You Build
- `app/services/repo_service.py`: `create_repository`, `get_repository`, `list_repositories`, `update_repository_status`
- `app/routers/repositories.py`: real implementation — validate input, call service, schedule task, return response
- `app/tasks/analysis_tasks.py`: update task to accept `repository_id`, set `status=analyzing` at start, `status=completed` at end, write dummy findings to DB
- `app/routers/tasks.py`: GET /tasks/{task_id}/status — query Celery `AsyncResult`, return `{state, progress, stage, result}`
- `app/services/audit_service.py`: called from repo router to log `repo.submitted`
- Redis caching on GET /repos/{id}: `cache_aside` pattern — check cache first, hit DB on miss, write to cache with TTL=300s

### Tech & Concepts
- **Cache-aside pattern**: check Redis → if hit return cached → if miss query DB → write to cache → return result
- **Celery task chaining**: `analyze_repository` triggers `embed_chunks` on completion via `.delay()` call inside the task
- **Task progress reporting**: use `self.update_state(state='STARTED', meta={'progress': 25, 'stage': 'cloning'})` to send real-time progress
- **Database session in tasks**: tasks create their own session — NOT the FastAPI `get_db()` dependency. Use `with SessionLocal() as db:`
- **Idempotency**: if repo already has a running task, don't submit another one — check `status IN (cloning, analyzing)`

### Concepts Learned
Why Celery tasks must manage their own DB sessions (they run in a different process), what the cache-aside pattern is and why we don't use cache-through, how Celery task progress metadata works.

### Milestone
POST /api/v1/repos with a GitHub URL → 202 with `{task_id}` → poll GET /tasks/{task_id}/status → watch progress go from 0 to 100 → final status SUCCESS → GET /repos/{id} shows status=completed.

---

## Day 7 — Structured Logging + Tests + Health Checks + README

### Goal
Production-quality observability. pytest suite passing. README good enough for a professor to run the project.

### What You Build
- `app/core/middleware.py`: correlation ID middleware — generates UUID per request, injects into all log lines, returns as `X-Correlation-ID` response header
- Structured JSON logging with `python-json-logger`: every log line is `{timestamp, level, correlation_id, path, method, duration_ms, user_id, message}`
- `GET /health/ready` — checks DB connection, Redis connection, returns `{status, database, redis, llm_api, version}`
- `tests/conftest.py`: fixtures for test DB (SQLite in-memory), test client (httpx), mock user, sample repositories
- `tests/unit/test_auth.py`: test `hash_password`, `verify_password`, `create_access_token`, token expiry
- `tests/unit/test_scorer.py`: test scoring formula `100 - (critical×15) - (major×7) - (minor×3)`
- `tests/integration/test_auth_flow.py`: register → login → get token → use token → refresh → use new token → old token rejected
- `tests/integration/test_repo_flow.py`: submit repo → get task ID → poll status → get review
- `README.md`: prerequisites, env setup, `docker-compose up`, running tests, API overview

### Tech & Concepts
- **Correlation IDs**: every request gets a UUID that follows it through all log lines — essential for debugging microservices
- **python-json-logger**: JSON-formatted logs parseable by Elasticsearch, Loki, CloudWatch
- **pytest + httpx**: `TestClient` for sync tests, `AsyncClient` for async. `conftest.py` fixtures are shared across test files
- **pytest-cov**: code coverage — `pytest --cov=app --cov-report=html` — target 70% minimum
- **Liveness vs Readiness**: liveness = is process alive (simple 200), readiness = is it ready to serve traffic (checks dependencies)

### Concepts Learned
What correlation IDs are and why every enterprise system uses them, difference between unit and integration tests, why health checks need two endpoints not one.

### Milestone
`pytest tests/ -v` → all tests pass. `GET /health/ready` returns JSON showing all dependencies healthy. Every log line in Docker logs is valid JSON with correlation_id.

---

# WEEK 2 — Code Intelligence & AI Engine
*Goal: A complete hybrid analysis pipeline — static tools, AST analysis, code chunking, LLM review — all wired together.*

---

## Day 8 — Repository Ingestion (GitPython)

### Goal
Clone a real GitHub repo, list its files by language, get the latest diff, confirm cleanup.

### What You Build
- `app/services/ingestion/git_cloner.py`:
  - `validate_url(url)` — regex check for GitHub/GitLab HTTPS URL format, check for path traversal attempts
  - `clone_repository(url, token=None)` — `git.Repo.clone_from(url, tmp_dir, depth=1)` — `depth=1` gets only latest commit, not full history
  - `get_file_list(repo_path)` — walk directory, return `[{path, language, size_kb}]`, skip `.git/`, skip files over `max_file_size_kb`
  - `get_latest_diff(repo_path)` — `repo.git.diff('HEAD~1', 'HEAD')` — returns changed files for incremental mode
  - `cleanup_repository(repo_path)` — `shutil.rmtree(path, ignore_errors=True)` — ALWAYS in `finally` block
- `app/services/ingestion/file_extractor.py`:
  - `detect_language(filepath)` — extension mapping: `.py`→python, `.js`→javascript, `.ts`→typescript, `.go`→go
  - `get_files_by_language(file_list)` — group files by language
  - `read_file_safely(path)` — read with UTF-8, fallback to latin-1, return None if binary
- `.githelpignore` parsing: read suppression patterns from `analysis_config.exclude_patterns`

### Tech & Concepts
- **GitPython**: `git.Repo`, `clone_from`, `depth=1` for shallow clone (faster, less disk)
- **Path traversal prevention**: validate that extracted file paths don't contain `../` — a security vulnerability where `../../etc/passwd` type paths escape the clone directory
- **`finally` block**: cleanup MUST happen even if analysis crashes — without it, `/tmp` fills up and crashes the server
- **Shallow clone (`depth=1`)**: only downloads the latest commit snapshot, not the entire git history — 10-100x faster for large repos
- **Language detection**: file extension mapping, not content detection — fast and good enough for our purposes

### Concepts Learned
Why shallow clones are critical for performance, what path traversal attacks are, why cleanup in finally is not optional.

### Milestone
`git_cloner.clone_repository("https://github.com/tiangolo/fastapi")` → files listed → cleanup confirmed → `/tmp` directory gone. Verify with a private repo using encrypted token.

---

## Day 9 — Static Analysis (pylint, flake8, radon, bandit, pip-audit, secrets scanner)

### Goal
`analyze_file(path)` returns a normalized list of findings from 6 different tools. Each finding has a consistent structure regardless of which tool produced it.

### What You Build
- `app/services/analysis/static_analyzer.py`:
  - `run_pylint(filepath)` → parse JSON output → normalize
  - `run_flake8(filepath)` → parse stdout line by line → normalize
  - `run_radon(filepath)` → cyclomatic complexity per function → normalize (complexity > 10 = major, > 20 = critical)
  - `run_bandit(filepath)` → parse JSON output → normalize with severity mapping
  - `run_pip_audit(requirements_path)` → parse JSON → normalize as security findings with CVE IDs in `rule_ids`
  - `run_secrets_scan(repo_path)` → `detect-secrets scan` or regex patterns for API keys, tokens, passwords → normalize as critical security findings
  - `normalize_finding(tool, raw)` → returns `{file_path, line_start, severity, category, source="static_only", rule_ids, title, explanation, suggestion}`
  - `analyze_file(filepath)` → calls all tools, merges results, deduplicates by (file, line, rule_id)
  - All subprocess calls: `subprocess.run([...], capture_output=True, timeout=60)` — NEVER `shell=True` (shell injection vulnerability)

### Tech & Concepts
- **subprocess without shell=True**: `shell=True` passes command to shell interpreter — vulnerable to injection if any user input reaches the command. Always use list form
- **pylint**: checks code style, naming, unused variables, imports — outputs JSON with `--output-format=json`
- **flake8**: PEP 8 style checker — line length, whitespace, imports
- **radon**: cyclomatic complexity — counts decision points (if/for/while/try) — measures how hard code is to test
- **bandit**: security-focused AST analysis — finds hardcoded passwords, SQL injection risks, weak crypto usage
- **pip-audit**: checks installed packages against OSV (Open Source Vulnerability) database — finds known CVEs
- **detect-secrets**: regex patterns for API keys, AWS credentials, private keys, connection strings
- **Finding normalization**: every tool outputs differently — our job is to produce one consistent schema

### Concepts Learned
What cyclomatic complexity measures (testability and maintainability), why `shell=True` is dangerous, what CVEs are and why dependency scanning matters, why deduplication by (file, line, rule_id) is needed when multiple tools flag the same issue.

### Milestone
`analyze_file("tests/fixtures/sample_bad_code.py")` returns at least 5 findings from at least 3 different tools. `run_pip_audit()` flags a known vulnerable package version. `run_secrets_scan()` flags a hardcoded API key in `tests/fixtures/sample_security_issues.py`.

---

## Day 10 — AST Parsing (Python ast module + networkx)

### Goal
`ast_analyzer.analyze_file(filepath)` detects structural issues invisible to static linters: god classes, circular imports, mutable defaults, overly complex nesting.

### What You Build
- `app/services/analysis/ast_analyzer.py`:
  - `ASTVisitor(ast.NodeVisitor)`: custom visitor detecting:
    - Functions > 50 lines → maintainability finding
    - Nesting depth > 4 → maintainability finding (counts nested if/for/while/try)
    - Classes with > 10 methods ("god class") → architecture finding
    - Bare `except:` clauses → reliability finding
    - Mutable default arguments `def f(x=[])` → bug finding
    - Magic numbers (numeric literals not in assignments or constants) → style finding
    - Functions with > 5 parameters → maintainability finding
  - `analyze_file(filepath)` → parse to AST, run visitor, return findings
- `app/services/analysis/dependency_mapper.py`:
  - `build_import_graph(file_list)` → parse all `import` statements, build `networkx.DiGraph`
  - `detect_circular_imports(graph)` → `networkx.simple_cycles(graph)` → circular dependency = architecture finding
  - `get_module_dependencies(module_name, graph)` → returns what a module imports and what imports it

### Tech & Concepts
- **Python `ast` module**: parses Python source into Abstract Syntax Tree — `ast.parse(source)`, `ast.NodeVisitor`, `ast.walk(tree)`. The AST represents code structure as a tree of nodes (FunctionDef, ClassDef, If, For, etc.)
- **`ast.NodeVisitor`**: visitor pattern — `visit_FunctionDef`, `visit_ClassDef` are called automatically when traversing
- **Nesting depth tracking**: maintain a depth counter that increments on entering If/For/While nodes and decrements on exit — max depth = worst-case cognitive complexity
- **Mutable defaults**: `def f(x=[])` is a famous Python bug — the list is created once and shared across all calls. The AST check: `FunctionDef.args.defaults` contains `List` or `Dict` node
- **networkx DiGraph**: directed graph where node = module, edge = "A imports B". Cycles in this graph = circular imports
- **`networkx.simple_cycles`**: finds all cycles in O(n+e) time — efficient even for large codebases

### Concepts Learned
What an AST is and how it's different from raw text parsing, why the visitor pattern is perfect for tree traversal, what circular imports cause (ImportError, hard-to-test modules), why mutable defaults are a Python-specific trap.

### Milestone
`ast_analyzer.analyze_file("tests/fixtures/sample_bad_code.py")` detects: at least one function > 50 lines, one bare except, one mutable default argument. `dependency_mapper` detects a manually created circular import between two test files.

---

## Day 11 — Code Chunking (tiktoken + AST boundaries)

### Goal
`chunk_repository(repo_path)` splits every file into chunks at function/class boundaries, each under 3000 tokens, stored in `code_chunks` table.

### What You Build
- `app/services/analysis/chunker.py`:
  - `count_tokens(text, model="gpt-4")` → `tiktoken.encoding_for_model(model).encode(text)` → length
  - `chunk_file_by_ast(filepath)` → parse AST, extract each `FunctionDef` and `ClassDef` as a chunk, remaining top-level code as a `module` chunk
  - Each chunk: `{file_path, chunk_type, name, start_line, end_line, source_code, token_count, language, imports_used, calls}`
  - `imports_used`: list of module names imported in the chunk's scope
  - `calls`: list of function names called within the chunk (from `ast.Call` nodes)
  - If a single function > 3000 tokens: split at logical boundaries (every N lines), mark as `chunk_type=module`
  - `chunk_repository(repo_path)` → calls `chunk_file_by_ast` on every Python/JS file → bulk insert to `code_chunks` table
- `app/tasks/analysis_tasks.py`: update `analyze_repository` task to call `chunk_repository` after static analysis

### Tech & Concepts
- **tiktoken**: OpenAI's tokenizer — counts tokens the same way GPT-4 does. Different models have different tokenizers. `cl100k_base` encoding used by GPT-4
- **Why 3000 token limit**: GPT-4 has 128k context window but we leave room for system prompt (~500 tokens), static findings (~1000 tokens), KB context (~1500 tokens), and response (~3000 tokens) — total ~6000 tokens per LLM call
- **AST-aware chunking**: splitting at function/class boundaries preserves semantic meaning. Splitting at N lines might cut a function in half, confusing the LLM
- **`calls` extraction**: `ast.walk(function_node)` looking for `ast.Call` nodes → `node.func.attr` for method calls, `node.func.id` for direct calls — feeds the cross-file context system later
- **Bulk insert**: `db.bulk_save_objects(chunks)` instead of individual inserts — 100x faster for large repos

### Concepts Learned
What tokenization is (text → integers the model processes), why chunking strategy matters (bad chunking = LLM sees incomplete code), why we track `calls` metadata (needed for cross-file context analysis in Week 3).

### Milestone
`chunk_repository("/tmp/fastapi_clone")` → chunks stored in DB → `SELECT COUNT(*), AVG(token_count), MAX(token_count) FROM code_chunks` — verify no chunk exceeds 3000 tokens.

---

## Day 12 — LLM Integration + Prompt Engineering I

### Goal
`llm_client.review_chunk(chunk, static_findings)` sends a code chunk to the LLM and gets back structured JSON findings.

### What You Build
- `app/prompts/system_prompt_v1.j2`: system prompt establishing senior engineer persona, output format specification
- `app/prompts/user_prompt.j2`: template receiving `{code, language, static_findings, file_path, function_name}`
- `app/prompts/few_shot_examples.j2`: 3 examples of code → expected JSON output (few-shot learning)
- `app/services/ai/prompt_builder.py`:
  - `build_system_prompt(version="v1")` → load and render Jinja2 template
  - `build_user_prompt(chunk, static_findings, kb_context=None)` → render user template
- `app/services/ai/llm_client.py`:
  - `call_llm(messages, max_tokens=1000)` → `anthropic.Anthropic().messages.create(...)` or OpenAI
  - Exponential backoff: `@retry(stop=stop_after_attempt(3), wait=wait_exponential(min=1, max=10))`
  - Parse response → validate against `FindingCreate` Pydantic schema
  - `review_chunk(chunk, static_findings)` → build prompts → call LLM → parse response → return `list[FindingCreate]`

### System Prompt Design
```
You are a senior software engineer conducting a formal code review.
Your job is to identify real bugs, security vulnerabilities, and 
maintainability issues — not style preferences.

Output ONLY valid JSON in this format:
{
  "findings": [
    {
      "title": "...",
      "explanation": "WHY this is a problem...",
      "suggestion": "HOW to fix it...",
      "severity": "critical|major|minor|info",
      "category": "security|performance|maintainability|architecture|style|documentation",
      "line_start": 12,
      "line_end": 18,
      "code_example": "corrected code here",
      "ai_confidence": 0.85
    }
  ]
}
```

### Tech & Concepts
- **Jinja2 templating**: `Environment(loader=FileSystemLoader("app/prompts/"))`, `template.render(chunk=chunk, findings=findings)` — never hardcode prompts in Python files
- **Few-shot prompting**: providing example input→output pairs before the real input — dramatically improves output format consistency
- **Structured output / JSON mode**: instruct the LLM to output only JSON, then validate with Pydantic. If validation fails → retry with error message in prompt
- **Exponential backoff**: `2^attempt` seconds between retries (1s, 2s, 4s) — handles LLM rate limits gracefully
- **Prompt injection defense**: user's code goes in a separate `<code>` XML tag so if code contains instructions like "ignore above and...", it's treated as content not instructions
- **`ai_confidence` field**: ask the LLM to self-rate its confidence (0-1) — low confidence findings get lower weight in scoring

### Concepts Learned
Why prompts are Jinja2 templates not Python strings (versioning, A/B testing), what few-shot prompting is and why it works (in-context learning), what prompt injection is and one defense against it.

### Milestone
`review_chunk(chunk_with_sql_injection, [])` → LLM returns valid JSON → Pydantic validates successfully → finding has `category=security, severity=critical`. Invalid LLM response → retry → success on second attempt.

---

## Day 13 — Hybrid Engine + Finding Lifecycle + Suppression

### Goal
`hybrid_engine.analyze(repository_id)` runs the complete pipeline: static → AST → chunking → selective LLM → merge → deduplicate → tag sources → cost guard.

### What You Build
- `app/services/ai/hybrid_engine.py`:
  - Pipeline: `ingest → static_analysis → ast_analysis → chunking → selective_llm → merge → deduplicate → score → store`
  - Selective LLM: only send chunks that have at least one static/AST finding (reduces cost by ~70%)
  - Cost guard: maximum 20 chunks sent to LLM per analysis run (configurable in `analysis_config`)
  - Source tagging: `static_only` / `ast_only` / `llm_only` / `hybrid` (hybrid = finding confirmed by multiple sources)
  - Confidence boosting: hybrid findings get `ai_confidence` boosted by 0.2 (capped at 1.0)
  - Deduplication: findings at the same (file, line_start, rule_id) deduplicated — keep the richest (most sources)
- `app/services/finding_service.py`:
  - `bulk_create_findings(review_id, findings)` — auto-set `severity_rank` from `SEVERITY_RANK` dict
  - `update_finding_status(finding_id, status, dismissed_reason, user_id)` — writes to `audit_logs`
  - `suppress_finding(finding_id, suppressor)` — sets `is_suppressed=True`
- `.githelpignore` parsing: before storing findings, filter out any finding whose file matches exclusion patterns

### Tech & Concepts
- **Hybrid engine pattern**: combining weak learners (static tools) with a strong learner (LLM) — static tools are fast/cheap, LLM is slow/expensive. Run static first, use findings to guide LLM
- **Selective LLM calls**: only chunks with existing findings need LLM review — pure clean code chunks can be skipped. This is the key cost optimization
- **Source confidence hierarchy**: `hybrid` (multiple sources agree) > `llm_only` > `static_only` = `ast_only` — used in scoring
- **Finding deduplication**: static tools and LLM often flag the same issue. We merge them into one finding with source=hybrid rather than showing duplicates
- **`.githelpignore` pattern**: `fnmatch` pattern matching — `migrations/*.py`, `*.test.py`, `vendor/**` — same syntax as `.gitignore`
- **`severity_rank` auto-set**: before inserting to DB, look up `SEVERITY_RANK[finding.severity]` — never trust client to provide this

### Concepts Learned
Why hybrid systems beat single-tool approaches, what selective sampling is and why it matters for cost, how finding deduplication works with multiple sources.

### Milestone
`hybrid_engine.analyze(test_repository_id)` on `tests/fixtures/sample_bad_code.py` → findings in DB with correct `source` tags → at least one `hybrid` finding → `llm_calls_made <= 20` in review record.

---

## Day 14 — Scoring + Reporting + GitHub PR Integration + Checks API

### Goal
Reviews produce quality scores. Findings posted as GitHub PR comments. GitHub Checks API creates pass/fail status on commits.

### What You Build
- `app/services/reporting/scorer.py`:
  - `calculate_quality_score(findings)` → `100 - (critical_count × 15) - (major_count × 7) - (minor_count × 3) - (info_count × 1)`, floor at 0
  - `calculate_security_score`, `calculate_performance_score`, `calculate_maintainability_score` — filter findings by category
  - `compute_score_delta(repository_id, current_score)` → fetch previous `commit_history.quality_score`, subtract
- `app/services/reporting/report_builder.py`:
  - `build_summary(findings)` → group by category, format as readable text for LLM to summarize
  - `generate_top_issues(findings)` → top 3 critical findings as JSONB for dashboard
  - `compute_review_counts(findings)` → `{critical_count, major_count, minor_count, info_count}`
- `app/services/integrations/github_client.py`:
  - `post_review_comment(repo, pr_number, commit_sha, findings, token)` → GitHub Review API — posts inline comments on changed lines
  - `create_check_run(repo, head_sha, name, conclusion, findings, token)` → GitHub Checks API — creates the ✅/❌ status on the commit
  - `verify_webhook_signature(payload, signature, secret)` → `hmac.compare_digest(expected, received)` — validates webhook authenticity
- `app/routers/webhooks.py`: real implementation — verify signature → queue `process_github_webhook` Celery task → return 200 immediately (GitHub webhooks have 10s timeout)
- `app/services/commit_history_service.py`: `record_commit(repository_id, review_id, commit_sha, scores)` — write to `commit_history` with `ON CONFLICT DO NOTHING`

### Tech & Concepts
- **Scoring formula**: penalties not grades — start at 100, subtract for problems. Critical issues cost 15 points each — 7 critical findings = score of 0
- **GitHub Review API vs Checks API**: Reviews = comments on specific lines of a PR diff. Checks = pass/fail status shown on the commit itself (the green/red checkmark). Both are different APIs
- **HMAC webhook verification**: GitHub signs every webhook with `HMAC-SHA256(payload, secret)`. We verify with `hmac.compare_digest` (constant-time comparison — prevents timing attacks)
- **Timing attack**: naive string comparison `a == b` leaks information via timing (longer match = longer comparison). `compare_digest` takes the same time regardless of match length
- **Async webhook response**: return 200 immediately, process in background. GitHub retries if no 200 within 10 seconds
- **`ON CONFLICT DO NOTHING`**: re-analyzing the same commit won't create duplicate history rows (UniqueConstraint on repository_id + commit_sha)

### Concepts Learned
What a timing attack is and why `compare_digest` prevents it, how GitHub's Checks API works and why it's more powerful than just comments, what score delta tells you (did this PR make things better or worse?).

### Milestone
Open a PR on a test repo → webhook fires → GitHelp posts inline review comments on changed lines → GitHub commit shows ✅ (score >= 60) or ❌ (score < 60) status from GitHelp.

---

# WEEK 3 — Embeddings, RAG, LangGraph & Frontend
*Goal: Semantic search, AI-powered context retrieval, full LangGraph pipeline, and working React frontend.*

---

## Day 15 — Embeddings + pgvector Similarity Search

### Goal
Every code chunk has an embedding stored in pgvector. `find_similar_chunks(text)` returns the most semantically similar chunks via cosine similarity.

### What You Build
- `app/services/ai/embeddings.py`:
  - `generate_embedding(text)` → `openai.embeddings.create(model="text-embedding-3-small", input=text)` → returns `list[float]` of length 1536
  - `batch_embed(texts, batch_size=100)` → embed in batches to avoid API rate limits
  - `find_similar_chunks(query_text, repository_id, top_k=5)` → raw SQL using pgvector operator:
    ```sql
    SELECT *, 1 - (embedding <=> :query_vec) AS similarity
    FROM code_chunks
    WHERE repository_id = :repo_id AND embedding IS NOT NULL
    ORDER BY embedding <=> :query_vec
    LIMIT :top_k
    ```
  - `find_similar_across_repos(query_text, top_k=10)` → search globally — used for KB retrieval
- `app/tasks/embedding_tasks.py`: real implementation of `embed_chunks(repository_id, force_reembed=False)` — batch embed all chunks, update `embedding` column
- pgvector operators: `<=>` cosine distance, `<->` L2 distance, `<#>` inner product

### Tech & Concepts
- **Word/code embeddings**: neural network encodes text into a dense vector — semantically similar text has similar vectors. "authentication bug" and "login vulnerability" end up close in vector space
- **Cosine similarity**: angle between vectors — 1.0 = identical, 0.0 = unrelated, -1.0 = opposite. We use `1 - cosine_distance` to get similarity from distance
- **pgvector operators**: `<=>` (cosine distance) is best for normalized embeddings. HNSW index makes this sub-linear time — no full table scan
- **`text-embedding-3-small`**: 1536 dimensions, cheaper than `text-embedding-3-large` (3072 dims), much better than `ada-002`. Dimensions match our `Vector(1536)` column
- **Batch embedding**: OpenAI API accepts up to 100 texts per request — batch to reduce HTTP round trips and cost
- **`force_reembed=False`**: by default skip chunks that already have embeddings. Set True after switching embedding model

### Concepts Learned
What embeddings are conceptually (compressed semantic representation), why cosine similarity works for semantic search, how HNSW makes similarity search fast (approximate nearest neighbor).

### Milestone
Embed all chunks from a test repository. `find_similar_chunks("SQL injection vulnerability")` returns chunks containing SQL queries at top — semantic match not keyword match.

---

## Day 16 — RAG Pipeline (Retrieval-Augmented Generation)

### Goal
`rag_retriever.retrieve_context(chunk, static_findings)` returns the most relevant best practices and past findings to inject into the LLM prompt, improving review quality.

### What You Build
- KB seeding script `scripts/seed_kb.py`: insert 100+ best practice rules as `KnowledgeBase` entries — embed each one
  - Example entries: "SQL injection prevention", "Never use shell=True in subprocess", "Avoid mutable default arguments", "Validate all user inputs", "Use parameterized queries", etc.
- `app/services/ai/rag_retriever.py`:
  - `retrieve_context(chunk, static_findings, top_k=3)` → embed the chunk → find similar KB entries → find similar past findings → merge and re-rank
  - Re-ranking formula: `final_score = similarity × (0.7 + 0.3 × usefulness_ratio)` where `usefulness_ratio = times_useful / max(times_retrieved, 1)`
  - `retrieve_similar_past_findings(chunk, top_k=3)` → find similar chunks from other reviews that had confirmed findings
  - `record_retrieval(kb_ids)` → increment `times_retrieved` for all retrieved entries
  - `record_useful_feedback(kb_ids)` → increment `times_useful` when developer marks finding as useful
- Update `app/prompts/rag_prompt.j2`: section injecting retrieved context into user prompt
- Update `hybrid_engine.py` to call `retrieve_context` before LLM call

### Tech & Concepts
- **RAG (Retrieval-Augmented Generation)**: instead of relying on LLM's training data, we retrieve relevant context from our own database and inject it into the prompt. The LLM doesn't hallucinate rules it doesn't know — we provide them
- **Flywheel effect**: more reviews → more confirmed findings in KB → better RAG context → better future reviews → more confident developers → more feedback → even better KB
- **Re-ranking**: raw similarity score is not enough. A highly retrieved but never-useful rule should rank lower. `usefulness_ratio` adjusts scores by historical utility
- **Cold start problem**: KB must be pre-seeded with 100+ rules before the flywheel can spin. The seed script is critical
- **Context window budget**: with RAG, we add ~1500 tokens of context. Must still fit in our 6000-token budget per call

### Concepts Learned
What RAG is and why it's better than pure LLM generation (grounding, up-to-date knowledge, domain specificity), what the cold start problem is in recommendation systems, why re-ranking matters (relevance ≠ utility).

### Milestone
RAG-enhanced review on `sample_bad_code.py` finds at least 2 issues that the basic prompt missed. `retrieve_context` returns non-empty results with similarity > 0.7 for security-related code.

---

## Day 17 — Prompt Engineering II (Chain-of-Thought + Versioning + A/B Testing)

### Goal
Prompt v2 with chain-of-thought produces measurably more specific findings than v1. A/B testing framework shows which prompt version performs better.

### What You Build
- `app/prompts/system_prompt_v2.j2`: upgraded prompt with chain-of-thought instruction:
  ```
  Before producing findings, reason through this code step by step:
  1. What is this function/class trying to do?
  2. What could go wrong? (edge cases, missing validation, error handling)
  3. Are there security implications? (input validation, authentication, authorization)
  4. Is this maintainable? (complexity, naming, documentation)
  5. Now, produce your findings based on this analysis.
  ```
- `app/services/ai/prompt_builder.py`: prompt version registry `{"v1": "system_prompt_v1.j2", "v2": "system_prompt_v2.j2"}`, A/B selector
- A/B testing framework: randomly assign 50% of reviews to v1, 50% to v2. Store `prompt_version` in `Review` table
- Metrics collection: `SELECT prompt_version, AVG(quality_score), COUNT(*) FROM reviews GROUP BY prompt_version` — compare versions
- `app/prompts/few_shot_examples.j2`: extend to 5 examples covering security, performance, maintainability, architecture, documentation
- Constitutional constraints in system prompt: "Do not flag issues in test files unless they are security vulnerabilities. Do not flag TODO comments as findings."

### Tech & Concepts
- **Chain-of-thought (CoT) prompting**: making the LLM reason step-by-step before answering — dramatically improves accuracy on complex tasks. "Think before you answer" for LLMs
- **Zero-shot vs few-shot vs chain-of-thought**: zero-shot = just ask, few-shot = examples, CoT = reasoning steps. Each improvement layer increases quality
- **Prompt versioning**: store `prompt_version` in the DB record — can correlate prompt versions with review quality metrics after the fact
- **A/B testing for prompts**: treat prompts like product features — measure impact empirically. Don't assume v2 is better — measure it
- **Constitutional constraints**: negative instructions in the prompt — "do not X, do not Y" — prevent common failure modes. Reduces false positives
- **Prompt injection defense**: wrapping user code in XML tags (`<code>...</code>`) so that if code contains instructions like "Ignore above", it's treated as string content, not instructions

### Concepts Learned
What chain-of-thought prompting is and the neuroscience-inspired reason it works (explicit reasoning traces), how A/B testing works in production systems, what constitutional AI constraints are.

### Milestone
Run 10 reviews with v1 and 10 with v2 on the same test repository. `SELECT prompt_version, AVG(major_count + critical_count) FROM reviews GROUP BY prompt_version` — v2 finds more issues with equal or fewer false positives.

---

## Day 18 — LangGraph (Replace Hybrid Engine with Graph)

### Goal
The entire analysis pipeline is now a compiled LangGraph `StateGraph`. Conditional edges skip unnecessary steps. Error routing prevents full pipeline failures.

### What You Build
- `app/services/graph/state.py`:
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
- `app/services/graph/nodes.py`: one function per node — `ingest_node`, `static_analysis_node`, `ast_analysis_node`, `chunking_node`, `embedding_node`, `rag_retrieval_node`, `ai_review_node`, `scoring_node`, `report_node`
- `app/services/graph/analysis_graph.py`:
  - Build `StateGraph(AnalysisState)`
  - Add all nodes
  - Conditional edges: `should_run_llm` → if no static/AST findings on chunk, skip to scoring
  - Error routing: any node failure → `error_node` → updates DB status to failed, records error
  - Compile: `graph = builder.compile()`
  - `run_analysis(repository_id)` → `graph.invoke(initial_state)`
- Replace `hybrid_engine.analyze()` call in Celery task with `run_analysis()`

### Tech & Concepts
- **LangGraph**: framework for building stateful multi-step AI pipelines as graphs. Built on top of LangChain. Nodes = functions, Edges = transitions, State = shared data
- **StateGraph**: the graph type for stateful pipelines. State is passed between nodes — each node receives state, returns updated state
- **Conditional edges**: `add_conditional_edges(node, condition_fn, {True: "next_node", False: "skip_node"})` — enables branching logic
- **Why LangGraph over plain Python**: checkpointing (resume failed pipelines), observability (trace every step), easy modification of pipeline structure without rewriting everything
- **Graph compilation**: `builder.compile()` validates the graph structure at startup, not at runtime — catches missing nodes/edges early
- **Error routing**: instead of crashing the entire graph on one node failure, route to an error handler that cleans up gracefully

### Concepts Learned
What LangGraph is and why it's better than a chain of function calls for complex pipelines, what directed acyclic graphs are (DAGs), how state machines work (nodes = states, edges = transitions).

### Milestone
`run_analysis(test_repo_id)` completes via LangGraph graph. LangSmith (or print statements) shows each node's execution with timing. Introduce a deliberate error in one node → error routing fires → DB status set to failed with error message.

---

## Day 19 — LangGraph Agents (ReAct Pattern + Tools)

### Goal
A `ReviewAgent` with 6 tools autonomously decides the order and which tools to call when reviewing a file, rather than following a fixed pipeline.

### What You Build
- `app/services/graph/agent.py`: `ReviewAgent` using ReAct (Reasoning + Acting) loop
- 6 tool definitions:
  1. `run_static_analysis(file_path)` → calls `static_analyzer.analyze_file()`
  2. `run_ast_analysis(file_path)` → calls `ast_analyzer.analyze_file()`
  3. `retrieve_similar_findings(code_snippet)` → calls `rag_retriever.retrieve_context()`
  4. `get_file_contents(file_path)` → reads file from cloned repo
  5. `check_function_callers(function_name, repository_id)` → queries `code_chunks.calls` JSONB — finds all chunks that call this function → cross-file context
  6. `post_finding(title, explanation, suggestion, severity, category, line_start, line_end)` → creates `FindingCreate` and adds to state
- Guard: `max_iterations=10` — agent cannot loop more than 10 times
- Agent prompt: "You are a code reviewer. Use the available tools to thoroughly review the provided code. Post findings using post_finding. Stop when you have reviewed all important aspects."

### Tech & Concepts
- **ReAct pattern (Reasoning + Acting)**: agent alternates between reasoning (thought) and tool use (action). Each step: "I see X, therefore I should call tool Y to learn Z"
- **Tool calling / Function calling**: LLMs can be given a list of function schemas, and will output structured JSON asking to call a function with specific arguments. The framework executes the function and feeds the result back
- **Why agents over fixed pipelines**: agents can decide "this function is suspicious, let me also check what calls it" — dynamic investigation rather than fixed steps
- **Cross-file context via `check_function_callers`**: the `calls` JSONB column (populated during chunking) enables this — "function X is called in files A, B, C" gives the agent cross-file context without loading the entire codebase
- **`max_iterations` guard**: without this, a confused agent can loop forever and consume unlimited LLM tokens
- **Tool schemas**: each tool is described with name, description, and parameter schema in JSON Schema format — the LLM uses these descriptions to decide when to call each tool

### Concepts Learned
What the ReAct pattern is and how it differs from chain-of-thought, what tool/function calling is at the API level (structured output with function schemas), why agents are powerful for open-ended investigation tasks.

### Milestone
Agent reviews a complex file with an inter-function dependency bug. Without `check_function_callers`, it misses the bug. With it, it calls the tool, discovers that a function with a security issue is called from 3 other files, posts findings referencing all call sites.

---

## Day 20 — React Frontend Setup + Auth Flow

### Goal
Full authentication flow in the browser: register, login, JWT stored, protected routes, auto-refresh, API calls working.

### What You Build
- Vite + React project scaffolding: `npm create vite@latest frontend -- --template react`
- Install: Tailwind CSS, React Router v6, Axios, `jwt-decode`
- `src/context/AuthContext.jsx`: `AuthProvider` with `user`, `login`, `logout`, `register` — stores JWT in `localStorage` (or httpOnly cookie)
- Axios interceptors in `src/api/axios.js`:
  - Request interceptor: auto-inject `Authorization: Bearer {token}` header
  - Response interceptor: on 401 → auto-call `/auth/refresh` → retry original request with new token → if refresh fails → logout
- `src/pages/LoginPage.jsx`, `src/pages/RegisterPage.jsx` — forms with validation
- `src/components/ProtectedRoute.jsx` — redirects to `/login` if not authenticated
- `src/pages/GitHubCallbackPage.jsx` — handles OAuth redirect, exchanges code for token
- React Router routes: `/login`, `/register`, `/auth/callback`, `/dashboard` (protected), `/repos/:id` (protected)

### Tech & Concepts
- **Vite**: modern build tool replacing Create React App — instant HMR (Hot Module Replacement), ES module native
- **React Router v6**: `createBrowserRouter`, `Outlet`, `useNavigate`, `useParams` — declarative routing
- **Axios interceptors**: middleware for HTTP requests/responses — perfect for auth token injection and auto-refresh
- **JWT in browser**: store in `localStorage` (simple, XSS vulnerable) vs `httpOnly` cookie (XSS safe, CSRF vulnerable). For this project, localStorage is acceptable
- **Auto-refresh flow**: intercept 401 → call /auth/refresh with refresh token → get new access token → retry failed request seamlessly
- **`Context + useReducer`**: auth state management without Redux — `AuthContext` provides `{user, token, login, logout}` to all components

### Concepts Learned
What XSS and CSRF are (browser security vulnerabilities), why token refresh must be transparent to the user, how React Context replaces prop drilling for global state.

### Milestone
Register a new user → redirected to dashboard → refresh page → still logged in → logout → try to access dashboard → redirected to login. GitHub OAuth button → GitHub → callback → auto-login.

---

## Day 21 — Frontend Dashboard (Submit, Progress, Results)

### Goal
Complete user-facing flow: submit a repo URL → watch real-time progress → see quality scores → browse findings with syntax highlighting.

### What You Build
- `src/pages/Dashboard.jsx`: list of submitted repos with status badges
- `src/components/RepoSubmitForm.jsx`: URL input with validation, submit button, loading state
- `src/components/AnalysisProgress.jsx`:
  - `setInterval` polling GET /tasks/{task_id}/status every 2 seconds
  - Progress bar (0-100%), current stage label ("Cloning...", "Running static analysis...", "AI reviewing...")
  - Auto-redirect to review page on SUCCESS, show error on FAILURE
  - `clearInterval` in cleanup — prevent memory leaks
- `src/pages/ReviewPage.jsx`:
  - Score gauges for overall, security, performance, maintainability (circular SVG gauges)
  - Finding counts by severity with color coding (red=critical, orange=major, yellow=minor, blue=info)
  - Top issues preview cards
  - `recharts` LineChart for commit history trend (quality score over time)
- `src/components/FindingCard.jsx`:
  - Expandable card per finding
  - `highlight.js` for syntax highlighting of `code_example`
  - Severity badge, category badge, source badge (static/AST/LLM/hybrid)
  - Status dropdown (open → acknowledged → fixed → dismissed)
  - Feedback buttons (👍 useful / 👎 false positive) → POST /findings/{id}/feedback
- `src/components/FindingFilters.jsx`: filter by severity, category, status, file path

### Tech & Concepts
- **`setInterval` + `clearInterval`**: polling pattern in `useEffect` — `return () => clearInterval(id)` in cleanup prevents stale intervals after component unmounts
- **`recharts`**: declarative charting library for React — `LineChart`, `XAxis`, `YAxis`, `Tooltip`, `ResponsiveContainer`
- **`highlight.js`**: syntax highlighting — `hljs.highlightElement(ref.current)` in `useEffect` after render
- **Optimistic UI**: when user changes finding status, update UI immediately, then confirm with API — makes UI feel instant
- **Controlled vs uncontrolled components**: filter form uses controlled inputs (`value + onChange`) so filter state drives the displayed list

### Concepts Learned
Why `clearInterval` in cleanup matters (memory leaks), what optimistic UI is and when to use it, how `recharts` React data binding works.

### Milestone
Submit a real GitHub URL from the browser → progress bar animates through stages → review page shows scores and findings → click a finding → see syntax-highlighted code example → mark as dismissed → finding shows dismissed status.

---

# WEEK 4 — DevOps, Scale, Security & Production
*Goal: Docker production build, CI/CD pipeline, security hardening, monitoring, cloud deployment.*

---

## Day 22 — Multi-Language Support + Commit History Analysis

### Goal
GitHelp analyzes JavaScript/TypeScript repos as well as Python. Dashboard shows quality trend chart across last 5 commits.

### What You Build
- Language strategy pattern in `app/services/analysis/`:
  - `LanguageAnalyzer` abstract base class: `analyze(filepath) -> list[FindingCreate]`
  - `PythonAnalyzer(LanguageAnalyzer)`: wraps existing pylint/flake8/radon/bandit
  - `JavaScriptAnalyzer(LanguageAnalyzer)`: wraps `ESLint` via subprocess, parses JSON output
  - `TypeScriptAnalyzer(JavaScriptAnalyzer)`: adds `tsc --noEmit` type check
  - `AnalyzerRegistry`: `{language: LanguageAnalyzer}` — `get_analyzer(language)` returns the right one
- `tree-sitter-python` and `tree-sitter-javascript`: for AST analysis on non-Python files
  - JS AST visitor: detect `eval()` calls, `==` instead of `===`, `var` instead of `let/const`, async without await, console.log left in production code
- `app/services/commit_history_service.py`:
  - `analyze_commit_range(repository_id, n_commits=5)` → `git log -n 5` → analyze each commit → store in `commit_history`
  - `compute_trend(history_points)` → "improving" / "declining" / "stable" / "insufficient_data"
- Frontend: commit history LineChart on ReviewPage now populated with real data, trend badge ("↑ Improving" in green, "↓ Declining" in red)

### Tech & Concepts
- **Strategy pattern**: define an interface (`LanguageAnalyzer`), multiple implementations. The caller doesn't know which implementation it's using — swap languages without changing calling code
- **tree-sitter**: incremental parsing library with grammars for 50+ languages — produces AST like Python's `ast` module but for any language
- **ESLint via subprocess**: `npx eslint --format=json filepath` — same pattern as pylint. Parse JSON output, normalize findings
- **`tsc --noEmit`**: TypeScript compiler in type-check-only mode — finds type errors without producing output files
- **Commit range analysis**: `git log --format="%H %s %ae %ai" -n 5` — get last 5 commit SHAs, checkout each, run analysis, restore HEAD

### Concepts Learned
Strategy pattern and why it enables extension without modification (Open/Closed Principle), how tree-sitter differs from language-specific parsers, what `tsc --noEmit` does.

### Milestone
Submit a JavaScript repo (e.g., `expressjs/express`) → findings include ESLint issues → commit history chart shows 5 data points. "Declining" badge visible if quality degraded over commits.

---

## Day 23 — Docker Production Build + Nginx + Multi-Stage

### Goal
`docker-compose -f docker-compose.prod.yml up` serves the entire application — React frontend + FastAPI backend — through Nginx, with no CORS errors.

### What You Build
- Multi-stage `Dockerfile` for frontend:
  ```
  Stage 1 (builder): node:18-alpine → npm ci → npm run build → /dist
  Stage 2 (nginx):   nginx:alpine → copy /dist → copy nginx.conf
  ```
- `nginx.conf`:
  - Serve React static files from `/`
  - Proxy `/api/` to FastAPI app service (`proxy_pass http://app:8000`)
  - `gzip on` for static assets
  - Cache headers for JS/CSS/images (1 year, immutable)
  - `try_files $uri $uri/ /index.html` — handles React Router client-side routing
- Multi-stage `Dockerfile` for backend:
  ```
  Stage 1 (builder): python:3.11-slim → pip install → wheel cache
  Stage 2 (runtime): python:3.11-slim → copy installed packages → copy app
  ```
- `docker-compose.prod.yml` with 5 services: `nginx`, `app`, `celery_worker`, `postgres`, `redis`
  - Remove dev volume mounts in prod
  - Add `restart: unless-stopped`
  - Set resource limits: `cpus: 0.5`, `memory: 512M` for worker

### Tech & Concepts
- **Multi-stage Docker builds**: separate build dependencies from runtime. The final image only contains what's needed to run — no Node.js, no build tools, no dev dependencies. Result: images 5-10x smaller
- **Nginx as reverse proxy**: single entry point for both frontend and API — browser talks to port 80 on Nginx, Nginx routes by URL prefix. Eliminates CORS because both frontend and API are same-origin from browser's perspective
- **`try_files $uri $uri/ /index.html`**: React Router uses HTML5 history API — `/repos/123` is a client-side route. Nginx must serve `index.html` for any unknown path, then React Router takes over
- **Build-time vs runtime**: `npm run build` happens at image build time (once) — produces static files that Nginx serves instantly with no runtime overhead
- **Resource limits**: prevent one service from consuming all memory — important in cloud where you pay by resource

### Concepts Learned
Why multi-stage builds exist and the attack surface reduction from smaller images, how reverse proxies work and why they solve CORS, what HTML5 history API routing requires from the server.

### Milestone
`docker-compose -f docker-compose.prod.yml up` → open `http://localhost` → React app loads → submit a repo URL → API call goes to `/api/v1/repos` through Nginx → 200 response → no CORS errors in browser console.

---

## Day 24 — CI/CD Pipeline (GitHub Actions)

### Goal
Every push to `main` triggers automated: lint → type check → test → build → Docker image push. Branch protection requires green CI before merge.

### What You Build
- `.github/workflows/ci.yml`:
  ```yaml
  on: [push, pull_request]
  jobs:
    backend:
      - checkout
      - setup Python 3.11
      - pip install (with cache)
      - flake8 app/ (lint)
      - mypy app/ (type check)  
      - pytest tests/ --cov=app --cov-fail-under=70
      - bandit -r app/ (security scan of our own code)
    frontend:
      - checkout
      - setup Node 18
      - npm ci (with cache)
      - npm run lint
      - npm test -- --coverage --watchAll=false
      - npm run build (verify no build errors)
    docker:
      needs: [backend, frontend]
      - docker build backend
      - docker build frontend
      - if branch=main: push to GitHub Container Registry (ghcr.io)
  ```
- `pyproject.toml`: mypy configuration, flake8 config, pytest config
- `app/py.typed`: marker file for mypy to treat the package as typed
- Branch protection rule: require `backend` and `frontend` checks to pass before merge to main
- README badge: `![CI](https://github.com/user/githelp/actions/workflows/ci.yml/badge.svg)`

### Tech & Concepts
- **GitHub Actions**: CI/CD platform — `.github/workflows/*.yml` defines pipelines triggered by git events
- **Job dependencies**: `needs: [backend, frontend]` — Docker build only runs if both tests pass
- **Caching in Actions**: `actions/cache@v3` for pip and npm — cache by hash of lockfile — cache invalidates only when dependencies change
- **mypy**: static type checker for Python — catches type errors before runtime. `--strict` mode
- **pytest-cov + `--cov-fail-under=70`**: CI fails if test coverage drops below 70% — enforces coverage discipline
- **GitHub Container Registry (ghcr.io)**: free Docker image hosting for GitHub repositories. Authenticated with `GITHUB_TOKEN` (auto-provided by Actions)
- **Branch protection**: GitHub setting that prevents direct push to main and requires status checks — enforces the CI gate

### Concepts Learned
What CI/CD means and why it matters (catch bugs before production), how GitHub Actions matrix builds work, what branch protection rules enforce.

### Milestone
Push a commit with a deliberate flake8 violation → CI fails on lint step → fix it → push again → CI passes → green badge appears in README → attempt to merge a PR with failing tests → GitHub blocks it.

---

## Day 25 — Performance: Caching + Rate Limiting + N+1 Fix + Connection Pooling

### Goal
GET /reviews/{id} hits Redis cache in < 5ms after first request. API rate limited to 5 repo submissions per hour per user. No N+1 queries in ORM calls.

### What You Build
- Redis cache-aside on read-heavy endpoints:
  - `GET /reviews/{id}` → cache key `review:{id}` → TTL 300s
  - `GET /repos/{id}` → cache key `repo:{id}` → TTL 60s
  - Cache invalidation: delete cache key on any write to that resource
- Rate limiting with `slowapi`:
  - `@limiter.limit("5/hour")` on `POST /api/v1/repos` — per user, not per IP
  - `@limiter.limit("100/hour")` on GET endpoints
  - Custom rate limit key: `lambda request: str(request.state.user.id)` — per authenticated user
  - On limit exceeded: 429 response with `Retry-After` header
- N+1 query fix:
  - Identify: `GET /reviews/{id}` with findings was doing 1 query for review + N queries for findings (one per finding for its chunk relationship)
  - Fix: `db.query(Review).options(selectinload(Review.findings)).filter(Review.id == review_id).first()` — loads all findings in 2 queries total
  - `EXPLAIN ANALYZE` in PostgreSQL to verify query plans
- Connection pooling in `database.py`:
  - `create_engine(DATABASE_URL, pool_size=10, max_overflow=20, pool_pre_ping=True)`
  - `pool_pre_ping=True` — test connections before using them (handles dropped connections)

### Tech & Concepts
- **N+1 problem**: loading N reviews then querying each review's findings separately = N+1 DB queries. `selectinload` solves with 2 queries: one for all reviews, one for all their findings (IN clause)
- **`selectinload` vs `joinedload`**: `joinedload` = SQL JOIN (one query but cartesian product). `selectinload` = separate query with IN clause (better for one-to-many). Use `selectinload` for collections
- **slowapi**: rate limiting library for FastAPI — built on Redis counter. `INCR key; EXPIRE key 3600` in Redis = sliding window rate limit
- **Connection pooling**: instead of opening a new DB connection per request (expensive), maintain a pool of persistent connections. `pool_size=10` = 10 permanent connections, `max_overflow=20` = up to 20 temporary extra
- **`EXPLAIN ANALYZE`**: PostgreSQL command showing query execution plan with actual timing — use it to verify indexes are being used
- **Cache TTL strategy**: shorter TTL (60s) for mutable resources (repos can be resubmitted), longer TTL (300s) for immutable resources (completed reviews don't change)

### Concepts Learned
What the N+1 problem is and how ORMs create it silently, how Redis-based rate limiting works with counters, why connection pooling matters under load.

### Milestone
Wrk/locust load test: GET /reviews/{id} → first request ~50ms (DB) → subsequent requests < 5ms (cache). POST /api/v1/repos 6 times as the same user → 6th request returns 429 with `Retry-After: 3600`.

---

## Day 26 — Security Hardening

### Goal
Refresh token rotation working. Secrets scanner flags test credentials. Path traversal prevented. All security findings from bandit resolved.

### What You Build
- Refresh token rotation implementation in `auth_service.py`:
  - `refresh_tokens(refresh_token)` → validate hash → generate NEW access + refresh tokens → invalidate old hash → return new pair
  - If old refresh token used again after rotation → both tokens invalidated (compromise detected)
- Path traversal prevention audit:
  - `git_cloner.py`: validate every extracted file path with `pathlib.Path(base).resolve()` — ensure path is inside clone directory
  - `file_extractor.py`: validate file paths from API requests against allowed base directory
  - Test: `../../../../etc/passwd` in file path → 400 error
- Secrets detection improvements:
  - Regex patterns for: AWS access keys (`AKIA[0-9A-Z]{16}`), GitHub tokens (`ghp_[a-zA-Z0-9]{36}`), private keys (`-----BEGIN RSA PRIVATE KEY-----`), database URLs (`postgresql://user:password@`), API keys in common variable names (`API_KEY=`, `SECRET=`, `PASSWORD=` in assignments)
  - `run_secrets_scan` now scans entire repository before anything else — if secrets found, they're returned as critical findings immediately
- `bandit -r app/` on GitHelp's own codebase → resolve all HIGH and MEDIUM severity findings
- Security headers middleware: add `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `X-XSS-Protection: 1; mode=block` to all responses
- Input validation: `max_file_size_kb` guard in file extractor, `max_url_length` guard in URL validator

### Tech & Concepts
- **Refresh token rotation**: single-use refresh tokens — each refresh invalidates the old token. Detected reuse = compromised refresh token → log out everywhere
- **Path traversal**: `../` sequences in file paths can escape intended directory. `pathlib.Path.resolve()` normalizes the path, then check it starts with the base path
- **`pathlib.Path.resolve()` defense**:
  ```python
  safe_base = Path("/tmp/clones/repo_123").resolve()
  requested = (safe_base / user_provided_path).resolve()
  if not str(requested).startswith(str(safe_base)):
      raise ValueError("Path traversal detected")
  ```
- **Security headers**: HTTP headers that instruct browsers to prevent certain attack classes. `X-Frame-Options: DENY` prevents clickjacking. `nosniff` prevents MIME sniffing attacks
- **Bandit on own code**: run GitHelp's security scanner against GitHelp itself — dogfooding and practical security audit in one

### Concepts Learned
Why refresh token rotation matters (stolen token becomes useless), the full path traversal attack vector, what security headers protect against.

### Milestone
`run_secrets_scan("tests/fixtures/sample_security_issues.py")` returns critical findings for hardcoded credentials. `git_cloner.clone_repository` with path `../../../../etc/passwd` → 400 error. Refresh token used twice → second use returns 401.

---

## Day 27 — Monitoring: Prometheus + Structured Logging + Admin Analytics

### Goal
Prometheus metrics visible. Custom metrics tracked. Correlation IDs in every log line. Admin analytics endpoint returning business metrics.

### What You Build
- `prometheus-fastapi-instrumentator`:
  - Auto-instruments all FastAPI routes: request count, latency histogram, response size
  - Exposes `GET /metrics` in Prometheus text format
- Custom Prometheus metrics in `app/core/metrics.py`:
  ```python
  analysis_duration = Histogram("githelp_analysis_duration_seconds", "Analysis pipeline duration", ["mode"])
  llm_tokens_used = Counter("githelp_llm_tokens_total", "Total LLM tokens consumed")
  findings_per_review = Histogram("githelp_findings_per_review", "Findings count per review", buckets=[0,5,10,20,50,100])
  active_analyses = Gauge("githelp_active_analyses", "Currently running analyses")
  ```
- Instrument LangGraph nodes: record timing with `analysis_duration.labels(mode="full").observe(duration)`
- Structured logging improvements:
  - Add `trace_id` (same as correlation ID), `span_id` (per-function), `user_id`, `repository_id` to all log lines inside analysis tasks
  - Log every LLM call: `{model, tokens_in, tokens_out, duration_ms, prompt_version, finding_count}`
- `GET /api/v1/admin/analytics` (admin role required):
  ```json
  {
    "total_repositories": 142,
    "total_reviews": 389,
    "avg_quality_score": 67.3,
    "total_llm_tokens_used": 2847392,
    "findings_by_severity": {"critical": 234, "major": 891, ...},
    "findings_by_category": {"security": 312, ...},
    "most_common_rules": ["pylint:C0301", "bandit:B105", ...],
    "kb_flywheel": {"times_retrieved": 8923, "times_useful": 6234, "usefulness_ratio": 0.698},
    "reviews_last_7_days": [{"date": "2024-01-01", "count": 12}, ...]
  }
  ```

### Tech & Concepts
- **Prometheus**: time-series metrics database. Scrapes `/metrics` endpoint periodically. Stores metrics with labels
- **Metric types**: Counter (only goes up: total requests), Gauge (can go up/down: active connections), Histogram (distribution: request latency buckets)
- **Labels**: dimensions on metrics — `mode="full"` vs `mode="incremental"` on analysis_duration — allows filtering in queries
- **PromQL**: Prometheus Query Language — `rate(githelp_llm_tokens_total[5m])` = tokens per second over 5 minutes
- **Structured logging vs metrics**: logs = what happened in detail (searchable), metrics = what happened aggregated (queryable). Both are needed
- **`span_id`**: within a request, different functions can have their own span ID — enables distributed tracing (seeing the call tree)

### Concepts Learned
When to use metrics vs logs vs traces (the three pillars of observability), what Prometheus scraping is, why Histograms are more useful than averages for latency.

### Milestone
`curl localhost:8000/metrics` returns Prometheus text format with custom metrics visible. Submit a repo, complete analysis → `githelp_analysis_duration_seconds_bucket` has new observations. Admin analytics endpoint returns correct business metrics.

---

## Day 28 — Cloud Deployment + Final Testing + Documentation + Self-Review

### Goal
GitHelp is live on the internet with HTTPS. Full test suite passes against production. GitHelp reviews its own codebase.

### What You Build
- Cloud deployment on Railway or Render:
  - Connect GitHub repository → auto-deploy on push to main
  - Set all environment variables from `.env.example`
  - Configure custom domain or use provided subdomain
  - HTTPS automatic via Let's Encrypt
  - Add `DATABASE_URL` (Railway PostgreSQL addon), `REDIS_URL` (Railway Redis addon)
  - Verify pgvector extension available in cloud PostgreSQL
- Post-deployment smoke tests:
  - Health check: `GET /health/ready` returns all green
  - Auth: register + login + JWT + protected route
  - Submit GitHelp's own GitHub repo URL → full analysis → review page loads
- `ARCHITECTURE.md` with Mermaid diagrams:
  - System architecture: browser → Nginx → FastAPI → PostgreSQL/Redis/Celery
  - Analysis pipeline: Ingestion → Static → AST → Chunking → Embedding → RAG → LLM → Score → Report
  - Database ER diagram overview (reference the artifact from earlier)
  - Data flow diagram: webhook → queue → analysis → GitHub Checks API
- Demo video (5-7 minutes):
  - Submit GitHelp's own repo → show real-time progress → show review results → show findings with code examples → show trend chart → show GitHub PR comment
- Self-review moment: the quality score and findings on GitHelp's own codebase — document what it found and what you fixed

### Tech & Concepts
- **Railway / Render**: Platform-as-a-Service (PaaS) — they manage servers, load balancers, TLS certificates. You provide Docker image or repo
- **Let's Encrypt**: free automated HTTPS certificate authority — Railway/Render handle renewal automatically
- **Mermaid diagrams**: Markdown-native diagramming — `graph TD`, `sequenceDiagram`, `erDiagram` — renders in GitHub README and documentation sites
- **Smoke tests**: minimal post-deployment tests confirming the system is operational — not comprehensive, just "does it start and respond"
- **Dogfooding**: using your own product — when GitHelp analyzes GitHelp, any real issues in your code are found. Shows the tool works on real codebases

### Final Concepts Learned
PaaS vs IaaS vs self-hosting tradeoffs, what TLS termination at the load balancer means, why dogfooding matters for product credibility.

### Milestone
Live URL accessible over HTTPS. GitHelp's own GitHub repo submitted → analysis completes → quality score visible → at least 10 real findings on your own code → ARCHITECTURE.md merged to main → demo video recorded.

---

# APPENDIX: AI Concept Learning Arc (Dependency Order)

```
Day 1-7   Foundation ──────────────────────────────────────────────────────
          Docker, PostgreSQL, FastAPI, SQLAlchemy, Celery, JWT

Day 8-9   Data Collection ────────────────────────────────────────────────
          GitPython clone → Static analysis tools → Normalized findings

Day 10    AST Parsing ──────────────────────────────────────────────────
          Python ast module → Code structure understanding → networkx graphs

Day 11    Code Chunking ────────────────────────────────────────────────
          tiktoken → Token counting → AST-aware boundary detection

Day 12    Basic LLM Integration ─────────────────────────────────────────
          Jinja2 prompts → Few-shot prompting → JSON output validation

Day 13    Hybrid Engine ────────────────────────────────────────────────
          Combining static + AST + LLM → Source confidence → Cost optimization

Day 14    GitHub Integration ────────────────────────────────────────────
          PR comments → Checks API → Webhook HMAC verification

Day 15    Embeddings ────────────────────────────────────────────────────
          Vector representations → pgvector → Cosine similarity → HNSW

Day 16    RAG ───────────────────────────────────────────────────────────
          Knowledge base → Retrieval → Re-ranking → Context injection → Flywheel

Day 17    Prompt Engineering ────────────────────────────────────────────
          Chain-of-thought → Versioning → A/B testing → Constitutional constraints

Day 18    LangGraph ─────────────────────────────────────────────────────
          StateGraph → Nodes → Conditional edges → Error routing

Day 19    Agents ────────────────────────────────────────────────────────
          ReAct pattern → Tool definitions → Function calling → Cross-file context

Day 20-21 Frontend ──────────────────────────────────────────────────────
          React + Axios interceptors → Real-time polling → Data visualization

Day 22-28 Production ────────────────────────────────────────────────────
          Multi-language → CI/CD → Security → Monitoring → Cloud deployment
```

---

# APPENDIX: Tech Stack Reference

| Layer | Technology | Purpose |
|-------|-----------|---------|
| API | FastAPI + Uvicorn | Async web framework |
| Database | PostgreSQL + pgvector | Relational data + vector similarity |
| ORM | SQLAlchemy 2.0 | Database models and queries |
| Validation | Pydantic v2 | Request/response validation |
| Auth | python-jose, passlib, cryptography | JWT, bcrypt, Fernet |
| Task Queue | Celery + Redis | Background jobs |
| Static Analysis | pylint, flake8, radon, bandit | Code quality tools |
| Dependency Audit | pip-audit, detect-secrets | Security scanning |
| AST Analysis | Python ast, networkx | Structural analysis |
| Tokenization | tiktoken | Token counting for LLM budget |
| LLM | Anthropic Claude / OpenAI GPT-4 | AI code review |
| Embeddings | OpenAI text-embedding-3-small | Semantic vectors |
| Vector DB | pgvector (HNSW index) | Similarity search |
| AI Pipeline | LangGraph | Stateful analysis graphs |
| Prompt Templates | Jinja2 | Prompt version management |
| Frontend | React + Vite + Tailwind | User interface |
| HTTP Client | Axios | API calls with interceptors |
| Charts | recharts | Data visualization |
| Syntax Highlighting | highlight.js | Code display |
| Reverse Proxy | Nginx | Static serving + API proxy |
| Containerization | Docker + Docker Compose | Environment isolation |
| CI/CD | GitHub Actions | Automated testing and deployment |
| Rate Limiting | slowapi | API abuse prevention |
| Monitoring | prometheus-fastapi-instrumentator | Metrics |
| Logging | python-json-logger | Structured logs |
| Cloud | Railway / Render | Production hosting |

---

# APPENDIX: Database Tables Reference

| Table | Purpose | Key Design Choice |
|-------|---------|------------------|
| users | Auth, profiles, OAuth tokens | Fernet-encrypted tokens, bcrypt refresh hash |
| repositories | Submitted repos, task tracking | metadata_ (reserved name fix), JSONB config |
| reviews | Analysis runs, scores, costs | Denormalized counts, prompt_version tracking |
| findings | Individual code issues | severity_rank int, source enum, full lifecycle |
| code_chunks | AST-aware chunks + embeddings | Vector(1536), HNSW index, server_default=[] |
| knowledge_base | RAG context source | Flywheel: times_retrieved/useful, soft delete |
| commit_history | Quality trend over time | Immutable snapshots, score_delta, UniqueConstraint |
| finding_feedback | Developer feedback | Powers KB flywheel, ondelete SET NULL |
| api_keys | Programmatic access | key_prefix shown, key_hash only stored |
| audit_logs | Immutable event trail | Append-only, survives user deletion |
