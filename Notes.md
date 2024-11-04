# C# Notes

---

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

### Entities

```csharp
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int DepartmentId { get; set; }
}

public class Department
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

### Generic Repository Interface

```csharp
public interface IRepository<T> where T : class
{
    IEnumerable<T> GetAll();
    T GetById(int id);
    void Add(T entity);
    void Update(T entity);
    void Delete(T entity);
    void Save();
}
```

### Generic Repository

```csharp
public class GenericRepository<T> : IRepository<T> where T : class
{
    private readonly DbContext _context;
    private readonly DbSet<T> _dbSet;

    public GenericRepository(DbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }

    public IEnumerable<T> GetAll()
    {
        return _dbSet.ToList();
    }

    public T GetById(int id)
    {
        return _dbSet.Find(id);
    }

    public void Add(T entity)
    {
        _dbSet.Add(entity);
    }

    public void Update(T entity)
    {
        _context.Entry(entity).State = EntityState.Modified;
    }

    public void Delete(T entity)
    {
        _dbSet.Remove(entity);
    }

    public void Save()
    {
        _context.SaveChanges();
    }
}
```

### Unit of Work Interface

```csharp
public interface IUnitOfWork
{
    IRepository<Employee> Employees { get; }
    IRepository<Department> Departments { get; }
    void Complete();
}
```

### Unit of Work

```csharp
public class UnitOfWork : IUnitOfWork
{
    private readonly DbContext _context;
    private GenericRepository<Employee> _employees;
    private GenericRepository<Department> _departments;

    public UnitOfWork(DbContext context)
    {
        _context = context;
    }

    public IRepository<Employee> Employees
    {
        get
        {
            if (_employees == null)
            {
                _employees = new GenericRepository<Employee>(_context);
            }
            return _employees;
        }
    }

    public IRepository<Department> Departments
    {
        get
        {
            if (_departments == null)
            {
                _departments = new GenericRepository<Department>(_context);
            }
            return _departments;
        }
    }

    public void Complete()
    {
        _context.SaveChanges();
    }
}
```

### Services

```csharp
public class EmployeeService
{
    private readonly IUnitOfWork _unitOfWork;

    public EmployeeService(IUnitOfWork unitOfWork)
    {
        _unitOfWork = unitOfWork;
    }

    public void AddEmployee(Employee employee)
    {
        _unitOfWork.Employees.Add(employee);
        _unitOfWork.Complete();
    }

    public Employee GetEmployee(int id)
    {
        return _unitOfWork.Employees.GetById(id);
    }
}
```
