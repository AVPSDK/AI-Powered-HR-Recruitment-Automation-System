# AI-Powered HR Recruitment Automation System

An automated recruitment management system designed to streamline candidate application collection, resume processing, job description analysis, AI-assisted candidate matching, candidate categorization, email notifications, and interview scheduling.

## Project Overview

The AI-Powered HR Recruitment Automation System automates key stages of the recruitment process using workflow automation and AI-assisted processing.

The system reduces repetitive HR tasks, organizes candidate information, supports recruitment decisions through AI-generated matching insights, and provides an HR-facing analytics dashboard.

## Project Objectives

- Automate candidate application and resume collection.
- Extract relevant information from submitted resumes.
- Analyze job descriptions and identify required skills.
- Generate AI-assisted candidate matching scores and explanations.
- Categorize candidates based on matching results.
- Automate candidate email notifications.
- Support interview scheduling and tracking.
- Provide recruitment analytics through a dashboard.

## System Workflows

| Workflow | Description |
|---|---|
| WF-01 | Candidate Application and Resume Collection |
| WF-02 | Resume Parsing and Data Extraction |
| WF-03 | Job Description Analysis |
| WF-04 | AI Candidate Matching |
| WF-05 | Candidate Categorization |
| WF-06 | Candidate Email Notification |
| WF-07 | Interview Scheduling |
| WF-08 | HR Dashboard Integration |

## Technology and Services

- **n8n:** Workflow automation and integration.
- **Google Forms:** Candidate application collection.
- **Google Drive:** Resume file storage.
- **Google Sheets:** Recruitment database.
- **AI integration:** Resume analysis and candidate matching.
- **Gmail:** Email notifications.
- **Google Looker Studio:** HR recruitment analytics dashboard.

## Recruitment Database

The project uses an existing Google Sheets database named `AI-HR-Recruitment-Database`, organized into three worksheets:

- **Applications:** Candidate information, extracted resume details, AI match scores, application status, and HR decisions.
- **Notification Logs:** Notification records, delivery status, timestamps, and error details.
- **Interviews:** Interview details, scheduling information, interview status, and HR notes.

## HR Analytics Dashboard

The HR dashboard is developed using Google Looker Studio and includes six retained KPIs:

1. Total Applications
2. Average AI Match Score
3. Total Interviews
4. HR Decisions
5. Job-wise Applications
6. Notification Reliability

The dashboard is designed for authorized HR users. Candidate contact details and resume links are excluded from the overview. AI-generated scores are advisory and do not replace human hiring decisions.

## Repository Structure

- `01 - Project Documentation/` — Workflow documentation and technical explanations.
- `02 - n8n Workflows/` — Exported n8n workflow JSON files.
- `03 - Test Data/` — Sample job descriptions and resumes used for testing.
- `04 - Database/` — Database-related documentation and evidence.
- `05 - Dashboard/` — HR dashboard documentation and evidence.
- `06 - Screenshots/` — Workflow configuration, execution, and validation screenshots.

## Testing and Validation

The individual workflows were implemented and tested through execution checks and verification of their expected outputs. Supporting screenshots and workflow exports are maintained in the repository.

## Important Notes

- AI-generated candidate match scores are decision-support information, not automatic hiring decisions.
- Access to recruitment information should remain restricted to authorized users.
- Sample resumes and test data should not contain unnecessary personal information.
- Credentials, API keys, passwords, and private candidate information must not be committed to the repository.

## Project Status

The implementation of workflows WF-01 through WF-07 and HR Dashboard Integration WF-08 has been completed.

The project documentation and final repository review are being finalized.