# Authentication Specification

## Purpose
Defines the single-user login, sessions, and how unauthenticated requests are handled.

## Requirements

### Requirement: Login Mode
The system SHALL require login for every request when a username and password are configured. When they are not configured, the system SHALL serve the gallery anonymously and SHALL NOT start a session.

#### Scenario: Login required
- **WHEN** a username and password are configured and the visitor has no session
- **THEN** the login form is shown instead of the gallery

#### Scenario: Anonymous access
- **WHEN** no username and password are configured
- **THEN** the gallery is served without a login

### Requirement: Credential Check
The system SHALL match the submitted username case-insensitively against the configured username and SHALL verify the submitted password against the stored password, rejecting the login if either does not match.

#### Scenario: Wrong password
- **WHEN** the submitted password does not match
- **THEN** the login is rejected and the form shows an "invalid login" alert

### Requirement: Login Form Token
The system SHALL issue a random per-session token, include it in the login form, and reject any login attempt whose token does not match the session.

#### Scenario: Missing or wrong token
- **WHEN** a login attempt is submitted without the session token
- **THEN** the attempt is rejected and the login form is shown with an "invalid login" alert

### Requirement: Login From The Application
The system SHALL accept a login submitted by the application as a JSON request with the `login` action and respond with a success flag.

#### Scenario: In-app login
- **WHEN** valid credentials and token are posted with the `login` action
- **THEN** the response is `{"success": true}` and the session is logged in

### Requirement: Logout
The system SHALL log the user out when a logged-in visitor requests the gallery with the `logout` parameter, clearing login state and showing the login form.

#### Scenario: Logout
- **WHEN** a logged-in visitor sends the `logout` parameter
- **THEN** the session is cleared and the login form shows a "you were logged out" alert

### Requirement: Unauthenticated Action Requests
The system SHALL reject action requests that require login without showing the form. POST requests SHALL receive a JSON error of `login`; GET requests SHALL receive an HTTP 401 page with a link to the login page. The diagnostics action is the exception and shows the form.

#### Scenario: Unauthenticated POST action
- **WHEN** a directory listing is requested by POST without a valid session and login is required
- **THEN** the response is `{"error": "login"}`

### Requirement: Session Ping
The system SHALL answer the `ping` action with the current username and session token so the application can detect that it has been logged out.

#### Scenario: Ping after logout
- **WHEN** the application pings after the session has expired
- **THEN** the response contains an empty username and no session token
