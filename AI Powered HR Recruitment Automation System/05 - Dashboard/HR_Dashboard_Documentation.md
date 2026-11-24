# HR Recruitment Analytics Dashboard

## 1. Overview

The HR Recruitment Analytics Dashboard is developed using Google Looker Studio to provide HR personnel with a centralized visual overview of recruitment activities.

The dashboard uses the existing `AI-HR-Recruitment-Database` Google Sheets file as its data source.

It is designed to support recruitment monitoring and analysis without modifying candidate records or making automatic hiring decisions.

## 2. Dashboard Objectives

- Monitor the total number of candidate applications.
- Review the average AI-generated candidate match score.
- Track the total number of scheduled interviews.
- Visualize HR recruitment decisions.
- Compare application volumes across different jobs.
- Monitor automated notification delivery statuses.

## 3. Data Sources

The dashboard uses the following worksheets:

| Worksheet | Dashboard usage |
|---|---|
| Applications | Application count, AI match score, HR decisions, and job-wise application analysis |
| Interviews | Total interview count |
| Notification Logs | Notification reliability analysis |

## 4. Dashboard KPIs

### 4.1 Total Applications

- Chart type: Scorecard
- Data source: Applications
- Metric: Application ID
- Aggregation: Count Distinct

Displays the total number of unique candidate applications.

### 4.2 Average AI Match Score

- Chart type: Gauge
- Data source: Applications
- Metric: AI Match Score
- Aggregation: Average

Displays the average AI-generated candidate matching score.

### 4.3 Total Interviews

- Chart type: Scorecard
- Data source: Interviews
- Metric: Interview ID
- Aggregation: Count Distinct

Displays the total number of unique interview records.

### 4.4 HR Decisions

- Chart type: Donut chart
- Data source: Applications
- Dimension: HR Decision
- Metric: Application ID
- Aggregation: Count Distinct

Visualizes the distribution of recorded HR decisions.

### 4.5 Job-wise Applications

- Chart type: Horizontal bar chart
- Data source: Applications
- Dimension: Job ID
- Metric: Application ID
- Aggregation: Count Distinct
- Sorting: Descending by application count

Compares application volumes across different job identifiers.

### 4.6 Notification Reliability

- Chart type: Stacked bar chart
- Data source: Notification Logs
- Dimension: Sent Status
- Metric: Notification ID
- Aggregation: Count Distinct

Displays notification records grouped by their recorded delivery status.

## 5. Dashboard Layout

The dashboard follows a two-row arrangement:

**Top row**
- Total Applications
- Average AI Match Score
- Job-wise Applications

**Bottom row**
- Total Interviews
- HR Decisions
- Notification Reliability

The layout was retained after implementation to provide a compact overview of the main recruitment indicators.

## 6. Data Privacy and Access Control

- Dashboard access is restricted to authorized users.
- Candidate contact details are not displayed on the dashboard overview.
- Resume links and resume files are not displayed.
- Aggregate recruitment analytics are preferred over exposing individual candidate records.
- AI match scores are advisory information and do not replace human HR decisions.

## 7. Testing and Validation

The completed dashboard was saved and refreshed successfully.

All six retained charts were verified to display correctly.

The final review confirmed restricted report access and no exposure of candidate contact details or resumes.

## 8. Implementation Status

The HR Recruitment Analytics Dashboard has been implemented and verified.

The dashboard uses the existing Google Sheets database and does not require modifications to the recruitment database or n8n workflows.