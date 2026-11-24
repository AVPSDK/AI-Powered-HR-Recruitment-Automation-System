# WF-06 — Candidate Email Notifications

## 1. Project Overview

WF-06 is an automated candidate email notification workflow developed as part of an AI-powered HR recruitment automation system using n8n.

The workflow manages candidate communication throughout the initial recruitment process by sending automated application acknowledgement, shortlisting, and rejection emails. It integrates Google Sheets for candidate status tracking and notification logging, and Gmail for email delivery.

The primary objective is to reduce manual communication workload, improve response consistency, maintain notification records, and ensure candidates receive timely updates based on recruitment decisions.

The workflow uses HR-controlled application status updates to determine whether shortlisted or rejected candidates should receive notifications. AI-generated candidate scores are not used as an independent rejection mechanism.

**Project Status:** Configured and structurally reviewed; pending controlled testing.

## 2. Objectives

- Automate application acknowledgement emails after a new application is recorded.
- Notify candidates when HR changes their application status to Shortlisted.
- Send respectful rejection notifications after HR confirms the Rejected status.
- Validate candidate information and consent before sending emails.
- Prevent duplicate successful notifications for the same application and notification type.
- Record successful, failed, and invalid notification attempts in Google Sheets.
- Reduce repetitive manual email communication.
- Maintain a consistent and professional candidate communication experience.

## 3. Technology Stack

| Technology | Purpose |
|---|---|
| n8n | Workflow automation and orchestration |
| Google Sheets | Candidate database, status tracking, notification log storage |
| Gmail | Automated candidate email delivery |
| JavaScript | Data validation, normalization, conditional logic, and expressions |
| Google Sheets Trigger | Detect new applications and HR status updates |

The workflow uses node-based automation with JavaScript expressions and validation logic to control notification eligibility, duplicate prevention, and logging.

## 4. Workflow Architecture

WF-06 consists of two Google Sheets triggers and three independent notification branches.

### 4.1 Trigger Configuration

| Trigger | Event | Purpose |
|---|---|---|
| Trigger - New Applications | New row added | Initiates application acknowledgement |
| Trigger - HR Status Updates | Row updated | Detects HR decisions for shortlisting and rejection |

The workflow uses only these two triggers. Interview-related notifications are handled separately in WF-07 — Interview Scheduling.

### 4.2 Notification Branches

**Branch A — Application Acknowledgement**

Triggered when a new candidate application is recorded. The candidate's information, email format, and consent are validated before the acknowledgement email is sent.

**Branch B — Shortlisting Notification**

Triggered by an HR status update. The workflow checks whether the application status is Shortlisted, validates candidate information, checks for a previous successful notification, and sends the shortlisting email if eligible.

**Branch C — Rejection Notification**

Triggered by an HR status update. The workflow checks whether the application status is Rejected, validates candidate information, checks for a previous successful notification, and sends the rejection email if eligible.

### 4.3 Human-Controlled Recruitment Decisions

Shortlisting and rejection notifications are based on HR-controlled application status changes.

AI-generated candidate match scores are advisory and do not independently trigger rejection emails. This approach keeps recruitment decisions under human control.

## 5. Notification Flow

### 5.1 Application Acknowledgement

1. Detect a newly added application in Google Sheets.
2. Validate Application ID, candidate name, email, and consent.
3. Check Notification Logs for an existing successful acknowledgement.
4. Skip sending if an acknowledgement has already been logged as Sent.
5. Send the acknowledgement email using Gmail if eligible.
6. Record the successful delivery attempt or failure.
7. Record invalid application data separately.

### 5.2 Shortlisting Notification

1. Detect an HR status update.
2. Check whether Application Status equals Shortlisted.
3. Validate Application ID, candidate name, email, consent, and current status.
4. Check Notification Logs for an existing successful shortlisting notification.
5. Skip duplicates.
6. Send the shortlisting email using Gmail.
7. Record success, failure, or invalid data in Google Sheets.

### 5.3 Rejection Notification

1. Detect an HR status update.
2. Check whether Application Status equals Rejected.
3. Validate Application ID, candidate name, email, consent, and current status.
4. Check Notification Logs for an existing successful rejection notification.
5. Skip duplicates.
6. Send the rejection email using Gmail.
7. Record success, failure, or invalid data in Google Sheets.

### 5.4 Duplicate Prevention

Before sending an email, the workflow checks the Notification Logs sheet using the Application ID, Notification Type, and Sent Status.

If an existing record with Sent status is found, the workflow skips the email to prevent duplicate successful notifications for the same application and notification type.

## 6. Notification Logging

All notification activities are recorded in the Notification Logs worksheet of the AI-HR-Recruitment-Database Google Sheets document.

### 6.1 Log Categories

| Status | Description |
|---|---|
| Sent | Gmail successfully completed the send operation |
| Failed | Gmail returned an error during sending |
| Invalid Data | Candidate information failed validation |

### 6.2 Notification Log Schema

| Column | Field Name | Description |
|---|---|---|
| A | Notification ID | Unique identifier for the notification record |
| B | Application ID | Reference to the candidate application |
| C | Notification Type | Acknowledgement, Shortlisting, or Rejection |
| D | Recipient Email | Candidate's email address |
| E | Sent Status | Sent, Failed, or Invalid Data |
| F | Sent At | Notification log timestamp |
| G | Workflow Name | Name of the automation workflow |
| H | Error Details | Validation errors or Gmail failure details |

### 6.3 Error Handling

Gmail sending nodes are configured with error handling to allow unsuccessful sending operations to be routed to dedicated failure-logging nodes.

Invalid candidate information is logged separately and is not passed to Gmail.

The logging mechanism provides traceability of notification attempts and supports future troubleshooting and monitoring.

## 7. Email Templates

WF-06 uses standardized email templates to maintain professional, consistent, and clear communication with candidates.

### 7.1 Application Acknowledgement

**Subject:** Application Received – Confirmation

Confirms that the candidate's application and resume have been received and that the recruitment team will review the application.

### 7.2 Shortlisting Notification

**Subject:** Application Update – Shortlisted

Informs the candidate that their application has been shortlisted for further consideration and that additional details regarding the next stage will be shared separately.

### 7.3 Rejection Notification

**Subject:** Application Update – Recruitment Decision

Politely informs the candidate that the recruitment team will not proceed with their application at this time. The message avoids unnecessary personal criticism or detailed evaluation.

### 7.4 Email Communication Principles

- Maintain a professional and respectful tone.
- Avoid exposing internal AI match scores or evaluation details.
- Avoid promising interviews or employment.
- Use clear, concise, candidate-friendly language.
- Send notifications only when the corresponding application status and validation requirements are satisfied.

## 8. Validation Rules

Candidate data is validated before email notification to reduce errors and prevent inappropriate or incomplete communication.

### 8.1 Required Fields

| Field | Validation Requirement |
|---|---|
| Application ID | Must not be empty |
| Candidate Name | Must not be empty |
| Email | Must not be empty and must match basic email format |
| Consent Status | Must be confirmed as Yes |
| Application Status | Must match the notification's required status |

### 8.2 Notification-Specific Status Requirements

| Notification | Required Application Status |
|---|---|
| Application Acknowledgement | Application is newly recorded and eligible for acknowledgement |
| Shortlisting Notification | Shortlisted |
| Rejection Notification | Rejected |

### 8.3 Validation Outcomes

When all required validation checks pass, the candidate proceeds to duplicate-notification checking.

If validation fails:

- The notification is not sent.
- The workflow routes the data to an invalid-data logging node.
- Validation errors are recorded in the Error Details field of Notification Logs.

### 8.4 Consent Protection

The workflow checks that the candidate's consent status is Yes before sending notifications. Applications without confirmed consent are not eligible for email communication through these notification branches.

## 9. Duplicate Prevention

WF-06 implements duplicate-notification checks to prevent repeatedly sending the same notification to a candidate for a given application.

### 9.1 Duplicate Check Criteria

The workflow searches the Notification Logs sheet using three criteria:

- Application ID
- Notification Type
- Sent Status = Sent

### 9.2 Duplicate Handling

If a previous successful notification exists for the same application and notification type, the workflow skips sending another email.

If no previous successful notification exists, the candidate proceeds to the Gmail sending node.

### 9.3 Notification Identification

Notification IDs use prefixes to identify notification categories:

| Prefix | Notification Category |
|---|---|
| ACK- | Application Acknowledgement |
| SHORT- | Shortlisting Notification |
| REJ- | Rejection Notification |
| INVALID- | Invalid Acknowledgement Data |
| INVALID-SHORT- | Invalid Shortlisting Data |
| INVALID-REJ- | Invalid Rejection Data |

These identifiers assist in organizing, distinguishing, and tracking notification records.

## 10. Error Handling & Reliability

The workflow incorporates separate error and validation handling mechanisms to improve traceability and reduce silent failures.

### 10.1 Gmail Errors

Gmail nodes use error handling to continue through the designated error output when sending fails. The failure path records the error message and notification details in Google Sheets.

### 10.2 Invalid Candidate Data

Validation failures are directed to dedicated invalid-data logging nodes. The workflow records validation messages rather than attempting to send an email with incomplete or invalid information.

### 10.3 Notification Logging

Every notification branch has separate logging paths for:

- Successful Gmail send operations
- Gmail sending failures
- Invalid candidate data

### 10.4 Reliability Considerations

The workflow provides structured validation, duplicate checking, and logging. However, notification delivery still depends on Gmail availability, valid recipient addresses, Google account permissions, and the reliability of Google Sheets operations.

## 11. Limitations & Future Improvements

### 11.1 Current Limitations

- The workflow has not yet undergone controlled end-to-end testing.
- Gmail send-operation success does not guarantee that an email reached the candidate's inbox.
- Google Sheets-based duplicate prevention may be vulnerable to concurrent executions checking the same record before either logs a successful notification.
- Google Sheets and Gmail service availability can affect workflow reliability.
- Notification retries and delivery monitoring are not yet implemented.
- Interview scheduling and interview-related communication are handled separately by WF-07.

### 11.2 Future Improvements

- Implement controlled end-to-end testing using test candidate records.
- Add retry strategies for temporary Gmail or Google Sheets failures.
- Introduce stronger duplicate protection if concurrent executions become possible.
- Add notification monitoring and failure alerts for HR.
- Improve notification history reporting and dashboard visibility.
- Integrate notification outcomes into the HR dashboard through WF-08.