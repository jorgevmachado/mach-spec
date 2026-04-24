## MODIFIED Requirements

### Requirement: Authenticated Pokemon List Page
The web application SHALL provide an authenticated `/pokemon` page that consumes the protected Pokemon catalog listing API and supports guarded navigation to the details experience for complete entries.

#### Scenario: User clicks a complete Pokemon card
- **WHEN** an authenticated user clicks a Pokemon card whose status is `COMPLETE`
- **THEN** the application navigates to `/pokemon/[name]`

#### Scenario: User clicks an incomplete Pokemon card
- **WHEN** an authenticated user clicks a Pokemon card whose status is not `COMPLETE`
- **THEN** the application does not navigate to the details page
