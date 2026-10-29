import java.util.*;
 
// ---------- Model class ----------
class Student {
    int rollNo;
    String name, dept;
    int marks;
 
    Student(int rollNo, String name, String dept, int marks) {
        this.rollNo = rollNo;
        this.name = name;
        this.dept = dept;
        this.marks = marks;
    }
 
    String getGrade() {
        if (marks >= 90) return "O";
        else if (marks >= 80) return "A";
        else if (marks >= 70) return "B";
        else if (marks >= 60) return "C";
        else if (marks >= 50) return "D";
        else return "F";
    }
 
    public String toString() {
        return rollNo + " | " + name + " | " + dept + " | " + marks + " | " + getGrade();
    }
}
 
// ---------- Main program ----------
public class StudentGradeSystem {
 
    static Scanner sc = new Scanner(System.in);
    static List<Student> students = new ArrayList<>();            // ordered list of students
    static Map<Integer, Student> studentMap = new HashMap<>();    // fast lookup by roll number
    static Set<Integer> rollNumbers = new HashSet<>();            // prevents duplicate roll numbers
    static Set<String> departments = new TreeSet<>();             // unique departments, sorted
    static Queue<Student> revaluationQueue = new LinkedList<>();  // FIFO revaluation requests
 
    // Reads an integer safely (keeps asking until valid)
    static int readInt(String prompt) {
        while (true) {
            System.out.print(prompt);
            try {
                return Integer.parseInt(sc.nextLine().trim());
            } catch (NumberFormatException e) {
                System.out.println("Please enter a valid number.");
            }
        }
    }
 
    static void addStudent() {
        int roll = readInt("Roll number: ");
        if (rollNumbers.contains(roll)) {
            System.out.println("Duplicate roll number. Student not added.");
            return;
        }
        System.out.print("Name: ");
        String name = sc.nextLine().trim();
        System.out.print("Department: ");
        String dept = sc.nextLine().trim().toUpperCase();
        if (name.isEmpty() || dept.isEmpty()) {
            System.out.println("Name and department cannot be empty.");
            return;
        }
        int marks = readInt("Marks (0-100): ");
        while (marks < 0 || marks > 100) {
            System.out.println("Marks must be between 0 and 100.");
            marks = readInt("Marks (0-100): ");
        }
        Student s = new Student(roll, name, dept, marks);
        rollNumbers.add(roll);
        students.add(s);
        studentMap.put(roll, s);
        departments.add(dept);
        System.out.println("Student added: " + s);
    }
 
    static void displayAll() {
        if (students.isEmpty()) {
            System.out.println("No students yet.");
            return;
        }
        System.out.println("Roll | Name | Dept | Marks | Grade");
        for (Student s : students) System.out.println(s);
    }
 
    static void searchStudent() {
        int roll = readInt("Enter roll number to search: ");
        Student s = studentMap.get(roll);
        System.out.println(s != null ? "Found: " + s : "Student not found (Roll " + roll + ")");
    }
 
    static void rankList() {
        if (students.isEmpty()) {
            System.out.println("No students yet.");
            return;
        }
        Comparator<Student> byMarks = (a, b) -> a.marks - b.marks;
        List<Student> ranked = new ArrayList<>(students);
        Collections.sort(ranked, byMarks.reversed());
        for (int i = 0; i < ranked.size(); i++) {
            Student s = ranked.get(i);
            System.out.println("Rank " + (i + 1) + ": " + s.name + " (" + s.marks + ")");
        }
        Student top = Collections.max(students, byMarks);
        Student low = Collections.min(students, byMarks);
        System.out.println("Topper: " + top.name + " (" + top.marks + ")");
        System.out.println("Lowest: " + low.name + " (" + low.marks + ")");
    }
 
    static void gradeWise() {
        if (students.isEmpty()) {
            System.out.println("No students yet.");
            return;
        }
        Map<String, List<String>> gradeMap = new TreeMap<>();
        for (Student s : students) {
            gradeMap.computeIfAbsent(s.getGrade(), k -> new ArrayList<>()).add(s.name);
        }
        for (Map.Entry<String, List<String>> e : gradeMap.entrySet()) {
            System.out.println(e.getKey() + " -> " + e.getValue());
        }
    }
 
    static void departmentAverage() {
        if (students.isEmpty()) {
            System.out.println("No students yet.");
            return;
        }
        System.out.println("Departments: " + departments);
        Map<String, List<Integer>> deptMarks = new TreeMap<>();
        for (Student s : students) {
            deptMarks.computeIfAbsent(s.dept, k -> new ArrayList<>()).add(s.marks);
        }
        for (Map.Entry<String, List<Integer>> e : deptMarks.entrySet()) {
            int sum = 0;
            for (int m : e.getValue()) sum += m;
            double avg = (double) sum / e.getValue().size();
            System.out.printf("%s : %.2f%n", e.getKey(), avg);
        }
    }
 
    static void requestRevaluation() {
        int roll = readInt("Enter roll number for revaluation: ");
        Student s = studentMap.get(roll);
        if (s == null) {
            System.out.println("Student not found (Roll " + roll + ")");
        } else if (revaluationQueue.contains(s)) {
            System.out.println("Request already pending for " + s.name + ".");
        } else {
            revaluationQueue.add(s);
            System.out.println("Request added for " + s.name + ". Pending requests: " + revaluationQueue.size());
        }
    }
 
    static void processRevaluation() {
        if (revaluationQueue.isEmpty()) {
            System.out.println("No pending revaluation requests.");
            return;
        }
        while (!revaluationQueue.isEmpty()) {
            Student s = revaluationQueue.poll();
            int old = s.marks;
            s.marks = Math.min(100, s.marks + 5);   // revaluation adds 5 marks (max 100)
            System.out.println("Revaluated " + s.name + ": " + old + " -> " + s.marks + " (" + s.getGrade() + ")");
        }
    }
 
    static void removeStudent() {
        int roll = readInt("Enter roll number to remove: ");
        boolean found = false;
        Iterator<Student> it = students.iterator();
        while (it.hasNext()) {
            Student s = it.next();
            if (s.rollNo == roll) {
                it.remove();                     // safe removal using Iterator
                studentMap.remove(roll);
                rollNumbers.remove(roll);
                revaluationQueue.remove(s);
                System.out.println("Removed: " + s.name);
                found = true;
                break;
            }
        }
        if (!found) {
            System.out.println("Student not found (Roll " + roll + ")");
            return;
        }
        departments.clear();                     // rebuild the department set
        for (Student s : students) departments.add(s.dept);
    }
 
    public static void main(String[] args) {
        int choice = 0;
        while (choice != 10) {
            System.out.println("\n--- STUDENT GRADE MANAGEMENT ---");
            System.out.println("1. Add student");
            System.out.println("2. Display all students");
            System.out.println("3. Search by roll number");
            System.out.println("4. Rank list");
            System.out.println("5. Grade-wise list");
            System.out.println("6. Department average");
            System.out.println("7. Request revaluation");
            System.out.println("8. Process revaluation requests");
            System.out.println("9. Remove student");
            System.out.println("10. Exit");
            choice = readInt("Enter choice: ");
 
            switch (choice) {
                case 1: addStudent(); break;
                case 2: displayAll(); break;
                case 3: searchStudent(); break;
                case 4: rankList(); break;
                case 5: gradeWise(); break;
                case 6: departmentAverage(); break;
                case 7: requestRevaluation(); break;
                case 8: processRevaluation(); break;
                case 9: removeStudent(); break;
                case 10: System.out.println("Thank you!"); break;
                default: System.out.println("Please choose 1-10.");
            }
        }
        sc.close();
    }
}
