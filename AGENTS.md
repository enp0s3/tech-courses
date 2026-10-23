# Technical Learning Courses

This repository contains self-paced Markdown courses generated from technical
questions and discussions.

## Course requirements

For each topic, create `courses/<topic-slug>/`.

Every course must include:

- `README.md`: audience, prerequisites, learning outcomes, estimated duration,
  recommended learning path, and a concise explanation of why the topic matters.
- Ordered lessons beginning at `00-`.
- Concrete examples, small use cases, commands or code where relevant.
- A glossary defining unfamiliar terms.
- A “check your understanding” section with questions and answers.
- `references.md` with authoritative sources and links.
- Clear separation between verified facts, assumptions, and simplifications.

Write for an engineer who knows Linux, containers, Kubernetes, and systems
programming, but may be new to the specific subject.

## Quality bar

- Explain concepts before introducing implementation details.
- Use diagrams only when they clarify a relationship or data flow.
- Prefer small runnable examples over abstract descriptions.
- Avoid unsupported claims and cite primary documentation or project sources.
- Do not reproduce large copyrighted passages.

## Publishing

Before modifying files, show the proposed course outline and ask for approval
unless the user explicitly says “create and publish the course”.

When explicitly asked to publish:
1. Create or update the course.
2. Review `git diff --check`.
3. Commit with `course: add <topic> learning guide`.
4. Push to `origin main`.
5. Report the commit hash and repository URL.

Do not alter unrelated courses.
