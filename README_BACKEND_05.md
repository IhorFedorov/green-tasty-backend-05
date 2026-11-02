# EPAM Backend Homework #5 — Owner Group Pre-order

Local implementation of Levels 1–3 in the existing .NET 10 backend.
Implements the EPAM Backend Homework #5 contract summarized below.
API reference: [Backend #5 API](docs/BACKEND_05_API.md). No cloud deployment is claimed.

## Implemented API

All four endpoints authorize the reservation Owner using authenticated `sub`.
Missing identity returns 401; another user returns 403. Invalid input returns
400; missing reservation/group/session returns 404 where applicable.

| Level | Endpoint | Success (200) |
|---|---|---|
| 1 | `GET /guest/groups/{groupReservationId}/sessions` | `participants` snapshot and `total`; draft and submitted sessions |
| 2 | `GET /reservations/{reservationId}/guest-session-id` | `{"guestSessionId":"owner-session-id"}` |
| 2 | `POST /group-reservations/{groupReservationId}/submit` | Array of consolidated `dishId`/`quantity` |
| 3 | `DELETE /guest/groups/{groupReservationId}/sessions/{guestSessionId}` | Updated snapshot and total, excluding the deleted ID |

POST body: `{"reservationId":"reservation-id"}`. Only submitted sessions are
aggregated by dishId. Existing order becomes `SubmittedByCustomer`; group becomes
`Submitted`. No submitted sessions, no dishes, or missing order returns 400.
Repeated submit returns 400 without adding dishes again. Wrong-group DELETE is
400; missing session is 404. Responses, including nested fields, use camelCase.

## Run local tests

Prerequisite: .NET SDK 10 and restored NuGet packages. From this directory:

```sh
dotnet restore Tests/APIHandler.Tests.csproj
dotnet test Tests/APIHandler.Tests.csproj --no-restore -m:1 -p:UseSharedCompilation=false --filter 'FullyQualifiedName~GroupSessionsLevel1Tests|FullyQualifiedName~GroupPreOrderLevel2Tests|FullyQualifiedName~DeleteGuestSessionLevel3Tests'
dotnet test Tests/APIHandler.Tests.csproj --no-build --no-restore -m:1 --filter 'FullyQualifiedName~GuestSessionServiceTests|FullyQualifiedName~GroupReservationServiceTests|FullyQualifiedName~OrderServiceTests|FullyQualifiedName~GroupSessionsLevel1Tests|FullyQualifiedName~GroupPreOrderLevel2Tests'
```

Restore is needed only on a clean checkout. Tests use mocks; AWS credentials
and deployed resources are not needed. No deployment commands are required.

Recorded local results on 2026-11-02 (not rerun when writing this README):

| Verification | Passed | Failed/skipped |
|---|---:|---:|
| Level 1 targeted | 17 | 0/0 |
| Level 2 targeted | 41 | 0/0 |
| Level 3 HTTP/service targeted | 18 | 0/0 |
| Latest adjacent regression, including Levels 1/2 | 144 | 0/0 |

The combined focused command above is a reproduction command; the targeted
results were collected separately. Regression includes the Level 1/2 tests;
these counts must not be summed as independent coverage. Existing NU1902/NU1903
dependency advisories remain. Full-suite and cloud/browser testing are unverified.

## Guarantees and limitations

- New Owner sessions use a deterministic key and conditional creation; a visible
  existing Owner session is reused. Owner counting reserves exactly one seat.
- One conditional DynamoDB transaction writes the consolidated Order and Group.
  Mock tests inspect the request and simulate failures; they do not verify AWS.
- GuestSessions are read through an eventually consistent GSI. Submitted sessions
  or legacy Owners can be missing temporarily. Global snapshot isolation and
  coordinated writers are deliberately not implemented.
- Concurrent legacy writers can overwrite data after the transaction. A timeout
  can mean both records committed; it does not prove rollback.
- DELETE filters its deleted ID from a stale GSI response. Other rows may lag;
  subsequent catalog/profile/read failure does not undo the deletion.
- Owner-ID GET does not issue a guest token or extend Owner cart authorization.
  No automatic deadline, database schema, index or IAM changes were introduced.
- Existing public API/S3 deployment security findings remain separate blockers
  for deployment. This package is source/test evidence, not a deployment guide.


## Standalone submission snapshot

This package contains only the API Lambda .NET project and its tests; other
Lambda projects, frontend, Terraform and deployment bundles are not required
for the local homework checks. Main-repository Git history is not included.
See [NOTICE](NOTICE.md) for EPAM attribution and the submission context.
No separate EPAM license or new license grant is claimed.
Use config/environment.example.json only as a variable-name template; do not
enter credentials. Unit tests use mocks and do not need those variables.
The original repository and its history/visibility were not changed.
