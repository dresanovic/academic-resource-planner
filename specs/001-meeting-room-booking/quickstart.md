# Quickstart: Meeting Room Booking Validation

## Prerequisites

- Python 3.11 available locally.
- Project dependencies installed for the backend service and tests.
- A local test database with at least two rooms and one known user.

## Run Validation

1. Start the backend service.
2. Confirm the service exposes the booking contract described in [contracts/booking-api.yaml](contracts/booking-api.yaml).
3. Run the automated test suite.

Expected result: all contract, integration, and unit tests pass.

## Manual Validation Scenarios

### Create an Available Booking

Request a booking for Room A from a future 09:00 to 10:00 time range aligned to 15-minute increments.

Expected result: the booking is accepted and the response includes the booking id, room id, user id, start time, end time, and `accepted` status.

### Reject Same-Room Overlap

With Room A already booked from 09:00 to 10:00, request Room A from 09:30 to 10:30.

Expected result: the request is rejected with an overlap reason that identifies Room A and the requested time range.

### Allow Adjacent Same-Room Booking

With Room A already booked from 09:00 to 10:00, request Room A from 10:00 to 11:00.

Expected result: the booking is accepted because the time ranges touch but do not overlap.

### Allow Different-Room Concurrent Booking

With Room A already booked from 09:00 to 10:00, request Room B from 09:00 to 10:00.

Expected result: the booking is accepted for Room B.

### Reject Invalid Time Rules

Submit booking requests with these invalid inputs:

- End time equal to start time.
- Start time in the past.
- Start or end time not aligned to a 15-minute increment.
- Unknown room id.
- Room not available for booking.

Expected result: each request is rejected with a clear reason and no booking is created.
