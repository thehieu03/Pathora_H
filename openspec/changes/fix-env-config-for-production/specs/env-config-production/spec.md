## ADDED Requirements

### Requirement: Backend CORS includes production frontend domain

The backend `.env` SHALL include `https://vivugo.me` in the CORS `AllowedOrigins` list so that the production frontend can call API endpoints.

#### Scenario: Frontend from production domain can call API
- **WHEN** a browser on `https://vivugo.me` makes a request to `https://api.vivugo.me`
- **THEN** the backend CORS policy permits the request with correct headers

#### Scenario: Localhost development still works
- **WHEN** a browser on `localhost:3000` or `localhost:3001` makes a request to the backend
- **THEN** the backend CORS policy permits the request

### Requirement: Backend JWT issuer and audience match production domain

The backend `.env` SHALL set `Jwt__Issuer` and `Jwt__Audience` to `https://api.vivugo.me` so that JWT tokens are valid when consumed by the production frontend.

#### Scenario: JWT token accepted at production
- **WHEN** frontend sends a request with Bearer token to `https://api.vivugo.me`
- **THEN** the JWT validation passes because issuer and audience match

### Requirement: Backend .env contains only variables actually read by code

The backend `.env` SHALL contain only environment variables that are actually read by the application code. Unused variables SHALL be removed.

#### Scenario: All .env variables are consumed by code
- **WHEN** backend starts with the updated `.env`
- **THEN** every variable in `.env` is read by at least one configuration GetSection or configuration[] call

### Requirement: Frontend NEXT_PUBLIC_API_GATEWAY points to production backend

The frontend `.env` SHALL set `NEXT_PUBLIC_API_GATEWAY` to `https://api.vivugo.me` so that API calls reach the production backend.

#### Scenario: Frontend API calls reach production backend
- **WHEN** the frontend makes an API request
- **THEN** the request is sent to `https://api.vivugo.me`

### Requirement: Frontend .env contains only variables actually read by code

The frontend `.env` SHALL contain only environment variables that are actually referenced in the codebase. Unused DEV_* variables SHALL be removed.

#### Scenario: All frontend .env variables are consumed
- **WHEN** frontend builds with the updated `.env`
- **THEN** every variable in `.env` is referenced in at least one source file
