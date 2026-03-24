Update the relevant `.brain/*.md` file(s) to reflect decisions made during the current conversation.

## Steps

1. Review the current conversation to identify decisions, changes, or new information about features
2. List the relevant `.brain/*.md` files that need updating
3. Read each file before editing
4. Update only the sections that changed — do not rewrite the entire file
5. If a new feature was discussed that has no brain file yet, create `.brain/[feature-name].md`
6. If DB schema changes were implied, update `.brain/db-schema.md` as well
7. Report what was changed and in which files

## What to capture

- Confirmed design/UX decisions
- Edge case handling
- Data structure changes
- Feature scope changes (added or removed)
- Open questions that were resolved
- New open questions that emerged

## Usage

User says: `/project:brain-update`

No arguments needed — scan the conversation automatically.
