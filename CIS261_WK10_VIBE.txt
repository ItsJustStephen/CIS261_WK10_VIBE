# Stephen Aldridge
# CIS261
# Week 10 VIBE Coding

import os
import sys


FILE_NAME = "student_grades.txt"


def calculate_average(test1, test2, test3):
	"""Return the average of three test scores."""
	return (test1 + test2 + test3) / 3


def calculate_grade(average):
	"""Return the letter grade for an average score."""
	if average >= 90:
		return "A"
	if average >= 80:
		return "B"
	if average >= 70:
		return "C"
	if average >= 60:
		return "D"
	return "F"


def get_non_empty_input(prompt):
	"""Prompt until the user enters a non-empty value."""
	while True:
		value = input(prompt).strip()
		if value:
			return value
		print("This value cannot be blank.")


def get_score(test_number):
	"""Prompt for a test score from 0 through 100."""
	while True:
		try:
			score = float(input(f"Test {test_number} score (0-100): "))
			if 0 <= score <= 100:
				return score
			print("Please enter a score from 0 through 100.")
		except ValueError:
			print("Please enter a valid number.")


def create_student():
	"""Collect student information and calculate the student's results."""
	name = get_non_empty_input("Student name: ")
	student_id = get_non_empty_input("Student ID: ")
	test1 = get_score(1)
	test2 = get_score(2)
	test3 = get_score(3)
	average = calculate_average(test1, test2, test3)

	return {
		"name": name,
		"id": student_id,
		"test1": test1,
		"test2": test2,
		"test3": test3,
		"average": average,
		"grade": calculate_grade(average),
	}


def save_students(students, file_name=FILE_NAME):
	"""Save student records in the required pipe-delimited format."""
	try:
		with open(file_name, "w", encoding="utf-8") as file:
			for student in students:
				file.write(
					f"{student['name']}|{student['id']}|"
					f"{student['test1']:.2f}|{student['test2']:.2f}|"
					f"{student['test3']:.2f}|{student['average']:.2f}|"
					f"{student['grade']}\n"
				)
		return True
	except OSError as error:
		print(f"Unable to save student records: {error}")
		return False


def load_students(file_name=FILE_NAME):
	"""Load valid student records from a pipe-delimited file."""
	students = []
	if not os.path.exists(file_name):
		return students

	try:
		with open(file_name, "r", encoding="utf-8") as file:
			for line_number, line in enumerate(file, start=1):
				fields = line.rstrip("\n").split("|")
				if len(fields) != 7:
					print(f"Skipping invalid record on line {line_number}.")
					continue

				name, student_id, test1, test2, test3, average, grade = fields
				try:
					scores = [float(test1), float(test2), float(test3)]
					stored_average = float(average)
				except ValueError:
					print(f"Skipping invalid record on line {line_number}.")
					continue

				calculated_average = calculate_average(*scores)
				students.append(
					{
						"name": name,
						"id": student_id,
						"test1": scores[0],
						"test2": scores[1],
						"test3": scores[2],
						"average": calculated_average,
						"grade": calculate_grade(calculated_average),
					}
				)
				if abs(stored_average - calculated_average) > 0.01 or grade != calculate_grade(calculated_average):
					print(f"Recalculated results for line {line_number}.")
	except OSError as error:
		print(f"Unable to load student records: {error}")

	return students


def display_students(students):
	"""Display all student records in a formatted table."""
	if not students:
		print("No student records found.")
		return

	print("\nStudent Records")
	print("-" * 92)
	print(
		f"{'Name':<22}{'ID':<14}{'Test 1':>10}{'Test 2':>10}"
		f"{'Test 3':>10}{'Average':>12}{'Grade':>8}"
	)
	print("-" * 92)
	for student in students:
		print(
			f"{student['name'][:21]:<22}{student['id'][:13]:<14}"
			f"{student['test1']:>10.2f}{student['test2']:>10.2f}"
			f"{student['test3']:>10.2f}{student['average']:>12.2f}"
			f"{student['grade']:>8}"
		)
	print("-" * 92)


def display_statistics(students):
	"""Display highest, lowest, and overall class averages."""
	if not students:
		print("No student records available for statistics.")
		return

	highest = max(students, key=lambda student: student["average"])
	lowest = min(students, key=lambda student: student["average"])
	class_average = sum(student["average"] for student in students) / len(students)

	print("\nClass Statistics")
	print(f"Highest average: {highest['average']:.2f} ({highest['name']})")
	print(f"Lowest average:  {lowest['average']:.2f} ({lowest['name']})")
	print(f"Class average:   {class_average:.2f}")


def search_students(students):
	"""Display records whose names contain the search text."""
	search_name = get_non_empty_input("Enter the name to search for: ").lower()
	matches = [student for student in students if search_name in student["name"].lower()]
	if matches:
		display_students(matches)
	else:
		print(f"No student found matching '{search_name}'.")


def display_menu():
	print("\nStudent Grade Calculator")
	print("1. Add New Student")
	print("2. Display all students")
	print("3. Search Student by Name")
	print("4. View Class Statistics")
	print("5. Save and Exit (or press ESC)")
	print("Press ESC to save and exit.")


def get_menu_choice():
	"""Read a menu choice and detect ESC immediately in a terminal."""
	if not sys.stdin.isatty():
		return input("Select an option: ")

	try:
		import termios
		import tty

		print("Select an option: ", end="", flush=True)
		terminal_settings = termios.tcgetattr(sys.stdin)
		tty.setraw(sys.stdin.fileno())
		characters = []
		while True:
			character = sys.stdin.read(1)
			if character == "\x1b":
				print()
				return "\x1b"
			if character in ("\r", "\n"):
				print()
				return "".join(characters)
			if character in ("\x7f", "\b"):
				if characters:
					characters.pop()
					print("\b \b", end="", flush=True)
			else:
				characters.append(character)
				print(character, end="", flush=True)
	except (ImportError, OSError):
		return input("Select an option: ")
	finally:
		if "terminal_settings" in locals():
			termios.tcsetattr(sys.stdin, termios.TCSADRAIN, terminal_settings)


def main():
	students = load_students()
	if students:
		print(f"Loaded {len(students)} student record(s) from {FILE_NAME}.")
	else:
		print("No saved student records found.")

	while True:
		display_menu()
		choice = get_menu_choice()

		if choice in ("5", "\x1b"):
			if save_students(students):
				print(f"Saved {len(students)} student record(s). Goodbye!")
			return
		if choice == "1":
			student = create_student()
			students.append(student)
			save_students(students)
			print(f"Added {student['name']} with an average of {student['average']:.2f} ({student['grade']}).")
		elif choice == "2":
			display_students(students)
		elif choice == "3":
			if students:
				search_students(students)
			else:
				print("No student records available to search.")
		elif choice == "4":
			display_statistics(students)
		else:
			print("Invalid option. Please choose 1, 2, 3, 4, 5, or press ESC.")


if __name__ == "__main__":
	main()
