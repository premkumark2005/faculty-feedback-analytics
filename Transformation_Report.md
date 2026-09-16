# Transformation Report
**Project:** Faculty Feedback & Course Rating Analysis Dashboard
**Scope:** transform four source CSVs into the 8-table (+Lab) target schema and expand to five academic years (2021-22 .. 2025-26).

## 1. Original dataset structure

| File | Rows | Cols | Key columns |
|---|---|---|---|
| students_part1_1000.csv | 1,000 | 12 | Student_ID (PK), Department, Year, Semester, Batch, CGPA |
| subjects_part2.csv | 30 | 5 | Subject_ID (PK), Semester, Credits, Faculty_ID (FK) |
| faculty_part2.csv | 40 | 7 | Faculty_ID (PK), Designation, Experience_Years |
| feedback_5000.csv | 5,000 | 13 | Feedback_ID (PK), Student_ID/Faculty_ID/Subject_ID (FK), 6 ratings, Comment, Sentiment, Date |

No missing values and no duplicate rows in any file. Ratings are already 1-5. Feedback dates run 2025-01-01 to 2025-12-31. Every feedback Student_ID, Faculty_ID and Subject_ID resolves to a master record, so referential integrity in the source is sound; the semantic problems are listed in the Data Quality Report.

## 2. Target schema
Department, Course, Faculty, Student (dimensions); FacultyFeedback, TheoryFeedback, LabFeedback, FeedbackCompletion (facts); Lab (new dimension, see section 12).

## 3. Source-to-target mapping

| Source table | Source column | Target table | Target column | Transformation | Original/Derived | Confidence |
|---|---|---|---|---|---|---|
| students | Student_ID | Student | S_ID | Rename | Original | High |
| students | Name | Student | Name | Copy | Original | High |
| students | Department | Student/Department | Dept_ID / Dept_Name | Normalised into a Department dimension | Original | High |
| students | Year | Student | Batch | FY/SY/TY -> cohort start year -> `YYYY-YYYY` | Derived | Medium |
| students | Batch | - | - | Dropped (inconsistent values) | - | High |
| students | Roll_No, Email, Gender, Division, CGPA, Attendance, Semester | - | - | Not in target schema; dropped | - | High |
| subjects | Subject_ID | Course | C_ID | SUB001 -> C001 | Original | High |
| subjects | Subject_Name | Course | Course_Name | Copy | Original | High |
| subjects | Semester | Course | Sem | Copy | Original | High |
| subjects | Subject_Name | Course | Type | Rule-based Theory/Lab classification | Derived | Medium |
| subjects | Subject_Name | Course | Dept_ID | Domain mapping to 4 departments | Derived | Medium |
| subjects | Faculty_ID | (teaching matrix) | FacultyFeedback.Faculty_ID | Course owner used as the base teacher per year | Derived | Medium |
| subjects | Credits | - | - | Not in target schema; dropped | - | High |
| faculty | Faculty_ID | Faculty | F_ID | Rename | Original | High |
| faculty | Faculty_Name | Faculty | Faculty_Name | Copy | Original | High |
| faculty | Designation | Faculty | Role | Professor -> Associate Professor; senior Professor per dept -> HOD | Derived | Medium |
| faculty | Experience_Years | (career model) | - | Joining year = 2026 - experience | Derived | Medium |
| faculty | Qualification, Email, Phone | - | - | Not in target schema; dropped | - | High |
| feedback | Feedback_ID | FacultyFeedback | Feedback_ID | Re-sequenced FB###### | Derived | High |
| feedback | Faculty_ID | FacultyFeedback | Faculty_ID | Re-mapped to teacher of record | Derived | Medium |
| feedback | Subject_ID | FacultyFeedback | Course_ID | SUB -> C | Original | High |
| feedback | Date | FacultyFeedback | Date | Kept unchanged | Original | High |
| feedback | Communication | FacultyFeedback | Communication | Copy | Original | High |
| feedback | Teaching | FacultyFeedback | Subject_Knowledge | Rename | Original | High |
| feedback | Interaction | FacultyFeedback | Interaction | Copy | Original | High |
| feedback | Notes | FacultyFeedback | Doubt_Resolution | Rename (notes/explanation quality) | Original | Medium |
| feedback | Overall | FacultyFeedback | Syllabus_Completion | Rename | Original | Low |
| feedback | Practical | FacultyFeedback | Practical_Example | Rename | Original | High |
| feedback | Student_ID | FeedbackCompletion | S_ID | Used to compute completion; not stored on the fact (target schema has no student key there) | Original | High |
| feedback | Comment, Sentiment | - | - | No target column; dropped | - | High |

## 4. Five-year expansion methodology
The source is a single-year snapshot, so four years had to be modelled rather than copied. The unit of expansion is a **feedback event** = (student, course, academic year). Events come from two places:
1. **Original events (5,000).** Every source feedback row is kept as an event. Its date places it in 2024-25 (Jan-Jun 2025) or 2025-26 (Jul-Dec 2025); its six ratings are carried over unchanged.
2. **Synthetic events.** For every student active in a year, a set of up to 6 courses from their two semesters is treated as the feedback drive for that year; whether they submit is drawn from a year-level engagement probability plus a per-student tendency.

Final volume: **26,272 FacultyFeedback rows** (4,686 / 4,834 / 5,088 / 6,017 / 5,647). That is roughly 5x the source, which is what ~1,000 active students x ~5 rated courses per year produces; it sits inside the 20,000-30,000 range you specified and was not fixed by hand. Each event also produces exactly one course-level row: TheoryFeedback (17,954) if the course is Theory, LabFeedback (8,318) if it is a Lab.

## 5. Date generation
Original dates are untouched. Synthetic dates fall in end-semester feedback windows: odd semesters 10 Nov - 20 Dec of the first calendar year, even semesters 5 Apr - 20 May of the second. Nothing is generated during May-June study break or the summer vacation. Every date was checked to fall inside its own academic year (1 July - 30 June).

## 6. Student cohort methodology
A 3-year UG programme is assumed (FY/SY/TY, semesters 1-6, exactly as the source encodes it). The 1,000 original students are placed in the cohorts implied by their `Year` as of 2025-26: TY -> 2023-26, SY -> 2024-27, FY -> 2025-28. Four synthetic cohorts of 330 students each (2019-22, 2020-23, 2021-24, 2022-25) cover the earlier years, giving 2,320 students and ~1,000 active in each year. A student only appears in years 1-3 of their own batch, so nobody rates a course before joining or after graduating.

## 7. Faculty methodology
All 40 faculty are carried across the period. Joining year = 2026 - Experience_Years, so the least experienced staff correctly have no records in the earliest years. Roles are fixed for the whole period (no promotions invented): one HOD per department, `Professor` re-labelled `Associate Professor` because the target schema allows only three roles. Teaching is assigned per course-year, starting from the course owner in `subjects.csv`, with a ~12% chance of reassignment within the same department each year - enough to make faculty trend analysis interesting without churning the whole timetable.

## 8. Course methodology
All 30 courses keep their ID, name, semester and type for the whole period. Type never changes between years. Two courses (Distributed Systems, Data Warehousing) are treated as discontinued after 2022-23, and four (Generative AI, Reinforcement Learning, Data Ethics, IoT Analytics) as introduced in 2024-25 under regulation R2024 - both are documented assumptions, not evidence from the source.

## 9. Rating generation
Synthetic ratings are drawn from a normal distribution, rounded and clipped to 1-5. The mean is a base of 3.85 plus: a year effect (-0.30, -0.10, +0.18, +0.05, +0.22), a per-parameter offset (e.g. Subject_Knowledge runs higher, Practical_Example and InternetConnectivity lower), and stable per-faculty, per-course and per-student effects. Those stable effects are what make some instructors and courses consistently strong or weak, which is the point of the dashboard. The year effect deliberately dips in 2024-25 rather than rising every year. Course_Difficulty is modelled separately around 3.1 and is not tied to satisfaction, so a hard course can still be well rated.

## 10. Feedback completion
For each active student-year, Completion = TRUE when the student rated at least 75% of their enrolled courses. It is computed from the generated events, not assigned: 80.7% / 84.0% / 87.6% / 85.4% / 93.3%.

## 11. Synthetic columns
Fully synthetic: Course.Regulation, Course.Type, Course.Dept_ID, Faculty.Dept_ID, the whole Lab table, all TheoryFeedback and LabFeedback ratings, FeedbackCompletion.Completion, and all feedback rows outside the 5,000 originals. Partly original: Faculty.Role, Student.Batch, FacultyFeedback ratings. The `Synthetic_or_Original` column in Data_Dictionary.csv flags every field.

## 12. Why a Lab master table was created
The target LabFeedback table keys on `LabID`, but no Lab table was defined and no source column resembles one. Inventing lab codes inside the fact table would leave `LabID` as an orphan key and make "which lab" unjoinable in Power BI. So `Lab.csv` was created with one lab per Lab-type course (9 rows: LabID, LabName, Course_ID, Dept_ID). This keeps LabFeedback reachable from Course and Department through a clean chain and does not invent any relationship the schema did not already imply.

## 13. Assumptions
1. Three-year UG programme, two semesters per year, feedback collected at end of semester.
2. Original feedback dates are trustworthy and define the 2024-25 / 2025-26 split.
3. `feedback.Overall` best fits `Syllabus_Completion` (lowest-confidence mapping in the whole exercise - the source has no syllabus question).
4. `feedback.Notes` (notes/explanation quality) maps to `Doubt_Resolution`.
5. Courses are offered by four departments in a service-teaching model; all students belong to Data Science & AI, as the source says.
6. Faculty experience implies joining year and nobody left during the period.
7. Course discontinuation/introduction as listed in section 8.

## 14. Limitations
- 80% of the feedback volume is synthetic; only the 5,000 original faculty ratings are real. Absolute rating levels in 2021-22 to 2023-24 reflect the generating model, not observed behaviour, so treat year-over-year movement as illustrative.
- Comment and Sentiment have nowhere to live in the target schema and are lost; add a column to FacultyFeedback if you want text analytics.
- TheoryFeedback and LabFeedback have no student key, so course-feedback cannot be tied back to an individual student.
- Three of the four departments are constructs; department comparisons measure the mapping as much as anything real.
- Student names in synthetic cohorts are placeholders.

## 15. Data quality results
All 32 automated checks pass - unique primary keys, no orphan foreign keys, all ratings 1-5, all dates inside the five-year window, Theory/Lab feedback routed by Course.Type, roles and Completion restricted to allowed values, and no student or faculty active outside their valid period. Full detail in Data_Quality_Report.md.


---

# Addendum - v2: 10-department restructure

The v1 dataset used four departments inferred from the source files. v2 replaces that structure with the ten institutional departments (CSE, ECE, EEE, AIML, AIDS, CSD, MECH, IT, EIE, CIVIL) and propagates the change. **No feedback record was regenerated, added or deleted**: all 26,272 FacultyFeedback rows, 17,954 TheoryFeedback rows and 8,318 LabFeedback rows keep their v1 IDs, dates and ratings, so every year-over-year trend is unchanged.

## What changed

1. **Department.csv** - rewritten to exactly 10 rows, D01..D10, in the order you listed. Old IDs D01-D04 are reused but now denote CSE/ECE/EEE/AIML, so any v1 report must be re-pointed at this file.
2. **Course.csv** - 30 -> 91 courses. The original 30 keep their C_IDs and names and were re-homed to the department that actually owns them (Data Structures -> CSE, Machine Learning -> AIML, SQL -> IT, IoT Analytics -> ECE, and so on). 61 new courses (C031-C091) were written for the departments the source had no coverage for, with realistic names, semesters 1-8 and Theory/Lab types (laboratory courses named "... Laboratory").
3. **Faculty.csv** - 40 -> 75. The 40 source faculty keep their IDs, names and roles and were placed in the department of the course they own in `subjects.csv`. 31 new faculty were created for ECE, EEE, MECH, EIE, CIVIL, CSD, CSE and IT, each with a `Specialization` matching the department. Exactly one HOD per department; the surplus HODs from v1 were demoted to Associate Professor.
4. **Student.csv** - the same 2,320 students, same IDs and batches, redistributed across the ten departments by the target share (CSE 11.5%, ECE 10%, EEE 9%, AIML 11.5%, AIDS 9%, CSD 8%, MECH 11%, IT 11%, EIE 8%, CIVIL 10%). FeedbackCompletion was not touched, so completion history follows each student unchanged.
5. **Lab.csv** - 9 -> 26 labs, one per Lab-type course.
6. **Feedback facts** - ratings and dates untouched. Each row's `Course_ID` was re-pointed to a course of the same type (Theory/Lab) that is active in that same academic year, drawn with probability equal to the departmental student share. `Faculty_ID` was then set to the teacher of record for that course-year, which guarantees the faculty always belongs to the course's own department.

## Assumptions added in v2
- Departmental feedback volume is proportional to student numbers, which is what a census-style feedback drive produces.
- Course and faculty department mapping is by subject domain; no interdisciplinary teaching is modelled, so an AIML lecturer never appears against a CIVIL course.
- The two discontinued courses and four late-introduced courses from v1 keep their availability windows.

## Limitations added in v2
- The ten-department structure is entirely synthetic. The source institution data covers one department only, so student, faculty and course department assignments carry no evidence from the original files.
- Because feedback was re-pointed rather than re-simulated, a department's average rating reflects the v1 rating model plus which rows happened to land there; departmental *differences* are therefore noise, not signal. Faculty, course and year comparisons remain meaningful.
- The 61 new courses and 31 new faculty have no original counterpart of any kind.
