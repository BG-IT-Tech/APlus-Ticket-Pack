# Suggested GitHub Repository Structure

```text
github-aplus-ticket-pack/
|-- .github/
|   `-- ISSUE_TEMPLATE/
|       |-- beginner-support-ticket.md
|       `-- student-submission.md
|-- tickets/
|   |-- APLUSA-001-account-lockout.md
|   |-- APLUSA-002-print-queue.md
|   |-- ...
|   `-- APLUSA-020-browser-hijack.md
|-- evidence/
|   |-- APLUSA-001/
|   |   |-- failed-logins.txt
|   |   `-- user-note.txt
|   |-- APLUSA-002/
|   |   |-- print-queue.txt
|   |   `-- printer-status.txt
|   `-- ...
|-- templates/
|   |-- student-response-template.md
|   `-- instructor-answer-template.md
|-- instructor/
|   |-- answer-key.md
|   `-- marking-guide.md
|-- student-pack/
|   |-- student-scenarios.md
|   `-- quick-start.md
`-- README.md
```

## Why this layout works
- Clean separation between learner content and instructor answers
- Easy to attach evidence to Issues
- Works well for forks, branches, and pull requests
- Scales from simple classroom use to small lab competitions

## Suggested workflow in GitHub
### Option 1 - GitHub Issues only
- Instructor opens each scenario as an Issue
- Students reply in comments using the response template
- Best for very early learners

### Option 2 - Branch and pull request
- Each student creates a branch
- Student completes the response template
- Student opens a pull request with their answer
- Best for adding basic GitHub practice

### Option 3 - Team troubleshooting
- One Issue per group
- Group discusses root cause and writes one final response
- Best for classroom collaboration
