# Entity Framework Core - Database Migrations

## Database Creation Approaches

### 1. EnsureCreated/EnsureDeleted Method
```csharp
using var dbContext = new CompanyDbContext();

// Delete existing database
dbContext.Database.EnsureDeleted();

// Create new database
dbContext.Database.EnsureCreated();
```

#### Limitations
- Deletes entire database
- No change tracking
- Not suitable for production
- Cannot track incremental changes

## Migration System

### Setting Up Migrations

1. **Install Required Package**
```mermaid
graph TD
    A[Visual Studio] --> B[Manage NuGet Packages]
    B --> C[Browse Tab]
    C --> D[Microsoft.EntityFrameworkCore.Tools]
    D --> E[Install v6.x]
```

#### Package Installation Steps
1. Right-click project
2. Select "Manage NuGet Packages"
3. Go to "Browse" tab
4. Search: `Microsoft.EntityFrameworkCore.Tools`
5. Install version 6.x
6. Accept terms

### Creating Migrations

```powershell
# Package Manager Console command
Add-Migration InitialCreate
```

### Migration Structure

```
ProjectName/
├── Migrations/
│   ├── YYYYMMDDHHMMSS_InitialCreate.cs
│   └── CompanyDbContextModelSnapshot.cs
```

#### Migration Class
```csharp
public partial class InitialCreate : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        // Changes to apply when upgrading
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        // Changes to apply when downgrading
    }
}
```

#### Snapshot Class
- Created with first migration
- Updated with subsequent migrations
- Tracks current model state
- Used for comparison to detect changes

```mermaid
graph TD
    A[Code Changes] --> B[Snapshot Comparison]
    B --> C[Detect Changes]
    C --> D[Generate Migration]
    D --> E[Up/Down Methods]
    style B fill:#f9f,stroke:#333,stroke-width:4px
```

## How Migrations Work

### Change Detection Process
```mermaid
graph LR
    A[Current Code] --> B[Compare]
    C[Snapshot] --> B
    B --> D[Generate Migration]
    D --> E[Update Snapshot]
```

### Migration Components
1. **Up Method**
   - Forward migration
   - Applies changes
   - Creates/modifies tables
   - Adds/removes columns

2. **Down Method**
   - Rollback migration
   - Reverts changes
   - Drops tables
   - Removes modifications

### Best Practices

1. **Migration Naming**
   - Use descriptive names
   - Include purpose
   - Follow naming convention
   - Example: `Add-Migration AddEmployeeSalaryColumn`

2. **Migration Management**
   - Regular migrations for changes
   - Small, focused migrations
   - Test migrations before production
   - Keep migrations in source control

3. **Development Workflow**
   - Make model changes
   - Add migration
   - Review generated code
   - Apply migration
   - Test changes

4. **Production Considerations**
   - Backup database before migration
   - Test migrations in staging
   - Plan for rollback scenarios
   - Monitor migration performance

## Common Scenarios

### Creating Initial Schema
```powershell
Add-Migration InitialCreate
```

### Adding New Properties
```powershell
Add-Migration AddEmployeeEmail
```

### Modifying Existing Schema
```powershell
Add-Migration UpdateEmployeeTable
```

### Rolling Back Changes
```powershell
# Use Package Manager Console
Update-Database LastGoodMigrationName
```



# Entity Framework Core - Migration Execution and Database Management

## Migration Files Structure

```
ProjectName/
├── Migrations/
│   ├── YYYYMMDDHHMMSS_InitialCreate.cs        # Generated migration
│   ├── InitialCreate.CustomLogic.cs           # Custom partial class (optional)
│   └── CompanyDbContextModelSnapshot.cs       # Current model state
```

## Migration Class Structure

```csharp
public partial class InitialCreate : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        // Creates tables only (not database)
        migrationBuilder.CreateTable(
            name: "Employees",
            columns: table => new table
            {
                // Column definitions
            });
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        // Drops tables only (not database)
        migrationBuilder.DropTable(
            name: "Employees");
    }
}
```

## Applying Migrations

### Method 1: Programmatic Approach
```csharp
// In Program.cs
using var dbContext = new CompanyDbContext();
dbContext.Database.Migrate();  // Applies all pending migrations
```

### Method 2: Package Manager Console (Recommended)
```powershell
Update-Database
```

```mermaid
graph TD
    A[Update-Database Command] --> B[Check Pending Migrations]
    B --> C[Create Database if Needed]
    C --> D[Execute Up Methods]
    D --> E[Record in __EFMigrationsHistory]
    style D fill:#f9f,stroke:#333,stroke-width:4px
```

## Database Creation Process

1. **First Migration Execution**
   - EF Core creates database
   - Executes Up method
   - Creates required tables

2. **Subsequent Migrations**
   - Only executes Up method
   - Modifies existing schema
   - Database remains intact

## Generated Database Structure

### System Tables
```mermaid
erDiagram
    __EFMigrationsHistory {
        string MigrationId PK
        string ProductVersion
    }
    Employees {
        int Id PK
        string Name
        double Salary
        int Age
    }
```

### 1. __EFMigrationsHistory Table
- Tracks applied migrations
- Contains:
  - MigrationId (Timestamp_MigrationName)
  - ProductVersion (EF Tools version)
- Ensures unique migrations

### 2. Entity Tables
- Generated from entity classes
- Follow defined schema
- Include constraints and relationships

## Monitoring and Debugging

### SQL Server Profiler
- Tracks executed SQL queries
- Shows migration commands
- Useful for development debugging

### Management Studio View
- Verify database creation
- Check table structures
- Monitor migration history

## Best Practices

1. **Migration Management**
   - Use Package Manager Console for migrations
   - Keep migrations small and focused
   - Review generated SQL before applying

2. **Partial Classes**
   - Don't modify generated migration files
   - Use partial classes for custom logic
   - Keep custom logic separate

3. **Version Control**
   - Include migrations in source control
   - Document significant changes
   - Track EF Tools version used

4. **Monitoring**
   - Review migration history
   - Check database schema
   - Validate data integrity

## Common Commands and Operations

```powershell
# Apply all pending migrations
Update-Database

# Roll back to specific migration
Update-Database MigrationName

# Generate SQL script
Script-Migration

# Remove last migration (if not applied)
Remove-Migration
```

## Migration History Table Example
```sql
SELECT MigrationId, ProductVersion
FROM __EFMigrationsHistory
ORDER BY MigrationId
```


# Entity Framework Core - Advanced Migration Management

## Handling Schema Changes

### Example: Renaming Column
```csharp
// Original
public string Name { get; set; }

// Changed to
public string EmpName { get; set; }
```

```mermaid
sequenceDiagram
    participant Code
    participant Migration
    participant Database
    Code->>Migration: Add-Migration ChangeEmployeeNameColumn
    Migration->>Database: Update-Database
    Note over Migration: Compares snapshot with current code
    Note over Database: Renames column Name to EmpName
```

## Migration Commands

### Adding New Migrations
```powershell
# Generate migration for column rename
Add-Migration ChangeEmployeeNameColumn
```

Generated Migration:
```csharp
public partial class ChangeEmployeeNameColumn : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.RenameColumn(
            name: "Name",
            table: "Employees",
            newName: "EmpName");
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.RenameColumn(
            name: "EmpName",
            table: "Employees",
            newName: "Name");
    }
}
```

## Migration Management Flow

```mermaid
graph TD
    A[Migration Stack] --> B{Applied?}
    B -->|Yes| C[Update-Database]
    B -->|No| D[Remove-Migration]
    C --> E[Roll Back]
    E --> D
    style B fill:#f9f,stroke:#333,stroke-width:4px
```

## Removing Migrations

### 1. Unapplied Migrations
```powershell
# Removes last unapplied migration
Remove-Migration
```

### 2. Applied Migrations
Step 1: Roll back to previous migration
```powershell
# Roll back to specific migration
Update-Database -Migration "PreviousMigrationName"
```

Step 2: Remove rolled back migration
```powershell
Remove-Migration
```

## Rolling Back Migrations

### To Specific Migration
```powershell
# Roll back to InitialCreate
Update-Database -Migration "InitialCreate"
```

### Remove All Migrations
```powershell
# Roll back all migrations
Update-Database 0

# Remove migration files
Remove-Migration
```

## Migration Stack Behavior

```mermaid
graph TD
    A[Migration3] --> B[Migration2]
    B --> C[Migration1]
    C --> D[InitialCreate]
    style A fill:#f9f,stroke:#333,stroke-width:4px
```

### Rules for Removal
1. Can only remove latest migration
2. Must roll back applied migrations first
3. Snapshot updates automatically
4. Stack-like behavior (LIFO)

## Common Scenarios

### Scenario 1: Remove Last Unapplied Migration
```powershell
Remove-Migration
```

### Scenario 2: Remove Multiple Migrations
```powershell
# Remove Migration3 and Migration2
Remove-Migration  # Removes Migration3
Remove-Migration  # Removes Migration2
```

### Scenario 3: Remove Applied Migration
```powershell
# Roll back to previous migration
Update-Database -Migration "Migration1"

# Remove rolled back migration
Remove-Migration
```

### Scenario 4: Complete Reset
```powershell
# Remove all migrations
Update-Database 0
Remove-Migration
```

## Best Practices

1. **Before Removing Migrations**
   - Check if migration is applied
   - Back up database if needed
   - Document changes

2. **Rolling Back**
   - Test rollback in development
   - Plan for data preservation
   - Consider dependencies

3. **Migration Management**
   - Keep migrations small
   - Regular cleanup of unused migrations
   - Maintain migration history

4. **Error Prevention**
   - Verify current database state
   - Check for dependent migrations
   - Test rollback procedures

## Common Issues and Solutions

| Issue | Solution |
|-------|----------|
| Migration already applied | Roll back first using Update-Database |
| Cannot remove middle migration | Roll back and remove in order |
| Snapshot out of sync | Remove unapplied migrations |
| Database inconsistency | Use Update-Database 0 to reset |
