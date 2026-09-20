# Sanctum Sanctorum Bookstore - Implementation Notes

## Live Deployment
- **API Live URL**: https://sanctum-sanctorum.onrender.com
- **Database**: Neon Serverless PostgreSQL (`ep-nameless-bonus-b5cmj7ql-pooler`)
- **Local Database**: SQLite (`sanctum.db` / in-memory for testing)

---

## Architecture & Technical Decisions

### 1. Database & Persistence Layer
- **Local Testing**: SQLite is used for fast, zero-dependency local testing (`uv run pytest`), ensuring 100% of the test suite (202 tests) runs isolated without external network calls.
- **Production**: Deployed with **Neon Serverless PostgreSQL**. Managed via SQLAlchemy 2.0 ORM with `SANCTUM_DATABASE_URL` dynamic connection string configuration.

### 2. Validation & Business Logic
- **Schema Validation**: All input validation (ISBN-13 check digit calculation via modulo 10 algorithm, email whitespace stripping & lowercasing, Order items uniqueness) is handled at the Pydantic v2 schema layer. This ensures bad requests are rejected with `422 Unprocessable Entity` before business logic or database queries run.
- **Transaction Safety**: All order placements (`create_order`) and returns (`cancel_order` / `return_loan`) handle inventory stock checks and decrements/increments atomically within database session transactions.

---

## AI Usage Log

- **Tools Used**: Google Antigravity IDE (Gemini 3.6 Flash & Claude)
- **Use Cases**:
  - **Architecture & Task Planning**: Generating incremental commit strategies aligned with project evaluation rubrics.
  - **Schema & Math Validation**: Writing the ISBN-13 check-digit algorithm and loan late fee calculation (25 cents/day, rounded up via `ceil`, capped at book price).
  - **Test Suite Verification**: Running pytest suites iteratively to ensure zero test regressions.
- **AI Override Example**:
  - When implementing `tier_at_least(tier, minimum)` in `app/services/members.py`, the initial draft used strictly `>` instead of `>=`. This caused `MASTER` tier members to be denied access to restricted books (requiring `MASTER` tier or above). I recognized the boundary comparison bug from `SPEC.md` and corrected it to `>=`.
