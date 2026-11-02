# Backend #5 API contract

Owner-only Cognito identity is checked against reservation.CustomerId.

- GET /guest/groups/{groupReservationId}/sessions returns participants and total.
  Participant fields: id, type, name, items, total, isSubmitted.
  Each item contains dish (including id/name/price) and quantity.
- GET /reservations/{reservationId}/guest-session-id returns guestSessionId.
- POST /group-reservations/{groupReservationId}/submit accepts
  {"reservationId":"reservation-id"}; returns [{"dishId":"dish-id","quantity":3}].
  Only submitted sessions are consolidated; statuses become SubmittedByCustomer
  for order and Submitted for group. Missing submitted sessions/dishes/order:400.
- DELETE /guest/groups/{groupReservationId}/sessions/{guestSessionId} returns
  the updated participants/total; wrong group:400, missing session:404.

All successful responses:200, camelCase. Missing identity:401, non-owner:403,
invalid path/body:400, missing reservation/group:404. Repeated group submit:400.
No deployment or production-concurrency guarantees are made by this package.
