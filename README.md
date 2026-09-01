# RaceDay — API Endpoint Plan

Roles in this system: **Organiser** (creates and manages events, categories, and results) and **Participant** (browses events and enrols in categories). **Any** means any logged-in user regardless of role. **None** means no authentication required.

## Authentication

| HTTP method | Route | Description | Role required | Request body | Expected response |
|---|---|---|---|---|---|
| POST | /api/auth/register | Creates a new user account as either an Organiser or a Participant. | None | { "fullName", "email", "password", "role" } | 201 Created — new user record (no password) <br> 400 Bad Request — validation failed <br> 409 Conflict — email already registered |
| POST | /api/auth/login | Authenticates a user and returns a JWT for subsequent requests. | None | { "email", "password" } | 200 OK — { token, user } <br> 401 Unauthorized — invalid credentials |

## User Profile

| HTTP method | Route | Description | Role required | Request body | Expected response |
|---|---|---|---|---|---|
| GET | /api/users/me | Returns the logged-in user's own profile. | Any | None | 200 OK — user profile <br> 401 Unauthorized |
| PUT | /api/users/me | Updates the logged-in user's own profile details. | Any | { "fullName", "email" } | 200 OK — updated profile <br> 400 Bad Request <br> 401 Unauthorized |
| PUT | /api/users/me/password | Changes the logged-in user's password. | Any | { "currentPassword", "newPassword" } | 200 OK — confirmation <br> 400 Bad Request <br> 401 Unauthorized |

## Events

| HTTP method | Route | Description | Role required | Request body | Expected response |
|---|---|---|---|---|---|
| GET | /api/events | Lists all published events, with optional filters (date, location). | None | None | 200 OK — array of events |
| GET | /api/events/{id} | Returns full details for a single event, including its categories. | None | None | 200 OK — event with nested categories <br> 404 Not Found |
| POST | /api/events | Creates a new event owned by the logged-in Organiser. | Organiser | { "eventName", "location", "eventDate", "status" } | 201 Created — new event <br> 400 Bad Request <br> 401 Unauthorized |
| PUT | /api/events/{id} | Updates an event the logged-in Organiser owns. | Organiser | { "eventName", "location", "eventDate", "status" } | 200 OK — updated event <br> 403 Forbidden — not the owner <br> 404 Not Found |
| DELETE | /api/events/{id} | Deletes an event the logged-in Organiser owns (only if it has no enrolments). | Organiser | None | 204 No Content <br> 403 Forbidden <br> 404 Not Found <br> 409 Conflict — event has active enrolments |

## Categories

| HTTP method | Route | Description | Role required | Request body | Expected response |
|---|---|---|---|---|---|
| GET | /api/events/{eventId}/categories | Lists all categories (distances/divisions) for a given event. | None | None | 200 OK — array of categories <br> 404 Not Found — event does not exist |
| POST | /api/events/{eventId}/categories | Adds a new category to an event the logged-in Organiser owns. | Organiser | { "categoryName", "distanceKm", "maxParticipants", "price" } | 201 Created — new category <br> 403 Forbidden <br> 404 Not Found |
| PUT | /api/categories/{id} | Updates a category belonging to an event the logged-in Organiser owns. | Organiser | { "categoryName", "distanceKm", "maxParticipants", "price" } | 200 OK — updated category <br> 403 Forbidden <br> 404 Not Found |
| DELETE | /api/categories/{id} | Deletes a category (only if it has no enrolments). | Organiser | None | 204 No Content <br> 403 Forbidden <br> 409 Conflict — category has enrolments |

## Event Enrolments

| HTTP method | Route | Description | Role required | Request body | Expected response |
|---|---|---|---|---|---|
| POST | /api/categories/{categoryId}/enrolments | Enrols the logged-in Participant into a category, if capacity allows. | Participant | None | 201 Created — enrolment record with bib number <br> 404 Not Found — category does not exist <br> 409 Conflict — category full or already enrolled |
| GET | /api/users/me/enrolments | Lists all enrolments made by the logged-in Participant. | Participant | None | 200 OK — array of enrolments |
| GET | /api/categories/{categoryId}/enrolments | Lists all enrolments for a category the logged-in Organiser owns (roster view). | Organiser | None | 200 OK — array of enrolments <br> 403 Forbidden |
| DELETE | /api/enrolments/{id} | Cancels an enrolment (by the enrolled Participant, or the owning Organiser). | Any (owner or Organiser) | None | 204 No Content <br> 403 Forbidden <br> 404 Not Found |

## Results

| HTTP method | Route | Description | Role required | Request body | Expected response |
|---|---|---|---|---|---|
| POST | /api/enrolments/{enrolmentId}/results | Records a result for an enrolment, in an event the logged-in Organiser owns. | Organiser | { "finishTime", "position", "status" } | 201 Created — new result <br> 400 Bad Request <br> 403 Forbidden <br> 404 Not Found |
| PUT | /api/results/{id} | Corrects a previously recorded result. | Organiser | { "finishTime", "position", "status" } | 200 OK — updated result <br> 403 Forbidden <br> 404 Not Found |
| GET | /api/categories/{categoryId}/results | Returns the results/leaderboard for a category, ordered by position. | None | None | 200 OK — array of results <br> 404 Not Found |
| GET | /api/users/me/results | Lists all results belonging to the logged-in Participant. | Participant | None | 200 OK — array of results |

