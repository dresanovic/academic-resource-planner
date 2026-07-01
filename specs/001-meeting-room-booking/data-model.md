# Data Model: Meeting Room Booking

## Room

Represents a meeting space that can be booked.

### Fields

- `id`: Stable unique room identifier.
- `name`: Human-readable room name shown in booking results and conflicts.
- `is_available_for_booking`: Indicates whether users can create new bookings for this room.

### Validation Rules

- `id` must be unique.
- `name` must be present.
- Booking requests for unknown rooms are rejected.
- Booking requests for rooms not available for booking are rejected.

## User

Represents a known person creating a booking.

### Fields

- `id`: Stable unique user identifier.
- `display_name`: Human-readable user name.

### Validation Rules

- `id` must identify a known user before booking creation.
- The feature assumes user identity is already established before booking creation.

## Booking

Represents a reservation for exactly one room during a defined future time range.

### Fields

- `id`: Stable unique booking identifier.
- `room_id`: The room reserved by the booking.
- `user_id`: The user who created the booking.
- `start_time`: Inclusive start of the reserved time range.
- `end_time`: Exclusive end of the reserved time range.
- `status`: Current booking state.

### Relationships

- A booking belongs to one room.
- A booking belongs to one user.
- A room can have many bookings over time.
- A user can create many bookings.

### Validation Rules

- `end_time` must be later than `start_time`.
- `start_time` must be in the future at request time.
- `start_time` and `end_time` must align to 15-minute increments.
- A booking may be created at any future time; business-hours and room operating-hours restrictions are out of scope.
- A booking is rejected if its requested time range overlaps an existing booking for the same room.
- A booking is allowed if its start time equals an existing same-room booking's end time.
- A booking is allowed if its end time equals an existing same-room booking's start time.
- A booking for one room does not conflict with bookings for other rooms.

## Booking State Transitions

```text
Requested -> Accepted
Requested -> Rejected
```

- `Accepted`: The booking passed validation and was reserved.
- `Rejected`: The request failed validation because of invalid time range, past start time, increment misalignment, unknown or unavailable room, or same-room overlap.

Cancellation, editing, recurring bookings, approvals, and expiration are outside this feature.
