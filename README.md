# Hospital Appointment Management (hospital_db)

A small DBMS project made in MySQL. It stores departments, doctors, patients, appointments and bills, and stops two patients from booking the same doctor at the same time.

The sample data (60 patients, 16 doctors, about 230 past appointments) is made up. Phone numbers are dummy numbers.

## Technologies used

| Part | Tool |
|------|------|
| Database | MySQL 8 |
| Tool for running SQL | MySQL Workbench |
| SQL features | Tables, keys, view, trigger, stored procedure, transaction, joins, GROUP BY |
| Data exchange | CSV (import doctors, export bills) |

## Files in this repository

- `hospital_appointments.sql` - creates the database, tables, sample data, trigger, procedure, view and report queries
- `doctors.csv` - extra doctors, imported into the `doctors` table
- `README.md` - this file

## Tables

```
departments (dept_id PK, name)
doctors     (doctor_id PK, name, dept_id FK, fee)
patients    (patient_id PK, name, age, phone)
appointments(appt_id PK, patient_id FK, doctor_id FK, appt_date, slot, status)
bills       (bill_id PK, appt_id FK UNIQUE, amount, billed_on)
```

Relationships:
- one department has many doctors
- one doctor has many appointments
- one patient has many appointments
- one appointment has at most one bill

## Data structures and data types

- **Primary key** with `AUTO_INCREMENT` gives every row a unique id.
- **Foreign key** keeps the tables linked, so an appointment cannot point to a doctor or patient that does not exist.
- **UNIQUE** on `bills.appt_id` means one appointment can only be billed once.
- **ENUM** on `appointments.status` allows only `PENDING`, `CONFIRMED` or `CANCELLED`.
- **CHECK** on `doctors.fee` does not allow a negative fee.
- **DECIMAL(8,2)** is used for money, so there are no rounding mistakes.
- **DATE** and **TIME** store the day and the slot separately.
- Tables are stored by MySQL as indexed tables (the primary key and foreign keys create indexes), so lookups by id are fast.

## How it works

**Trigger `trg_block_double_slot`**
Runs before every insert into `appointments`. If the same doctor already has an active appointment on the same date and slot, it raises error 45000 and the insert is refused. Cancelled appointments are ignored.

**Transaction in `confirm_appointment`**
Changes the status to `CONFIRMED` and creates the bill in one step. The row is locked with `FOR UPDATE`. If anything fails, `ROLLBACK` undoes everything, so there is never a confirmed appointment without a bill.

**View `todays_appointments`**
Shows how many appointments each doctor has today, along with the department.

**Reports**
- patients per department
- doctor workload
- revenue per month

**CSV**
- Import: `doctors.csv` is loaded into the `doctors` table (`LOAD DATA` or the Workbench import wizard).
- Export: the bills report (bill, patient, doctor, date, amount) is exported to a CSV file.

## How we made it

1. Read the assignment and listed what the system needs to store.
2. Drew the tables and relationships (ER diagram), then decided the keys.
3. Wrote the `CREATE TABLE` statements.
4. Added sample data, using relative dates so it always looks recent.
5. Wrote the trigger, then the procedure with the transaction.
6. Created the view and the report queries.
7. Tested the error cases: a double booking and confirming an appointment twice.
8. Did the CSV import and export in Workbench.

## How to run

1. Open MySQL Workbench and connect to your local server.
2. Open `hospital_appointments.sql` and run the whole script (lightning button).
3. Check the result with `SELECT * FROM todays_appointments;`.
4. To test the trigger, insert an appointment for a doctor and slot that is already taken. It should give the error "Doctor already has an appointment in this slot".

Running the script again starts from a clean database, because it drops and recreates `hospital_db`.
