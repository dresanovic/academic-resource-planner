# Tasks: Meeting Room Booking

**Input**: Design documents from `specs/001-meeting-room-booking/`

**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/booking-api.yaml`, `quickstart.md`

**Tests**: Test tasks are included because the plan, quickstart, and success criteria require contract, integration, and unit validation of booking behavior.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel because it touches different files or only depends on completed setup/foundation
- **[Story]**: Maps task to the user story from `spec.md`
- Every task includes an exact file path

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Initialize the Python backend project and test structure.

- [ ] T001 Create project directories `src/resource_planner/api`, `src/resource_planner/models`, `src/resource_planner/services`, `src/resource_planner/storage`, `tests/contract`, `tests/integration`, and `tests/unit`
- [ ] T002 Create Python package markers in `src/resource_planner/__init__.py`, `src/resource_planner/api/__init__.py`, `src/resource_planner/models/__init__.py`, `src/resource_planner/services/__init__.py`, and `src/resource_planner/storage/__init__.py`
- [ ] T003 Configure project metadata and dependencies for FastAPI, Pydantic, SQLAlchemy, pytest, and test client support in `pyproject.toml`
- [ ] T004 [P] Configure pytest defaults and import path in `pytest.ini`
- [ ] T005 [P] Add environment example for local SQLite configuration in `.env.example`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Establish shared app, persistence, schemas, and error handling required by all user stories.

**CRITICAL**: No user story work can begin until this phase is complete.

- [ ] T006 Define shared SQLAlchemy engine/session setup and database initialization in `src/resource_planner/storage/repository.py`
- [ ] T007 [P] Create Room persistence model with `id`, `name`, and `is_available_for_booking` fields in `src/resource_planner/models/room.py`
- [ ] T008 [P] Create User persistence model with `id` and `display_name` fields in `src/resource_planner/models/user.py`
- [ ] T009 [P] Create Booking persistence model with `id`, `room_id`, `user_id`, `start_time`, `end_time`, and `status` fields in `src/resource_planner/models/booking.py`
- [ ] T010 Define request and response schemas matching `contracts/booking-api.yaml` in `src/resource_planner/api/bookings.py`
- [ ] T011 Create FastAPI application factory and include booking routes in `src/resource_planner/app.py`
- [ ] T012 Implement shared rejection response helper for validation and conflict failures in `src/resource_planner/api/bookings.py`
- [ ] T013 [P] Create reusable test database and seeded Room/User fixtures in `tests/conftest.py`

**Checkpoint**: Foundation ready; user story implementation can now begin.

---

## Phase 3: User Story 1 - Book an Available Room (Priority: P1) MVP

**Goal**: A user can create a booking for a known available room when the requested future 15-minute time range is valid and non-conflicting.

**Independent Test**: Submit a valid future booking request for an empty room schedule and verify a `201` accepted response with the selected room and time range.

### Tests for User Story 1

- [ ] T014 [P] [US1] Add contract test for successful `POST /bookings` response shape in `tests/contract/test_booking_api.py`
- [ ] T015 [P] [US1] Add unit tests for strict future-at-submission validation, end-after-start validation, and 15-minute increment validation in `tests/unit/test_booking_rules.py`
- [ ] T016 [P] [US1] Add contract test for invalid time validation rejection response shape and clear validation messages in `tests/contract/test_booking_api.py`
- [ ] T017 [P] [US1] Add integration tests proving past-start, current-time start, and end-before-or-equal-start requests are rejected without creating bookings in `tests/integration/test_booking_conflicts.py`
- [ ] T018 [P] [US1] Add integration test for creating a valid booking in an empty room schedule in `tests/integration/test_booking_conflicts.py`
- [ ] T019 [P] [US1] Add integration tests for `unknown_room` and `room_unavailable` booking rejections in `tests/integration/test_booking_conflicts.py`

### Implementation for User Story 1

- [ ] T020 [US1] Implement booking validation rules for end-after-start, strict future-at-submission start time, and 15-minute increments in `src/resource_planner/services/booking_service.py`
- [ ] T021 [US1] Implement room existence and availability rejection handling in `src/resource_planner/services/booking_service.py`
- [ ] T022 [US1] Implement repository operations to fetch rooms, fetch users, and create accepted bookings in `src/resource_planner/storage/repository.py`
- [ ] T023 [US1] Implement booking creation service path for valid non-conflicting requests in `src/resource_planner/services/booking_service.py`
- [ ] T024 [US1] Implement `POST /bookings` accepted response handling in `src/resource_planner/api/bookings.py`
- [ ] T025 [US1] Implement `GET /rooms` room listing endpoint from the contract in `src/resource_planner/api/bookings.py`

**Checkpoint**: User Story 1 is fully functional and testable independently.

---

## Phase 4: User Story 2 - Prevent Same-Room Overlaps (Priority: P1)

**Goal**: A user cannot create a booking when any part of the requested time range overlaps an existing booking for the same room.

**Independent Test**: Seed Room A with a 09:00-10:00 booking, submit overlapping requests for Room A, and verify each request returns a rejected outcome with `same_room_overlap`.

### Tests for User Story 2

- [ ] T026 [P] [US2] Add contract test for `409` overlap rejection response shape in `tests/contract/test_booking_api.py`
- [ ] T027 [P] [US2] Add unit tests for half-open overlap detection cases in `tests/unit/test_booking_rules.py`
- [ ] T028 [P] [US2] Add integration tests for same-room partial, enclosing, exact-match, and adjacent booking cases in `tests/integration/test_booking_conflicts.py`
- [ ] T029 [P] [US2] Add integration test proving only one overlapping concurrent request for the same room succeeds in `tests/integration/test_booking_conflicts.py`

### Implementation for User Story 2

- [ ] T030 [US2] Implement half-open time range overlap detection in `src/resource_planner/services/booking_service.py`
- [ ] T031 [US2] Implement repository query for existing same-room overlapping bookings in `src/resource_planner/storage/repository.py`
- [ ] T032 [US2] Integrate transactional conflict check before booking creation in `src/resource_planner/services/booking_service.py`
- [ ] T033 [US2] Return `same_room_overlap` rejection responses with requested room and time range from `src/resource_planner/api/bookings.py`

**Checkpoint**: User Stories 1 and 2 both work independently.

---

## Phase 5: User Story 3 - Book Different Rooms Concurrently (Priority: P2)

**Goal**: A user can book one room at the same time another room is booked, as long as the requested room is available.

**Independent Test**: Seed Room A with a 09:00-10:00 booking, submit a 09:00-10:00 booking for Room B, and verify the Room B booking is accepted.

### Tests for User Story 3

- [ ] T034 [P] [US3] Add integration test for concurrent same-time bookings in different rooms in `tests/integration/test_booking_conflicts.py`
- [ ] T035 [P] [US3] Add unit test proving overlap checks are scoped by `room_id` in `tests/unit/test_booking_rules.py`

### Implementation for User Story 3

- [ ] T036 [US3] Ensure conflict queries filter by requested `room_id` only in `src/resource_planner/storage/repository.py`
- [ ] T037 [US3] Ensure booking service allows same-time bookings for different rooms in `src/resource_planner/services/booking_service.py`
- [ ] T038 [US3] Verify accepted response for different-room concurrent booking remains contract-compliant in `src/resource_planner/api/bookings.py`

**Checkpoint**: All user stories are independently functional.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final validation, documentation, and cleanup across all user stories.

- [ ] T039 [P] Update `README.md` with local setup, run, and test commands for the booking service
- [ ] T040 [P] Align `specs/001-meeting-room-booking/quickstart.md` with final commands and expected responses
- [ ] T041 Add integration validation for 50 rooms and 500 future bookings with under-1-second booking response expectation in `tests/integration/test_booking_conflicts.py`
- [ ] T042 Run the full pytest suite and record any required fixes in `tests/`
- [ ] T043 Validate `contracts/booking-api.yaml` against implemented request and response behavior in `tests/contract/test_booking_api.py`
- [ ] T044 Review implementation against out-of-scope items in `specs/001-meeting-room-booking/spec.md` to ensure editing, cancellation, recurring bookings, approvals, capacity, equipment filtering, and notifications were not added

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 Setup**: No dependencies; can start immediately.
- **Phase 2 Foundational**: Depends on Phase 1; blocks all user stories.
- **Phase 3 User Story 1**: Depends on Phase 2 and delivers the MVP.
- **Phase 4 User Story 2**: Depends on Phase 2 and the booking creation path from Phase 3.
- **Phase 5 User Story 3**: Depends on Phase 2 and can proceed after the booking creation path from Phase 3.
- **Phase 6 Polish**: Depends on all desired user stories being complete.

### User Story Dependencies

- **User Story 1 (P1)**: Starts after Foundational; no dependency on other stories.
- **User Story 2 (P1)**: Starts after Foundational and uses the booking creation path from US1.
- **User Story 3 (P2)**: Starts after Foundational and uses the booking creation path from US1; can run in parallel with US2 after US1 service creation exists.

### Within Each User Story

- Tests are written before implementation tasks for that story.
- Validation and repository behavior precede service integration.
- Service behavior precedes endpoint response handling.
- Each story reaches its checkpoint before moving to the next priority unless parallel staffing is available.

### Parallel Opportunities

- T004 and T005 can run in parallel after T003.
- T007, T008, T009, and T013 can run in parallel after T006.
- Tests within each user story marked `[P]` can be written in parallel.
- US2 and US3 can proceed in parallel after US1 establishes the booking creation path.
- T039 and T040 can run in parallel during polish.

---

## Parallel Example: User Story 1

```bash
Task: "T014 [P] [US1] Add contract test for successful POST /bookings response shape in tests/contract/test_booking_api.py"
Task: "T015 [P] [US1] Add unit tests for strict future-at-submission validation, end-after-start validation, and 15-minute increment validation in tests/unit/test_booking_rules.py"
Task: "T016 [P] [US1] Add contract test for invalid time validation rejection response shape and clear validation messages in tests/contract/test_booking_api.py"
Task: "T017 [P] [US1] Add integration tests proving past-start, current-time start, and end-before-or-equal-start requests are rejected without creating bookings in tests/integration/test_booking_conflicts.py"
Task: "T018 [P] [US1] Add integration test for creating a valid booking in an empty room schedule in tests/integration/test_booking_conflicts.py"
Task: "T019 [P] [US1] Add integration tests for unknown_room and room_unavailable booking rejections in tests/integration/test_booking_conflicts.py"
```

## Parallel Example: User Story 2

```bash
Task: "T026 [P] [US2] Add contract test for 409 overlap rejection response shape in tests/contract/test_booking_api.py"
Task: "T027 [P] [US2] Add unit tests for half-open overlap detection cases in tests/unit/test_booking_rules.py"
Task: "T028 [P] [US2] Add integration tests for same-room partial, enclosing, exact-match, and adjacent booking cases in tests/integration/test_booking_conflicts.py"
Task: "T029 [P] [US2] Add integration test proving only one overlapping concurrent request for the same room succeeds in tests/integration/test_booking_conflicts.py"
```

## Parallel Example: User Story 3

```bash
Task: "T034 [P] [US3] Add integration test for concurrent same-time bookings in different rooms in tests/integration/test_booking_conflicts.py"
Task: "T035 [P] [US3] Add unit test proving overlap checks are scoped by room_id in tests/unit/test_booking_rules.py"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1 setup.
2. Complete Phase 2 foundational infrastructure.
3. Complete Phase 3 User Story 1.
4. Stop and validate the available-room booking flow independently.

### Incremental Delivery

1. Deliver US1 so users can create valid bookings.
2. Add US2 to enforce same-room conflict prevention.
3. Add US3 to prove different-room concurrent bookings remain allowed.
4. Finish polish tasks and run quickstart validation.

### Parallel Team Strategy

1. Complete setup and foundational tasks as a shared baseline.
2. Write US1 tests while implementing shared models and fixtures.
3. After US1 booking creation exists, split US2 conflict prevention and US3 room-scoped concurrency validation.

## Notes

- `[P]` tasks are parallelizable because they touch different files or are independent test additions.
- `[US1]`, `[US2]`, and `[US3]` labels map directly to user stories in `spec.md`.
- Keep implementation within the feature scope: no booking editing, cancellation, recurrence, approvals, room capacity, equipment filtering, or notifications.
