---
name: technical-course-publisher
description: Create and publish a self-paced Markdown course from a prior technical question or answer. Use when the user asks to teach, prepare a course, create a study guide, learning path, educational guide, or publish learning material about software engineering, systems, programming languages, open source, or architecture.
---

# Technical Course Publisher

Create a structured course in the configured `tech-courses` Git repository.

## Source context

Use the current conversation as the primary source when it contains the original
technical question and answer.

If this is a new Codex session and the prior discussion is unavailable, ask the
user to provide one of:
- the original question and answer;
- a short topic brief;
- a source file or link.

Do not pretend to remember a previous CLI session.

## Workflow

1. Identify the learner's assumed background and the topic boundaries.
2. Propose a course outline unless the user explicitly requests immediate creation
   and publishing.
3. Research authoritative primary sources where freshness or accuracy matters.
4. Create `courses/<topic-slug>/` following the repository's `AGENTS.md`.
5. Include:
   - learning objectives;
   - prerequisites;
   - concepts in dependency order;
   - examples and realistic use cases;
   - small exercises;
   - answers or solution guidance;
   - a glossary;
   - links to primary references.
6. Run `git diff --check`.
7. Commit and push only when the user explicitly asks to publish, push, or commit.
8. Return the created path, a concise outline, commit hash, and repository URL.
