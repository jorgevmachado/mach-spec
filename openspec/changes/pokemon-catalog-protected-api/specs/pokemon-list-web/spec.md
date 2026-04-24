## ADDED Requirements

### Requirement: Authenticated Pokemon List Page
The web application SHALL provide an authenticated `/pokemon` page that consumes the protected Pokemon catalog listing API.

#### Scenario: Authenticated user opens Pokemon page
- **WHEN** an authenticated user navigates to `/pokemon`
- **THEN** the page requests the protected Pokemon listing API and renders the returned Pokemon entries

### Requirement: Frontend Pagination Uses Existing Pattern
The web application SHALL use the project’s existing paginated list behavior for the Pokemon page rather than introducing a new pagination contract.

#### Scenario: User navigates between pages
- **WHEN** a user changes the current page in the Pokemon listing UI
- **THEN** the page requests the next paginated dataset using the existing list hook pattern

### Requirement: Pokemon List Filters
The web application SHALL allow users to filter the Pokemon list by `name` and `order`.

#### Scenario: User filters by name
- **WHEN** a user applies a `name` filter in the Pokemon list UI
- **THEN** the page requests filtered Pokemon results and renders the filtered dataset

#### Scenario: User filters by order
- **WHEN** a user applies an `order` filter in the Pokemon list UI
- **THEN** the page requests filtered Pokemon results and renders the filtered dataset

### Requirement: Related Resources Stay Out of Frontend Scope
The web application SHALL not consume moves, abilities, types, or growth-rate endpoints in this change.

#### Scenario: User browses Pokemon list
- **WHEN** the Pokemon list page is rendered in this delivery
- **THEN** the frontend only depends on the protected Pokemon listing capability and not on related-resource UI integration
