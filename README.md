# Common Misuses of JavaScript functionals

JavaScript provides some useful tools for iterating over arrays. Let's see what they are and when we should use them.

1. **for-loop**: Use when your looping logic needs to know the index of the item in the array
2. **forEach()**: Use when your looping logic must perform an action for the item in the array, but does not need to return anything.
3. **map()**: Use when your looping logic must compute or extract a value based on the item in the array.
4. **filter()**: Use when you want to operate on a subset of the array.
5. **reduce()**: Use when you are computing a value that depends on all the items in the array


The `map()`, `filter()`, `reduce()`, and `forEach()` functionals in JavaScript allow you to elegantly express iteration logic. However, these functionals are easy to misuse.

This document outlines some common misuses of JavaScript functionals.

For the examples below, assume that `employees` is an array of JavaScript objects, each of which is an object of the form:

```
{
  "name": String, name of employee,
  "salary": Number, salary of employee in dollars,
  "save": Function(no args), saves the employee information to the server,
  "yearsAtCompany": Number, the number of years the employee has spent at the company,
  "promote": Function(no args), promotes the employee
}
```

## Using for-loop when indices don't matter.
**Goal**: Save every employee to the server.

### Bad
```
for (var i = 0; i < employees.length; i++) {
  employee[i].save();
}
```

### Bad
```
employees.map(function(employee) {
  employee.save();
});
```

### Good
```
employees.forEach(function(employee) {
  employee.save();
});
```

### Explanation:
The for-loop is bad because we don't need to know the employees index in order to call `save()`. The `map` is bad because we're not producing a value for each employee, we're just calling `save()`.


## Using `forEach` to build an array
**Goal**: Create an array called `names` that contains the names of each employee.

### Bad
```
var names = [];
for (var i = 0; i < employees.length; i++) {
  names.push(employees[i].name);
});
```

### Good
```
var names = employees.map(function(employee) {
  return employee.name;
})
```

### Explanation
The `forEach` is bad because we're not simply iterating over employees and performing an action. We are producing a value - the name of the employee. Using a for-loop does not fix this issue and also introduces an index, which we don't need.

Since `map` can produce a value (the name) for each employee, we use it instead.

## Using an if-statement to perform filtering
**Goal**: Get the names of employees (in array `earlyEmployees`) who have been at the company more than 4 years.

### Bad
```
var earlyEmployees = [];
employees.forEach(function(employee) {
  if (employee.yearsAtCompany > 4) {
    earlyEmployees.push(employee.name);
  }
});
```

### Good
```
var earlyEmployees = employees.filter(function(employee) {
    return employee.yearsAtCompany > 4;
  }).map(function(employee) {
    return employee.name;
  });
```

### Explanation
First, we want to focus on a subset of the employees - those who have worked at the company for more than 4 years, so we should use a `filter` to indicate that. The `forEach` + if-statement doesn't make that clear.

Second, we're producing a value (the name) for each employee in our filtered array, so we should use `map`. The `forEach` doesn't return value, so we have to a weird thing where we define an array beforehand and append to it in the `forEach`.

## Using an accumulator variable to produce a computation
**Goal**: Compute the sum the salaries for employees that have been at the company for more than 4 years.

### Bad
```
var salarySum = 0;
employees.forEach(function(employee) {
  if (employee.yearsAtCompany > 4) {
    salarySum += employee.salary;
  }
})
```

### Also Bad
```
var salarySum = 0;
employees.filter(function(employee) {
  return employee.yearsAtCompany > 4
}).forEach(function(employee) {
  salarySum += employee.salary;
})
```

### Good
```
var salarySum = employees.filter(function(employee) {
  return employee.yearsAtCompany > 4;
}).map(function(employee) {
  return employee.salary;
}).reduce(function(totalSalarySoFar, employeeSalary) {
  return totalSalarySoFar + employeeSalary; 
}, 0);
```

### Explanation
Although the first example is the fewest in lines of code, the final example is the clearest in terms of what it is doing:

* Step 1: Restrict our focus to employees who have worked at the company for over 4 years
* Step 2: Get the salaries for these employees
* Step 3: Add up the salaries

## More examples
**Goal**: Promote employees who have been at the company more than 4 years.
```
employees.filter(function(employee) {
  return employee.yearsAtCompany > 4;
}).forEach(function(employee) {
  employee.promote();
})
```

**Goal**: Ensure there are no employees making less than 15,000 dollars.
```
var employeesBelowMinimumWage = employees.filter(function(employee) {
  return employee.salary < 15000;
});
var isMinimumWageViolated = employeesBelowMinimumWage.length !== 0;
```

## FAQ

* Why does this matter?

Proper use of functionals makes your iteration logic clearer. 

They can give you a performance boost in some languages, like [Java](https://docs.oracle.com/javase/tutorial/collections/streams/parallelism.html#executing_streams_in_parallel) if you perform operations in parallel.

In environments where you compute on huge collections of data, such as in [Apache Spark](http://spark.apache.org/docs/latest/programming-guide.html#basics), for-loops may not be available. Similarly, they may not be available in environments where you process [realtime streams of data](http://spark.apache.org/docs/latest/structured-streaming-programming-guide.html#basic-operations---selection-projection-aggregation).
 
They can help you understand large scale data processing systems like [MapReduce](https://en.wikipedia.org/wiki/MapReduce).

* map() creates a new array, isn't it more performant to mutate the existing array?

Yes, but in 6.170 you probably won't run into such a performance issue. Don't prematurely optimize.

* Isn't a for-loop more performant?

Yes, but it's highly unlikely that this will be a performance bottleneck in your 6.170 projects. 