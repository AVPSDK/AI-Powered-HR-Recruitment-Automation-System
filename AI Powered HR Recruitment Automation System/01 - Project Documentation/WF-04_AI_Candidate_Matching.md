# WF-04 – AI Candidate Matching

## 1. Project Overview

The **AI Candidate Matching** workflow is the fourth workflow in the AI-Powered HR Recruitment Automation System.

Its purpose is to evaluate candidate profiles against job requirements using AI-powered analysis and a weighted scoring mechanism. It retrieves candidate information from the Applications worksheet and job requirements from the Jobs worksheet, validates the available information, and generates an evidence-based candidate match assessment.

The workflow uses Groq AI to assess relevant skills, experience, education, preferred skills, and project evidence. It calculates an overall match score and generates an explanation that HR personnel can review.

The workflow also includes a manual review mechanism for incomplete, invalid, or insufficient candidate information.

AI-generated scores are advisory only. All recruitment decisions remain under human control.

## 2. Objectives

The main objectives of WF-04 are:

- Automatically retrieve candidate application information from Google Sheets.
- Retrieve the corresponding job requirements using the Job ID.
- Validate candidate and job information before AI evaluation.
- Compare candidate qualifications and experience with job requirements.
- Generate structured AI evaluation results using Groq API.
- Calculate a weighted candidate match score.
- Generate an understandable explanation of the matching results.
- Route applications with insufficient information or invalid evaluations for manual review.
- Update the existing Applications worksheet without modifying unrelated candidate information.
- Support processing multiple candidate applications independently.

## 3. Workflow Architecture

The workflow follows a structured sequence of data retrieval, validation, AI evaluation, scoring, and Google Sheets integration.

### 3.1 Main Workflow Components

1. **Trigger — New Candidate Application:** Detects updated candidate application records in Google Sheets.
2. **Get Candidate Application:** Retrieves the corresponding candidate record using the Application ID.
3. **Get Job Requirements:** Retrieves the associated job information using the Job ID.
4. **Validate Matching Data:** Checks candidate information, job requirements, Job ID consistency, and job status.
5. **Prepare Matching Input:** Prepares the candidate and job information required for AI evaluation.
6. **Check AI Matching Eligibility:** Determines whether sufficient validated information is available for AI evaluation.
7. **AI — Evaluate Candidate Match:** Sends the prepared information to Groq AI for evidence-based assessment.
8. **Parse AI Matching Result:** Parses and validates the AI-generated JSON response.
9. **Check AI Evaluation Validity:** Determines whether the AI evaluation is valid and sufficiently supported by evidence.
10. **Calculate Final Match Score:** Calculates the weighted candidate match score.
11. **Check Final Score Validity:** Verifies whether the calculated score is valid.
12. **Format Matching Explanation:** Generates a structured explanation of the candidate's matching results.
13. **Prepare Manual Review Result:** Generates a manual review explanation when automated evaluation cannot produce a reliable score.
14. **Update Application Record:** Updates the candidate's AI Match Score and Matching Explanation in Google Sheets.

### 3.2 Workflow Architecture Diagram

The workflow contains two main processing paths:

- **AI Evaluation Path:** Processes candidates with sufficient information and generates a weighted match score.
- **Manual Review Path:** Handles missing information, invalid AI responses, insufficient evidence, or scoring errors.

Both paths connect to the final Google Sheets update node.

## 4. Technology Stack

| Technology | Purpose |
|---|---|
| n8n | Workflow automation and orchestration |
| Google Sheets | Candidate and job information storage |
| Groq API | AI-powered candidate evaluation |
| GPT-OSS-20B | AI model used for structured matching analysis |
| JavaScript | Data validation, transformation, scoring, and explanation generation |
| Google Sheets Trigger | Detecting updated candidate records |

## 5. Google Sheets Data Sources

WF-04 uses the existing Google Sheets database created for the recruitment automation system.

### 5.1 Applications Worksheet

The Applications worksheet provides candidate information.

Relevant fields include:

- Application ID
- Candidate Name
- Job ID
- Education
- Parsed Skills
- Experience
- AI Match Score
- Matching Explanation
- Candidate Category
- Application Status

Candidate profile information such as Education, Parsed Skills, and Experience is populated by WF-02.

### 5.2 Jobs Worksheet

The Jobs worksheet provides job requirements.

Relevant fields include:

- Job ID
- Required Skills
- Preferred Skills
- Experience Required
- Education Required
- Job Description
- Job Status
- Created Date

Job requirement information is populated by WF-03.

### 5.3 Data Relationship

The Job ID connects candidate applications with their corresponding job requirements.

WF-04 uses the Application ID to retrieve the candidate record and the Job ID to retrieve the associated job record.

The workflow verifies that both records refer to the same Job ID before proceeding.

The existing spreadsheet schema is preserved. No additional columns are required for WF-04.

## 6. Candidate and Job Data Validation

Before AI evaluation, the workflow checks whether the retrieved candidate and job information is suitable for matching.

### 6.1 Candidate Validation

The following candidate fields are checked:

- Application ID
- Job ID
- Parsed Skills
- Education
- Experience

Missing candidate profile information prevents AI evaluation and routes the application for manual review.

### 6.2 Job Validation

The following job fields are checked:

- Job ID
- Required Skills
- Experience Required
- Education Required
- Job Description

The workflow also verifies that the candidate's Job ID matches the retrieved job's Job ID.

### 6.3 Job Status Validation

Only jobs with the status **Open** are eligible for matching.

Applications associated with Closed or On Hold jobs are routed for manual review rather than being evaluated automatically.

## 7. AI Candidate Evaluation

### 7.1 AI Integration

The workflow uses the Groq API with the `openai/gpt-oss-20b` model.

The AI request is sent through an HTTP Request node using the Chat Completions API.

The request contains:

- Candidate skills
- Candidate education
- Candidate experience
- Required job skills
- Preferred job skills
- Required experience
- Required education
- Job description

Personal identifiers such as candidate name, email address, phone number, and resume Drive URL are excluded from the AI matching input.

### 7.2 Evaluation Criteria

The AI evaluates five criteria:

1. Required Skills
2. Relevant Experience
3. Education
4. Preferred Skills
5. Projects

Each applicable criterion receives a score between 0 and 100, supported by an explanation of the available evidence.

### 7.3 Evidence-Based Evaluation

The AI is instructed to:

- Evaluate only information supported by the candidate profile.
- Avoid inventing skills, qualifications, experience, or projects.
- Identify missing or unverified evidence.
- Provide reasoning for each assessment.
- Avoid using protected characteristics in candidate evaluation.
- Avoid making automatic hiring or rejection decisions.

### 7.4 Project Evidence Handling

There is no separate Projects column in the existing Applications worksheet.

Therefore, the AI evaluates identifiable project evidence contained within the candidate's Experience field.

If project evidence is available, it can be assessed.

If no project evidence is available and the job description does not explicitly require it, the Projects criterion is marked as Not Applicable.

If the job description explicitly requires project evidence but the candidate information is insufficient, the criterion is marked as Insufficient Evidence and routed for manual review.

The absence of a separate Projects field is not automatically treated as a negative result.

### 7.5 Preferred Skills Handling

If a job does not specify preferred skills, the Preferred Skills criterion is marked as Not Applicable rather than receiving an artificial score.

## 8. Weighted Match Score Calculation

After receiving and validating the AI evaluation, the workflow calculates the final candidate match score.

### 8.1 Scoring Weights

The following weights are used:

| Evaluation Criterion | Weight |
|---|---:|
| Required Skills | 40% |
| Relevant Experience | 20% |
| Education | 15% |
| Preferred Skills | 15% |
| Projects | 10% |
| **Total** | **100%** |

These weights are configurable project-level heuristics and have not been scientifically validated.

### 8.2 Score Calculation

The final score is calculated using the weighted average of applicable criteria.

\[
\text{Final Score} =
\frac{\sum(\text{Criterion Score}\times\text{Criterion Weight})}
{\sum(\text{Applicable Criterion Weights})}
\]

Criteria marked as Not Applicable are excluded from the calculation, and the remaining applicable weights are normalized.

A criterion marked as Insufficient Evidence does not receive an assumed score. Instead, the application is routed for manual review.

### 8.3 Score Validation

The workflow verifies that:

- Every required evaluation criterion exists.
- Each assessed score is numeric.
- Scores are within the range of 0–100.
- Not Applicable criteria have null scores.
- No criterion has insufficient evidence.
- At least one applicable criterion is available.

If validation fails, the workflow does not produce a final match score.

## 9. Matching Explanation Generation

The Format Matching Explanation node generates a structured explanation for each successfully evaluated candidate.

The explanation contains:

- Final Match Score
- Criterion Breakdown
- Criterion Weights
- Individual Criterion Scores
- Evidence Assessment
- Evaluation Uncertainties
- AI Summary
- Human Review Disclaimer

The explanation helps HR personnel understand the evidence behind the score rather than relying only on a numerical result.

The generated explanation explicitly states that the score is advisory and does not represent a hiring decision.

## 10. Manual Review Handling

The workflow includes a separate manual review path for cases where reliable automated evaluation is not possible.

### 10.1 Manual Review Conditions

An application can be routed for manual review when:

- Candidate profile information is missing.
- Candidate or job information is invalid.
- Candidate and job IDs do not match.
- The associated job is not Open.
- The AI response is missing or invalid.
- The AI response does not satisfy the required JSON structure.
- The evaluation identifies insufficient evidence.
- The final score calculation encounters an error.

### 10.2 Manual Review Output

The Prepare Manual Review Result node generates:

- Candidate Application ID
- Blank AI Match Score
- Manual Review Explanation
- Matching Outcome

The explanation identifies the reason for manual review and instructs HR to verify the application and job information.

### 10.3 Human Decision-Making

The workflow does not automatically reject candidates or make final recruitment decisions.

HR personnel remain responsible for independently reviewing candidate applications and deciding whether to shortlist, interview, select, or reject candidates.

## 11. Google Sheets Integration

The final Update Application Record node updates the existing Applications worksheet.

Only the following fields are modified:

| Field | Updated Value |
|---|---|
| Application ID | Existing candidate Application ID |
| AI Match Score | Calculated score or blank value for manual review |
| Matching Explanation | AI assessment or manual review explanation |

The Application ID is used as the matching column.

The workflow does not modify:

- Candidate Name
- Job ID
- Education
- Parsed Skills
- Experience
- Resume Drive URL
- Candidate Category
- Application Status

Candidate Category is reserved for WF-05, while Application Status remains unchanged.

## 12. Multi-Candidate Processing

WF-04 supports processing multiple candidate applications in the same execution.

To achieve this, the six Code nodes were configured to use **Run Once for Each Item**:

1. Validate Matching Data
2. Prepare Matching Input
3. Prepare Manual Review Result
4. Parse AI Matching Result
5. Calculate Final Match Score
6. Format Matching Explanation

Each candidate is processed independently using its corresponding application and job information.

This prevents candidate data from being unintentionally mixed between different applications.

Both successful AI evaluation results and manual review results are passed to the final Google Sheets update node.

## 13. Testing and Verification

WF-04 was tested using dedicated dummy candidate records and a dummy job record.

### 13.1 Test Job

A dedicated test job was created with:

- Job ID: `TEST-WF04-001`
- Job Title: Junior Full Stack Developer — Test
- Department: Engineering
- Job Status: Open
- Required Skills: JavaScript, React, Node.js, HTML, CSS
- Preferred Skills: TypeScript, Next.js, Git
- Experience Required: 0–1 years
- Education Required: B.Tech Computer Engineering or equivalent

### 13.2 Test Candidate Records

Two dummy candidate applications were created:

- `TEST-APP-WF04-001`
- `TEST-APP-WF04-002`

Both applications were associated with the same test job but contained different education, skills, and experience information.

### 13.3 Functional Testing

The following tests were performed:

| Test | Result |
|---|---|
| Candidate application retrieval | Successful |
| Job requirement retrieval | Successful |
| Candidate and job ID validation | Successful |
| Candidate data readiness validation | Successful |
| AI eligibility routing | Successful |
| Groq AI request execution | Successful during initial testing |
| AI response parsing and validation | Successful |
| Weighted score calculation | Successful |
| Matching explanation generation | Successful |
| Manual review routing | Successful |
| Google Sheets update | Successful |
| Candidate data separation | Successful |
| Multi-candidate processing | Successful |
| Existing candidate fields preservation | Successful |

### 13.4 Manual Review Test

The Experience field of the first dummy candidate was temporarily cleared.

The workflow correctly routed the application to the manual review path.

The AI Match Score remained blank, and the Matching Explanation contained the manual review reason.

The original Experience value was then restored, and the normal AI evaluation path was successfully tested again.

### 13.5 Multi-Candidate Test

Both dummy applications were processed in the same execution.

The workflow successfully processed the two records through their respective evaluation paths.

Both corresponding Google Sheets records were updated correctly, and their explanations remained associated with the correct Application IDs.

Candidate Category and Application Status remained unchanged.

### 13.6 Final Configuration Verification

The temporary test filter was removed after multi-candidate testing.

The Google Sheets Trigger was reconnected directly to Get Candidate Application.

The final workflow configuration was inspected and saved.

The workflow remains inactive.

## 14. Error Handling and Limitations

### 14.1 Data Validation Errors

Missing or invalid candidate and job information is detected before AI evaluation.

Such applications are routed to manual review.

### 14.2 AI Response Errors

The AI response parser validates the returned JSON structure, evidence statuses, scores, reasoning, and required arrays.

Invalid responses are not used to calculate a final score.

### 14.3 Score Calculation Errors

The score calculation node checks the validity of all applicable criterion scores and prevents invalid results from being written as final scores.

### 14.4 Groq API Rate Limiting

During multi-candidate testing, the Groq API returned an HTTP 429 — Too Many Requests error because the workflow exceeded the model's Tokens Per Minute (TPM) limit.

The configured limit was 8,000 TPM. The error indicated that 7,404 tokens had already been used and the next request required 2,303 additional tokens.

To address this issue, request batching was configured in the AI — Evaluate Candidate Match HTTP Request node.

Final configuration:

- Items per Batch: 1
- Batch Interval: 15,000 milliseconds
- Retry On Fail: Enabled
- Maximum Tries: 3
- Wait Between Tries: 5,000 milliseconds

The workflow was subsequently tested with multiple dummy candidate applications, and the execution completed successfully without encountering the previous rate-limit error.

The 15-second batching interval is therefore retained as the tested configuration. Rate limits may still vary with prompt size, candidate data, and API usage, so continued monitoring is recommended.

### 14.5 Current Operational Limitation

WF-04 is saved but remains inactive.

It must not be activated for production use until the Groq rate-limit issue is investigated, appropriate request pacing and retry behavior are verified, and final integration testing is completed.

## 15. Privacy, Security and Human Oversight

The workflow follows several privacy and responsible-AI practices:

- Personal identifiers are excluded from the AI matching request.
- Candidate information is retrieved from the existing recruitment database.
- AI evaluation is based on job-related candidate evidence.
- Protected characteristics are excluded from matching criteria.
- Missing evidence is not replaced with invented information.
- Unreliable evaluations are routed for manual review.
- AI scores are advisory and do not automatically determine hiring outcomes.
- Candidate Category and Application Status are not modified by WF-04.
- HR personnel retain responsibility for all final recruitment decisions.

The workflow should operate with appropriately restricted access to candidate information and secure API credential storage.

## 16. Conclusion

WF-04 – AI Candidate Matching was developed to automate the comparison of candidate profiles with job requirements using structured AI evaluation and weighted scoring.

The workflow integrates candidate information from WF-02 and job requirements from WF-03, validates the available data, evaluates candidates using Groq AI, calculates match scores, generates explanations, and updates the existing Google Sheets database.

It also includes evidence validation, manual review handling, and multi-candidate processing.

Functional testing with dummy records verified the main processing paths and confirmed that unrelated candidate information remains unchanged.

The workflow is currently saved in an inactive state. Further investigation and testing of the Groq API rate-limit issue are required before production activation.