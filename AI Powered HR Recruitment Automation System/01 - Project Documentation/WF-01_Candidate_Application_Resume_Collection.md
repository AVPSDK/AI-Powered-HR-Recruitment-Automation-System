# WF-01: Candidate Application & Resume Collection

## 1. Workflow Overview

**Workflow Name:** WF-01 - Candidate Application & Resume Collection

**Project:** AI-Powered HR Recruitment Automation System

**Purpose:**

This workflow automates the initial candidate application process. It collects candidate information through an online application form, validates required consent, checks for duplicate applications, uploads resumes to Google Drive, generates a unique Application ID, and stores application details in Google Sheets.

**Primary Objectives:**
- Collect candidate information through an online form.
- Receive and securely store uploaded resumes.
- Prevent duplicate applications for the same email address and job.
- Generate a unique Application ID for each valid application.
- Store application records in the central recruitment database.
- Ensure candidate consent before processing application data.

**Workflow Outcome:**

A valid, non-duplicate application is recorded in the Applications sheet, and its resume is uploaded to the designated Google Drive folder. Duplicate submissions are stopped and shown an appropriate response.

## 2. Technology Stack

| Technology | Purpose in WF-01 |
|---|---|
| n8n | Workflow automation and execution |
| n8n Form Trigger | Collects candidate information and resumes |
| Google Drive | Stores uploaded candidate resumes |
| Google Sheets | Stores application records |
| JavaScript | Generates unique Application IDs and normalizes candidate data |
| Google Drive API integration | Enables automated resume uploads |
| Google Sheets integration | Enables application lookup and database insertion |

### 2.1 Why These Technologies Were Selected

**n8n:** Provides a visual, node-based automation environment that connects different services and manages the application workflow.

**n8n Form Trigger:** Provides an application form without requiring a separate frontend application during the initial development stage.

**Google Drive:** Provides centralized storage for uploaded resumes, with access controlled through Google Drive permissions.

**Google Sheets:** Acts as the initial structured database for candidate applications and supports duplicate-record lookup.

**JavaScript:** Handles data normalization, consent normalization, and unique Application ID generation within the workflow.

### 2.2 Integration Summary

The workflow connects n8n with Google Drive and Google Sheets. Candidate information is received through the form, duplicate records are checked against the Applications sheet, resumes are uploaded to Drive, and valid application records are appended to the spreadsheet.

## 3. Workflow Logic and Node Description

### 3.1 Form Trigger

The Form Trigger collects candidate information through an online application form.

Information collected:
- Full Name
- Email Address
- Phone Number
- Position Applying For
- Highest Qualification
- Resume
- Data Processing Consent

The form validates required fields and prevents submission when mandatory consent is not provided.

### 3.2 Normalize Duplicate Check Fields

This Code node normalizes candidate information before duplicate checking.

Responsibilities:
- Converts the candidate's email address to lowercase.
- Removes unnecessary spaces from the email address.
- Extracts the Job ID from the selected job option.
- Preserves the original form data and uploaded resume.

This ensures consistent data during duplicate checking.

### 3.3 Check Existing Application

This Google Sheets node searches the Applications sheet for an existing application.

Matching conditions:
- Candidate Email
- Job ID

Both conditions must match.

This allows candidates to apply for different jobs while preventing duplicate applications for the same job using the same email address.

### 3.4 Restore Original Application Data

This Merge node combines:
- Original candidate information and resume binary data.
- Output from the existing application lookup.

It ensures that the original application data and uploaded resume remain available for subsequent processing.

### 3.5 Is Duplicate Application?

This IF node checks whether an existing Application ID was returned by the Google Sheets lookup.

**True branch:**
- Routes the candidate to Duplicate Application Response.
- Displays a message explaining that an application already exists.
- Stops further processing.

**False branch:**
- Allows the new application to continue.
- Proceeds to resume upload.

### 3.6 Duplicate Application Response

This Form Ending node displays an appropriate message when a duplicate application is detected.

The candidate is informed that another application for the same position using the same email address has already been received.

### 3.7 Upload File

This Google Drive node uploads the candidate's resume.

Destination folder:

AI-HR-Recruitment-System/01_Candidate_Resumes/New_Applications

The node returns file metadata, including the uploaded file's ID and web view link.

Resume access remains restricted to authorized personnel.

### 3.8 Merge

This Merge node combines:
- Original candidate information and application data.
- Google Drive resume upload metadata.

This produces a consolidated record for application ID generation and database insertion.

### 3.9 Generate Application ID

This JavaScript Code node generates a unique Application ID for each valid new application.

ID format:

APP-YYYYMMDD-XXXXXX

Responsibilities:
- Generates the application date component.
- Creates a unique random suffix.
- Normalizes the Job ID.
- Converts the consent response into Yes or No.
- Preserves candidate information, Drive metadata, and resume binary data.

### 3.10 Append Row in Google Sheets

This Google Sheets node inserts the new application into the Applications sheet.

Stored information includes:
- Application ID
- Candidate Name
- Email
- Phone
- Job ID
- Resume Link
- Applied Date
- Education
- Application Status
- Consent Status

Fields related to resume parsing, AI matching, candidate categorization, and HR decisions remain blank until their respective workflows process them.

### 3.11 Overall Workflow Outcome

WF-01 successfully collects candidate applications, validates required consent, prevents duplicate applications, uploads resumes to Google Drive, generates unique Application IDs, and stores application records in Google Sheets.

## 4. Application Data Flow

### 4.1 Candidate Information Collection

The candidate submits the application form containing personal contact information, job selection, educational qualification, resume, and consent.

### 4.2 Data Normalization

The workflow normalizes the candidate's email address and extracts the Job ID from the selected position.

### 4.3 Duplicate Application Verification

The workflow searches the Applications sheet using the normalized email address and Job ID.

- If a matching application exists, the duplicate response is displayed.
- If no matching application exists, processing continues.

### 4.4 Resume Upload

The candidate's resume is uploaded to the designated Google Drive folder.

The upload response provides the resume file ID and web view link.

### 4.5 Application ID Generation

A unique Application ID is generated using the current date and a random suffix.

The workflow also normalizes consent information and preserves the candidate's application data.

### 4.6 Database Insertion

The consolidated application information is inserted into the Applications sheet.

The new record receives the initial Application Status:

Submitted

### 4.7 Data Flow Summary

The application data flow follows these steps:

**Step 1: Candidate Submission**

The candidate submits personal information, job details, resume, and consent through the application form.

↓

**Step 2: Data Normalization**

The workflow normalizes the candidate's email address and extracts the Job ID.

↓

**Step 3: Duplicate Application Check**

Google Sheets is searched using the normalized email address and Job ID.

↓

**Step 4: Duplicate Decision**

- If a duplicate exists, the candidate receives the Duplicate Application Response.
- If no duplicate exists, the application proceeds to resume upload.

↓

**Step 5: Resume Upload**

The resume is uploaded to the designated Google Drive folder.

↓

**Step 6: Data Merging**

Original candidate information is combined with the uploaded resume metadata.

↓

**Step 7: Application ID Generation**

A unique Application ID is generated and application data is normalized.

↓

**Step 8: Database Insertion**

The completed application record is appended to the Applications sheet in Google Sheets.

**Final Outcome:**

A valid, non-duplicate application is stored successfully, and its resume is available in the designated Google Drive folder.

### 4.8 Expected Results

| Scenario | Expected Result |
|--------|--------|
| New application | Resume uploaded and application recorded |
| Duplicate application | Duplicate response displayed |
| Missing required information | Form validation prevents submission |
| Missing consent | Form submission blocked |
| Successful application | Unique Application ID generated |

## 5. Testing and Validation

### 5.1 Testing Overview

WF-01 was tested using synthetic candidate information to verify the candidate application process, resume upload, duplicate prevention, consent validation, and database insertion.

### 5.2 Test Cases and Results

| Test ID | Test Scenario | Expected Result | Actual Result | Status |
|---|---|---|---|---|
| TC-01 | Submit a new candidate application | Application should be processed successfully | Application recorded and resume uploaded to Google Drive | Passed |
| TC-02 | Submit a duplicate application using the same email and Job ID | Duplicate application should be detected and stopped | Duplicate response displayed; no additional resume uploaded or application row created | Passed |
| TC-03 | Submit the form without required consent | Form submission should be blocked | Form blocked submission because consent was not provided | Passed |
| TC-04 | Generate an Application ID | A unique Application ID should be generated | Application ID generated in the expected format | Passed |
| TC-05 | Verify candidate information after merging | Original candidate data and resume binary should be preserved | Candidate information and resume binary were available after merging | Passed |
| TC-06 | Verify Google Sheets insertion | Valid application details should be stored correctly | Application details were successfully appended to the Applications sheet | Passed |
| TC-07 | Verify Job ID normalization | The selected job should be stored using its Job ID | Job ID was normalized and stored as JOB001 | Passed |
| TC-08 | Verify consent normalization | Consent should be stored as Yes or No | Consent was normalized and stored as Yes | Passed |

### 5.3 Testing Summary

All eight test cases were successfully completed.

The testing confirmed that:

- New candidate applications are processed successfully.
- Duplicate applications for the same email address and job are prevented.
- Required consent is enforced by the application form.
- Unique Application IDs are generated.
- Candidate information and resume binary data are preserved during workflow execution.
- Resumes are uploaded to the designated Google Drive folder.
- Application records are stored correctly in Google Sheets.

### 5.4 Conclusion

The testing and validation of WF-01 confirmed that the candidate application and resume collection process operates as expected under the tested scenarios.

The workflow is ready to serve as the input stage for subsequent recruitment automation workflows.

## 6. Security, Privacy, and Data Handling

### 6.1 Candidate Consent

The application form requires candidates to provide consent before submitting their applications.

The consent statement informs candidates that their application information and resumes will be collected and processed for recruitment purposes and accessed by authorized recruitment personnel.

Applications cannot be submitted through the form without providing the required consent.

### 6.2 Resume Storage Security

Candidate resumes are stored in Google Drive under the designated folder:

AI-HR-Recruitment-System/01_Candidate_Resumes/New_Applications

The uploaded resumes are not made publicly accessible. Access should remain restricted to authorized recruitment personnel.

### 6.3 Application Data Protection

Candidate application information is stored in the Applications sheet of the AI-HR-Recruitment-Database spreadsheet.

Access to the spreadsheet should be limited to authorized personnel who require the information for recruitment activities.

### 6.4 Data Minimization

The workflow collects information required for the initial recruitment application process.

Only the necessary candidate information is transferred to the recruitment database and resume storage.

### 6.5 Test Data and Development Safety

Synthetic candidate information is used during development and workflow testing.

Real candidate information should not be used for testing without appropriate authorization and consent.

### 6.6 Credential Security

Google Drive and Google Sheets integrations use credentials configured within n8n.

Authentication credentials and API secrets should not be included in publicly shared workflow exports, screenshots, or GitHub repositories.

### 6.7 Future Security Improvements

The following improvements can be considered in future development:

- Define and implement a candidate data retention policy.
- Introduce controlled access based on HR roles and responsibilities.
- Implement a secure process for candidate data deletion or withdrawal requests.
- Maintain appropriate records of access and data-processing activities.
- Review credential permissions and periodically rotate secrets.

### 6.8 Security Summary

WF-01 incorporates required candidate consent, restricted resume storage, controlled spreadsheet access, and secure credential management practices.

Additional security controls and formal data-retention procedures can be introduced as the recruitment system develops.

## 7. Limitations and Future Enhancements

### 7.1 Current Limitations

Although WF-01 successfully automates candidate application collection and resume storage, it has some limitations:

1. **Manual Resume Analysis:** The workflow collects and stores resumes but does not currently extract skills, education, or work experience. This functionality will be implemented in WF-02.

2. **Basic Application Database:** Google Sheets is used as the initial database. It may become less suitable as application volume and system complexity increase.

3. **Basic Duplicate Detection:** Duplicate applications are identified using the candidate's normalized email address and Job ID. Applications submitted using different email addresses are not automatically identified as belonging to the same person.

4. **Limited Job Information:** The current application form uses a configured job option. Dynamic job listings and job availability validation can be enhanced in future development.

5. **No AI-Based Candidate Evaluation:** WF-01 does not evaluate candidate suitability. Resume analysis and AI-based matching will be handled by subsequent workflows.

6. **Development Environment:** The workflow is currently developed and tested in a local n8n environment. Production deployment, availability, monitoring, and backup arrangements require additional configuration.

### 7.2 Future Enhancements

The following improvements can extend the capabilities of WF-01:

- Introduce dynamic job selection using active job records.
- Implement stronger duplicate detection and application identity verification.
- Integrate a scalable database when application volume increases.
- Add improved error handling and notifications for failed uploads or database operations.
- Implement automated resume file validation and security scanning.
- Introduce configurable application retention and deletion procedures.
- Deploy n8n in a reliable production environment with appropriate monitoring and backups.

### 7.3 Integration with Subsequent Workflows

WF-01 acts as the initial data collection stage of the recruitment automation system.

Its output will be used by subsequent workflows:

| Workflow | Planned Responsibility |
|---|---|
| WF-02 | Resume Parsing and Data Extraction |
| WF-03 | Job Description Analysis |
| WF-04 | AI Candidate Matching |
| WF-05 | Candidate Categorization |
| WF-06 | Candidate Email Notifications |
| WF-07 | Interview Scheduling |
| WF-08 | Dashboard Data Integration |

### 7.4 Human-in-the-Loop Recruitment

The recruitment system is designed to support HR personnel rather than replace human hiring decisions.

AI-generated candidate insights and matching scores will serve as decision-support information. Authorized HR personnel will remain responsible for reviewing applications and making recruitment decisions.

### 7.5 Conclusion

WF-01 establishes the foundation for the AI-Powered HR Recruitment Automation System by automating candidate information collection, resume storage, duplicate application prevention, and initial database insertion.

The limitations identified in this section provide a roadmap for extending the system through subsequent workflows and future improvements.