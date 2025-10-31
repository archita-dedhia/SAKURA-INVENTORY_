(https://github.com/archita-dedhia/SAKURA-INVENTORY_/new/main?readme=1)
# Sakura Inventory Management System

## System Overview
Sakura Inventory is a web-based inventory management system built using Jakarta EE (formerly Java EE) that allows multiple businesses to manage their products and categories.

## Technical Stack
- Jakarta EE 6.0
- Apache Tomcat 11.0.13
- MySQL 8.0
- JSP (Jakarta Server Pages)
- JDBC for database connectivity

## Database Structure

### Database Tables
1. **users**
   - Stores business account information
   - Contains authentication details
   - Fields: id, business_name, email, password_hash, role

2. **categories**
   - Stores product categories for each business
   - Fields: id, name, description, business_name

3. **products**
   - Stores product information
   - Links to categories via category_id
   - Fields: id, name, description, price, quantity, category_id, business_name

## System Flow

### 1. User Authentication Flow
1. User visits the application at `http://localhost:8080/SAKURA-INVENTORY/`
2. System displays login page (index.jsp)
3. User enters email and password
4. AuthServlet processes the login:
   - Validates credentials against users table
   - Creates session with business_name if successful
   - Redirects to products.jsp on success
   - Returns to login with error message on failure

### 2. Product Management Flow
1. After login, user sees products.jsp
2. User can:
   - View all products for their business
   - Add new products via product-form.jsp
   - Edit existing products
   - Delete products
3. ProductServlet handles all product operations
4. Products are filtered by business_name to ensure data isolation

### 3. Category Management Flow
1. Access categories via categories.jsp
2. User can:
   - View all categories for their business
   - Add new categories via category-form.jsp
   - Edit existing categories
   - Delete categories (products will have category set to NULL)
3. CategoryServlet handles all category operations

## Setup Instructions

### 1. Database Setup
1. Install MySQL 8.0 or higher
2. Run the following commands:
   ```sql
   CREATE DATABASE sakura_inventory;
   USE sakura_inventory;
   ```
3. Run all commands from database_setup.sql to create tables

### 2. Creating Users
1. **Method 1: Direct Database Insert**
   ```sql
   INSERT INTO users (business_name, email, password_hash, role)
   VALUES ('Your Business Name', 'your@email.com', 'your_password', 'admin');
   ```

2. **Method 2: Using Application**
   - Currently, users must be created through database directly
   - Future versions will include user registration

### 3. Table Auto-generation
- Tables are NOT auto-generated
- Must be created using database_setup.sql
- Run the CREATE TABLE statements in order:
  1. users
  2. categories
  3. products (depends on categories for foreign key)

### 4. Data Flow and Access Control

#### Login Process
1. User enters email/password
2. System checks users table:
   ```sql
   SELECT business_name, password_hash 
   FROM users 
   WHERE email = ?
   ```
3. If credentials match:
   - Creates session with business_name
   - This business_name is used to filter all future queries

#### Data Isolation
- All tables have business_name column
- All queries include WHERE business_name = ? clause
- Example product query:
  ```sql
  SELECT * FROM products 
  WHERE business_name = 'Your Business'
  ```

## File Structure
```
SAKURA-INVENTORY/
├── WEB-INF/
│   ├── classes/          # Compiled Java files
│   ├── lib/             # Required JAR files
│   └── web.xml          # Web application configuration
├── servlet/             # Servlet source files
│   ├── AuthServlet.java
│   ├── ProductServlet.java
│   └── CategoryServlet.java
├── *.jsp               # Web pages
├── database_setup.sql  # Database setup commands
└── README.md          # This documentation
```

## Common Operations

### Adding Products
1. Click "Add Product" on products.jsp
2. Fill form in product-form.jsp
3. ProductServlet processes form:
   ```sql
   INSERT INTO products (name, description, price, quantity, category_id, business_name)
   VALUES (?, ?, ?, ?, ?, ?)
   ```

### Managing Categories
1. Access Categories page
2. Add/Edit/Delete categories
3. Changes reflect in products through foreign key relationship

## Troubleshooting

### Common Issues
1. **404 Error**
   - Check if servlets are compiled in WEB-INF/classes
   - Verify web.xml mappings

2. **Database Connection Errors**
   - Verify MySQL is running
   - Check connection string in JSP/Servlet files
   - Ensure MySQL JDBC driver is in WEB-INF/lib

3. **Login Issues**
   - Verify user exists in database
   - Check email and password match
   - Ensure session handling is working

### Database Verification
To verify user data:
```sql
SELECT * FROM users WHERE email = 'user@email.com';
```

To check product access:
```sql
SELECT p.*, c.name as category_name 
FROM products p 
LEFT JOIN categories c ON p.category_id = c.id 
WHERE p.business_name = 'Your Business';
```

## Security Notes
- Passwords are currently stored as plain text (not recommended for production)
- Future updates should include:
  - Password hashing
  - Input validation
  - XSS protection
  - CSRF tokens
