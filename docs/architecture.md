# OctoFit Tracker architecture

OctoFit Tracker is a three-tier application for a school fitness program. The starter code includes API contracts and intentionally incomplete implementation points so you can use GitHub Copilot agent mode in a test-driven loop.

![Three-tier OctoFit architecture](../.github/images/octofit-architecture.svg)

## Tier boundaries

- **Presentation tier**: React 19, Vite, React Router, Bootstrap, and a shared API client in `octofit-tracker/frontend/src`.
- **Logic tier**: Express and TypeScript routes/services in `octofit-tracker/backend/src` expose `/api/*` behavior on port `8000`.
- **Data tier**: Mongoose models and seed data in `octofit-tracker/backend/src/models` and `src/scripts/seed.ts` persist to MongoDB on port `27017`.

## Ports and environment rules

- MongoDB: `27017`
- Express API: `8000`
- Vite frontend: `5173`
- The browser calls relative `/api` paths. Vite proxies them to `http://localhost:8000`, so the unauthenticated API port stays private in Codespaces. `VITE_API_BASE_URL` can override the base URL.

## API contract

| Capability | Method and path | Expected behavior |
| --- | --- | --- |
| Activity tracking | `GET /api/activities`, `POST /api/activities` | List and create valid activity records. POST accepts `userId`, `teamId`, `type`, and `minutes`; `userId` and `teamId` must be well-formed IDs for an existing user and team; otherwise respond with a deliberate 4xx (never a 500 from a cast error). A created activity stores the submitted `user`, `team`, `type`, and `minutes` plus computed `points` and `occurredAt`. |
| Teams | `GET /api/teams`, `POST /api/teams` | List and create teams with members. Created and listed teams keep the submitted `name` and `mascot`. Reject a team without a name with `400`. |
| Leaderboard | `GET /api/leaderboard` | Return teams ordered by the sum of their activity points, highest first. Each entry has `teamName`, `totalPoints`, and a 1-based `rank`. |
| Workout suggestions | `GET /api/workouts/suggestions/:userEmail` | Recommend workouts from recent activity patterns for the matching seeded user's email address. The first suggestion's `focus` matches that user's goal or recent activity, with a `reason` explaining why. An email with no matching user returns a deliberate 4xx (for example, `404`). |

## Implementation decisions

Record your plan here before editing code.

- Presentation tier decision:Add React routes for Activities, Teams, Leaderboard, and Workouts; use a shared API client and accessible loading, empty, and error states.
- Logic tier decision:Implement Express /api/* handlers with typed request validation and deliberate 4xx responses for invalid IDs or missing records.
- Data tier decision: Complete the Mongoose schemas and retain seed data that is representative and idempotent.
- Activity tracking approach:Validate user/team IDs and references before creating activities. Store computed positive points and an occurredAt timestamp.
- Team management approach:Require name and mascot; default members to an empty array and expose team members in API responses.
- Leaderboard approach: Aggregate activity points by team, sort highest first, and assign one-based ranks.
- Workout suggestions approach:Match the user by email, infer a focus from goal or recent activity, filter matching workouts, and include a reason.

## Test plan

Record the commands you run and what each failure means.

- Data-tier command: `npm run test:data --prefix octofit-tracker/backend`
Verifies Mongoose validation, linked seed records, and idempotent seeding.
Run first; failures indicate incomplete or overstrict data schemas.
- API command: `npm run test:api --prefix octofit-tracker/backend`
Verifies activity/team CRUD, leaderboard aggregation, workout matching, and 4xx handling.
Run after data contracts pass; failures indicate missing or incorrect Express behavior.
- Frontend command: `npm test --prefix octofit-tracker/frontend`
Verifies API data rendering and loading, empty, and accessible error states.
Run after API contracts pass.
- Backend build command: `npm run build --prefix octofit-tracker/backend`
Catches TypeScript and backend build issues.
- Frontend build command: `npm run build --prefix octofit-tracker/frontend`
Catches TypeScript, React, and Vite integration issues.
