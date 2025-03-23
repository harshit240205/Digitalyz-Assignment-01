# Digitalyz-Assignment-01
Milestone 1: Data Cleaning & Validation
Overview
This milestone focuses on cleaning the provided raw dataset, structuring it into a usable format, and performing initial validation checks. The final output includes a structured JSON file and an insights report highlighting potential scheduling conflicts.
Approach
Data Loading:
Extracted data from dataset.xlsx containing multiple sheets: Lecturer Details, Rooms Data, Course List, Student Requests, and Rules.
Data Cleaning & Structuring:


Standardized column names for consistency.
Handle missing values by filling with appropriate defaults.
Converted the structured data into JSON format for further processing.
Validation Checks:


Lecturer Double-Booking: Ensured no lecturer is assigned to multiple courses in the same time block.
Student Schedule Conflicts: Verified that students are not assigned to overlapping courses.
Room Overcapacity: Checked that classroom assignments do not exceed their capacities.
Assumptions
Each course is assigned to only one lecturer.
If a course does not have available time blocks, it was skipped in scheduling validation.
Missing room capacities were assumed to be unlimited for validation purposes.
Data was cleaned but not modified; scheduling conflicts were only identified, not resolved.

Output Files
cleaned_data.json → Structured and cleaned dataset.
insights_report.json → Identified conflicts and unresolved requests.
Relevant Code
1. Data Loading
import pandas as pd
import json

# Load the dataset
file_path = "dataset.xlsx"
xls = pd.ExcelFile(file_path)

# Load each sheet into a DataFrame
lecturers_df = pd.read_excel(xls, sheet_name='Lecturer Details')
rooms_df = pd.read_excel(xls, sheet_name='Rooms data')
courses_df = pd.read_excel(xls, sheet_name='Course list')
requests_df = pd.read_excel(xls, sheet_name='Student requests')
rules_df = pd.read_excel(xls, sheet_name='RULES')

2. Data Cleaning & Structuring
# Standardizing column names
for df in [lecturers_df, rooms_df, courses_df, requests_df, rules_df]:
    df.columns = df.columns.str.strip().str.lower().str.replace(" ", "_")

# Fill missing values
rooms_df.fillna("", inplace=True)
courses_df.fillna("", inplace=True)
requests_df.fillna("", inplace=True)

# Convert cleaned data to JSON
cleaned_data = {
    "lecturers": lecturers_df.to_dict(orient="records"),
    "rooms": rooms_df.to_dict(orient="records"),
    "courses": courses_df.to_dict(orient="records"),
    "student_requests": requests_df.to_dict(orient="records"),
    "rules": rules_df.to_dict(orient="records")
}

# Save JSON to file
json_file_path = "cleaned_data.json"
with open(json_file_path, "w") as json_file:
    json.dump(cleaned_data, json_file, indent=4)

3. Validation Checks
conflicts = []
unresolved_requests = []
lecturer_schedule = {}
room_capacity = {}
room_assignments = {}

# Check lecturer double-booking
for _, row in courses_df.iterrows():
    course_code = row['course_code']
    available_blocks = row['available_blocks'].split(",") if row['available_blocks'] else []
    lecturer_ids = lecturers_df.loc[lecturers_df['lecture_code'] == course_code, 'lecturer_id'].values
    
    if len(lecturer_ids) == 0 or not available_blocks:
        continue
    
    lecturer_id = lecturer_ids[0]
    if lecturer_id not in lecturer_schedule:
        lecturer_schedule[lecturer_id] = []
    
    for block in available_blocks:
        if block in lecturer_schedule[lecturer_id]:
            conflicts.append(f"Lecturer {lecturer_id} is double-booked in block {block}.")
        lecturer_schedule[lecturer_id].append(block)

# Check student schedule conflicts
student_schedules = {}
for _, row in requests_df.iterrows():
    student_id = row['student_id']
    course_code = row['course_code']
    course_data = courses_df[courses_df['course_code'] == course_code]
    if course_data.empty:
        continue
    available_blocks = course_data['available_blocks'].iloc[0].split(",")
    
    if student_id not in student_schedules:
        student_schedules[student_id] = []
    
    for block in available_blocks:
        if block in student_schedules[student_id]:
            unresolved_requests.append(f"Student {student_id} has a conflict in block {block}.")
        student_schedules[student_id].append(block)

# Check room overcapacity
for _, row in rooms_df.iterrows():
    if 'capacity' in rooms_df.columns:
        room_capacity[row['room_number']] = row['capacity']
    room_assignments[row['room_number']] = 0

for _, row in courses_df.iterrows():
    room_number = row.get('room_number', None)
    section_size = row.get('target_section_size', 0)
    
    if room_number and room_number in room_capacity:
        room_assignments[room_number] += section_size
        if room_assignments[room_number] > room_capacity[room_number]:
            conflicts.append(f"Room {room_number} is overbooked.")

# Save validation results
insights = {
    "conflicts": conflicts,
    "unresolved_requests": unresolved_requests,
    "lecturer_schedule": lecturer_schedule
}

insights_file_path = "insights_report.json"
with open(insights_file_path, "w") as f:
    json.dump(insights, f, indent=4)

print("Data cleaned and validated successfully.")

Cleaning the dataset.
Converting data into JSON.
Performing validation checks.

