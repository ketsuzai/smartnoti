Compare a built HTML file against its brain spec and report gaps or inconsistencies.

## Steps

1. Identify the target HTML file and its corresponding `.brain/*.md` file
2. Read both files
3. Check the following:

### Functional gaps
- Are all listed components present in the HTML?
- Are all user flows implemented (including edge cases)?
- Are all role-based views handled correctly?
- Are all filter/search behaviors implemented?

### Data gaps
- Does the mock data match the DB schema in `.brain/db-schema.md`?
- Are all fields referenced in the brain present in the mock data?

### UX gaps
- Are empty states handled?
- Are error states handled?
- Are loading states considered?

### Design gaps
- Does the color scheme match the brain spec?
- Are all specified components visually present?

4. Output a checklist:
   - ✅ Implemented
   - ⚠️ Partial or unclear
   - ❌ Missing

5. Ask the user which gaps to fix, then fix them.

## Usage

User says: `/project:review [page-name]`

Example: `/project:review dashboard` → compares `dashboard.html` against `.brain/dashboard-main.md`
