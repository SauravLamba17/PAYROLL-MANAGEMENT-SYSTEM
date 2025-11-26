import java.io.*;
import java.util.ArrayList;
import java.util.InputMismatchException;
import java.util.Scanner;

public class PayrollSystem {
    private static ArrayList<Employee> employeeList = new ArrayList<>();
    private static ArrayList<PayrollRecord> payrollHistory = new ArrayList<>();
    private static final String EMP_FILE = "employees.dat";
    private static final String HIST_FILE = "payroll_history.dat";
    private static Scanner scanner = new Scanner(System.in);
    private static int nextEmployeeId = 1001;

    public static void main(String[] args) {
        loadData();
        
        boolean running = true;
        while (running) {
            displayMenu();
            try {
                int choice = scanner.nextInt();
                scanner.nextLine();
                
                switch (choice) {
                    case 1: addEmployee(); break;
                    case 2: calculateAllPayroll(); break;
                    case 3: generatePayslip(); break;
                    case 4: viewData(); break;
                    case 5: running = false; saveData(); break;
                    default: System.out.println("Invalid choice. Please try again.");
                }
            } catch (InputMismatchException e) {
                System.err.println("❌ Invalid input! Please enter a number from the menu.");
                scanner.nextLine();
            }
            System.out.println("\n----------------------------------------\n");
        }
        System.out.println("Payroll System Shut Down. Data saved.");
    }

    private static void displayMenu() {
        System.out.println("========================================");
        System.out.println("  💰 Payroll Management System Menu 💰");
        System.out.println("========================================");
        System.out.println("1. Add New Employee");
        System.out.println("2. Calculate Monthly Payroll for ALL");
        System.out.println("3. Generate Payslip (View History)");
        System.out.println("4. View Employee Data");
        System.out.println("5. Exit and Save Data");
        System.out.print("Enter your choice: ");
    }

    private static void addEmployee() {
        System.out.println("\n--- Add Employee ---");
        System.out.print("Enter Name: ");
        String name = scanner.nextLine();
        System.out.print("Enter Designation: ");
        String designation = scanner.nextLine();
        System.out.print("Enter Bank Account Number: ");
        String bankAcct = scanner.nextLine();
        
        double taxRate = 0.15;

        System.out.print("Is the employee (S)alaried or (H)ourly? (S/H): ");
        String type = scanner.nextLine().toUpperCase();

        try {
            if (type.equals("S")) {
                System.out.print("Enter Monthly Salary: ");
                double salary = Double.parseDouble(scanner.nextLine()); 
                employeeList.add(new SalariedEmployee(nextEmployeeId++, name, taxRate, designation, bankAcct, salary));
                System.out.println("✅ Salaried Employee added with ID: " + (nextEmployeeId - 1));
            } else if (type.equals("H")) {
                System.out.print("Enter Hourly Rate: ");
                double rate = Double.parseDouble(scanner.nextLine());
                System.out.print("Enter Hours Worked for the month: ");
                double hours = Double.parseDouble(scanner.nextLine());
                employeeList.add(new HourlyEmployee(nextEmployeeId++, name, taxRate, designation, bankAcct, rate, hours));
                System.out.println("✅ Hourly Employee added with ID: " + (nextEmployeeId - 1));
            } else {
                System.err.println("❌ Invalid employee type. Employee not added.");
            }
        } catch (NumberFormatException e) {
            System.err.println("❌ Invalid input for salary/rate! Please enter a numeric value.");
        }
    }

    private static void calculateAllPayroll() {
        System.out.println("\n--- Calculating Monthly Payroll ---");
        if (employeeList.isEmpty()) {
            System.out.println("⚠️ No employees found to process payroll.");
            return;
        }

        System.out.print("Enter payroll month (e.g., OCT-2025): ");
        String month = scanner.nextLine();

        Deduction standardDeduction = new Deduction(2000.0, 3000.0);

        for (Employee emp : employeeList) {
            PayrollRecord record = PayrollProcessor.calculateSalary(emp, standardDeduction, month);
            payrollHistory.add(record);
            System.out.println("Processed: " + record);
        }
        System.out.println("✅ Payroll calculation complete for all employees.");
    }

    private static void generatePayslip() {
        System.out.println("\n--- Payslip / History ---");
        if (payrollHistory.isEmpty()) {
            System.out.println("⚠️ No historical payroll records found.");
            return;
        }
        for (PayrollRecord pr : payrollHistory) {
            System.out.println(pr);
        }
    }

    private static void viewData() {
        System.out.println("\n--- Employee Data ---");
        if (employeeList.isEmpty()) {
            System.out.println("⚠️ No employees in the system.");
            return;
        }
        for (Employee emp : employeeList) {
            System.out.println(emp);
        }
    }

    private static void saveData() {
        try (ObjectOutputStream oosEmp = new ObjectOutputStream(new FileOutputStream(EMP_FILE));
             ObjectOutputStream oosHist = new ObjectOutputStream(new FileOutputStream(HIST_FILE))) {
            
            oosEmp.writeObject(employeeList);
            oosHist.writeObject(payrollHistory);
            
            System.out.println("\n💾 Data saved successfully to " + EMP_FILE + " and " + HIST_FILE);

        } catch (IOException e) {
            System.err.println("❌ File Save Error: Could not save data. " + e.getMessage());
        }
    }

    private static void loadData() {
        try (ObjectInputStream oisEmp = new ObjectInputStream(new FileInputStream(EMP_FILE));
             ObjectInputStream oisHist = new ObjectInputStream(new FileInputStream(HIST_FILE))) {
            
            employeeList = (ArrayList<Employee>) oisEmp.readObject();
            payrollHistory = (ArrayList<PayrollRecord>) oisHist.readObject();
            
            if (!employeeList.isEmpty()) {
                nextEmployeeId = employeeList.get(employeeList.size() - 1).getEmployeeId() + 1;
            }

            System.out.println("✅ Data loaded successfully. " + employeeList.size() + " employees loaded.");

        } catch (FileNotFoundException e) {
            System.out.println("ℹ️ Data files not found. Starting with a fresh system.");
        } catch (IOException | ClassNotFoundException e) {
            System.err.println("❌ File Load Error: An error occurred while loading data. " + e.getMessage());
        }
    }
    
    // --- Nested Classes (Employees) ---
    public static abstract class Employee implements Serializable {
        private static final long serialVersionUID = 1L;
        private int employeeId;
        private String name;
        private double taxRate;
        private String designation;
        private String bankAccountNumber;

        public Employee(int employeeId, String name, double taxRate, String designation, String bankAccountNumber) {
            this.employeeId = employeeId;
            this.name = name;
            this.taxRate = taxRate;
            this.designation = designation;
            this.bankAccountNumber = bankAccountNumber;
        }
        
        public abstract double calculateGrossPay();

        public int getEmployeeId() { return employeeId; }
        public String getName() { return name; }
        public double getTaxRate() { return taxRate; }
        public String getDesignation() { return designation; }

        @Override
        public String toString() {
            return "ID: " + employeeId + ", Name: " + name + ", Designation: " + designation;
        }
    }

    public static class SalariedEmployee extends Employee {
        private double monthlySalary;

        public SalariedEmployee(int employeeId, String name, double taxRate, String designation, String bankAccountNumber, double monthlySalary) {
            super(employeeId, name, taxRate, designation, bankAccountNumber);
            this.monthlySalary = monthlySalary;
        }

        @Override
        public double calculateGrossPay() {
            return monthlySalary;
        }
    }

    public static class HourlyEmployee extends Employee {
        private double hourlyRate;
        private double hoursWorked;

        public HourlyEmployee(int employeeId, String name, double taxRate, String designation, String bankAccountNumber, double hourlyRate, double hoursWorked) {
            super(employeeId, name, taxRate, designation, bankAccountNumber);
            this.hourlyRate = hourlyRate;
            this.hoursWorked = hoursWorked;
        }

        @Override
        public double calculateGrossPay() {
            return hourlyRate * hoursWorked;
        }
    }
    
    // --- Nested Classes (Calculators/Records) ---
    
    public static class Deduction {
        private double healthInsurance;
        private double pensionContribution;

        public Deduction(double healthInsurance, double pensionContribution) {
            this.healthInsurance = healthInsurance;
            this.pensionContribution = pensionContribution;
        }

        public double getTotalDeduction() {
            return healthInsurance + pensionContribution;
        }
    }

    public static class PayrollRecord implements Serializable {
        private static final long serialVersionUID = 1L;
        private int employeeId;
        private String month;
        private double grossPay;
        private double netPay; 

        public PayrollRecord(int employeeId, String month, double grossPay, double netPay) {
            this.employeeId = employeeId;
            this.month = month;
            this.grossPay = grossPay;
            this.netPay = netPay;
        }

        public double getNetPay() { return netPay; }

        @Override
        public String toString() {
            return "Employee ID: " + employeeId + ", Month: " + month + ", Gross: " + String.format("%.2f", grossPay) + ", Net: " + String.format("%.2f", netPay);
        }
    }

    public static class PayrollProcessor {
        public static PayrollRecord calculateSalary(Employee employee, Deduction deduction, String month) {
            double grossPay = employee.calculateGrossPay();
            double totalDeductions = deduction.getTotalDeduction();
            double tax = 0;

            if (grossPay > 50000) {
                tax = grossPay * employee.getTaxRate(); 
            } else if (grossPay > 20000) {
                tax = grossPay * (employee.getTaxRate() * 0.7); 
            }
            
            double netPay = grossPay - totalDeductions - tax;

            return new PayrollRecord(employee.getEmployeeId(), month, grossPay, netPay);
        }
    }
}