# Architecture

This document explains how EvoPIE works at a system level.

## EvoPIE in one sentence

EvoPIE is a web application for asynchronous peer instruction. Instructors
build multiple-choice quiz activities, students answer and revise those
activities in stages, and the system records the resulting answers,
justifications, likes, and grades for instruction and analysis.

## The learning flow

EvoPIE is organized around courses, questions, quizzes, and students.

An instructor creates questions. Each question has a correct answer and a set
of distractors, which are wrong but plausible answer choices. The instructor
then uses those questions to build quizzes and attaches quizzes to courses.

Students take a quiz through a staged workflow.

In Step 1, each student answers the quiz individually. For each answer choice
they do not select, students provide a justification explaining why that choice
seems incorrect.

In Step 2, students revisit the quiz after other students have completed Step
\ 1. They can see selected peer justifications for the alternatives and can
revise their answers. Students may also like useful peer justifications.

Step 3 is optional. When enabled, students can propose new distractors and
justify why those distractors are plausible wrong answers. Instructors can
later review and grade that work.

The Solutions stage lets students see the correct answers and instructor
justifications after the activity is complete.

## Roles

EvoPIE currently uses three main roles:

- students, who take quizzes and participate in peer instruction;
- instructors, who create and manage courses, questions, quizzes, and grades;
- admins, who can access administrative views.

On an empty database, the first account created through the sign-up page
becomes an instructor account. Later accounts become student accounts.

## Main application parts

The `evopie` package is the Flask web application. It defines the application,
routes, templates, quiz pages, instructor pages, authentication, and dashboard
registration.

The `datalayer` package contains database setup and model definitions. EvoPIE
stores users, courses, questions, distractors, quizzes, attempts,
justifications, likes, grading data, quiz model state, and dashboard data in
the database.

The `analysislayer` package supports dashboard and analysis features. It builds
derived views of quiz and student data that instructors can inspect through the
data dashboard.

The `updater.py` process refreshes analysis and dashboard data. In local
development it can be run once or in a loop. In Docker deployment it runs as a
separate service.

The `nginx` directory contains the production reverse-proxy configuration used
by the Docker Compose deployment.

## Database-centered state

EvoPIE persists quiz state in the database. This is important because web
requests are stateless: each page load or submission has to restore the current
state from stored records.

The database stores the structure of a quiz, the students attached to a course,
the answers students submitted, the justifications they wrote, the
justifications shown in Step 2, likes on justifications, and grading-related
values. The schema diagram is included in the
[reference](reference.md#database-schema).

Some operations are intentionally cached in the database. For example, once a
student receives a sampled quiz attempt or a sampled set of peer
justifications, that state is stored so refreshing the page does not create a
different activity for the same student.

## Quiz models

A quiz model is the part of EvoPIE that decides which distractors or quiz
variants should be used for students. The default behavior relies on instructor
selection, but EvoPIE also contains model implementations that can sample or
adapt quiz content based on collected interactions.

At a high level, quiz models exist so EvoPIE can use student responses and
justifications as data. That data can help expose misconceptions and guide how
future quiz content is selected or analyzed.

## Justifications and peer instruction

Justifications are central to EvoPIE. In Step 1, students explain why
unselected alternatives are incorrect. In Step 2, other students can see
selected peer justifications while revising their answers.

EvoPIE stores these justifications and tracks which justifications were shown
to which student. Students can like useful justifications. Those likes
contribute to participation and justification-related grading.

## Grading and analytics

EvoPIE grades more than whether a final answer is correct. Depending on quiz
configuration, grades may include Step 1 performance, Step 2 performance,
justification quality, participation through likes, and optional Step 3
distractor-design work.

The data dashboard and grading pages help instructors inspect student
performance, peer-instruction activity, and derived analysis views.

## Deployment shape

For local development, EvoPIE can run directly with Pipenv, Flask, SQLite, and
the updater process.

For the current production deployment, Docker Compose runs separate services
for the web application, the updater, and nginx. The current Docker
configuration is specific to `evopie.cse.usf.edu` and expects its data
directory and TLS certificate paths to match that deployment.
