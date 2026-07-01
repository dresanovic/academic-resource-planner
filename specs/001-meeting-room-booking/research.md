# Research: Meeting Room Booking

## Decision: Implement as a single backend HTTP service

**Rationale**: The feature requires a clear booking creation contract, validation of time ranges, and conflict prevention. A backend service provides a focused boundary for contract tests and future clients without requiring frontend scope that the specification does not request.

**Alternatives considered**:

- Full web application: rejected because the specification does not require UI screens beyond user-facing booking outcomes.
- Internal library only: rejected because the system must expose booking behavior to users or client applications.

## Decision: Use Python 3.11 with FastAPI, Pydantic, SQLAlchemy, and pytest

**Rationale**: This stack supports concise validation, clear HTTP contracts, and straightforward automated tests for domain rules. It is suitable for a small new repository without existing source code or framework commitments.

**Alternatives considered**:

- TypeScript/Node service: viable, but no existing project files indicate a Node stack.
- Framework-free Python service: rejected because request validation and contract clarity would require more custom code.

## Decision: Use SQLite for local validation with a transactional repository boundary

**Rationale**: SQLite keeps local quickstart and test setup simple while still supporting persistent room and booking records. A repository boundary keeps storage concerns isolated if a later environment moves to a server database.

**Alternatives considered**:

- In-memory-only storage: rejected because bookings need durable identity and conflict validation across requests.
- Server database as a mandatory dependency: rejected because the initial repository has no deployment or infrastructure context.

## Decision: Enforce half-open time ranges and 15-minute boundaries in the booking service

**Rationale**: The spec clarifies that start time is included and end time is excluded. This makes adjacent bookings non-overlapping and provides deterministic conflict checks. The 15-minute boundary rule belongs in the domain service so every interface applies it consistently.

**Alternatives considered**:

- Closed intervals: rejected because adjacent bookings would incorrectly conflict.
- Interface-only validation: rejected because bypassing an interface could create invalid bookings.

## Decision: Handle concurrent same-room booking attempts through a transactional create flow

**Rationale**: The spec requires only one successful outcome when multiple users attempt overlapping bookings for the same room. The create flow must check conflicts and commit the accepted booking as a single operation from the caller perspective.

**Alternatives considered**:

- Best-effort conflict check without transaction semantics: rejected because concurrent requests could double-book a room.
- Manual administrative conflict resolution: rejected because the system must prevent overlaps at booking time.
