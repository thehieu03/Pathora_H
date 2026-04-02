# Tour Status Update API — Capability Spec

## Overview

A dedicated REST API endpoint for updating a tour's status field. Replaces the current broken approach that sends a partial payload to the full-tour update endpoint. Uses `ExecuteUpdateAsync` for single-SQL performance and follows the existing Clean Architecture + CQRS pattern.

## ADDED Requirements

### Requirement: Dedicated status update endpoint

The system SHALL provide a `PUT /api/tour/{id}/status` endpoint that accepts a JSON body with a `status` field and updates only the tour's status. All other tour fields remain unchanged.

#### Scenario: Successful status update
- **WHEN** an authenticated admin sends `PUT /api/tour/{id}/status` with `{ "status": 2 }` and a valid tour ID
- **THEN** the server updates the tour's status to `2`, sets `LastModifiedOnUtc` to the current UTC timestamp, sets `LastModifiedBy` to the current user's ID, and returns HTTP 200 with `{ "success": true, "message": "Status updated successfully" }`

#### Scenario: Tour not found
- **WHEN** an authenticated admin sends `PUT /api/tour/{invalid-guid}/status` with `{ "status": 2 }`
- **THEN** the server returns HTTP 404 with `{ "type": "...", "title": "Not Found", "status": 404, "detail": "Tour not found." }`

#### Scenario: Invalid status enum value
- **WHEN** an authenticated admin sends `PUT /api/tour/{id}/status` with `{ "status": 999 }`
- **THEN** the server returns HTTP 400 with a validation error indicating the status value is invalid

#### Scenario: Missing status field in request body
- **WHEN** an authenticated admin sends `PUT /api/tour/{id}/status` with an empty body or `{ }`
- **THEN** the server returns HTTP 400 with a validation error indicating `status` is required

#### Scenario: Unauthenticated request
- **WHEN** an unauthenticated user sends `PUT /api/tour/{id}/status` with `{ "status": 2 }`
- **THEN** the server returns HTTP 401 Unauthorized

#### Scenario: Unauthorized role (e.g., regular user)
- **WHEN** an authenticated user without SuperAdmin, Admin, TourManager, or TourOperator role sends `PUT /api/tour/{id}/status`
- **THEN** the server returns HTTP 403 Forbidden

#### Scenario: Malformed JSON body
- **WHEN** an authenticated admin sends `PUT /api/tour/{id}/status` with `{ "status": }` (invalid JSON)
- **THEN** the server returns HTTP 400 Bad Request

### Requirement: Audit trail on status update

The system SHALL update `LastModifiedBy` and `LastModifiedOnUtc` on the tour entity when a status update succeeds.

#### Scenario: Audit fields populated
- **WHEN** an authenticated admin updates a tour's status
- **THEN** `LastModifiedBy` is set to the current user's ID (from JWT `sub` claim) and `LastModifiedOnUtc` is set to the current UTC timestamp

### Requirement: No entity load on status update

The system SHALL update a tour's status using a single SQL UPDATE statement without loading the entity into memory.

#### Scenario: Single SQL execution
- **WHEN** a status update request is processed
- **THEN** the repository executes exactly one `UPDATE` statement against the `Tours` table, setting `Status`, `LastModifiedBy`, and `LastModifiedOnUtc` in a single operation
- **AND** the tour entity is NOT loaded into EF Core's change tracker before the update
