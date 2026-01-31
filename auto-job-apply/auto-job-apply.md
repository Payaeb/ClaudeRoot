# Auto-job-apply Skill
> Version: 1.1.0
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

## Update Protocol

### about-applicant.md Updates — File Versioning

When updates are approved, use this versioning system:

**Naming Convention:** `about-applicant-YYYYMMDDNNN.md`
- `YYYYMMDD` = Year, month, day (e.g., 20250131)
- `NNN` = Serial number starting at 001 each day (e.g., 001, 002, 003)

**Update Process:**
1. Only update after session report and explicit user approval
2. Show exact changes before applying
3. Instruct user to:
   - Rename current `about-applicant.md` → `about-applicant-YYYYMMDDNNN.md`
   - Example: `about-applicant.md` → `about-applicant-20250131001.md`
4. Provide complete new `about-applicant.md` content with:
   - Incremented version number
   - Updated "Last Updated" date
   - All existing information preserved
   - New information added
5. User creates fresh `about-applicant.md` with the provided content
6. User uploads new file to Project knowledge

**Version History:**
- Archived files are kept for reference and rollback
- Serial number increments if multiple updates occur same day
- All history is preserved, nothing is lost

### Skill Self-Improvement — File Versioning

**Naming Convention:** `auto-job-apply-YYYYMMDDNNN.md`

**Update Process:**
1. Identify process improvements during session
2. Document in session report under "Lessons Learned"
3. Propose specific changes with rationale
4. After user approval, instruct user to:
   - Rename current `auto-job-apply.md` → `auto-job-apply-YYYYMMDDNNN.md`
5. Provide complete new `auto-job-apply.md` content with:
   - Incremented version number
   - Updated changelog
   - Improvements incorporated
6. User creates fresh file and updates Project custom instructions

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

### v1.1.0 — 2025-01-31
- Added file versioning system for about-applicant.md and skill updates
- Naming convention: `filename-YYYYMMDDNNN.md` for archived versions
- Preserves complete history, enables rollback
- Clear instructions for user to rename and recreate files

### v1.0.0 — 2025-01-31
- Initial release
- Core form-filling workflow
- Page-by-page review process
- Unknown field handling with save preferences
- Salary field handling with job description context
- Pre-submit review checklist
- Post-submit session report
- Update approval workflow
