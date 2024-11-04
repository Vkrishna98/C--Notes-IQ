## IEnumerable

- IEnumerable is an interface in C# that represents a sequence of data that you can iterate through, like a list of items. It's a very common way to handle collections of data, especially when you don’t need to modify the collection itself but only want to read it.

### What is IEnumerable?

- IEnumerable allows you to enumerate or loop through a collection, such as a list or array, using a foreach loop.
- It only provides read-only access to the collection, so you can’t add, remove, or change items directly in the collection.
- It’s commonly used when you just need to fetch or iterate through data without needing to manipulate it.
- Think of IEnumerable as a one-way street where you can go through each item one by one but can’t go back, modify, or add items directly to the street itself.

### Why Use IEnumerable?

- Efficiency: IEnumerable is often more efficient when you only need to read data without modifying it. It’s ideal for looping through large data sets.
- Deferred Execution: IEnumerable can defer execution. This means that if you use LINQ to query data (e.g., Where, Select), it won’t execute the query until you actually need the data.
- Abstraction: It abstracts the collection type, so your code can work with any collection type (e.g., arrays, lists, database results) as long as it implements IEnumerable.

### Example: Using IEnumerable in Simple Terms

- Imagine you have a collection of Employee objects and just want to list all the employees without changing anything.

```csharp
public class Employee
{
public int Id { get; set; }
public string Name { get; set; }
}
```

#### Let's create a method that returns an IEnumerable<Employee> and use it to display the employees.

```csharp
public class EmployeeService
{
public IEnumerable<Employee> GetEmployees()
{
// Assume this data comes from a database
List<Employee> employees = new List<Employee>
{
new Employee { Id = 1, Name = "Alice" },
new Employee { Id = 2, Name = "Bob" },
new Employee { Id = 3, Name = "Charlie" }
};

        return employees; // We are returning IEnumerable<Employee>
    }

}
```

### Using IEnumerable to List Employees

- In the code below, we get the list of employees and loop through each one to display their names. We use IEnumerable<Employee> because we only need to read the data.

```csharp
public class Program
{
public static void Main()
{
EmployeeService employeeService = new EmployeeService();

        // Get employees as IEnumerable
        IEnumerable<Employee> employees = employeeService.GetEmployees();

        // Loop through each employee
        foreach (var employee in employees)
        {
            Console.WriteLine($"Employee ID: {employee.Id}, Name: {employee.Name}");
        }
    }

}
```

### Output

```yaml
Employee ID: 1, Name: Alice
Employee ID: 2, Name: Bob
Employee ID: 3, Name: Charlie
```

### Key Points About IEnumerable

- IEnumerable is read-only; you can loop through items but can’t modify the collection.
- It’s flexible and can work with any collection (lists, arrays, database results, etc.).
- It’s ideal when you only need to fetch or display data without modifying it.
- Deferred Execution: If used with LINQ queries, IEnumerable can delay execution until you need the results, making it more efficient.

### When to Use IEnumerable vs. List

- Use IEnumerable when you only need to read the data.
- Use List if you need to add, remove, or modify items in the collection directly.

#### In our example, we used IEnumerable<Employee> for GetEmployees() because it’s efficient for returning a collection when we only need to display it without modifying the original data.
