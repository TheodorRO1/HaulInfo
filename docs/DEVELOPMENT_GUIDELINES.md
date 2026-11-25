# Development guidelines

## Language and platform

Rust is the primary programming language. Target platform is Windows 10/11 x64.

## Quality expectations

- Low resource usage
- Stable performance
- Smooth rendering
- No noticeable impact on ETS2 performance
- Background processing for demanding tasks
- Reliable error handling
- Maintainable and testable code
- Offline functionality where required

Avoid unnecessary dependencies. Each dependency must be justified.

## Writing style

Documentation, code comments, commit messages, and other project text are written like a developer documenting their own project:

- Use natural, straightforward language.
- Keep sentences clear and reasonably short.
- Avoid generic introductions and unnecessary explanations.
- Do not repeat the same information across multiple files.
- Avoid marketing language, exaggerated claims and buzzwords.
- Use technical terms when they add useful meaning, not just to sound professional.
- Keep headings simple and descriptive.
- Do not add comments that simply repeat what the code already says.

## Git workflow

- Keep the repository at https://github.com/TheodorRO1/HaulInfo.
- Use the `main` branch unless there is a good reason not to.
- Keep changes focused and meaningful.
- Verify changes before committing.
- Run relevant tests when applicable.
- Commit every meaningful completed change.
- Push completed commits to GitHub before moving on to the next task.
- Use short, descriptive commit messages.
- Never overwrite or discard existing user work without permission.
- Avoid unnecessary dependencies.
- Document important decisions.

Do not create meaningless commits for every individual line edit. Each meaningful completed task or change should have its own verified Git update.

## Roadmap status synchronisation

docs/ROADMAP.md is the detailed source of truth for project progress. The roadmap table in README.md must always match it.

Whenever a phase starts, progresses, enters review or is completed:

1. Update its status in docs/ROADMAP.md.
2. Update the corresponding status in README.md.
3. Make sure both files agree.
4. Include the changes in the same meaningful Git commit as the work that caused the status change.

Never leave the README roadmap outdated.

Do not mark a phase as completed simply because its files have been created. Completion requires the phase's work to be checked and reviewed.