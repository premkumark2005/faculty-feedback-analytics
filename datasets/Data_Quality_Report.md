# Data Quality Report (v2 - 10 departments)

Same five academic years as v1 (2021-22 .. 2025-26). Only the department structure and the objects that depend on it changed.

## 1. Table inventory

| Table | Rows | Columns |
|---|---|---|
| Department | 10 | 2 |
| Course | 91 | 6 |
| Faculty | 75 | 5 |
| Student | 2,320 | 4 |
| FacultyFeedback | 26,272 | 10 |
| TheoryFeedback | 17,954 | 9 |
| LabFeedback | 8,318 | 13 |
| FeedbackCompletion | 4,993 | 3 |
| Lab | 26 | 4 |

## 2. Validation

| Check | Result | Detail |
|---|---|---|
| Department count is exactly 10 | PASS |  |
| Department codes exact | PASS |  |
| Course.Dept_ID -> Department | PASS |  |
| Faculty.Dept_ID -> Department | PASS |  |
| Student.Dept_ID -> Department | PASS |  |
| Lab.Dept_ID -> Department | PASS |  |
| FacultyFeedback.Faculty_ID -> Faculty | PASS |  |
| FacultyFeedback.Course_ID -> Course | PASS |  |
| TheoryFeedback.Course_ID -> Course | PASS |  |
| LabFeedback.LabID -> Lab | PASS |  |
| Lab.Course_ID -> Course | PASS |  |
| FeedbackCompletion.S_ID -> Student | PASS |  |
| Faculty teaches only own-department courses | PASS |  |
| TheoryFeedback only for Theory courses | PASS |  |
| LabFeedback only for Lab courses | PASS |  |
| Course.Type in {Theory,Lab} | PASS | {'Theory': 65, 'Lab': 26} |
| Course.Sem within 1-8 | PASS | [np.int64(1), np.int64(2), np.int64(3), np.int64(4), np.int64(5), np.int64(6), np.int64(7), np.int64(8)] |
| Faculty.Role allowed | PASS | {'Assistant Professor': 36, 'Associate Professor': 29, 'HOD': 10} |
| Exactly one HOD per department | PASS |  |
| FacultyFeedback ratings 1-5 and non-null | PASS |  |
| TheoryFeedback ratings 1-5 and non-null | PASS |  |
| LabFeedback ratings 1-5 and non-null | PASS |  |
| All five academic years present | PASS |  |
| Completion TRUE/FALSE only | PASS |  |
| No duplicate PKs anywhere | PASS |  |
| Students active only within their batch window | PASS |  |
| Every department has courses, faculty, students and feedback | PASS |  |

## 3. Department summary

| Department | Students | Faculty | Courses (Theory/Lab) | Faculty FB | Theory FB | Lab FB |
|---|---|---|---|---|---|---|
| CSE | 275 | 10 | 10 (8/2) | 3,115 | 2,094 | 915 |
| ECE | 234 | 7 | 9 (6/3) | 2,739 | 1,873 | 844 |
| EEE | 210 | 6 | 8 (6/2) | 2,403 | 1,569 | 759 |
| AIML | 269 | 9 | 11 (8/3) | 3,016 | 2,084 | 978 |
| AIDS | 210 | 11 | 13 (9/4) | 2,319 | 1,593 | 735 |
| CSD | 187 | 7 | 8 (5/3) | 2,070 | 1,464 | 698 |
| MECH | 257 | 6 | 8 (6/2) | 2,873 | 1,985 | 925 |
| IT | 257 | 9 | 9 (6/3) | 3,001 | 2,022 | 885 |
| EIE | 187 | 5 | 7 (5/2) | 2,077 | 1,451 | 688 |
| CIVIL | 234 | 5 | 8 (6/2) | 2,659 | 1,819 | 891 |

## 4. Year distribution

| Academic Year | Faculty FB | Theory FB | Lab FB | Completion |
|---|---|---|---|---|
| 2021-22 | 4,686 | 3,292 | 1,394 | 80.7% |
| 2022-23 | 4,834 | 3,378 | 1,456 | 84.0% |
| 2023-24 | 5,088 | 3,464 | 1,624 | 87.6% |
| 2024-25 | 6,017 | 4,023 | 1,994 | 85.4% |
| 2025-26 | 5,647 | 3,797 | 1,850 | 93.3% |

Row counts per year are identical to v1 - no feedback record was added or removed.
