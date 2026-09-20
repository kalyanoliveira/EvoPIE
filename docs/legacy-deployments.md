# Legacy deployment archive notes

These notes preserve useful operational knowledge from the historical
`deployments/` directory. The active Docker deployment workflow is documented
in `README.md` and `docs/docker-tls.md`; the legacy deployment files predate
that workflow and should not be used as current deployment instructions.

## Historical context

The archived deployment folders correspond to course offerings and field tests,
including COP2513 Fall 2020, COP2512 Spring 2021, COP2513 Fall 2021,
COP2512 Spring 2022, and COP2513 Fall 2022.

Those folders mixed several concerns:

- one-off Dockerfiles;
- hand-built SQLite database snapshots;
- curl scripts for creating test users and quizzes;
- scripts for advancing a live quiz between steps;
- ad hoc SQLite queries for grading and validation;
- ad hoc SQLite migration scripts.

The SQLite snapshots were removed from Git because they are data artifacts and
may contain user/account rows. Keep production or field-test databases outside
source control.

## Deployment lessons superseded by the current workflow

Older deployment notes show the app being cloned on a target server and
started with `docker-compose up --build -d`. They also show experimental
Dockerfiles for Flask development mode, gunicorn, and gunicorn plus nginx.

The current Compose workflow supersedes those files by making the deployment
mode explicit:

- use the `local` profile for direct local HTTP startup;
- use the `production` profile for nginx/TLS deployment;
- use environment variables for data, certificate, server-name, and git-ref
  configuration;
- use `docs/docker-tls.md` for certificate guidance.

Do not copy hardcoded historical hostnames, absolute paths, or server-specific
volume mounts from the archived deployments.

## Live quiz operations

Several historical scripts only logged in as an instructor and changed a quiz
status from `STEP1` to `STEP2`.

The reusable operational idea is:

1. create or upload a prepared database before the live event;
2. log in as the instructor account;
3. release the quiz to the intended status through the application route or UI;
4. verify the status before students begin the next step.

The old scripts used direct curl calls such as:

```bash
curl -L -c ./mycookies \
  -d '{ "email": "instructor@usf.edu", "password": "..." }' \
  -H "Content-Type: application/json" \
  http://host:5000/login

curl -L -b ./mycookies \
  -d '{ "status": "STEP2" }' \
  -H "Content-Type: application/json" \
  http://host:5000/quizzes/1/status
```

Prefer the current UI or maintained API tests over these old scripts.

## Historical grading queries

The archived SQL files show how instructors inspected quiz performance directly
from SQLite before gradebook tooling matured.

Common checks included:

- joining `user`, `quiz_attempt`, and `quiz` to list initial and revised
  responses for a quiz;
- counting `-1` answer IDs in response blobs as a quick estimate of correct
  answers;
- listing submitted justifications for specific quiz-question IDs;
- counting likes received by each student's justifications for a quiz;
- counting likes given by each student for a quiz.

Example shape for likes received:

```sql
SELECT user.last_name, COUNT(*)
FROM likes4_justifications, justification, user,
     relation_questions_vs_quizzes, quiz
WHERE likes4_justifications.justification_id = justification.id
  AND justification.student_id = user.id
  AND justification.quiz_question_id =
      relation_questions_vs_quizzes.quiz_question_id
  AND quiz.id = relation_questions_vs_quizzes.quiz_id
  AND quiz.title = 'PLQ3'
GROUP BY user.last_name;
```

These queries are useful for debugging and data recovery, but current grading
should be verified through the application gradebook and maintained tests.

## Historical schema migrations

The 2022 deployment folders contained ad hoc SQLite migrations that added quiz
configuration and grading columns after initial field deployments.

The migrated `quiz` columns were:

- `limiting_factor`
- `initial_score_weight`
- `revised_score_weight`
- `justification_grade_weight`
- `participation_grade_weight`
- `participation_grade_threshold`
- `max_likes`
- `num_justifications_shown`
- `first_quartile_grade`
- `second_quartile_grade`
- `third_quartile_grade`
- `fourth_quartile_grade`

The migrated `distractor` column was:

- `justification`

Those columns are now represented in the SQLAlchemy models. The historical
SQL is useful as a record of how old SQLite databases were upgraded, but it
is not a replacement for a real migration framework.

## Archived course setup scripts

Several folders contained curl scripts that created course-specific quiz
content for live tests. The useful pattern was:

1. sign up the instructor first;
2. create one or more student accounts;
3. create questions with distractors;
4. compose quiz questions by choosing distractor IDs;
5. create a quiz with ordered question IDs;
6. release the quiz to the desired step.

The content itself was course-specific and should not be treated as maintained
seed data.

## Cleanup guidance

When removing old deployment files, preserve only knowledge that is not already
covered by current docs or maintained code. Files that only contain hardcoded
hosts, obsolete Docker commands, old course content, generated SQLite data,
or manual curl invocations can usually be removed once any unique operational
notes have been captured here.
