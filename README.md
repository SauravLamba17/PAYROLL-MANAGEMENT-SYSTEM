💰 Payroll Management System (Java)

A robust, console-based application designed to automate employee salary processing and data management. This project demonstrates a strong command of Object-Oriented Programming (OOP), File I/O, and Data Persistence in Java.

🌟 Key Features
Hybrid Employee Support: Utilizes inheritance to manage both Salaried and Hourly employees with distinct pay structures.
Automated Payroll Engine: Calculates gross pay, tax deductions (tiered based on income), and standard contributions (Health/Pension).
Data Persistence: Implements Java Serialization to save and load records from .dat files, ensuring data is retained across sessions.
Historical Tracking: Generates and stores monthly payslips for easy retrieval and audit.
Robust Error Handling: Features comprehensive try-catch blocks and InputMismatchException handling for a crash-proof user experience.

🛠️ Technical Stack
Language: Java (JDK 8+)
Core Concepts: * OOP: Abstraction, Polymorphism, and Encapsulation.
Collections: ArrayList for dynamic data management.
I/O: ObjectOutputStream / ObjectInputStream for binary file storage.
Logic: Tiered tax algorithms and monthly record snapshots.

📂 Project Structure
Employee: Abstract base class defining the core identity of a staff member.
PayrollProcessor: Logic layer that handles complex financial calculations.
PayrollRecord: A serializable data object representing a processed payment.
PayrollSystem: The main driver class managing the CLI and file operations.
