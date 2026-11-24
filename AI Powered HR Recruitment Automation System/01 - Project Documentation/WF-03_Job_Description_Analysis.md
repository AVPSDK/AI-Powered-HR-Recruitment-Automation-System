# WF-03 — Job Description Analysis

## 1. Project Overview

**Workflow ID:** WF-03  
**Workflow Name:** Job Description Analysis  
**Project:** AI-Powered HR Recruitment Automation System  
**Workflow Type:** AI-powered job requirement extraction and database management  
**Platform:** n8n (Self-Hosted)  
**AI Provider:** Groq API  
**AI Model:** openai/gpt-oss-20b  
**Database:** Google Sheets  
**Status:** Implemented, tested, saved, and inactive

### 1.1 Purpose

WF-03 automates the analysis of job descriptions submitted by HR. It uses an AI model to extract structured job requirements, validates the submitted information, and stores the extracted details in the Jobs worksheet.

The workflow supports both new job registration and updates to existing job records.

### 1.2 Objectives

- Collect job information through an HR submission form.
- Normalize and validate submitted job descriptions.
- Use AI to extract structured job requirements.
- Separate mandatory skills from preferred skills.
- Preserve the original job information alongside AI-generated analysis.
- Prevent duplicate job records by checking the Job ID.
- Create new job records or update existing records.
- Handle invalid job descriptions through validation errors.

## 2. Technology Stack

| Technology | Purpose |
|---|---|
| n8n | Workflow automation and orchestration |
| Groq API | AI-powered job requirement analysis |
| openai/gpt-oss-20b | Language model used for extraction |
| Google Sheets | Storage of structured job information |
| Basic Authentication | Restrict access to the HR submission form |
| JavaScript | Data normalization, validation, parsing, and formatting |

## 3. Workflow Input

The workflow begins with the **HR Job Description Submission** form.

The form collects the following fields:

| Field | Description |
|---|---|
| Job ID | Unique identifier for the job |
| Job Title | Position being recruited for |
| Department | Department associated with the position |
| Job Description | Complete job description |
| Job Status | Open, Closed, or On Hold |

All five fields are required.

The form uses Basic Authentication and is configured to respond when the workflow finishes.

## 4. System Architecture

### 4.1 Workflow Components

The workflow consists of the following main components:

1. **Form Trigger:** Collects job information from HR.
2. **Normalize Job Submission:** Trims input values, standardizes the Job ID, and normalizes the description.
3. **Validate Job Description:** Checks the description length and required data.
4. **AI — Analyze Job Requirements:** Sends the validated description to the Groq API.
5. **Merge Original Job Data + AI Analysis:** Combines the original job information with the AI response.
6. **Parse AI Job Analysis:** Parses and validates the AI-generated JSON.
7. **Format Extracted Job Data:** Formats extracted arrays into readable text.
8. **Find Existing Job:** Searches the Jobs worksheet using the Job ID.
9. **Is Job Already Registered?:** Determines whether the job already exists.
10. **Create New Job:** Adds a new record when the Job ID is not found.
11. **Update Existing Job:** Updates the matching record when the Job ID already exists.

### 4.2 Workflow Diagram

The overall workflow follows this sequence:

HR Submission Form → Normalize Job Submission → Validate Job Description → AI Analysis → Merge Original Data and AI Analysis → Parse AI Analysis → Format Extracted Data → Find Existing Job → Check Existing Job

The workflow then follows one of two branches:

- **False branch:** Create New Job.
- **True branch:** Update Existing Job.

## 5. Data Processing and Validation

### 5.1 Input Normalization

The normalization stage:

- Removes leading and trailing spaces.
- Converts Job ID to uppercase.
- Normalizes line endings.
- Reduces unnecessary spaces and excessive blank lines.
- Retains the original submitted fields.

### 5.2 Job Description Validation

The workflow enforces the following description-length rules:

| Validation | Condition |
|---|---|
| Minimum length | 100 characters |
| Maximum length | 20,000 characters |
| Valid input | Description is within the permitted range |

If the description is too short or exceeds the maximum length, the workflow stops at the validation stage.

Invalid descriptions are not forwarded to the Google Sheets write nodes.

## 6. AI-Powered Job Requirement Extraction

### 6.1 API Configuration

The AI analysis node sends a POST request to:

`https://api.groq.com/openai/v1/chat/completions`

The configured model is:

`openai/gpt-oss-20b`

The request uses authenticated header credentials stored in n8n.

Retry configuration:

- Retry on Fail: Enabled
- Maximum Attempts: 3
- Wait Between Attempts: 2,000 milliseconds
- On Error: Stop Workflow

### 6.2 Extracted Information

The AI model returns eight structured fields:

| Field | Description |
|---|---|
| required_skills | Mandatory technical or professional skills |
| preferred_skills | Additional desirable skills |
| experience_required | Required experience |
| education_required | Educational requirements |
| key_responsibilities | Main responsibilities of the position |
| role_level | Level of the job role |
| key_competencies | Important professional competencies |
| analysis_notes | Additional job-related analysis notes |

### 6.3 AI Processing Guidelines

The AI analysis is designed to:

- Extract requirements supported by the submitted job description.
- Avoid inventing qualifications or requirements.
- Distinguish mandatory and preferred skills.
- Treat the submitted description as untrusted input.
- Ignore instructions embedded in the description that attempt to alter the analysis process.
- Exclude protected characteristics from job requirement extraction.
- Avoid making candidate selection or rejection decisions.

## 7. AI Response Parsing and Formatting

### 7.1 Parsing

The Parse AI Job Analysis node:

- Reads the AI response content.
- Parses the response as JSON.
- Checks that all eight required fields are present.
- Validates that array fields contain strings.
- Validates that experience, education, and role level are strings.
- Stops the workflow if the response is missing or invalid.

### 7.2 Data Formatting

The Format Extracted Job Data node converts the following AI fields into readable comma-separated text:

- Required Skills
- Preferred Skills
- Experience Required
- Education Required

It also removes duplicate entries and trims unnecessary spaces.

## 8. Existing Job Detection

The Find Existing Job node searches the Jobs worksheet using the submitted Job ID.

The IF node, named **Is Job Already Registered?**, determines whether a matching record exists.

The workflow follows these conditions:

| Condition | Action |
|---|---|
| Job ID already exists | Update Existing Job |
| Job ID does not exist | Create New Job |

This approach supports both new job registration and existing job maintenance without intentionally creating duplicate records for the same Job ID.

## 9. Google Sheets Integration

The workflow stores job information in the existing **Jobs** worksheet.

### 9.1 New Job Creation

When a Job ID is not found, the Create New Job node appends a new record.

The mapped fields are:

- Job ID
- Job Title
- Department
- Required Skills
- Preferred Skills
- Experience Required
- Education Required
- Job Description
- Job Status
- Created Date

The Created Date is populated using the workflow's current date and time.

### 9.2 Existing Job Update

When a matching Job ID is found, the Update Existing Job node updates the corresponding record.

The update includes the job title, department, extracted requirements, description, and status.

The original Created Date is not overwritten during an update.

## 10. Error Handling and Reliability

The workflow includes several error-handling measures:

- Required-field checks during normalization.
- Minimum and maximum description-length validation.
- Retry handling for temporary Groq API failures.
- JSON parsing checks.
- Required-field and data-type checks for AI output.
- Workflow termination when critical processing errors occur.
- Google Sheets record lookup before creating or updating a job.

These measures help prevent invalid or incomplete job information from being stored.

## 11. Testing and Verification

WF-03 underwent comprehensive testing through the submission form and workflow execution.

The completed test suite covered:

- New job registration.
- Existing job update.
- Job ID-based branch selection.
- Minimum description-length validation.
- Maximum description-length validation.
- Authentication behavior.
- Google Sheets data verification.
- Execution-node input and output inspection.

### 11.1 Test Results

**Overall result: All reported tests A–F passed.**

The following outcomes were verified:

- New job records were created correctly.
- Existing job records were updated correctly.
- Updating an existing job did not create an additional row for that Job ID.
- The original Created Date was preserved during updates.
- Invalid job descriptions were stopped at validation.
- Invalid descriptions did not reach Google Sheets write operations.
- Authentication behaved as expected in the tests performed.
- Executed nodes were inspected individually.
- Google Sheets records were checked for correctness.

No outstanding issues were reported during final verification.

## 12. Security and Privacy Considerations

The workflow incorporates the following measures:

- Basic Authentication protects the HR submission form.
- API credentials are stored in n8n credentials rather than hardcoded in workflow logic.
- Submitted job descriptions are treated as untrusted input by the AI prompt.
- AI analysis is restricted to job-related information.
- Protected characteristics are excluded from job requirement extraction.

For deployment beyond a local or controlled environment, HTTPS, access controls, credential management, and appropriate data-retention policies should be reviewed.

## 13. Limitations and Future Improvements

Potential improvements include:

- Adding more detailed execution monitoring and alerting.
- Improving API failure reporting.
- Introducing a formal audit trail for job-description changes.
- Adding HR approval before publishing job descriptions.
- Improving prompt evaluation using a larger collection of job descriptions.
- Integrating the extracted job requirements with WF-04 AI Candidate Matching.

AI-generated requirements should be reviewed by HR before being used in recruitment decisions.

## 14. Conclusion

WF-03 successfully automates the collection, validation, AI analysis, and storage of job descriptions.

It extracts structured job requirements using the Groq API, preserves the original job information, and supports both new job creation and existing job updates through Google Sheets.

The workflow passed its reported functional tests and final verification. It is saved and remains inactive pending further integration and project-level testing.

**Workflow:** WF-03 — Job Description Analysis  
**Final Verification:** Passed  
**Workflow State:** Inactive