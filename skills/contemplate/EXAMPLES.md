# Contemplate — Examples

---

## Example 1: Full mode, no prior knowledge

**User message:** `contemplate`

**Intake:**

> Agent: "Tell me anything you know about this project — what it does, who built it, why it exists. It's OK if you don't know much."
>
> User: "Honestly no idea. I inherited this repo last week and the team that built it left the company."

**Tracing finds:**
- Decision: Uses SQLite locally instead of a remote database (`src/db/client.ts:8`)
- Constraint: `.env.example` lists `SYNC_URL` — suggests data syncs somewhere (`.env.example`)
- Absence: No caching anywhere in the codebase (no cache deps/config/imports found)
- Module boundary: `core/`, `adapters/`, `handlers/` — hexagonal architecture (`src/core/index.ts:1`, `src/adapters/db.ts:1`)
- Git: early commits say "initial POC" and "need offline support" (`a1b2c3d`, `d4e5f6a`)

**Infer produces:**

🔵 Conclusion: This is a local-first app. SQLite + sync URL = offline-first architecture (`src/db/client.ts:8`, `.env.example`)
🟡 Hypothesis: No caching because local reads are already fast — cache would add complexity for no gain. Supporting evidence: SQLite local reads. Undermining evidence: no benchmark exists.
🟡 Hypothesis: No caching because data freshness was prioritized over read performance. Supporting evidence: sync URL + offline commits. Undermining evidence: no explicit consistency comment.

**Map produces cascade:**

Decision (local-first SQLite) → needed sync → built bidirectional sync protocol → skipped caching (local reads fast enough, data freshness priority)

**Output pages:**
- `contemplate/README.md`
- `contemplate/local-first-architecture.md`
- `contemplate/sync-protocol.md`
- `contemplate/caching-absence.md`
- `contemplate/module-structure.md`

---

## Example 2: Topic-filtered mode

**User message:** `contemplate auth flow, token handling`

**Intake:**

> Agent: "Tell me anything you know about this project."
>
> User: "It's an internal dashboard for our sales team. I'm trying to understand why auth is so complicated."

**Tracing scopes to auth and tokens only.**

Finds:
- Custom JWT implementation (not using Auth0/Clerk) (`src/auth/jwt.ts:1`)
- Middleware runs on every route — centralized auth (`src/routes/index.ts:9`)
- Refresh token rotation with sliding window (`src/auth/refresh-token.ts:35`)
- Custom claims in JWT payload (role, team, region) (`src/auth/jwt.ts:22`)

**Infer produces:**

🔵 Conclusion: Custom auth exists to issue JWTs with custom claims (role/team/region) (`src/auth/jwt.ts:22`)
🟡 Hypothesis: The app was built before managed auth was mature enough for their needs. Supporting evidence: old auth commits. Undermining evidence: no commit explicitly rejects managed auth.
🟡 Hypothesis: Cost or vendor lock-in concerns. Supporting evidence: no external auth dependency. Undermining evidence: no billing/vendor-lock-in evidence.

**Output pages:**
- `contemplate/README.md` (scoped to auth and token topics)
- `contemplate/auth-decisions.md`
- `contemplate/token-architecture.md`

---

## Example 3: Rich intake

**User message:** `contemplate`

**Intake:**

> Agent: "Tell me anything you know about this project."
>
> User: "It's a video processing pipeline that our ML team built about 2 years ago. It takes uploads, transcodes them, and stores them in S3. I think they used FFmpeg but I'm not sure about the details."

**Intake gives rich context.** The trace now has a starting point — video processing, FFmpeg, S3 storage, ML team, ~2 years old.

This collapses the inference space significantly. The tracer can look for:
- FFmpeg usage and its configuration (which codecs, what flags, why those choices)
- Upload handling (streaming? buffered? size limits? — reveals constraints)
- S3 integration patterns (direct upload? signed URLs? — reveals security decisions)
- Whether ML processing happens inline or async (reveals architecture decisions)

The intake doesn't replace tracing — it gives it direction.