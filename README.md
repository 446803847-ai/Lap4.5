import java.time.LocalDateTime;
import java.util.Scanner;

// Enum for Roles
enum Role {
    INTERN,
    ENGINEER,
    MANAGER,
    HR,
    FINANCE,
    SALES;

    public static Role fromString(String s) {
        if (s == null) throw new IllegalArgumentException("Role cannot be null");
        return Role.valueOf(s.trim().toUpperCase());
    }
}

// Employee class
class Employee {
    // Fields
    private String name;
    private int employeeId;
    private Role role;                
    private double basicSalary;
    private double bonus;

    // Constructors (overloaded)
    public Employee(String name, int employeeId, Role role) {
        this(name, employeeId, role, 5000.0); // default basicSalary = 5000
    }

    public Employee(String name, int employeeId, Role role, double basicSalary) {
        if (name == null || name.isBlank()) throw new IllegalArgumentException("Name is required");
        if (employeeId <= 0) throw new IllegalArgumentException("Employee ID must be positive");
        if (role == null) throw new IllegalArgumentException("Role is required");
        if (basicSalary < 0) throw new IllegalArgumentException("Basic salary cannot be negative");
        this.name = name;
        this.employeeId = employeeId;
        this.role = role;
        this.basicSalary = basicSalary;
        this.bonus = 0.0;
    }

    // Methods
    public double calculateTotalSalary() {
        return basicSalary + bonus;
    }

    public void displayDetails() {
        System.out.printf(
            "Employee: %s | ID: %d | Role: %s | Basic: %.2f | Bonus: %.2f | Total: %.2f%n",
            name, employeeId, role, basicSalary, bonus, calculateTotalSalary()
        );
    }

    // Getters/Setters (only what we need)
    public int getEmployeeId() { return employeeId; }
    public String getName() { return name; }
    public Role getRole() { return role; }
    public double getBasicSalary() { return basicSalary; }
    public double getBonus() { return bonus; }

    public void setBonus(double bonus) {
        if (bonus < 0) throw new IllegalArgumentException("Bonus cannot be negative");
        this.bonus = bonus;
    }
}

// HR class (manages employee actions)
class HR {
    // Fields
    private final Employee[] employees;
    private int count = 0;

    // capacity 2–5 as required (default to 5)
    public HR() {
        this(5);
    }

    public HR(int capacity) {
        if (capacity < 2 || capacity > 5) {
            throw new IllegalArgumentException("Capacity must be between 2 and 5.");
        }
        this.employees = new Employee[capacity];
        logAction("Initialized HR system with capacity " + capacity);
    }

    // Methods
    public boolean addEmployee(Employee employee) {
        if (employee == null) {
            logAction("Attempted to add null employee");
            return false;
        }
        if (count >= employees.length) {
            logAction("System is at full capacity. Cannot add employee ID " + employee.getEmployeeId());
            return false;
        }
        if (isEmployeeExists(employee.getEmployeeId())) {
            logAction("Employee ID " + employee.getEmployeeId() + " already exists");
            return false;
        }
        employees[count++] = employee;
        logAction("Added employee: " + employee.getName() + " (ID " + employee.getEmployeeId() + ")");
        return true;
    }

    public boolean updateBonus(int employeeId, double bonus) {
        if (bonus < 0) {
            logAction("Rejected negative bonus update for ID " + employeeId);
            return false;
        }
        int idx = findIndexById(employeeId);
        if (idx == -1) {
            logAction("Employee not found for bonus update. ID " + employeeId);
            return false;
        }
        employees[idx].setBonus(bonus);
        logAction("Updated bonus for ID " + employeeId + " to " + bonus);
        return true;
    }

    public void displayAllEmployees() {
        if (count == 0) {
            System.out.println("No employees in the system.");
            return;
        }
        System.out.println("=== All Employees (" + count + "/" + employees.length + ") ===");
        for (int i = 0; i < count; i++) {
            employees[i].displayDetails();
        }
    }

    // Required: protected exists-check
    protected boolean isEmployeeExists(int employeeId) {
        return findIndexById(employeeId) != -1;
    }

    // Logger method
    public void logAction(String action) {
        System.out.printf("[LOG %s] %s%n", LocalDateTime.now(), action);
    }

    // Helpers
    private int findIndexById(int employeeId) {
        for (int i = 0; i < count; i++) {
            if (employees[i].getEmployeeId() == employeeId) return i;
        }
        return -1;
    }

    // Just in case you want to expose a read-only snapshot
    public Employee[] snapshot() {
        return new Employee[count];
    }
}

// Main application with menu
public class EMSApp {
    private static final Scanner SC = new Scanner(System.in);

    public static void main(String[] args) {
        HR hr = new HR(); // default capacity 5
        boolean running = true;

        while (running) {
            printMenu();
            int choice = readInt("Choose an option: ");

            switch (choice) {
                case 1 -> handleAddEmployee(hr);
                case 2 -> handleUpdateBonus(hr);
                case 3 -> hr.displayAllEmployees();
                case 4 -> {
                    hr.logAction("Shutting down");
                    running = false;
                }
                default -> System.out.println("Invalid choice. Try again.");
            }
            System.out.println();
        }
        System.out.println("Goodbye!");
    }

    private static void printMenu() {
        System.out.println("=========== Employee Management System ===========");
        System.out.println("1) Add Employee");
        System.out.println("2) Update Bonus");
        System.out.println("3) Display All Employees");
        System.out.println("4) Exit");
        System.out.println("==================================================");
    }

    private static void handleAddEmployee(HR hr) {
        System.out.println("--- Add Employee ---");
        String name = readLine("Enter name: ");
        int id = readInt("Enter employee ID (positive integer): ");
        Role role = readRole();
        double customBasic = readOptionalDouble(
            "Enter basic salary or press ENTER to use default 5000: ", 5000.0
        );

        try {
            Employee emp = (customBasic == 5000.0)
                    ? new Employee(name, id, role)
                    : new Employee(name, id, role, customBasic);
            boolean ok = hr.addEmployee(emp);
            System.out.println(ok ? "Employee added." : "Failed to add employee.");
        } catch (IllegalArgumentException ex) {
            System.out.println("Error: " + ex.getMessage());
        }
    }

    private static void handleUpdateBonus(HR hr) {
        System.out.println("--- Update Bonus ---");
        int id = readInt("Enter employee ID: ");
        double bonus = readDouble("Enter new bonus (>= 0): ");
        boolean ok = hr.updateBonus(id, bonus);
        System.out.println(ok ? "Bonus updated." : "Failed to update bonus.");
    }

    // Input helpers
    private static int readInt(String prompt) {
        while (true) {
            System.out.print(prompt);
            try {
                int val = Integer.parseInt(SC.nextLine().trim());
                return val;
            } catch (NumberFormatException ex) {
                System.out.println("Please enter a valid integer.");
            }
        }
    }

    private static double readDouble(String prompt) {
        while (true) {
            System.out.print(prompt);
            try {
                double val = Double.parseDouble(SC.nextLine().trim());
                return val;
            } catch (NumberFormatException ex) {
                System.out.println("Please enter a valid number.");
            }
        }
    }

    private static double readOptionalDouble(String prompt, double defaultVal) {
        System.out.print(prompt);
        String line = SC.nextLine();
        if (line == null || line.isBlank()) return defaultVal;
        try {
            return Double.parseDouble(line.trim());
        } catch (NumberFormatException ex) {
            System.out.println("Invalid number. Using default " + defaultVal);
            return defaultVal;
        }
    }

    private static String readLine(String prompt) {
        System.out.print(prompt);
        return SC.nextLine().trim();
    }

    private static Role readRole() {
        while (true) {
            System.out.print("Enter role (Intern/Engineer/Manager/HR/Finance/Sales): ");
            String s = SC.nextLine();
            try {
                return Role.fromString(s);
            } catch (Exception e) {
                System.out.println("Invalid role. Try again.");
            }
        }
    }
}
