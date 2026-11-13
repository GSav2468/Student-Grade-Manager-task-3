#include <iostream>
#include <fstream>
#include <sstream>
#include <string>
#include <vector>
using namespace std;

class Student {
private:
    string id;
    string name;
    double coursework;
    double exam;

public:
    Student(string studentId, string studentName, double courseworkMark, double examMark) {
        id = studentId;
        name = studentName;
        coursework = courseworkMark;
        exam = examMark;
    }

    string getId() {
        return id;
    }

    double finalMark() {
        return (coursework * 0.4) + (exam * 0.6);
    }

    string classification() {
        double mark = finalMark();
        if (mark >= 70) {
            return "First";
        } else if (mark >= 60) {
            return "Upper Second";
        } else if (mark >= 50) {
            return "Lower Second";
        } else if (mark >= 40) {
            return "Third";
        } else {
            return "Fail";
        }
    }

    void display() {
        cout << "ID: " << id << " | Name: " << name;
        cout << " | Coursework: " << coursework << " | Exam: " << exam << endl;
    }

    void displayResult() {
        display();
        cout << "Final module mark: " << finalMark();
        cout << " | Classification: " << classification() << endl;
    }
};

string trim(string text) {
    int first = 0;
    int last = text.length();
    last--;
    while (first <= last && (text[first] == ' ' || text[first] == '\t' || text[first] == '\r')) {
        first++;
    }
    while (last >= first && (text[last] == ' ' || text[last] == '\t' || text[last] == '\r')) {
        last--;
    }
    return text.substr(first, last - first + 1);
}

bool readMark(string text, double& mark) {
    stringstream input(text);
    input >> mark;
    if (input.fail() || !(mark >= 0 && mark <= 100)) {
        return false;
    }
    string extra;
    if (input >> extra) {
        return false;
    }
    return true;
}

int findStudent(vector<Student>& students, string id) {
    int studentCount = students.size();
    for (int i = 0; i < studentCount; i++) {
        if (students[i].getId() == id) {
            return i;
        }
    }
    return -1;
}

bool loadStudents(vector<Student>& students) {
    ifstream file("resources/studentMarks.txt");
    if (!file.is_open()) {
        cout << "Could not open resources/studentMarks.txt." << endl;
        return false;
    }
    string line;
    int lineNumber = 0;
    bool firstRecord = true;
    while (getline(file, line)) {
        lineNumber++;
        line = trim(line);
        if (line == "") {
            continue;
        }
        int commaCount = 0;
        int lineLength = line.length();
        for (int i = 0; i < lineLength; i++) {
            if (line[i] == ',') {
                commaCount++;
            }
        }
        if (commaCount != 3) {
            cout << "Expected four fields on line " << lineNumber << "." << endl;
            return false;
        }
        stringstream row(line);
        string id, name, courseworkText, examText;
        getline(row, id, ',');
        getline(row, name, ',');
        getline(row, courseworkText, ',');
        getline(row, examText);
        id = trim(id);
        name = trim(name);
        courseworkText = trim(courseworkText);
        examText = trim(examText);
        if (firstRecord && id == "StudentID" && name == "Name" &&
            courseworkText == "Coursework" && examText == "Exam") {
            firstRecord = false;
            continue;
        }
        firstRecord = false;
        double coursework = 0;
        double exam = 0;
        if (id == "" || name == "" || !readMark(courseworkText, coursework) ||
            !readMark(examText, exam) || findStudent(students, id) != -1) {
            cout << "Invalid or duplicate student record on line " << lineNumber << "." << endl;
            return false;
        }
        Student student(id, name, coursework, exam);
        students.push_back(student);
    }
    file.close();
    if (students.empty()) {
        cout << "The student file contains no student records." << endl;
        return false;
    }
    return true;
}

int main() {
    vector<Student> students;
    if (!loadStudents(students)) {
        return 1;
    }
    string choice;
    while (true) {
        cout << "\n1. View all students\n2. View an individual student\n";
        cout << "3. Calculate a student's final module mark\n0. Quit\nChoice: ";
        if (!getline(cin, choice)) {
            break;
        }
        choice = trim(choice);
        if (choice == "0") {
            break;
        } else if (choice == "1") {
            int studentCount = students.size();
            for (int i = 0; i < studentCount; i++) {
                students[i].display();
            }
        } else if (choice == "2" || choice == "3") {
            cout << "Student ID: ";
            string id;
            if (!getline(cin, id)) {
                break;
            }
            int index = findStudent(students, trim(id));
            if (index == -1) {
                cout << "Student not found." << endl;
            } else if (choice == "2") {
                students[index].display();
            } else {
                students[index].displayResult();
            }
        } else {
            cout << "Please choose 0, 1, 2, or 3." << endl;
        }
    }
    cout << "Goodbye!" << endl;
    return 0;
}
