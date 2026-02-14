# Requirements Document: Student Opportunity Navigator

## Introduction

The Student Opportunity Navigator is an MVP platform designed to help students discover government exams and scholarships they are eligible for. The system analyzes student profile inputs (age, state, education, income, category) to filter relevant opportunities, provides binary eligibility confirmation, simplifies complex guidelines using AI translation, and displays registration links and deadlines. The goal is to demonstrate a working proof-of-concept that eliminates confusion and improves access to verified academic and career opportunities for students from diverse communities.

## MVP Scope Limitations

This is a hackathon MVP with the following explicit limitations:

- **No authentication system**: Student profiles are stateless session inputs, not persisted accounts
- **No notification infrastructure**: No email, SMS, or push notifications for deadlines or new opportunities
- **No automated verification**: Opportunities are manually curated and marked as verified; no background jobs or API integrations with government portals
- **No background schedulers**: Deadline expiration is checked at query time, not via scheduled jobs
- **No audit trails**: No logging of verification history or data source tracking
- **No ranking algorithms**: Opportunities are displayed in simple chronological or alphabetical order
- **Binary eligibility only**: Students are either ELIGIBLE or NOT_ELIGIBLE; no partial matching or scoring

## System Architecture

The MVP uses a simplified three-tier architecture:

```
┌─────────────────┐
│  React Frontend │
│   (User Input)  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ FastAPI Backend │
│ (Eligibility    │
│  Filtering)     │
└────┬───────┬────┘
     │       │
     ▼       ▼
┌─────────┐ ┌──────────────┐
│Firebase │ │  Gemini API  │
│Firestore│ │ (Translation)│
└─────────┘ └──────────────┘
```

- **Frontend**: React application for profile input and opportunity display
- **Backend**: FastAPI service for eligibility logic and data retrieval
- **Database**: Firebase Firestore for storing opportunity data
- **AI Service**: Gemini API for multilingual guideline simplification

## Glossary

- **System**: The Student Opportunity Navigator MVP platform
- **Student**: A user who inputs profile information to discover opportunities (no account creation)
- **Profile**: A temporary collection of student attributes including age, state, education, income, and category
- **Opportunity**: A government exam or scholarship program stored in Firestore
- **Eligibility_Criteria**: The set of requirements that determine if a student qualifies for an opportunity
- **Guideline**: Official documentation describing an opportunity's requirements and application process
- **Registration_Link**: A verified URL where students can apply for an opportunity
- **Deadline**: The last date by which an application must be submitted
- **Category**: A classification such as General, OBC, SC, ST, EWS
- **Verification_Status**: A boolean flag indicating whether an opportunity has been manually verified

## Requirements

### Requirement 1: Student Profile Input

**User Story:** As a student, I want to input my profile information in a simple form, so that the system can show me relevant opportunities.

#### Acceptance Criteria

1.1 WHEN a student accesses the system, THE System SHALL display a profile input form with fields for age, state, education level, income, and category

1.2 WHEN a student submits the form, THE System SHALL validate that all required fields are filled

1.3 IF a required field is missing or invalid, THEN THE System SHALL display an error message and prevent submission

1.4 THE System SHALL support education levels: High School, Undergraduate, Postgraduate, Doctoral

1.5 THE System SHALL support categories: General, OBC, SC, ST, EWS

### Requirement 2: Eligibility Filtering

**User Story:** As a student, I want the system to automatically show me only the opportunities I'm eligible for, so that I don't waste time reviewing irrelevant options.

#### Acceptance Criteria

2.1 WHEN a student submits their profile, THE System SHALL retrieve all opportunities from Firestore

2.2 WHEN filtering opportunities, THE System SHALL evaluate age, state, education, income (for scholarships only), and category against each opportunity's eligibility criteria

2.3 WHEN all criteria are satisfied, THE System SHALL mark the opportunity as ELIGIBLE

2.4 WHEN any criterion is not satisfied, THE System SHALL mark the opportunity as NOT_ELIGIBLE

2.5 THE System SHALL display only ELIGIBLE opportunities to the student

2.6 WHEN no opportunities match the profile, THE System SHALL display a message: "No opportunities currently match your profile"

### Requirement 3: Deadline Management

**User Story:** As a student, I want to see only active opportunities with upcoming deadlines, so that I don't see expired listings.

#### Acceptance Criteria

3.1 WHEN filtering opportunities, THE System SHALL exclude any opportunity whose deadline has passed

3.2 WHEN displaying an opportunity, THE System SHALL show the application deadline date

3.3 WHEN a deadline is within 7 days, THE System SHALL display a visual indicator (e.g., "Deadline Soon" badge)

3.4 THE System SHALL compare deadlines against the current server date and time

### Requirement 4: Verification Status Display

**User Story:** As a student, I want to see which opportunities have been verified, so that I can trust the information.

#### Acceptance Criteria

4.1 WHEN displaying an opportunity, THE System SHALL show a verification badge if the opportunity's `is_verified` field is true

4.2 WHEN displaying an opportunity, THE System SHALL show the source organization name

4.3 THE System SHALL only display opportunities where `is_verified` is true

### Requirement 5: Multilingual Guideline Simplification

**User Story:** As a student from a diverse linguistic background, I want to read simplified opportunity guidelines in my preferred language, so that I can understand the requirements.

#### Acceptance Criteria

5.1 THE System SHALL support guideline translation into Hindi, Bengali, Tamil, Telugu, Marathi, Gujarati, Kannada, Malayalam, Punjabi, and Urdu

5.2 WHEN a student selects a language preference, THE System SHALL send the guideline text to Gemini API with a simplification and translation prompt

5.3 WHEN Gemini API returns translated content, THE System SHALL display it to the student

5.4 WHEN translation fails or times out, THE System SHALL display the original English guideline text

5.5 THE System SHALL allow students to switch languages and re-translate guidelines on demand

### Requirement 6: Registration Links and Opportunity Details

**User Story:** As a student, I want to access direct registration links and see key opportunity details, so that I can apply without searching multiple websites.

#### Acceptance Criteria

6.1 WHEN displaying an opportunity, THE System SHALL show the opportunity name, description, and eligibility criteria

6.2 WHEN displaying an opportunity, THE System SHALL provide a clickable registration link to the official application portal

6.3 WHEN displaying an opportunity, THE System SHALL indicate the opportunity type (Exam or Scholarship)

6.4 WHEN displaying a scholarship, THE System SHALL show the award amount if available

6.5 THE System SHALL display opportunities in a card or list format with clear visual separation

### Requirement 7: Basic Search and Filtering

**User Story:** As a student, I want to filter displayed opportunities by type, so that I can focus on exams or scholarships.

#### Acceptance Criteria

7.1 THE System SHALL allow students to filter results by opportunity type: All, Exams, Scholarships

7.2 WHEN a filter is applied, THE System SHALL update the displayed list to show only matching opportunities

7.3 THE System SHALL display the count of opportunities matching the current filter

### Requirement 8: Data Model Constraints

**User Story:** As a developer, I want clear data models for student profiles and opportunities, so that the system processes data consistently.

#### Acceptance Criteria

8.1 THE System SHALL define a StudentProfile model with fields: age (integer), state (string), education (enum), income (integer, optional), category (enum)

8.2 THE System SHALL define an Opportunity model with fields: id, name, type (Exam/Scholarship), description, eligibility_criteria (object), deadline (date), registration_link (URL), is_verified (boolean), source_organization (string)

8.3 THE System SHALL define an EligibilityCriteria model with fields: min_age (integer, optional), max_age (integer, optional), states (array of strings, optional), education_levels (array of enums, optional), max_income (integer, optional), categories (array of enums, optional)

8.4 WHEN evaluating eligibility, THE System SHALL treat missing criteria fields as "no restriction"
