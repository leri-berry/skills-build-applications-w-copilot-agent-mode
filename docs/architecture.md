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

- Presentation tier decision: Use React Router and React 19 components for the fitness capabilities, Bootstrap for layout and feedback styling, and the shared `src/api/client.ts` module for every API request. Preserve loading, success, empty, and accessible error states.
- Logic tier decision: Keep Express route handlers under `/api/` thin, put validation and business behavior in explicit TypeScript service functions, and return deliberate 4xx responses for invalid identifiers or missing resources.
- Data tier decision: Use Mongoose schemas for required fields and relationships, the shared database configuration for MongoDB connections, and stable upsert-based seed records. Do not clear collections or leave demo-team membership inconsistent.
- Activity tracking approach: Validate `userId` and `teamId` before Mongoose writes, verify that both referenced records exist, and create an activity with the submitted values plus computed `points` and `occurredAt`.
- Team management approach: Validate the team name in the schema and service, preserve submitted `name` and `mascot`, and keep team membership represented by the team model.
- Leaderboard approach: Aggregate activity points by team, sort totals from highest to lowest, and add deterministic 1-based ranks for the API response.
- Workout suggestions approach: Find the seeded user by email, inspect that user's recent activities to derive a goal or activity-based focus, and return an explanatory reason. Return a 404 for an unknown email.

## Test plan

Record the commands you run and what each failure means.

- Data-tier command: `npm run test:data --prefix octofit-tracker/backend` - checks Mongoose validation, stable seed behavior, and demo fixture reconciliation.
- API command: `npm run test:api --prefix octofit-tracker/backend` - checks activity, team, leaderboard, and workout API contracts and deliberate 4xx handling.
- Frontend command: `npm test --prefix octofit-tracker/frontend` - checks shared API usage and visible loading, success, empty, and error states.
- Backend build command: `npm run build --prefix octofit-tracker/backend` - verifies that the Express and TypeScript implementation has no build/type errors.
- Frontend build command: `npm run build --prefix octofit-tracker/frontend` - verifies that React, Router, and Vite compile successfully.
