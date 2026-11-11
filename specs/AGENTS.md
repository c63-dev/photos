# Spec Instructions

This directory holds the behavioural specification for the C63 Photos server.

- `project.md` describes the project, conventions and what is in and out of scope.
- `<capability>.md` is the current truth for each capability.

Workflow:
1. Before changing server behaviour, update the affected capability spec first, describing the new behaviour as requirements and scenarios.
2. Implement against the updated spec.

Rules:
- Never install new software, packages or tools, globally or via one-off runners such as npx, without explicit permission from the user.
- Specs describe behaviour only, never implementation.
- Requirement headings use `### Requirement: <Name>` with SHALL/MUST wording, and every requirement has at least one `#### Scenario:` block.
