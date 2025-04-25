import java.util.*;

/*
 * LectureTracker3000 - Surveillance Edition
 * 
 * This is a Java-based simulation for tracking student attendance at lectures in a university.
 * The system is designed with the following features:
 * 
 * - Students are monitored for their lecture attendance.
 * - If a student misses too many lectures and does not respect the basics, they might be flagged for deportation (if they are non-EU).
 * - The system uses different types of judges (Brutal, Merciful, AI) to evaluate if students pass the course.
 * - There is a simulated external API (BorderGuardAPI) to check if a student should be deported based on their attendance.
 * - The code demonstrates the usage of Inheritance, Interfaces, Enums, Collections, and Custom Exceptions.
 * 
 * Features:
 * - Custom exception handling using DeportationAlertException.
 * - Randomized attendance simulation for students.
 * - Plot twist feature granting diplomatic immunity to non-EU students.
 * 
 * Technical Requirements Implemented:
 * - Enum (Nationality)
 * - Interface (LectureJudge)
 * - Abstract Class (LectureContent)
 * - Collections (ArrayList, HashMap)
 * - Custom Exception (DeportationAlertException)
 * - Randomization and UUID generation for student IDs.
 * - Try-catch block for exception handling.
 * 
 * Date: 2025
 * 
 * Author: [Your Name]
 */

public class LectureTracker3000 {

    // ===== ENUM =====
    enum Nationality {
        POLISH,
        EU,
        NON_EU
    }

    // ===== STUDENT CLASS =====
    static class Student {
        private String name;
        private int lecturesMissed;
        private boolean respectsBasics;
        private Nationality nationality;
        private UUID studentID;

        public Student(String name, int lecturesMissed, boolean respectsBasics, Nationality nationality) {
            this.name = name;
            this.lecturesMissed = lecturesMissed;
            this.respectsBasics = respectsBasics;
            this.nationality = nationality;
            this.studentID = UUID.randomUUID();
        }

        public void attendLecture() {
            if (lecturesMissed > 0) {
                lecturesMissed--;
            }
        }

        public boolean isAtRisk() {
            return lecturesMissed > 4 && !respectsBasics;
        }

        public String getName() { return name; }
        public int getLecturesMissed() { return lecturesMissed; }
        public boolean respectsBasics() { return respectsBasics; }
        public Nationality getNationality() { return nationality; }
        public UUID getStudentID() { return studentID; }
    }

    // ===== IMMUNE STUDENT =====
    static class ImmuneStudent extends Student {
        public ImmuneStudent(String name, int lecturesMissed, boolean respectsBasics, Nationality nationality) {
            super(name, lecturesMissed, respectsBasics, nationality);
        }

        @Override
        public boolean isAtRisk() {
            return false;
        }
    }

    // ===== LECTURECONTENT ABSTRACT =====
    static abstract class LectureContent {
        public abstract String getTopic();

        public void displayLecture() {
            System.out.println("Today's missed knowledge: " + getTopic());
        }
    }

    static class IntroLecture extends LectureContent {
        public String getTopic() {
            return "Introduction to Java";
        }
    }

    static class LoopsLecture extends LectureContent {
        public String getTopic() {
            return "Loops and Iteration";
        }
    }

    static class InheritanceLecture extends LectureContent {
        public String getTopic() {
            return "Object-Oriented Inheritance";
        }
    }

    static class InterfaceLecture extends LectureContent {
        public String getTopic() {
            return "Interfaces in Java";
        }
    }

    // ===== DATABASE =====
    static class LectureDatabase {
        public static Map<String, LectureContent> lectures = new HashMap<>();

        static {
            lectures.put("Week 1", new IntroLecture());
            lectures.put("Week 2", new LoopsLecture());
            lectures.put("Week 3", new InheritanceLecture());
            lectures.put("Week 4", new InterfaceLecture());
        }

        public static void displayLectureForWeek(String week) {
            LectureContent lecture = lectures.get(week);
            if (lecture != null) {
                lecture.displayLecture();
            } else {
                System.out.println("No lecture found for " + week);
            }
        }
    }

    // ===== JUDGES =====
    interface LectureJudge {
        boolean shouldPass(Student s);
    }

    static class BrutalJudge implements LectureJudge {
        public boolean shouldPass(Student s) {
            return s.getLecturesMissed() <= 2 && s.respectsBasics();
        }
    }

    static class MercifulJudge implements LectureJudge {
        public boolean shouldPass(Student s) {
            return s.getLecturesMissed() <= 6 || s.respectsBasics();
        }
    }

    static class AIJudge implements LectureJudge {
        public boolean shouldPass(Student s) {
            double score = s.getLecturesMissed() * 0.8 + (s.respectsBasics() ? -2 : 2);
            return score < 5;
        }
    }

    // ===== EXCEPTION =====
    static class DeportationAlertException extends Exception {
        public DeportationAlertException(Student s) {
            super("⚠️ Border Alert: Student " + s.getName() + " flagged for deportation. Too many skipped lectures.");
        }
    }

    // ===== API MOCK =====
    static class BorderGuardAPI {
        public static void checkDeportationRisk(Student s) throws DeportationAlertException {
            if (s.isAtRisk() && s.getNationality() == Nationality.NON_EU) {
                throw new DeportationAlertException(s);
            }
        }
    }

    // ===== MAIN SIMULATOR =====
    public static void main(String[] args) {
        Random rand = new Random();

        List<Student> students = new ArrayList<>();
        students.add(new Student("Anna", 3, true, Nationality.POLISH));
        students.add(new Student("Carlos", 5, false, Nationality.NON_EU));
        students.add(new Student("Emily", 2, true, Nationality.EU));
        students.add(new Student("Fatima", 6, false, Nationality.NON_EU));
        students.add(new Student("Lukas", 4, true, Nationality.POLISH));

        plotTwist(students);

        List<LectureJudge> judges = Arrays.asList(new BrutalJudge(), new MercifulJudge(), new AIJudge());

        for (int week = 1; week <= 4; week++) {
            System.out.println("\n📚 Week " + week + ":");
            LectureDatabase.displayLectureForWeek("Week " + week);

            for (Student student : students) {
                boolean attended = rand.nextBoolean();
                if (attended) {
                    student.attendLecture();
                    System.out.println(student.getName() + " attended the lecture.");
                } else {
                    System.out.println(student.getName() + " skipped the lecture.");
                }
            }
        }

        System.out.println("\n🧑‍⚖️ Final Evaluation:\n");

        for (Student student : students) {
            LectureJudge judge = judges.get(rand.nextInt(judges.size()));
            try {
                BorderGuardAPI.checkDeportationRisk(student);

                if (judge.shouldPass(student)) {
                    System.out.println("✅ Student " + student.getName() + " passed. The Republic thanks you for your loyalty.");
                } else {
                    System.out.println("❌ Student " + student.getName() + " failed. Consider respecting the basics next time.");
                }
            } catch (DeportationAlertException e) {
                System.out.println(e.getMessage());
                System.out.println("💀 " + student.getName() + " has been reported. Maybe come to loops lecture next life.");
            }
        }
    }

    // ===== BONUS PLOT TWIST =====
    public static void plotTwist(List<Student> students) {
        List<Student> nonEU = new ArrayList<>();
        for (Student s : students) {
            if (s.getNationality() == Nationality.NON_EU) {
                nonEU.add(s);
            }
        }

        if (!nonEU.isEmpty()) {
            Random rand = new Random();
            Student chosen = nonEU.get(rand.nextInt(nonEU.size()));

            Student immune
