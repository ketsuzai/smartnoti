Scan all `.brain/*.md` files and synchronize `.brain/db-schema.md` to ensure consistency.

## Steps

1. Read `.brain/db-schema.md`
2. Read all other `.brain/*.md` files
3. For each brain file, extract implied data structures:
   - Entities mentioned (users, classes, children, etc.)
   - Fields referenced (name, date, status, etc.)
   - Relationships implied (1:N, N:M)
   - Edge cases that imply constraints (nullable, unique, enum values)
4. Cross-check against current db-schema.md:
   - Missing tables or fields
   - Mismatched types or relationships
   - Inconsistent naming
5. Update db-schema.md with any additions or corrections
6. Report a summary of what changed

## Rules

- Never remove existing schema entries — only add or flag as potentially outdated
- Flag conflicts as comments rather than silently overwriting
- Keep the existing formatting style of db-schema.md
