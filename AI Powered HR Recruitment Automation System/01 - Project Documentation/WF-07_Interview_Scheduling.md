# WF-07 – Interview Scheduling

## 1. Project Overview

**Workflow Name:** WF-07 – Interview Scheduling

**Project:** AI-Powered HR Recruitment Automation System

**Platform:** n8n

**Primary Purpose:** Automate candidate interview notification emails when HR schedules or reschedules an interview through the Google Sheets-based recruitment database.

### 1.1 Overview

WF-07 is designed to streamline the interview communication stage of the recruitment process. It monitors changes in the Interviews sheet, validates interview information, checks candidate consent, prevents duplicate notifications, and sends interview-related emails.

The workflow is designed to support HR operations while keeping interview scheduling and hiring decisions under human control.

### 1.2 Scope

The workflow supports:

- Interview confirmation email notifications.
- Interview rescheduling notifications.
- Interview data validation.
- Candidate consent verification.
- Duplicate notification prevention.
- Email success and failure logging.

The workflow does not automatically make hiring decisions or create calendar events.

### 1.3 Workflow Status

**Configuration Status:** Configured and structurally reviewed

**Execution Status:** Not executed

**Activation Status:** Inactive

## 2. Technology Stack & Integrations

WF-07 uses automation, spreadsheet-based data management, and email services to manage interview notifications.

| Technology | Purpose |
|---|---|
| n8n | Workflow automation and conditional routing |
| Google Sheets | Stores applications, interview schedules, and notification logs |
| Google Sheets Trigger | Detects updates in the Interviews sheet |
| Gmail | Sends interview confirmation and rescheduling emails |
| JavaScript | Validates interview information and generates notification IDs |
| Google OAuth credentials | Authenticates access to connected Google services |

### 2.1 Google Sheets Integration

The workflow uses the existing `AI-HR-Recruitment-Database` spreadsheet.

**Applications sheet:** Used to retrieve candidate application information and consent status.

**Interviews sheet:** Used as the source of interview details entered and maintained by HR.

**Notification Logs sheet:** Used to record email notification outcomes, timestamps, recipients, and error details.

### 2.2 Gmail Integration

Gmail is used to deliver interview-related messages to candidates. The workflow prepares email subjects and bodies dynamically using candidate and interview information.

Email delivery errors are routed to a separate failure logging branch for tracking and troubleshooting.

### 2.3 JavaScript Processing

JavaScript Code nodes handle required-field validation, email format validation, interview mode validation, notification eligibility, and notification ID generation.

### 2.4 Authentication & Access

Google integrations use configured Google credentials within n8n. Credentials are managed through n8n's credential system rather than being embedded directly into workflow code.

## 3. Workflow Architecture & Logic

### 3.1 Architecture Overview

WF-07 follows a sequential automation architecture with validation, eligibility checks, consent verification, duplicate prevention, email delivery, and logging.

The workflow is initiated when HR updates an interview record in the Google Sheets `Interviews` sheet.

### 3.2 Processing Stages

**Stage 1 — Trigger**

The Google Sheets Trigger monitors row updates in the Interviews sheet.

**Stage 2 — Validation**

The Validate Interview Data Code node checks required fields, email format, interview mode, and online meeting link requirements.

**Stage 3 — Eligibility Check**

The workflow allows notifications only when the interview status is `Scheduled` or `Rescheduled`.

**Stage 4 — Application Lookup & Consent Verification**

The candidate's application record is retrieved from the Applications sheet using the Application ID. The workflow checks that Consent Status is `Yes`.

**Stage 5 — Notification ID Generation**

A notification ID is generated based on Interview ID and notification type. Rescheduling notifications include a date-time fingerprint to distinguish updated interview details.

**Stage 6 — Duplicate Prevention**

The workflow checks Notification Logs for an existing notification ID with `Sent` status. Previously sent notifications are skipped.

**Stage 7 — Email Preparation & Delivery**

The workflow dynamically prepares the subject and body, then sends the notification using Gmail.

**Stage 8 — Outcome Logging**

Successful emails and failed email attempts are routed to separate logging nodes and recorded in Notification Logs.

### 3.3 Decision Gates

| Gate | Condition | Outcome |
|---|---|---|
| Validation | Required data valid | Continue |
| Eligibility | Scheduled / Rescheduled | Continue |
| Consent | Consent Status = Yes | Continue |
| Duplicate | Existing ID already Sent | Skip |
| Email | Gmail success | Log Sent |
| Email | Gmail error | Log Failed |

### 3.4 Architecture Principles

- Sequential processing with explicit validation.
- Consent-first communication safeguards.
- Duplicate notification protection.
- Separate success and failure handling.
- Human-controlled interview scheduling and hiring decisions.

## 4. Validation Rules & Notification ID Logic

### 4.1 Interview Data Validation

The workflow validates interview information before proceeding with candidate notifications.

The following fields are required:

- Interview ID
- Application ID
- Candidate Name
- Candidate Email
- Job Title
- Interview Date
- Interview Time
- Interview Mode

The workflow also validates the candidate email format and restricts Interview Mode to `Online` or `In-person`.

For online interviews, a Meeting Link is mandatory. For in-person interviews, the meeting link is optional.

### 4.2 Notification Eligibility

Only interview records with the following statuses are eligible for email notifications:

- `Scheduled`
- `Rescheduled`

Other statuses do not proceed to the notification process.

### 4.3 Candidate Consent Validation

Before preparing an interview email, the workflow retrieves the candidate's application record and checks the `Consent Status` field.

Only candidates with consent status `Yes` are allowed to proceed. Missing consent data or a `No` value prevents the notification from continuing.

### 4.4 Notification ID Strategy

Notification IDs are used to identify individual interview notification events and support duplicate prevention.

**Interview Confirmation ID:**

`INT-<InterviewID>-CONF`

**Interview Rescheduling ID:**

`INT-<InterviewID>-RESCHED-<DateTimeFingerprint>`

The rescheduling identifier includes the interview date and time fingerprint. This allows an updated reschedule event to have a different identifier from a previously sent notification.

### 4.5 Duplicate Notification Prevention

Before sending an email, the workflow checks the Notification Logs sheet using the generated Notification ID.

If a matching record has a `Sent` status, the notification is skipped. If no successful record exists, the notification is permitted to continue, allowing failed attempts to be retried.

### 4.6 Validation Outcomes

Invalid interview data is recorded in the Notification Logs sheet with status `Invalid Data` and associated validation error details.

Valid but ineligible interview records do not proceed to candidate notification.

These safeguards help maintain data quality, consent compliance, and reliable communication.

## 5. Email Notification Design & Templates

### 5.1 Notification Types

WF-07 supports two types of candidate email notifications:

| Notification Type | Purpose |
|---|---|
| Interview Confirmation | Notify candidates when HR schedules an interview |
| Interview Rescheduled | Notify candidates when HR changes an interview schedule |

### 5.2 Dynamic Email Content

Email content is generated dynamically using interview and candidate information retrieved from the workflow.

Dynamic fields include:

- Candidate Name
- Job Title
- Interview Date
- Interview Time
- Interview Mode
- Meeting Link (for online interviews)
- Location / Additional Information (for in-person interviews, when provided)

### 5.3 Email Subject Design

**Confirmation Email:**

`Interview Invitation – <Job Title>`

**Rescheduling Email:**

`Interview Rescheduled – <Job Title>`

### 5.4 Email Body Structure

The email body contains:

1. Candidate greeting.
2. Confirmation or rescheduling message.
3. Job title.
4. Interview date and time.
5. Interview mode.
6. Meeting link or in-person location information, where applicable.
7. Preparation and punctuality reminder.
8. Reply instructions for questions.
9. HR Recruitment Team sign-off.

### 5.5 Email Delivery Integration

The Gmail node sends the prepared notification to the candidate's email address.

The Gmail node is configured to continue using its error output if a sending error occurs, allowing the workflow to route the outcome to the appropriate logging branch.

### 5.6 Delivery Outcome Handling

**Success branch:** Records notification details with `Sent` status and timestamp.

**Failure branch:** Records notification details with `Failed` status, timestamp, and error information.

This separation improves troubleshooting and provides an audit trail for email delivery attempts.

### 5.7 Duplicate Communication Protection

The notification ID lookup prevents previously successful notifications from being sent again under the same notification ID. Failed notifications may be retried through the configured duplicate-check logic.

## 6. Logging, Error Handling & Reliability

### 6.1 Notification Logging System

WF-07 uses the `Notification Logs` sheet in the `AI-HR-Recruitment-Database` spreadsheet to record notification attempts and their outcomes.

The log contains the following fields:

| Field | Description |
|---|---|
| Notification ID | Unique identifier for the notification event |
| Application ID | Links the notification to the candidate application |
| Notification Type | Confirmation, rescheduling, or validation event |
| Recipient Email | Candidate's email address |
| Sent Status | Sent, Failed, or Invalid Data |
| Sent At | Timestamp of the recorded attempt |
| Workflow Name | Identifies the originating workflow |
| Error Details | Stores error or validation information |

### 6.2 Logging Statuses

**Sent:** The Gmail node completed successfully, and the workflow recorded a success log.

**Failed:** Gmail returned an error, and the workflow recorded the failure details.

**Invalid Data:** Interview data did not pass the required validation rules.

### 6.3 Error Handling

The workflow uses separate error-routing paths for email delivery.

If Gmail sends successfully, the success branch records the notification outcome.

If Gmail encounters an error, the error branch records the failure and available error details.

Invalid interview records are routed to a dedicated validation logging node before proceeding is prevented.

### 6.4 Retry Logic

The duplicate notification check is based on notification ID and successful delivery status.

- Existing notification with `Sent` status is skipped.
- Existing notification with `Failed` status does not block a retry.
- A new rescheduling date/time generates a different notification ID.

This allows retrying failed notification attempts while reducing the risk of duplicate successful emails.

### 6.5 Reliability & Audit Trail

The logging system provides an operational history of interview-related notifications. It helps HR identify successful communications, investigate email delivery failures, review invalid records, and trace notification activity to its originating workflow.

The system improves transparency and troubleshooting without automatically making recruitment decisions.

## 7. Database Structure & Data Flow

### 7.1 Database Overview

WF-07 uses the existing `AI-HR-Recruitment-Database` Google Sheets spreadsheet as its data source and notification record store.

The workflow interacts with three existing sheets:

| Sheet | Role in WF-07 |
|---|---|
| Applications | Retrieves candidate application details and consent status |
| Interviews | Stores HR-managed interview scheduling details and triggers workflow updates |
| Notification Logs | Records validation outcomes, email attempts, and notification statuses |

### 7.2 Interview Record

The Interviews sheet contains the following fields:

| Field | Purpose |
|---|---|
| Interview ID | Unique identifier for an interview record |
| Application ID | Links the interview to a candidate's application |
| Candidate Name | Candidate's display name |
| Candidate Email | Notification recipient |
| Job Title | Position for which the candidate is interviewing |
| Interview Date | Scheduled interview date |
| Interview Time | Scheduled interview time |
| Interview Mode | Online or In-person |
| Meeting Link | Online interview joining link |
| Interview Status | Scheduling status |
| HR Notes | Location or additional HR information |

### 7.3 Data Matching

The workflow uses Application ID to retrieve the corresponding candidate application from the Applications sheet.

Interview information and application information are merged to support consent verification and notification preparation.

Notification ID is used to look up previously logged notification events and determine whether a successful notification already exists.

### 7.4 Data Flow

1. HR updates an interview record in Interviews.
2. The workflow reads and validates the updated record.
3. Application ID is used to retrieve the associated candidate application.
4. Interview and application data are combined.
5. Consent Status is checked.
6. Notification ID is generated.
7. Notification Logs are checked for an existing successful notification.
8. Gmail sends the email when the safeguards allow it.
9. The outcome is recorded in Notification Logs.

### 7.5 Schema Preservation

The workflow uses the existing Google Sheets structure and does not require additional database columns.

The design preserves the existing HR database while extending its functionality through automation.

## 8. Workflow Limitations, Assumptions & Future Improvements

### 8.1 Current Limitations

The current version of WF-07 has the following limitations:

- **No calendar integration:** Interview scheduling and date/time management are handled manually by HR through Google Sheets.
- **No automatic calendar invitations:** The workflow sends email notifications but does not create Google Calendar or other calendar events.
- **No cancellation notifications:** Interview cancellation emails are not included in the current workflow.
- **No automated execution testing:** The workflow configuration has been reviewed, but end-to-end execution has not been performed.
- **No production deployment:** The workflow remains inactive and has not been deployed for live recruitment operations.
- **Email delivery limitations:** Gmail execution success does not guarantee that the recipient has received, opened, or read the email.
- **Rescheduling dependency:** HR must update the interview details and set the appropriate interview status to trigger the intended notification process.

### 8.2 Design Assumptions

The workflow is designed based on the following assumptions:

- HR maintains accurate candidate and interview information in Google Sheets.
- Each interview has a valid Interview ID and Application ID.
- Candidate application records can be matched using Application ID.
- Candidate consent must be explicitly recorded as `Yes` before notification.
- Interview status must be `Scheduled` or `Rescheduled` to qualify for notification.
- Online interviews require a valid meeting link.
- HR is responsible for interview scheduling decisions and data accuracy.

### 8.3 Future Improvements

The following features may be considered in future versions:

| Enhancement | Description |
|---|---|
| Google Calendar integration | Automatically create and update calendar events |
| Cancellation notifications | Notify candidates when interviews are cancelled |
| Automated testing | Perform controlled tests covering success, failure, validation and duplicate scenarios |
| Advanced monitoring | Improve visibility into workflow errors and execution history |
| HR dashboard | Display interview schedules, notification status and recruitment metrics |
| Time-zone handling | Standardize interview time-zone display for candidates |
| Retry and recovery enhancements | Improve recovery controls for failed notifications |
| Deployment readiness | Add credential, access, operational and security checks before production use |

### 8.4 Future Scope

WF-07 establishes a foundation for interview communication automation. Future enhancements could reduce manual work for HR, improve candidate communication, increase operational visibility and integrate recruitment scheduling with other business tools.

These enhancements are outside the scope of the current configured version.

## 9. Configuration Summary & Node Inventory

### 9.1 Workflow Configuration Summary

| Configuration | Value |
|---|---|
| Workflow Name | WF-07 - Interview Scheduling |
| Automation Platform | n8n |
| Trigger Source | Google Sheets — Interviews |
| Trigger Event | Row Updated |
| Polling Interval | 1 minute |
| Candidate Application Source | Applications sheet |
| Notification Record Source | Notification Logs sheet |
| Email Service | Gmail |
| Eligible Interview Statuses | Scheduled, Rescheduled |
| Consent Requirement | Consent Status = Yes |
| Duplicate Prevention | Notification ID + successful Sent status check |
| Execution Mode | Configuration only |
| Active Status | Inactive |
| Testing Status | Not executed |

### 9.2 Node Inventory

| No. | Node Name | Function |
|---:|---|---|
| 1 | Trigger - Interview Sheet Updates | Detects updated interview records |
| 2 | Validate Interview Data | Validates required fields and interview information |
| 3 | Check Interview Validation | Routes valid and invalid records |
| 4 | Log Invalid Interview Data | Records validation failures |
| 5 | Check Interview Notification Eligibility | Checks whether status is eligible for notification |
| 6 | Lookup Candidate Application | Retrieves candidate application and consent data |
| 7 | Merge Interview and Application Data | Combines interview and candidate data |
| 8 | Check Candidate Consent | Allows notification only when consent is Yes |
| 9 | Generate Interview Notification ID | Creates confirmation or reschedule identifier |
| 10 | Lookup Existing Interview Notification | Searches notification logs for an existing ID |
| 11 | Merge Interview and Log Lookup | Combines interview data and existing log results |
| 12 | Check Duplicate Interview Notification | Prevents repeated successful notifications |
| 13 | Prepare Interview Email | Creates dynamic email subject and message |
| 14 | Send Interview Email | Sends candidate notification through Gmail |
| 15 | Log Interview Email Success | Records successful Gmail execution |
| 16 | Log Interview Email Failure | Records email-send errors |

### 9.3 Branch Summary

| Branch | Destination | Purpose |
|---|---|---|
| Validation failure | Log Invalid Interview Data | Records invalid interview details |
| Ineligible status | No further processing | Stops notifications for unsupported statuses |
| Consent not granted | No further processing | Prevents notification without consent |
| Duplicate already sent | Stops processing | Avoids repeat successful notifications |
| Gmail success | Log Interview Email Success | Records successful email execution |
| Gmail error | Log Interview Email Failure | Records send failure for review/retry |

### 9.4 Configuration Status

All 16 nodes have been added, configured, named, and structurally reviewed. The workflow connections and safeguards have been reviewed.

The workflow remains inactive and unexecuted. This inventory represents the current configuration and does not imply that end-to-end runtime behavior has been verified.

## 10. Conclusion & Project Outcome

### 10.1 Conclusion

WF-07 - Interview Scheduling extends the AI-Powered HR Recruitment Automation System by automating candidate interview email notifications based on HR-managed interview records in Google Sheets.

The workflow is designed to validate interview information, retrieve associated application data, verify candidate consent, generate notification identifiers, prevent duplicate successful notifications, and send interview confirmation or rescheduling emails through Gmail.

The implementation incorporates structured validation, eligibility checks, consent verification, duplicate prevention, and separate success and failure logging paths to improve reliability and maintainability.

### 10.2 Project Outcome

The workflow configuration establishes a structured approach to interview communication automation with the following capabilities:

- Automated processing of updated interview records.
- Interview information validation.
- Candidate application matching.
- Consent-based notification control.
- Interview confirmation and rescheduling email preparation.
- Duplicate notification prevention.
- Success, failure, and invalid-data logging.
- Clear separation between HR-managed decisions and automation responsibilities.

### 10.3 Completion Status

WF-07 has been configured, saved, and structurally reviewed. Its node configuration, workflow connections, notification logic, safeguards, and documentation have been reviewed.

However, the workflow has not been executed or activated. Therefore, runtime behavior, end-to-end email delivery, and operational performance have not yet been verified.

WF-07 is currently considered **configuration-complete and documentation-complete, pending controlled testing and deployment**.

### 10.4 Learning & Technical Value

This workflow demonstrates practical experience with workflow automation, event-based processing, data validation, Google Sheets integration, Gmail automation, conditional routing, duplicate prevention, error handling, and documentation.

It also reinforces the importance of consent, data integrity, human oversight, and responsible automation when developing recruitment-related systems.