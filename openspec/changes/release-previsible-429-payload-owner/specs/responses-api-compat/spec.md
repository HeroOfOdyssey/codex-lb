## ADDED Requirements

### Requirement: Pre-visible rate-limit or quota rejection does not establish a transient dispatch owner

Notwithstanding the account-bound retry requirement, a pre-visible rejection
classified as rate-limit or quota MUST NOT establish a new dispatch-owner
binding while owner registration is pending. This exception MUST NOT clear or
move an independently established file, previous-response, turn-state, or
existing dispatch owner.

#### Scenario: Classified HTTP 429 does not establish a transient owner

- **GIVEN** a streaming Responses body is not a canonical account-neutral fresh
  replay
- **AND** the body has no independently established required account owner
- **AND** account B is eligible and the request retry budget remains
- **WHEN** account A rejects the request with an HTTP 429 classified as
  rate-limit or quota before any response event
- **THEN** the proxy does not record account A as the dispatch owner
- **AND** the proxy MUST attempt dispatch on account B before returning account
  A's 429

#### Scenario: First-event limit rejection does not establish a transient owner

- **GIVEN** a streaming Responses body is not a canonical account-neutral fresh
  replay
- **AND** the body has no independently established required account owner
- **AND** account B is eligible and the request retry budget remains
- **WHEN** account A's first upstream event is `response.failed` with a code
  classified as rate-limit or quota
- **THEN** the proxy does not expose that event or record account A as the
  dispatch owner
- **AND** the proxy MUST attempt dispatch on account B

#### Scenario: Existing required owner remains fail-closed after a limit rejection

- **GIVEN** a streaming Responses body has a file, previous-response,
  turn-state, or existing dispatch owner on account A
- **WHEN** account A returns a pre-visible rejection classified as rate-limit or
  quota
- **THEN** the proxy does not clear or move that owner
- **AND** the retained body is not dispatched on account B
