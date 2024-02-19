from datetime import date

class StudyField:
    MECHANICAL_ENGINEERING = "Mechanical Engineering"
    SOFTWARE_ENGINEERING = "Software Engineering"
    FOOD_TECHNOLOGY = "Food Technology"
    URBANISM_ARCHITECTURE = "Urbanism Architecture"
    VETERINARY_MEDICINE = "Veterinary Medicine"

class Student:
    def __init__(self, first_name, last_name, email, enrollment_date, date_of_birth):
        self.first_name = first_name
        self.last_name = last_name
        self.email = email
        self.enrollment_date = enrollment_date
        self.date_of_birth = date_of_birth

class Faculty:
    def __init__(self, name, abbreviation, study_field):
        self.name = name
        self.abbreviation = abbreviation
        self.students = []
        self.study_field = study_field

class FileManager:
    def save_data(self, faculties):
        with open("data.txt", "w") as file:
            for faculty in faculties:
                file.write(f"{faculty.name},{faculty.abbreviation},{faculty.study_field}\n")
                for student in faculty.students:
                    file.write(f"{student.first_name},{student.last_name},{student.email},{student.enrollment_date},{student.date_of_birth}\n")

    def load_data(self):
        try:
            with open("data.txt", "r") as file:
                faculties = []
                lines = file.readlines()
                for line in lines:
                    data = line.strip().split(",")
                    faculty = Faculty(data[0], data[1], data[2])
                    faculties.append(faculty)
                return faculties
        except FileNotFoundError:
            print("File 'data.txt' not found. Creating empty list of faculties.")
            return []

class StudentManagementSystem:
    def __init__(self):
        self.faculties = []
        self.file_manager = FileManager()
        self.log_file = "log.txt"

    def create_student(self, faculty, first_name, last_name, email, day, month, year):
        enrollment_date = date.today()
        date_of_birth = date(year, month, day)
        student = Student(first_name, last_name, email, enrollment_date, date_of_birth)
        self.assign_student_to_faculty(student, faculty)
        self.log_operation(f"Created student: {first_name} {last_name}, Email: {email}, Faculty: {faculty.name}")
        return student

    def assign_student_to_faculty(self, student, faculty):
        faculty.students.append(student)

    def graduate_student(self, student, faculty):
        if student in faculty.students:
            faculty.students.remove(student)
            self.log_operation(f"Graduated student: {student.first_name} {student.last_name}, Faculty: {faculty.name}")
        else:
            self.log_operation(f"Failed to graduate student: {student.first_name} {student.last_name}, Faculty: {faculty.name}")

    def display_current_students(self, faculty):
        print(f"Faculty: {faculty.name}")
        for student in faculty.students:
            print(f"{student.first_name} {student.last_name}")

    def display_alumni(self, faculty):
        print(f"Faculty: {faculty.name} - Alumni:")
        for student in faculty.students:
            print(f"{student.first_name} {student.last_name}")

    def check_student_belongs_to_faculty(self, student, faculty):
        return student in faculty.students

    def create_faculty(self, name, abbreviation, study_field):
        faculty = Faculty(name, abbreviation, study_field)
        self.faculties.append(faculty)
        self.log_operation(f"Created faculty: {name}, Abbreviation: {abbreviation}, Field: {study_field}")
        return faculty

    def search_faculty_by_student_id(self, student_id):
        for faculty in self.faculties:
            for student in faculty.students:
                if student.email == student_id:
                    return faculty
        return None

    def display_all_faculties(self):
        print("All faculties:")
        for faculty in self.faculties:
            print(f"Faculty: {faculty.name}, Abbreviation: {faculty.abbreviation}, Field: {faculty.study_field}")

    def display_faculties_by_field(self, study_field):
        print(f"Faculties in field {study_field}:")
        for faculty in self.faculties:
            if faculty.study_field == study_field:
                print(f"Faculty: {faculty.name}, Abbreviation: {faculty.abbreviation}")

    def save_system_state(self):
        self.file_manager.save_data(self.faculties)
        self.log_operation("Saved system state.")

    def load_system_state(self):
        self.faculties = self.file_manager.load_data()
        self.log_operation("Loaded system state.")

    def log_operation(self, message):
        with open(self.log_file, "a") as file:
            file.write(f"{date.today()} - {message}\n")

    def register_students_from_file(self, file_path):
        with open(file_path, "r") as file:
            lines = file.readlines()
            for line in lines:
                data = line.strip().split(",")
                self.create_student(*data)

def main():
    system = StudentManagementSystem()
    system.load_system_state()

    while True:
        print("\nWelcome to TUM's student management system!")
        print("What do you want to do?")
        print("g - General operations")
        print("f - Faculty operations")
        print("s - Student operations")
        print("q - Quit Program")

        user_input = input("Your input> ").lower()

        if user_input == 'g':
            general_operations(system)
        elif user_input == 'f':
            faculty_operations(system)
        elif user_input == 's':
            student_operations(system)
        elif user_input == 'q':
            system.save_system_state()
            print("Exiting the program.")
            break
        else:
            print("Invalid input. Please try again.")

def general_operations(system):
    while True:
        print("\nGeneral operations What do you want to do?")
        print("nf/<faculty name>/<faculty abbreviation>/<field>")
        print("ss/<student email>")
        print("search student and show faculty")
        print("df display faculties")
        print("df/<field> display all faculties of a field")
        print("create faculty")
        print("b - Back")
        print("q - Quit Program")

        user_input = input("Your input> ").lower()

        if user_input == 'b':
            break
        elif user_input.startswith('nf/'):
            _, name, abbreviation, field = user_input.split('/')
            system.create_faculty(name, abbreviation, field)
            print(f"Faculty {name} created.")
        elif user_input.startswith('ss/'):
            _, email = user_input.split('/')
            faculty = system.search_faculty_by_student_id(email)
            if faculty:
                print(f"The student with email {email} belongs to {faculty.name}.")
            else:
                print(f"No student found with email {email}.")
        elif user_input == 'df':
            system.display_all_faculties()
        elif user_input.startswith('df/'):
            _, field = user_input.split('/')
            system.display_faculties_by_field(field)
        elif user_input == 'create faculty':
            name = input("Enter faculty name: ")
            abbreviation = input("Enter faculty abbreviation: ")
            field = input("Enter faculty field: ")
            system.create_faculty(name, abbreviation, field)
            print(f"Faculty {name} created.")
        elif user_input == 'q':
            system.save_system_state()
            print("Exiting the program.")
            exit()
        else:
            print("Invalid input. Please try again.")

def faculty_operations(system):
    while True:
        print("\nFaculty operations What do you want to do?")
        print("ns/<faculty abbreviation>/<first name>/<last name>/<email>/<day>/<month>/<year>")
        print("gs/<email> (g)raduate (s)tudent")
        print("ds/<faculty abbreviation> (d)isplay enrolled (s)tudents")
        print("dg/<faculty abbreviation> (d)isplay (g)raduated students.")
        print("bf/<faculty abbreviation>/<email> check if student (b)elongs to (f)aculty")
        print("D Back")
        print("q Quit Program")

        user_input = input("Your input> ").lower()

        if user_input == 'd':
            break
        elif user_input.startswith('ns/'):
            _, faculty_abbreviation, first_name, last_name, email, day, month, year = user_input.split('/')
            faculty = next((f for f in system.faculties if f.abbreviation == faculty_abbreviation), None)
            if faculty:
                system.create_student(faculty, first_name, last_name, email, int(day), int(month), int(year))
                print(f"Student {first_name} {last_name} created.")
            else:
                print(f"Faculty with abbreviation {faculty_abbreviation} not found.")
        elif user_input.startswith('gs/'):
            _, email = user_input.split('/')
            faculty = system.search_faculty_by_student_id(email)
            if faculty:
                student = next((s for s in faculty.students if s.email == email), None)
                if student:
                    system.graduate_student(student, faculty)
                    print(f"Student {student.first_name} {student.last_name} graduated.")
                else:
                    print(f"No student found with email {email} in {faculty.name}.")
            else:
                print(f"No faculty found for student with email {email}.")
        elif user_input.startswith('ds/'):
            _, faculty_abbreviation = user_input.split('/')
            faculty = next((f for f in system.faculties if f.abbreviation == faculty_abbreviation), None)
            if faculty:
                system.display_current_students(faculty)
            else:
                print(f"Faculty with abbreviation {faculty_abbreviation} not found.")
        elif user_input.startswith('dg/'):
            _, faculty_abbreviation = user_input.split('/')
            faculty = next((f for f in system.faculties if f.abbreviation == faculty_abbreviation), None)
            if faculty:
                system.display_alumni(faculty)
            else:
                print(f"Faculty with abbreviation {faculty_abbreviation} not found.")
        elif user_input.startswith('bf/'):
            _, faculty_abbreviation, email = user_input.split('/')
            faculty = next((f for f in system.faculties if f.abbreviation == faculty_abbreviation), None)
            if faculty:
                student = next((s for s in faculty.students if s.email == email), None)
                if student:
                    print(f"Yes, {email} belongs to {faculty.name}.")
                else:
                    print(f"No, {email} does not belong to {faculty.name}.")
            else:
                print(f"Faculty with abbreviation {faculty_abbreviation} not found.")
        elif user_input == 'q':
            system.save_system_state()
            print("Exiting the program.")
            exit()
        else:
            print("Invalid input. Please try again.")

def student_operations(system):
    while True:
        print("\nStudent operations What do you want to do?")
        print("as/<faculty abbreviation>/<first name>/<last name>/<email>/<day>/<month>/<year> add (s)tudent")
        print("b - Back")
        print("q - Quit Program")

        user_input = input("Your input> ").lower()

        if user_input == 'b':
            break
        elif user_input.startswith('as/'):
            _, faculty_abbreviation, first_name, last_name, email, day, month, year = user_input.split('/')
            faculty = next((f for f in system.faculties if f.abbreviation == faculty_abbreviation), None)

            if not faculty:
                print(f"Faculty with abbreviation {faculty_abbreviation} not found. Creating faculty.")
                system.create_faculty(faculty_abbreviation, f"{faculty_abbreviation}_name", "Some Field")

            faculty = next((f for f in system.faculties if f.abbreviation == faculty_abbreviation), None)

            if faculty:
                student_exists = any(stud.email == email for stud in faculty.students)
                if not student_exists:
                    system.create_student(faculty, first_name, last_name, email, int(day), int(month), int(year))
                    print(f"Student {first_name} {last_name} created.")
                else:
                    print(f"Student with email {email} already exists in {faculty.name}.")
            else:
                print(f"Faculty with abbreviation {faculty_abbreviation} not found.")
        elif user_input == 'q':
            system.save_system_state()
            print("Exiting the program.")
            exit()
        else:
            print("Invalid input. Please try again.")
            
class FileManager:
    def save_data(self, faculties):
        with open("data.txt", "w") as file:
            for faculty in faculties:
                file.write(f"{faculty.name},{faculty.abbreviation},{faculty.study_field}\n")
                for student in faculty.students:
                    file.write(f"{student.first_name},{student.last_name},{student.email},{student.enrollment_date},{student.date_of_birth}\n")

    def load_data(self):
        try:
            with open("data.txt", "r") as file:
                faculties = []
                lines = file.readlines()
                for line in lines:
                    data = line.strip().split(",")
                    faculty = Faculty(data[0], data[1], data[2])
                    faculties.append(faculty)
                return faculties
        except FileNotFoundError:
            print("File 'data.txt' not found. Creating empty list of faculties.")
            return []

if __name__ == "__main__":
    main()
