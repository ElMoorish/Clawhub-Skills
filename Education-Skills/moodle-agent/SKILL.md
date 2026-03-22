---
name: moodle-agent
version: 1.0.0
description: >
  Full LMS integration with Moodle via its REST Web Services API. Use this
  skill whenever the user wants to interact with Moodle: browse courses,
  check grades, submit assignments, post forum messages, manage users,
  create activities, or pull reports. Trigger on phrases like "Moodle",
  "check my grades", "post to the forum", "create a quiz in Moodle",
  "enroll a student", "LMS", "my course assignments", or "learning management
  system". Works with any self-hosted Moodle 3.x or 4.x instance.
metadata:
  clawdbot:
    requires:
      bins: ["curl", "python3"]
      env:
        - name: MOODLE_URL
          description: "Base URL of your Moodle instance, e.g. https://moodle.yourschool.edu"
        - name: MOODLE_TOKEN
          description: "Moodle Web Services token (see setup below)"
---

# Moodle Agent Skill

Interact with any Moodle 3.x/4.x LMS instance via the REST Web Services API.

---

## Authentication setup

1. In Moodle admin: **Site administration → Plugins → Web services → Enable web services** ✓
2. Enable **REST protocol** under Manage protocols.
3. Create a service and assign required functions (see Functions below).
4. Generate a token: **Site administration → Plugins → Web services → Manage tokens → Add**
5. Set environment variables:

```bash
export MOODLE_URL="https://moodle.yourschool.edu"
export MOODLE_TOKEN="your_token_here"
```

---

## Base request pattern

All calls use this URL shape:

```
GET/POST $MOODLE_URL/webservice/rest/server.php
  ?wstoken=$MOODLE_TOKEN
  &wsfunction=FUNCTION_NAME
  &moodlewsrestformat=json
  &[params...]
```

Helper function:

```bash
moodle_call() {
  local fn=$1; shift
  curl -s "$MOODLE_URL/webservice/rest/server.php" \
    --data-urlencode "wstoken=$MOODLE_TOKEN" \
    --data-urlencode "wsfunction=$fn" \
    --data-urlencode "moodlewsrestformat=json" \
    "$@"
}
```

---

## Common functions

### Course management

```bash
# List courses the user is enrolled in
moodle_call core_enrol_get_users_courses \
  --data-urlencode "userid=USER_ID"

# Get course details
moodle_call core_course_get_courses_by_field \
  --data-urlencode "field=id" \
  --data-urlencode "value=COURSE_ID"

# Get course content (sections + activities)
moodle_call core_course_get_contents \
  --data-urlencode "courseid=COURSE_ID"
```

### Grades

```bash
# Get grades for a user in a course
moodle_call gradereport_user_get_grades_table \
  --data-urlencode "courseid=COURSE_ID" \
  --data-urlencode "userid=USER_ID"

# Get overview of all grades across courses
moodle_call gradereport_overview_get_course_grades \
  --data-urlencode "userid=USER_ID"
```

### Assignments

```bash
# List assignments in a course
moodle_call mod_assign_get_assignments \
  --data-urlencode "courseids[0]=COURSE_ID"

# Get submission status for a user
moodle_call mod_assign_get_submission_status \
  --data-urlencode "assignid=ASSIGN_ID" \
  --data-urlencode "userid=USER_ID"

# Submit an assignment (text online)
moodle_call mod_assign_save_submission \
  --data-urlencode "assignmentid=ASSIGN_ID" \
  --data-urlencode "plugindata[onlinetext_editor][text]=YOUR_SUBMISSION_TEXT" \
  --data-urlencode "plugindata[onlinetext_editor][format]=1"
```

### Forum

```bash
# Get forum discussions in a course
moodle_call mod_forum_get_forums_by_courses \
  --data-urlencode "courseids[0]=COURSE_ID"

# Get posts in a discussion
moodle_call mod_forum_get_discussion_posts \
  --data-urlencode "discussionid=DISCUSSION_ID"

# Post a reply
moodle_call mod_forum_add_discussion_post \
  --data-urlencode "postid=PARENT_POST_ID" \
  --data-urlencode "subject=Re: Your subject" \
  --data-urlencode "message=Your reply text here" \
  --data-urlencode "messageformat=1"
```

### Users (admin)

```bash
# Get user by username
moodle_call core_user_get_users \
  --data-urlencode "criteria[0][key]=username" \
  --data-urlencode "criteria[0][value]=USERNAME"

# Enroll a user in a course
moodle_call enrol_manual_enrol_users \
  --data-urlencode "enrolments[0][roleid]=5" \
  --data-urlencode "enrolments[0][userid]=USER_ID" \
  --data-urlencode "enrolments[0][courseid]=COURSE_ID"
# roleid: 5=student, 3=teacher, 4=non-editing teacher
```

### Quizzes

```bash
# Get quizzes in a course
moodle_call mod_quiz_get_quizzes_by_courses \
  --data-urlencode "courseids[0]=COURSE_ID"

# Get a user's quiz attempts
moodle_call mod_quiz_get_user_attempts \
  --data-urlencode "quizid=QUIZ_ID" \
  --data-urlencode "userid=USER_ID"
```

### Notifications / Messages

```bash
# Send a message to a user
moodle_call core_message_send_messages \
  --data-urlencode "messages[0][touserid]=TARGET_USER_ID" \
  --data-urlencode "messages[0][text]=Hello from the Moodle agent!" \
  --data-urlencode "messages[0][textformat]=0"
```

---

## Resolving user IDs

Most calls need numeric IDs. If you only have a username or email:

```bash
moodle_call core_user_get_users \
  --data-urlencode "criteria[0][key]=email" \
  --data-urlencode "criteria[0][value]=student@example.com" \
| python3 -c "import json,sys; u=json.load(sys.stdin)['users']; print(u[0]['id'] if u else 'Not found')"
```

---

## Displaying results

**Grade report:**
```
Course: Introduction to Python (CS101)
──────────────────────────────────────
Assignment 1: Functions     92/100  A
Assignment 2: Loops         88/100  B+
Midterm Quiz                76/100  C+
Final Project               —  (due 2026-04-15)
──────────────────────────────────────
Current grade:  85.3%  B
```

**Upcoming deadlines (pull from all enrolled courses):**
```
Due soon:
  Today     Assignment 3 — Data Structures (CS201)
  Mar 26    Forum post — Ethics in AI (PHIL305)
  Mar 28    Quiz 4 — Organic Chemistry (CHEM201)
```

---

## Error handling

| Error | Meaning | Fix |
|---|---|---|
| `invalidtoken` | Token expired or wrong | Regenerate token in Moodle admin |
| `accessexception` | Function not in service | Add function to the web service |
| `invalidparameter` | Wrong param name/type | Check Moodle API docs for function |
| 403 / HTML response | Web services not enabled | Enable in Site administration |
