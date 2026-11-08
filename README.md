# Warehouse Order Management System

A Java desktop application for managing the clients, products and orders of a warehouse, backed by a PostgreSQL database and built on a strict layered architecture. Its core is a **generic, reflection-based data access layer** that generates SQL CRUD operations for any entity, so adding a new table requires no new SQL code.

## Tech Stack
- **Language:** Java (Maven)
- **Database:** PostgreSQL, accessed through JDBC with `PreparedStatement`
- **GUI:** Java Swing (`JTable`)
- **Key concepts:** Layered Architecture, Generics, Java Reflection, Java Records, Streams & Lambda expressions, Singleton, Javadoc

## Features
- **Client management:** add, edit, delete and view clients, with validation for email format and age limits.
- **Product management:** manage the warehouse inventory (add, edit, delete, view products).
- **Order processing:** create orders by selecting a client and a product; orders that exceed available stock are rejected, and stock is decremented automatically.
- **Immutable billing:** every valid order generates an immutable `Bill` (Java Record) saved in a read-only log table.
- **Dynamic tables:** `TableGenerator` uses Reflection to build `JTable` headers and rows for any list of objects.

## Generic Data Access (the interesting part)
`AbstractDAO<T>` inspects the fields of the model class at runtime with Java Reflection and builds the SQL queries dynamically:

```java
public class ClientDAO extends AbstractDAO<Client> { }   // full CRUD, no SQL written
```

- `findAll()`, `findById()`, `insert()`, `update()`, `delete()` work for **any** entity
- Column lists and `?` placeholders are generated with Streams from the class fields
- All queries use `PreparedStatement`, protecting against SQL injection

## Architecture
```
src/main/java
├── Model/          Client, Product, Orders, Bill (record)
├── Connection/     ConnectionFactory (Singleton, JDBC connection)
├── DataAccess/     AbstractDAO<T> (generic CRUD via Reflection), ClientDAO,
│                   ProductDAO, OrdersDAO, BillDAO
├── BusinessLogic/  ClientBLL, ProductBLL, OrdersBLL
│   └── Validators/ Validator<T>, EmailValidator, ClientAgeValidator,
│                   OrderQuantityValidator, ProductStockValidator
└── Presentation/   MainView, ClientView, ProductView,
                    OrdersManagementView, TableGenerator
```
- A UML diagram is included (`Diagrame.drawio`).
- Full Javadoc documentation is available in the `Javadoc/` folder.

## How to Run
**Prerequisites:** Java (JDK 25), Maven, PostgreSQL

1. Clone the repository:
```bash
   git clone https://github.com/daniilban01/warehouse-order-management.git
```
2. Create the database by importing the SQL dump from `src/main/resources/`:
```bash
   psql -U postgres -d postgres -f src/main/resources/postgres_localhost-2026_05_21_09_15_10-dump.sql
```
3. Set your database credentials as environment variables:
```bash
   export DB_URL=jdbc:postgresql://localhost:5432/postgres
   export DB_USER=postgres
   export DB_PASSWORD=your_password
```
4. Open the project as a Maven project in IntelliJ IDEA and run `Presentation.MainView`.
