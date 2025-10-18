Title: softwareAndComputeTwo — Task 1: "Compute Two" library and endpoints

Goal
- Implement a small, well-typed compute module and corresponding HTTP endpoints that perform addition and multiplication of exactly two numbers, with input validation and consistent error handling. Provide unit tests for the pure functions and integration tests for the endpoints.

Summary
- Create a TypeScript-based Node.js service exposing:
  - POST /api/v1/compute/two/sum — returns the sum of two numbers
  - POST /api/v1/compute/two/product — returns the product of two numbers
- Backed by a library module exporting sumTwo(a, b) and productTwo(a, b) with strict input validation.
- Include a basic health endpoint returning service status and version.

Affected files/paths
- New (Node + TypeScript project scaffold):
  - package.json (scripts: build, start, dev, test)
  - tsconfig.json
  - .eslintrc.cjs (or .eslintrc.js) and .prettierrc (optional but preferred)
  - src/index.ts (server bootstrap)
  - src/routes/computeTwo.ts (binds routes)
  - src/controllers/computeTwoController.ts (HTTP handlers)
  - src/lib/computeTwo.ts (pure functions: sumTwo, productTwo, and validation utilities)
  - src/routes/health.ts (GET /api/health)
  - tests/lib/computeTwo.test.ts (unit tests)
  - tests/integration/computeTwo.api.test.ts (supertest-based API tests)
- Modified/Added (if present):
  - README.md: add quick start instructions and endpoints table

Specific functions and endpoints
- Library (src/lib/computeTwo.ts):
  - export function sumTwo(a: number, b: number): number
    - Validates that a and b are finite numbers (Number.isFinite).
    - Returns a + b (ensure result is finite; otherwise throw a domain error).
  - export function productTwo(a: number, b: number): number
    - Validates that a and b are finite numbers.
    - Returns a * b (ensure result is finite; otherwise throw a domain error).
  - export function assertFiniteNumbers(...values: unknown[]): asserts values are number[]
    - Throws a typed Error with a stable message when validation fails.

- HTTP endpoints (Express):
  - POST /api/v1/compute/two/sum
    - Request body: { a: number, b: number }
    - 200: { result: number }
    - 400: { error: 'Invalid payload: "a" and "b" must be finite numbers.' }
  - POST /api/v1/compute/two/product
    - Request body: { a: number, b: number }
    - 200: { result: number }
    - 400: { error: 'Invalid payload: "a" and "b" must be finite numbers.' }
  - GET /api/health
    - 200: { status: 'ok', version: string, uptime: number }

- Server bootstrap (src/index.ts):
  - Creates an Express app with JSON body parsing, mounts routes:
    - /api/health
    - /api/v1/compute/two/*
  - Listens on PORT from env (default 3000).

Acceptance criteria
- Project
  - Node.js v20+ and TypeScript configured.
  - npm scripts:
    - dev: ts-node-dev (or nodemon) for local development
    - build: tsc compiles to /dist
    - start: node dist/index.js
    - test: runs unit and integration tests with Jest + ts-jest
  - Lint passes (ESLint) and formatting applied (Prettier if added).

- Library behavior
  - sumTwo(1, 2) === 3
  - sumTwo(-5, 2.5) === -2.5
  - productTwo(3, 4) === 12
  - productTwo(-2, 3.5) === -7
  - For floating-point sums/products that cannot be represented exactly, tests use toBeCloseTo with precision 10.
  - Passing NaN, Infinity, -Infinity, null, undefined, strings, or objects throws a validation error with message: 'Invalid payload: "a" and "b" must be finite numbers.'
  - Results that are not finite (overflow to Infinity) produce a domain error with message: 'Computation result is not finite.'

- API behavior
  - POST /api/v1/compute/two/sum with { a: 1, b: 2 } -> 200 { result: 3 }
  - POST /api/v1/compute/two/product with { a: 3, b: 4 } -> 200 { result: 12 }
  - Invalid body (missing a or b, non-numeric, non-finite) -> 400 with { error: 'Invalid payload: "a" and "b" must be finite numbers.' }
  - Overflows or non-finite results -> 422 with { error: 'Computation result is not finite.' }
  - GET /api/health -> 200 with { status: 'ok', version: <from package.json>, uptime: <number> }

- Testing
  - Unit tests for sumTwo and productTwo covering: integers, negatives, floats, large numbers, invalid inputs.
  - Integration tests for both endpoints: happy paths, validation errors, and overflow handling.
  - Minimum 80% line coverage for src/lib/computeTwo.ts.

Edge cases
- Inputs include:
  - Non-finite numbers: NaN, Infinity, -Infinity -> reject with 400.
  - Non-numeric types: string, boolean, object, array, null, undefined -> reject with 400.
  - Extremely large magnitudes where a*b or a+b exceeds Number.MAX_VALUE -> respond 422 with domain error.
  - Floating point precision anomalies (e.g., 0.1 + 0.2) are allowed; tests should assert with toBeCloseTo rather than strict equality when appropriate.
  - Payloads with extra properties should be ignored but not cause failure (only 'a' and 'b' are read).

Constraints
- Use Express and TypeScript; avoid introducing heavy math libraries (no decimal.js/bignumber.js) unless strictly necessary.
- Keep public API stable and messages identical to those specified above.
- Uniform error response shape: { error: string } with the exact messages provided.
- No global mutable state.
- Do not add a database; this service is stateless.

Non-goals/Out of scope
- Support for more than two operands.
- Division/subtraction endpoints.
- Authentication or rate limiting.

Example requests
- Sum
  curl -s -X POST http://localhost:3000/api/v1/compute/two/sum \
    -H 'Content-Type: application/json' \
    -d '{"a": 0.1, "b": 0.2}'

- Product
  curl -s -X POST http://localhost:3000/api/v1/compute/two/product \
    -H 'Content-Type: application/json' \
    -d '{"a": 2, "b": 5}'

- Health
  curl -s http://localhost:3000/api/health
