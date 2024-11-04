## Repository Pattern in C# DotNet 8

- The Repository Pattern is a design pattern that provides an abstraction layer between the data access layer and the business logic layer of an application. It helps to manage data operations in a consistent way, making the code more maintainable and testable.

## Unit of Work

- The Unit of Work pattern is used to group multiple operations into a single transaction. This ensures that either all operations succeed or none of them do, maintaining the integrity of the database. It's particularly used when you have multiple repositories interacting with the same database context.

## Generic Repository

- The Generic Repository pattern abstracts common data operations into a single repository, reducing redundancy. Instead of having separate repositories for each entity type, you have one generic repository that handles CRUD operations for all entities.

### Uses

- Separation of Concerns: These patterns help to separate the concerns of data access and business logic, making the code more modular and easier to maintain.
- Testability: By abstracting data operations, it becomes easier to mock these operations in unit tests.
- Maintainability: Changes to the data access logic can be made in one place without affecting the rest of the application.
- Consistency: Ensures consistent data operations across the application.

---

### Entities

```csharp
// Models/Employee.cs
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int DepartmentId { get; set; }
    public Department Department { get; set; }
}

// Models/Department.cs
public class Department
{
    public int Id { get; set; }
    public string DepartmentName { get; set; }
    public ICollection<Employee> Employees { get; set; }
}
```

### AppDbContext

```csharp
// Data/AppDbContext.cs
using Microsoft.EntityFrameworkCore;

public class AppDbContext : DbContext
{
    public DbSet<Employee> Employees { get; set; }
    public DbSet<Department> Departments { get; set; }

    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
}
```

### Generic Repository Interface

```csharp
// Repositories/IGenericRepository.cs
public interface IGenericRepository<T> where T : class
{
    Task<IEnumerable<T>> GetAllAsync();
    Task<T> GetByIdAsync(int id);
    Task AddAsync(T entity);
    void Update(T entity);
    void Delete(T entity);
}
```

### Generic Repository

```csharp
// Repositories/GenericRepository.cs
using Microsoft.EntityFrameworkCore;

public class GenericRepository<T> : IGenericRepository<T> where T : class
{
    private readonly AppDbContext _context;
    private readonly DbSet<T> _dbSet;

    public GenericRepository(AppDbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }

    public async Task<IEnumerable<T>> GetAllAsync() => await _dbSet.ToListAsync();

    public async Task<T> GetByIdAsync(int id) => await _dbSet.FindAsync(id);

    public async Task AddAsync(T entity) => await _dbSet.AddAsync(entity);

    public void Update(T entity) => _dbSet.Update(entity);

    public void Delete(T entity) => _dbSet.Remove(entity);
}
```

### Unit of Work Interface

```csharp
// Repositories/IUnitOfWork.cs
public interface IUnitOfWork : IDisposable
{
    IGenericRepository<Employee> Employees { get; }
    IGenericRepository<Department> Departments { get; }
    Task<int> CompleteAsync();
}
```

### Unit of Work

```csharp
// Repositories/UnitOfWork.cs
public class UnitOfWork : IUnitOfWork
{
    private readonly AppDbContext _context;
    private GenericRepository<Employee> _employees;
    private GenericRepository<Department> _departments;

    public UnitOfWork(AppDbContext context)
    {
        _context = context;
    }

    public IGenericRepository<Employee> Employees => _employees ??= new GenericRepository<Employee>(_context);
    public IGenericRepository<Department> Departments => _departments ??= new GenericRepository<Department>(_context);

    public async Task<int> CompleteAsync() => await _context.SaveChangesAsync();

    public void Dispose() => _context.Dispose();
}
```

### Service Layer

```csharp
// Services/EmployeeService.cs
public class EmployeeService
{
    private readonly IUnitOfWork _unitOfWork;

    public EmployeeService(IUnitOfWork unitOfWork)
    {
        _unitOfWork = unitOfWork;
    }

    public async Task<IEnumerable<Employee>> GetAllEmployeesAsync()
    {
        return await _unitOfWork.Employees.GetAllAsync();
    }

    public async Task<Employee> GetEmployeeByIdAsync(int id)
    {
        return await _unitOfWork.Employees.GetByIdAsync(id);
    }

    public async Task AddEmployeeAsync(Employee employee)
    {
        await _unitOfWork.Employees.AddAsync(employee);
        await _unitOfWork.CompleteAsync();
    }

    public async Task UpdateEmployeeAsync(Employee employee)
    {
        _unitOfWork.Employees.Update(employee);
        await _unitOfWork.CompleteAsync();
    }

    public async Task DeleteEmployeeAsync(int id)
    {
        var employee = await _unitOfWork.Employees.GetByIdAsync(id);
        if (employee != null)
        {
            _unitOfWork.Employees.Delete(employee);
            await _unitOfWork.CompleteAsync();
        }
    }
}
```

### API Controller

```csharp
// Controllers/EmployeeController.cs
using Microsoft.AspNetCore.Mvc;

[Route("api/[controller]")]
[ApiController]
public class EmployeeController : ControllerBase
{
    private readonly EmployeeService _employeeService;

    public EmployeeController(EmployeeService employeeService)
    {
        _employeeService = employeeService;
    }

    [HttpGet]
    public async Task<IActionResult> GetAllEmployees()
    {
        var employees = await _employeeService.GetAllEmployeesAsync();
        return Ok(employees);
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetEmployeeById(int id)
    {
        var employee = await _employeeService.GetEmployeeByIdAsync(id);
        if (employee == null)
            return NotFound();

        return Ok(employee);
    }

    [HttpPost]
    public async Task<IActionResult> CreateEmployee([FromBody] Employee employee)
    {
        if (!ModelState.IsValid)
            return BadRequest(ModelState);

        await _employeeService.AddEmployeeAsync(employee);
        return CreatedAtAction(nameof(GetEmployeeById), new { id = employee.Id }, employee);
    }

    [HttpPut("{id}")]
    public async Task<IActionResult> UpdateEmployee(int id, [FromBody] Employee employee)
    {
        if (id != employee.Id)
            return BadRequest();

        await _employeeService.UpdateEmployeeAsync(employee);
        return NoContent();
    }

    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteEmployee(int id)
    {
        await _employeeService.DeleteEmployeeAsync(id);
        return NoContent();
    }
}
```

### Configure Dependency Injection

```csharp
// Program.cs
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Configure database connection
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Register repositories and unit of work
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
builder.Services.AddScoped<EmployeeService>();

// Add controllers and other services
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseAuthorization();
app.MapControllers();
app.Run();
```

### Configure Connection String in appsettings.json

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "YourDatabaseConnectionString"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```
