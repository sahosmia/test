-- Repo: erp-sql-database
-- This single file contains all SQL files, README.md, and instructions for the GitHub-ready ERP SQL database project.
-- When copying to separate files, split by the "-- FILE: <path>" markers below.

-- FILE: README.md
-- =================
# 📦 ERP System – MySQL Database (Full SQL Schema)

A complete, production-level ERP system database built with MySQL.
This repository demonstrates advanced SQL skills, relational design, constraints, indexing, and ERP architecture.

## 🚀 Included ERP Modules
- User Management (users, roles, permissions)
- HRM (departments, employees, attendance, payroll)
- CRM (customers, leads)
- Inventory (products, categories, warehouses, stock)
- Procurement (suppliers, purchase_orders, goods_received)
- Sales (sales_orders, invoices, payments)
- Accounting (chart_of_accounts, journal_entries)
- Projects (projects, tasks, time_logs)

## 📁 Project Structure
```
erp-sql-database/
├── README.md
├── schema/
│   ├── 01_create_database.sql
│   ├── 02_users_roles.sql
│   ├── 03_hrm.sql
│   ├── 04_crm.sql
│   ├── 05_inventory.sql
│   ├── 06_procurement.sql
│   ├── 07_sales.sql
│   ├── 08_accounting.sql
│   ├── 09_projects.sql
│   ├── 10_relationships.sql
│   └── 11_sample_data.sql
├── diagrams/
│   └── erp_er_diagram.png  (generate with dbdiagram.io / draw.io)
└── docs/
    ├── module_overview.md
    └── table_descriptions.md
```

## 🧪 How to Use
1. Create DB in phpMyAdmin or MySQL client:
```sql
CREATE DATABASE erp_system; -- or import 01_create_database.sql
USE erp_system;
```
2. Import schema files in numeric order (02 → 09). Finally import 11_sample_data.sql for sample data.
3. Run queries from `queries/` (you can create a `queries/` folder in the repo for practice queries).

## 📊 ER Diagram
I included an ER diagram template path `diagrams/erp_er_diagram.png` — generate a PNG using any ERD tool (dbdiagram.io, draw.io, MySQL Workbench) from the schema files and save it there.

## 📅 Roadmap
- Add views for reports (revenue, ageing receivables)
- Add stored procedures for invoice generation
- Add triggers for audit logs
- Add indexing and EXPLAIN notes

---

-- FILE: schema/01_create_database.sql
CREATE DATABASE IF NOT EXISTS erp_system CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE erp_system;

-- FILE: schema/02_users_roles.sql
-- Users, Roles, Permissions (simple RBAC)
CREATE TABLE roles (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL UNIQUE,
    description VARCHAR(255)
);

CREATE TABLE permissions (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE,
    description VARCHAR(255)
);

CREATE TABLE role_permissions (
    role_id INT NOT NULL,
    permission_id INT NOT NULL,
    PRIMARY KEY(role_id, permission_id),
    FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE,
    FOREIGN KEY (permission_id) REFERENCES permissions(id) ON DELETE CASCADE
);

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(100) NOT NULL UNIQUE,
    email VARCHAR(150) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    full_name VARCHAR(150),
    active TINYINT(1) DEFAULT 1,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE user_roles (
    user_id INT NOT NULL,
    role_id INT NOT NULL,
    PRIMARY KEY(user_id, role_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE
);

-- FILE: schema/03_hrm.sql
-- Departments
CREATE TABLE departments (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE,
    description VARCHAR(255)
);

-- Employees
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    employee_code VARCHAR(50) NOT NULL UNIQUE,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    department_id INT,
    manager_id INT,
    hire_date DATE,
    salary DECIMAL(12,2) DEFAULT 0,
    email VARCHAR(150),
    phone VARCHAR(50),
    active TINYINT(1) DEFAULT 1,
    FOREIGN KEY (department_id) REFERENCES departments(id),
    FOREIGN KEY (manager_id) REFERENCES employees(id)
);

-- Attendance
CREATE TABLE attendance (
    id INT AUTO_INCREMENT PRIMARY KEY,
    employee_id INT NOT NULL,
    date DATE NOT NULL,
    check_in TIME,
    check_out TIME,
    status ENUM('Present','Absent','Leave','Remote') DEFAULT 'Present',
    FOREIGN KEY (employee_id) REFERENCES employees(id)
);

-- Payroll (simplified)
CREATE TABLE payrolls (
    id INT AUTO_INCREMENT PRIMARY KEY,
    employee_id INT NOT NULL,
    period_start DATE,
    period_end DATE,
    gross DECIMAL(12,2),
    deductions DECIMAL(12,2),
    net_pay DECIMAL(12,2),
    paid_date DATE,
    FOREIGN KEY (employee_id) REFERENCES employees(id)
);

-- FILE: schema/04_crm.sql
-- Customers & Leads
CREATE TABLE customers (
    id INT AUTO_INCREMENT PRIMARY KEY,
    customer_code VARCHAR(50) UNIQUE,
    name VARCHAR(150) NOT NULL,
    email VARCHAR(150),
    phone VARCHAR(50),
    address VARCHAR(255),
    city VARCHAR(100),
    country VARCHAR(100),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE leads (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(150),
    email VARCHAR(150),
    phone VARCHAR(50),
    source VARCHAR(100),
    status ENUM('New','Contacted','Qualified','Lost') DEFAULT 'New',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- FILE: schema/05_inventory.sql
-- Product catalog
CREATE TABLE product_categories (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    parent_id INT,
    FOREIGN KEY (parent_id) REFERENCES product_categories(id)
);

CREATE TABLE products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    sku VARCHAR(100) UNIQUE,
    name VARCHAR(255) NOT NULL,
    category_id INT,
    unit_price DECIMAL(12,2),
    description TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (category_id) REFERENCES product_categories(id)
);

CREATE TABLE warehouses (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(150) NOT NULL,
    location VARCHAR(255)
);

-- Stock ledger per warehouse
CREATE TABLE stocks (
    id INT AUTO_INCREMENT PRIMARY KEY,
    product_id INT NOT NULL,
    warehouse_id INT NOT NULL,
    qty INT DEFAULT 0,
    last_updated DATETIME DEFAULT CURRENT_TIMESTAMP,
    UNIQUE KEY product_warehouse (product_id, warehouse_id),
    FOREIGN KEY (product_id) REFERENCES products(id),
    FOREIGN KEY (warehouse_id) REFERENCES warehouses(id)
);

-- Stock transactions (audit trail)
CREATE TABLE stock_movements (
    id INT AUTO_INCREMENT PRIMARY KEY,
    product_id INT NOT NULL,
    warehouse_id INT NOT NULL,
    movement_type ENUM('IN','OUT','ADJUST') NOT NULL,
    qty INT NOT NULL,
    reference VARCHAR(100),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (product_id) REFERENCES products(id),
    FOREIGN KEY (warehouse_id) REFERENCES warehouses(id)
);

-- FILE: schema/06_procurement.sql
CREATE TABLE suppliers (
    id INT AUTO_INCREMENT PRIMARY KEY,
    supplier_code VARCHAR(50) UNIQUE,
    name VARCHAR(150) NOT NULL,
    contact_name VARCHAR(150),
    phone VARCHAR(50),
    email VARCHAR(150),
    address VARCHAR(255)
);

CREATE TABLE purchase_orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    po_number VARCHAR(100) UNIQUE,
    supplier_id INT,
    order_date DATE,
    expected_date DATE,
    status ENUM('Draft','Ordered','Received','Cancelled') DEFAULT 'Draft',
    total DECIMAL(12,2) DEFAULT 0,
    FOREIGN KEY (supplier_id) REFERENCES suppliers(id)
);

CREATE TABLE purchase_order_items (
    id INT AUTO_INCREMENT PRIMARY KEY,
    purchase_order_id INT NOT NULL,
    product_id INT NOT NULL,
    qty INT NOT NULL,
    unit_price DECIMAL(12,2) NOT NULL,
    FOREIGN KEY (purchase_order_id) REFERENCES purchase_orders(id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);

CREATE TABLE goods_received_notes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    grn_number VARCHAR(100) UNIQUE,
    purchase_order_id INT,
    received_date DATE,
    warehouse_id INT,
    FOREIGN KEY (purchase_order_id) REFERENCES purchase_orders(id),
    FOREIGN KEY (warehouse_id) REFERENCES warehouses(id)
);

-- FILE: schema/07_sales.sql
CREATE TABLE sales_orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    so_number VARCHAR(100) UNIQUE,
    customer_id INT,
    order_date DATE,
    status ENUM('Draft','Confirmed','Shipped','Completed','Cancelled') DEFAULT 'Draft',
    total DECIMAL(12,2) DEFAULT 0,
    FOREIGN KEY (customer_id) REFERENCES customers(id)
);

CREATE TABLE sales_order_items (
    id INT AUTO_INCREMENT PRIMARY KEY,
    sales_order_id INT NOT NULL,
    product_id INT NOT NULL,
    qty INT NOT NULL,
    unit_price DECIMAL(12,2) NOT NULL,
    FOREIGN KEY (sales_order_id) REFERENCES sales_orders(id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);

CREATE TABLE invoices (
    id INT AUTO_INCREMENT PRIMARY KEY,
    invoice_number VARCHAR(100) UNIQUE,
    sales_order_id INT,
    invoice_date DATE,
    due_date DATE,
    total DECIMAL(12,2) DEFAULT 0,
    status ENUM('Unpaid','Paid','Partial','Cancelled') DEFAULT 'Unpaid',
    FOREIGN KEY (sales_order_id) REFERENCES sales_orders(id)
);

CREATE TABLE payments (
    id INT AUTO_INCREMENT PRIMARY KEY,
    invoice_id INT NOT NULL,
    payment_date DATE,
    amount DECIMAL(12,2) NOT NULL,
    method VARCHAR(50),
    reference VARCHAR(100),
    FOREIGN KEY (invoice_id) REFERENCES invoices(id)
);

-- FILE: schema/08_accounting.sql
CREATE TABLE chart_of_accounts (
    id INT AUTO_INCREMENT PRIMARY KEY,
    account_code VARCHAR(50) UNIQUE,
    name VARCHAR(150) NOT NULL,
    type ENUM('Asset','Liability','Equity','Revenue','Expense') NOT NULL
);

CREATE TABLE journal_entries (
    id INT AUTO_INCREMENT PRIMARY KEY,
    entry_date DATE NOT NULL,
    description VARCHAR(255)
);

CREATE TABLE journal_lines (
    id INT AUTO_INCREMENT PRIMARY KEY,
    journal_entry_id INT NOT NULL,
    account_id INT NOT NULL,
    debit DECIMAL(12,2) DEFAULT 0,
    credit DECIMAL(12,2) DEFAULT 0,
    FOREIGN KEY (journal_entry_id) REFERENCES journal_entries(id),
    FOREIGN KEY (account_id) REFERENCES chart_of_accounts(id)
);

-- FILE: schema/09_projects.sql
CREATE TABLE projects (
    id INT AUTO_INCREMENT PRIMARY KEY,
    project_code VARCHAR(100) UNIQUE,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    start_date DATE,
    end_date DATE,
    status ENUM('Planned','Active','Completed','On Hold','Cancelled') DEFAULT 'Planned'
);

CREATE TABLE tasks (
    id INT AUTO_INCREMENT PRIMARY KEY,
    project_id INT NOT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    assigned_to INT,
    status ENUM('Todo','In Progress','Done','Blocked') DEFAULT 'Todo',
    estimated_hours DECIMAL(6,2),
    spent_hours DECIMAL(6,2) DEFAULT 0,
    FOREIGN KEY (project_id) REFERENCES projects(id),
    FOREIGN KEY (assigned_to) REFERENCES employees(id)
);

CREATE TABLE time_logs (
    id INT AUTO_INCREMENT PRIMARY KEY,
    task_id INT NOT NULL,
    employee_id INT NOT NULL,
    log_date DATE NOT NULL,
    hours DECIMAL(5,2) NOT NULL,
    notes VARCHAR(255),
    FOREIGN KEY (task_id) REFERENCES tasks(id),
    FOREIGN KEY (employee_id) REFERENCES employees(id)
);

-- FILE: schema/10_relationships.sql
-- (Most relationships already declared inline; extra indexes and constraints)

-- Indexes for faster lookup
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_employees_dept ON employees(department_id);
CREATE INDEX idx_products_sku ON products(sku);
CREATE INDEX idx_stocks_prod_wh ON stocks(product_id, warehouse_id);
CREATE INDEX idx_sales_orders_customer ON sales_orders(customer_id);
CREATE INDEX idx_invoices_so ON invoices(sales_order_id);

-- FILE: schema/11_sample_data.sql
-- Sample: Roles & Permissions
INSERT INTO roles (name, description) VALUES
('Admin', 'Full system access'),
('Manager', 'Manager level access'),
('Staff', 'General staff');

INSERT INTO permissions (name, description) VALUES
('manage_users', 'Manage users and roles'),
('view_reports', 'View financial and operational reports'),
('manage_inventory', 'Add/Edit stock and products'),
('process_sales', 'Create sales orders and invoices');

INSERT INTO role_permissions (role_id, permission_id) VALUES
(1,1),(1,2),(1,3),(1,4),
(2,2),(2,3),(2,4),
(3,3),(3,4);

-- Sample users
INSERT INTO users (username, email, password_hash, full_name) VALUES
('admin','admin@example.com','$2y$10$examplehash','System Admin'),
('manager','manager@example.com','$2y$10$examplehash','Project Manager');

INSERT INTO user_roles (user_id, role_id) VALUES (1,1),(2,2);

-- Departments & Employees
INSERT INTO departments (name, description) VALUES
('Engineering','Software Engineering'),
('Sales','Sales Department'),
('Finance','Finance & Accounts');

INSERT INTO employees (employee_code, first_name, last_name, department_id, hire_date, salary, email) VALUES
('EMP001','Amin','Ahmed',1,'2021-01-10',60000,'amin@example.com'),
('EMP002','Karim','Hossain',2,'2020-05-20',45000,'karim@example.com'),
('EMP003','Rina','Khatun',3,'2019-11-01',55000,'rina@example.com');

-- Attendance sample
INSERT INTO attendance (employee_id, date, check_in, check_out, status) VALUES
(1,'2024-08-01','09:05:00','17:10:00','Present'),
(2,'2024-08-01','09:15:00','17:00:00','Present');

-- Payroll sample
INSERT INTO payrolls (employee_id, period_start, period_end, gross, deductions, net_pay, paid_date) VALUES
(1,'2024-07-01','2024-07-31',60000,5000,55000,'2024-08-05');

-- CRM sample
INSERT INTO customers (customer_code, name, email, phone, city, country) VALUES
('CUST001','Acme Corp','contact@acme.com','0123456789','Dhaka','Bangladesh'),
('CUST002','Beta LLC','info@beta.com','0987654321','Chittagong','Bangladesh');

INSERT INTO leads (name, email, phone, source) VALUES
('Lead One','lead1@example.com','01700000001','Website');

-- Inventory sample
INSERT INTO product_categories (name) VALUES ('Electronics'),('Office Supplies');
INSERT INTO products (sku, name, category_id, unit_price) VALUES
('SKU-001','Laptop',1,65000),
('SKU-002','Office Chair',2,8000);

INSERT INTO warehouses (name, location) VALUES ('Main Warehouse','Dhaka'),('Secondary Warehouse','Chittagong');

INSERT INTO stocks (product_id, warehouse_id, qty) VALUES (1,1,10),(1,2,5),(2,1,20);

-- Procurement sample
INSERT INTO suppliers (supplier_code, name, contact_name, phone) VALUES
('SUP001','Tech Supplier','Mr Tech','01911111111');

INSERT INTO purchase_orders (po_number, supplier_id, order_date, expected_date, status, total) VALUES
('PO-1001',1,'2024-07-01','2024-07-10','Ordered',520000);

INSERT INTO purchase_order_items (purchase_order_id, product_id, qty, unit_price) VALUES (1,1,10,50000);

-- Sales sample
INSERT INTO sales_orders (so_number, customer_id, order_date, status, total) VALUES
('SO-2001',1,'2024-07-15','Confirmed',130000);

INSERT INTO sales_order_items (sales_order_id, product_id, qty, unit_price) VALUES (1,1,2,65000);

INSERT INTO invoices (invoice_number, sales_order_id, invoice_date, due_date, total, status) VALUES
('INV-3001',1,'2024-07-16','2024-08-16',130000,'Unpaid');

INSERT INTO payments (invoice_id, payment_date, amount, method) VALUES (1,'2024-07-20',50000,'Bank Transfer');

-- Accounting sample
INSERT INTO chart_of_accounts (account_code, name, type) VALUES
('1000','Cash','Asset'),
('2000','Accounts Payable','Liability'),
('4000','Sales Revenue','Revenue');

INSERT INTO journal_entries (entry_date, description) VALUES
('2024-07-16','Invoice INV-3001');

INSERT INTO journal_lines (journal_entry_id, account_id, debit, credit) VALUES
(1,1,130000,0),(1,3,0,130000);

-- Projects sample
INSERT INTO projects (project_code, name, start_date, status) VALUES
('PRJ001','Website Redesign','2024-07-01','Active');

INSERT INTO tasks (project_id, title, assigned_to, estimated_hours) VALUES
(1,'Design Mockups',1,40),(1,'Frontend Implementation',1,120);

INSERT INTO time_logs (task_id, employee_id, log_date, hours) VALUES
(1,1,'2024-07-05',5),(2,1,'2024-07-06',6);

-- End of sample data

-- Helpful note: split this file into the listed files when creating your repository.
-- You can generate ER diagram with dbdiagram.io using the table definitions above or use MySQL Workbench to export PNG.
