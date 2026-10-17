

import csv
import os

input_file = "students.csv"
output_file = "student_performance_report.csv"

sample_students = [
    ["101", "Anu", 85, 90, 88, 95],
    ["102", "Ravi", 45, 55, 60, 70],
    ["103", "Priya", 95, 92, 98, 99],
    ["104", "Kiran", 30, 25, 40, 60],
    ["105", "Sneha", 72, 78, 80, 85],
    ["106", "Arjun", 65, 58, 70, 75],
]

if not os.path.exists(input_file):
    with open(input_file, "w", newline="") as file:
        writer = csv.writer(file)
        writer.writerow([
            "ID", "Name", "Python", "SQL",
            "Excel", "Attendance"
        ])
        writer.writerows(sample_students)

students = []

with open(input_file, "r", newline="") as file:
    reader = csv.DictReader(file)

    for row in reader:
        try:
            marks = [
                float(row["Python"]),
                float(row["SQL"]),
                float(row["Excel"])
            ]
            attendance = float(row["Attendance"])

            if not all(0 <= mark <= 100 for mark in marks):
                raise ValueError("Marks must be between 0 and 100")

            if not 0 <= attendance <= 100:
                raise ValueError("Attendance must be between 0 and 100")

            if not row["ID"].strip() or not row["Name"].strip():
                raise ValueError("Student ID and name are required")

            total = sum(marks)
            average = total / 3

            if min(marks) < 35:
                result = "Fail"
            else:
                result = "Pass"

            if result == "Fail":
                grade = "F"
            elif average >= 90:
                grade = "A+"
            elif average >= 80:
                grade = "A"
            elif average >= 70:
                grade = "B"
            elif average >= 60:
                grade = "C"
            elif average >= 50:
                grade = "D"
            else:
                grade = "E"

            students.append({
                "ID": row["ID"],
                "Name": row["Name"],
                "Python": marks[0],
                "SQL": marks[1],
                "Excel": marks[2],
                "Attendance": attendance,
                "Total": round(total, 2),
                "Average": round(average, 2),
                "Grade": grade,
                "Result": result
            })

        except (ValueError, KeyError) as error:
            print(f"Skipping invalid record: {error}")

if not students:
    print("No valid student records found.")
    raise SystemExit

students.sort(key=lambda student: student["Average"], reverse=True)

print("\nSTUDENT PERFORMANCE REPORT")
print("-" * 75)
print(f"{'Name':<12}{'Total':<10}{'Average':<12}{'Grade':<10}{'Result':<10}")

for student in students:
    print(
        f"{student['Name']:<12}"
        f"{student['Total']:<10.1f}"
        f"{student['Average']:<12.2f}"
        f"{student['Grade']:<10}"
        f"{student['Result']:<10}"
    )

top_student = students[0]
passed = sum(student["Result"] == "Pass" for student in students)
failed = len(students) - passed
class_average = sum(student["Average"] for student in students) / len(students)

print("\nCLASS SUMMARY")
print("-" * 30)
print("Total students:", len(students))
print("Passed:", passed)
print("Failed:", failed)
print("Class average:", round(class_average, 2))
print("Top performer:", top_student["Name"])
print("Average attendance:", round(
    sum(student["Attendance"] for student in students) / len(students), 2
), "%")

print("\nStudents needing attention:")

for student in students:
    if student["Result"] == "Fail" or student["Attendance"] < 75:
        print(
            student["Name"],
            "- Result:", student["Result"],
            "- Attendance:", student["Attendance"], "%"
        )

with open(output_file, "w", newline="") as file:
    fieldnames = list(students[0].keys())
    writer = csv.DictWriter(file, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerows(students)

print("\nReport saved to", output_file)
