# Auto-job-apply Skill
> Version: 1.3.0
> Last Updated: 2025-01-31

---

## Purpose
Assist user in completing job applications accurately and efficiently using Chrome extension form-filling capabilities. Fill all forms, upload documents, and pause before final submit for user review.

## Invocation
- Command: `/auto-job-apply`
- Natural language: "auto-job-apply" or "help me apply to this job"

---

## Required Inputs

| Input | Required | Notes |
|-------|----------|-------|
| Job application link | Yes | URL to the application page |
| Resume | Yes | PDF/document for this specific application |
| Cover letter | No | Ask if available; proceed without if not |

### Input Collection Flow
1. Check what user has provided
2. If **link missing**: "To proceed, I need the job application link. Please provide the URL."
3. If **resume missing**: "To proceed, I need your resume. Please upload it."
4. If **cover letter not mentioned**: "Do you have a cover letter for this application?"
   - If yes: "Please upload it."
   - If no: "Understood, proceeding without cover letter."

---

## Data Sources (Priority Order)
1. **Resume** (provided this session) — primary source
2. **Cover letter** (provided this session, if available)
3. **about-applicant.md** (persistent knowledge base)
4. **Job description** (for context like salary range)

**CRITICAL RULE:** Never assume or fabricate information. Only use data explicitly found in these sources. When information is not available, STOP and ask the user.

---

## Process Flow

### Phase 1: Setup
1. Collect required inputs (link, resume, optional cover letter)
2. Load `about-applicant.md` from project knowledge
3. Parse resume and cover letter for key information
4. Navigate to job application link
5. Identify the ATS platform if recognizable

### Phase 2: Page-by-Page Form Filling

**For each page of the application:**

1. **Scan** — Identify all fields on the current page
2. **Categorize** each field:
   - Known (answer exists in data sources)
   - Unknown (no answer in data sources)
   - Skippable (optional with no known answer)
3. **Fill** known fields
4. **Handle** unknown fields (see Unknown Field Protocol below)
5. **Upload** documents when file upload fields are encountered
6. **Present** completed page summary to user
7. **Wait** for user approval before proceeding to next page

### Phase 3: Pre-Submit Review
Before user clicks submit, provide comprehensive review (see Pre-Submit Review section)

### Phase 4: Post-Submit Report
After user confirms submission, provide session report and request approval for updates

---

## Unknown Field Protocol

When a required field has no known answer:

### Step 1: Pause and Ask
```
"I need information for: [Field Name]

[If applicable] The job description indicates: [relevant context]
[If applicable] Common answers include: [suggestions]

What would you like me to enter? (Or provide your own answer)"
```

### Step 2: Receive Answer
User provides their answer

### Step 3: Save Preference Check
```
"Would you like me to save this for future applications?

1. Yes, update about-applicant.md
2. No, don't update (use for this session only)
3. No, and never ask about saving this type of information"
```

**Important distinctions:**
- **Option 1**: Answer is saved to about-applicant.md for future use
- **Option 2**: Answer is used this session only, not saved
- **Option 3**: Answer is used this session only, NOT saved, AND preference to never ask about saving this topic is recorded in about-applicant.md under Preferences

---

## Salary/Compensation Field Handling

When salary/compensation fields are encountered:

1. **Check job description** for posted salary/range
2. **Check about-applicant.md** for saved expectations
3. **Present both** to help user decide:

```
"This application asks for salary expectations.

Job posting indicates: [amount/range or 'Not listed in job description']
Your saved expectation: [amount/range or 'Not yet saved']

What would you like to enter?"
```

4. Proceed with standard save preference question

---

## File Upload Handling

When file upload fields are encountered:

1. **Resume upload**: Upload the provided resume file
2. **Cover letter upload**:
   - If cover letter was provided: Upload it
   - If no cover letter: Inform user "This application has a cover letter upload field. You mentioned proceeding without one. Should I leave it empty or would you like to provide one now?"
3. **Other documents**: Ask user if they have the requested document

---

## Pre-Submit Review

Before asking user to click submit, provide:

### Summary
```
## Application Summary
- **Company**: [Name]
- **Position**: [Title]
- **Platform**: [ATS name if known]

## Entered Information
[List all fields and values entered, organized by page/section]

## Uploaded Files
- Resume: [filename] ✓
- Cover Letter: [filename or 'Not uploaded']

## Review Checklist — Please Verify:
- [ ] [Any fields where answer came from user input this session, not from documents]
- [ ] [Salary/compensation if entered]
- [ ] [Custom/open-ended question responses]
- [ ] [Any dates — verify accuracy]
- [ ] [Any numerical values — verify accuracy]
- [ ] [Anything marked as "suggested" vs confirmed from your documents]

## Confidence Notes
[Any fields where the skill is less certain, or where user should double-check]
```

---

## Post-Submit Session Report

After user confirms they clicked submit, provide:

```
## Session Report: [Company] - [Role]

### Application Details
- Platform: [ATS name]
- Date: [Date]
- Pages completed: [X]
- Total fields filled: [X]

### New Information Captured
- [List any new info user provided this session]

### Issues Encountered
- [List any problems or unusual situations]

### Lessons Learned
- [Specific, actionable improvements identified]
- [Process observations]
- [Platform-specific notes if applicable]

---

**Pending Updates (require your approval):**

### about-applicant.md Changes:
[Show exact additions/changes in diff format]

### Skill Improvements:
[Describe any process improvements identified]

Approve updates? (Yes to all / Yes to specific items / No)
```

---

## Quality Standards

### Accuracy
- Every piece of information must trace back to a source document or user confirmation
- No assumptions about skills, experience, dates, or qualifications
- When uncertain: ASK, don't guess

### Formatting
- **Dates**: Use format requested by form; default to MM/YYYY if not specified
- **Phone**: Use format requested; default to (XXX) XXX-XXXX
- **Names**: Match exactly as shown in resume
- **Spelling**: Match source documents exactly

### Professional Standards
- Proper capitalization
- No typos
- Consistent formatting throughout

---

## File Storage Configuration

**Storage Location:** Google Drive (via MCP) or connected file system
**Base Path:** `[Configure: e.g., /Job Applications/auto-job-apply/]`

Files managed:
- `about-applicant.md` — current applicant data
- `about-applicant-YYYYMMDDNNN.md` — archived versions
- `auto-job-apply.md` — current skill instructions
- `auto-job-apply-YYYYMMDDNNN.md` — archived skill versions

---

### Storage Setup Check

**On first run or when file operations fail, check for storage access:**

1. Attempt to list files in the configured base path
2. If successful → proceed normally
3. If fails → trigger Setup Required flow

### Setup Required Flow

If file storage is not configured, pause and display:

```
## File Storage Setup Required

To enable automatic file versioning, I need access to a file storage location
(Google Drive recommended).

**Current status:** File storage not configured

**To set up Google Drive access:**

1. Install the Google Drive MCP server:
   - Open Claude Desktop settings
   - Go to: Settings → Developer → MCP Servers
   - Add new server with configuration below

2. MCP Server Configuration:
   ```json
   {
     "mcpServers": {
       "gdrive": {
         "command": "npx",
         "args": ["-y", "@anthropic-ai/mcp-server-gdrive"]
       }
     }
   }
   ```

3. Restart Claude Desktop

4. Authorize Google Drive access when prompted

5. Create a folder for this skill:
   - Recommended: `/Job Applications/auto-job-apply/`
   - This folder will store your applicant data and version history

6. Tell me the folder path you created, and I'll save it for future sessions.

**Alternative options:**
- If you prefer a different MCP file server, let me know which one
- If you cannot set up MCP now, I can provide file contents for manual saving (less convenient but still works)

Would you like help with any of these steps?
```

### Setup Verification

After user reports setup is complete:

1. Attempt to create a test file: `_setup-test.txt`
2. If successful:
   - Delete test file
   - Confirm: "File storage configured successfully! Ready to proceed."
   - Save base path for future sessions
3. If fails:
   - Report specific error
   - Offer troubleshooting guidance

### Fallback Mode (No File Storage)

If user cannot or chooses not to set up file storage:

```
"Understood. I'll operate in manual mode:
- I'll provide updated file contents in chat
- You can copy and save them manually
- Version history will be your responsibility

Note: You can set up automatic file storage anytime by saying 'setup file storage'."
```

In fallback mode:
- After approval, provide complete file contents in a code block
- User copies content to their preferred storage
- Skill continues to function, just without automatic file operations

---

## Update Protocol

### about-applicant.md Updates — Automatic File Versioning

When updates are approved, the skill automatically handles all file operations:

**Naming Convention:** `about-applicant-YYYYMMDDNNN.md`
- `YYYYMMDD` = Year, month, day (e.g., 20250131)
- `NNN` = Serial number starting at 001 each day (e.g., 001, 002, 003)

**Automatic Update Process:**
1. Show session report with proposed changes
2. Wait for user approval
3. Upon approval, automatically:
   - Check for existing archives from today to determine next serial number
   - Rename current `about-applicant.md` → `about-applicant-YYYYMMDDNNN.md`
   - Create new `about-applicant.md` with:
     - Incremented version number
     - Updated "Last Updated" date
     - All existing information preserved
     - New information added
4. Confirm to user: "Files updated successfully. Archived previous version as about-applicant-YYYYMMDDNNN.md"

**No user action required** — all file operations are automatic.

**Version History:**
- Archived files are kept for reference and rollback
- Serial number increments if multiple updates occur same day
- All history is preserved, nothing is lost

### Skill Self-Improvement — Automatic File Versioning

**Naming Convention:** `auto-job-apply-YYYYMMDDNNN.md`

**Automatic Update Process:**
1. Identify process improvements during session
2. Document in session report under "Lessons Learned"
3. Propose specific changes with rationale
4. Wait for user approval
5. Upon approval, automatically:
   - Rename current `auto-job-apply.md` → `auto-job-apply-YYYYMMDDNNN.md`
   - Create new `auto-job-apply.md` with:
     - Incremented version number
     - Updated changelog
     - Improvements incorporated
6. Confirm to user: "Skill updated successfully. Archived previous version as auto-job-apply-YYYYMMDDNNN.md"

**No user action required** — all file operations are automatic.

---

## Error Handling

### Form Errors
- If form shows validation error: Report to user, suggest correction, wait for guidance

### Navigation Issues
- If page doesn't load or is unexpected: Pause, describe situation, ask user how to proceed

### Missing Information
- Never skip required fields
- Never enter placeholder text
- Always pause and ask user

---

## Changelog

### v1.3.0 — 2025-01-31
- Added Storage Setup Check — detects when file storage is not configured
- Added Setup Required Flow with step-by-step Google Drive MCP instructions
- Added Setup Verification to confirm successful configuration
- Added Fallback Mode for users who cannot set up file storage
- Skill guides user through setup if needed, then proceeds automatically

### v1.2.0 — 2025-01-31
- Automatic file operations — no user engagement required for file updates
- Added File Storage Configuration section
- Skill automatically renames old files and creates new versions
- User only needs to approve; file handling is automatic

### v1.1.0 — 2025-01-31
- Added file versioning system for about-applicant.md and skill updates
- Naming convention: `filename-YYYYMMDDNNN.md` for archived versions
- Preserves complete history, enables rollback

### v1.0.0 — 2025-01-31
- Initial release
- Core form-filling workflow
- Page-by-page review process
- Unknown field handling with save preferences
- Salary field handling with job description context
- Pre-submit review checklist
- Post-submit session report
- Update approval workflow
