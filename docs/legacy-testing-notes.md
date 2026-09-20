# Legacy testing notes

These notes preserve useful behavior captured in removed deprecated shell
tests. The scripts themselves were removed because they depended on old
endpoints, manual curl flows, generated CSV outputs, and local database
state.

## Manual API flow

The historical shell tests exercised this sequence:

1. Reboot the database.
2. Sign up one instructor first.
3. Sign up several students.
4. Log in as the instructor.
5. Create questions, distractors, and quiz-question mappings.
6. Create a quiz from those questions.
7. Release the quiz to `STEP1`.
8. Log in as each student and submit initial answers and justifications.
9. Release the quiz to `STEP2`.
10. Submit revised answers and justification likes.
11. Download the gradebook CSV and compare it with a verified CSV.

The scripts assumed that the first user became an instructor. That
assumption is historical and should not be used as a design requirement
for new tests.

## Test-data conventions

The legacy tests used deterministic synthetic content so expected gradebooks
could be checked by hand.

- Instructor account: `instructor@usf.edu`.
- Student accounts: `student1@usf.edu`, `student2@usf.edu`, and so on.
- Simple question/distractor labels such as `Question 1 distractor 1`.
- Student justifications encoded the student, question, and distractor, such as
  `s1q1d1` or `s3q2sol`.

Some setup scripts intentionally avoided code snippets, quotes, and HTML-like
text. That made them useful for grading-flow checks but not for escaping or XSS
regression coverage.

## Grading and likes behavior

Several removed tests documented the intent of the participation and
justification-like scoring model.

For a student who sees 13 alternatives and has 2 peers in the scripted test,
the maximum number of possible outgoing likes was treated as 26. The quiz-level
limiting factor was then used to calculate the target number of likes. For
example, a 50% limiting factor produced a target of 7 likes after rounding.

The intended behavior was:

- Likes up to the target count at full value.
- Likes above the target progressively worth less.
- Like value reaches zero once the student likes every justification they saw.
- Received likes can score a student's justifications.
- In small tests, students may receive fewer shown justifications because there
  are not enough peers to fill all configured justification slots.

The deleted scripts also recorded expected gradebook checks for cases such as:

- one liked justification;
- maximum likes on selected justifications;
- question-level maximum-like scenarios;
- justifications for one question with no likes;
- all four grade quartiles represented.

These cases are useful candidates for future automated regression tests.

## Escaping regression scenario

One removed local test captured a historical escaping bug for programming
questions. The scenario used answers such as:

```text
ArrayList<Book> list = new ArrayList<>();
ArrayList<EBook> list = new ArrayList<>();
ArrayList<AudioBook> list = new ArrayList<>();
```

It also used Java code containing escaped quotes. The original bug was that
angle-bracketed generic types or quotes could disappear or break JSON parsing
when rendered into older student pages.

This scenario remains useful as a regression idea after the rich-HTML storage
and rendering policy changes. New tests should assert that programming content
is stored, sanitized, serialized, and rendered according to the current field
policy.

## Gradebook verification

The old shell tests downloaded `/quiz/1/grades?q=csv` while logged in as the
instructor, then compared the CSV against a checked-in verified gradebook.

Future tests should prefer Python test code or Flask CLI helpers over shell
curl flows, but the invariant remains valuable: deterministic student answers,
likes, and grading weights should produce a reproducible gradebook CSV.
