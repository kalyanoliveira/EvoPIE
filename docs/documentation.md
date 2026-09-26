# Documentation guide

This document explains where EvoPIE documentation belongs.

## Documentation map

- `README.md`: project identity, short overview, acknowledgement, and links to
  the main documentation.
- `docs/how-to-run.md`: local setup and deployment steps.
- `docs/architecture.md`: system design, application workflow, data flow,
  roles, quiz lifecycle, background processing, and deployment shape.
- `docs/user-guide.md`: task-oriented instructions for instructors, students,
  and admins using EvoPIE through the web interface.
- `docs/reference.md`: exact facts, such as role names, quiz statuses,
  commands, environment variables, paths, configuration values, and formulas.
- `docs/troubleshooting.md`: FAQ-style fixes for common problems.
- `docs/decisions/`: short records explaining important project and
  documentation decisions.

## Updating documentation

When a change alters how EvoPIE works, how it is run, how it is used, or how it
is troubleshot, update the relevant documentation in the same change.

Prefer updating an existing document over creating a new one. Create a new
document only when the information has a clearly different purpose from the
existing documents.

