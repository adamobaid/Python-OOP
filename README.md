# Employee Management system using Python & Pandas

Object-Oriented programming is based on the idea of storing data and behavior by wrapping them with structures called Objects. In OOP theory, objects are created and customized using classes. Classes can be thought of as a blueprint for a behavior that needs to be implemented. 

In this post I'm going to explore how OOP concepts such as abstraction, inheritance, encapsulation and other nitty-gritty details are applied in Python by building an Employee Management Library for company X. Employees will be represented as objects and each employee will have attributes and functions they can perform. After the basics, I will add the ability to retreive employee data from a remote url, explore the structure of the remote data and store it in a Pandas DataFrame for further work. 

- The documented source code can be found in this [repository](https://github.com/adamobaid/Python-OOP)


# Employee Management System

**Summary of concepts / featurs:**

- Object attributes and class attributes
- Object methods, class methods, static methods and special (dunder) methods
- Customized repr() and str() to give class objects more readability
- Customized add() and len() to emulate builtin behavior. 
- Inheritance: Developer and Manager sub-classes
- Connecting to company API and creating objects using retrieved data
- Encapsulation: using the methods of one class as attributes in another
- Setter and Deleter functionality using property decorators 
- Storing Employee data in Pandas DataFrames for futher analysis


```python
# Imports used
import json
import requests
import datetime
from datetime import date, timedelta
import pandas as pd
```

### Object attributes and class attributes
Object attributes or instances attributes can be unqiue from one object to another, while class attributes are shared amongest all objects. In this scenario, we can think of the anual raise the company gives to employees as a class attribute. Another class attribute that is the same amongest all objects is the number of employees in the company. 


```python
class Employee:
    
    num_of_emps = 0
    raise_amount = 1.04
    
    def __init__(self, first, last, pay):
        self.first = first
        self.last = last
        self.email = first + '.' + last + '@email.com'
        self.pay = pay
        Employee.num_of_emps += 1
```


```python
# Creating instancs of the class
employee_1 = Employee('Adam', 'Obaid', 50000)
employee_2 = Employee('Test', 'Employee', 60000)
# Accessing instance attributes
print(employee_1.first)
print(employee_1.pay)
```

    Adam
    50000


Class attributes can be accesssed the name of the class, i.e., `Employee.raise_amount` or through the instance `self.raise_amount` because when accessing an attribute in python, it first checks if the instance has that attribute, if not, it searches for it in the class attributes. To see the attributes associated with an object, the __dict__ method can be used.



```python
# print the namespace of employee 1
print(employee_1.__dict__)
```

    {'first': 'Adam', 'last': 'Obaid', 'email': 'Adam.Obaid@email.com', 'pay': 50000}


### Object methods
Object or instance methods are functions that perform actions associated with the object itself, not some general operation related to the class. An example of this in this application would be the below `fullname()` method, where it combines the first and last name attributes of the instance and returns the full name. Another example would a function that changes the raise amount for a specific employee, not for the whole class. This can be accomplished by passing `self` to the function, which is convention for the class instance, i.e., `employee1`. Using `self` in the `apply_raise()` also allows other classes to override this class if need be. 



```python
class Employee:
    
    num_of_emps = 0
    raise_amount = 1.04
    
    def __init__(self, first, last, pay):
        self.first = first
        self.last = last
        self.email = first + '.' + last + '@email.com'
        self.pay = pay
        Employee.num_of_emps += 1

    def fullname(self):
        return '{} {}'.format(self.first, self.last)

    def apply_raise(self):
        self.pay = int(self.pay * self.raise_amount)

```


```python
employee_1 = Employee('Adam', 'Obaid', 50000)
employee_2 = Employee('Test', 'Employee', 60000)
print('Raise amount', Employee.raise_amount)
print('Salary before raise',employee_1.pay)
employee_1.apply_raise()
print('Salary after raise',employee_1.pay)
```

    Raise amount 1.04
    Salary before raise 50000
    Salary after raise 52000


### Special (dunder) Methods
There's an excellent lecture (5) on the topic of using these special methods to impelement some higher-level behavior that is builtin in Python. Some example of these methods includ `__rep__` and `__str__`, which when customized, give class objects more readability. but this can be extended further to emulate  more advanced behaviors like implementing custom addition and subtraction operations for class objects. This is known and called by the name of - The Data Model in Python Documentations(3) 
    


```python
class Employee:

    def __init__(self, first, last, pay):
        self.first = first
        self.last = last
        self.email = first + '.' + last + '@email.com'
        self.pay = pay

    def fullname(self):
        return '{} {}'.format(self.first, self.last)

    def apply_raise(self):
        self.pay = int(self.pay * self.raise_amt)

    def __repr__(self):
        """
        unambigous representation of the obj for developers
        """
        
        return "Employee('{}', '{}', {})".format(self.first, self.last, self.pay)
   
    def __str__(self):
        """
        a string to copy & past and recreate the object with
        """
        
        return '{} - {}'.format(self.fullname(), self.email)

    def __add__(self, other):
        """
        Returns the sum of two employee's salaries
        """
        return self.pay + other.pay

    def __len__(self):
        """
        Returns the number of characters in an employee's full name
        """
        return len(self.fullname())


```


```python
emp_1 = Employee('Adam', 'Obaid', 50000)
emp_2 = Employee('Test', 'Employee', 60000)

print('Addtion two employees is ',emp_1 + emp_2)

print('The length of an employee is ', len(emp_1))
```

    Addtion two employees is  110000
    The length of an employee is  10


### Propertry Decortators
Allows us to give the class, getter, setter and deleter. This is useful whenever you have a method that changes the value of an attribute. For example, In the previous email method in the Employee class, it used to depend on first and last names properties such that if the first name was changed at a later point, the email will still keep the first name.


```python
class Employee:

    def __init__(self, first, last):
        self.first = first
        self.last = last

    @property
    def email(self):
        return '{}.{}@email.com'.format(self.first, self.last)
 
    @property
    def fullname(self):
        return '{} {}'.format(self.first, self.last)
    # if we wanna be able to do emp_1.fullname = "Adam Obaid"
    
    @fullname.setter
    def fullname(self, name):
        first, last = name.split(' ')
        self.first = first
        self.last = last
    
    @fullname.deleter
    def fullname(self):
        print('Delete Name!')
        self.first = None
        self.last = None
```

**Note**
- why not set setters and getters instead of using classmethod to set raise amount?
- because raise amount propertry is not correlated with any other attribute

### Inheritance 

Inheritance gives the ability to create new classes that are already loaded with attribute and methods from another, more mature class. This allows one to upgrade a class or add functionality without affecting the parent class.


In this case, good candidates for child classes would be Intern, Developer and Manager Employees which can have different attributes and methods. For example, to add `prog_lang` attribute to employees who are developers. This is done by giving the class `Developer` its own `__init__` constructor method, while at the same time extending from the constructor of the parent's class, i.e., `super().__ini__(attrs)`.
   


```python
class Developer(Employee):
    # customize raise amount for this class
    raise_amt = 1.10
    
    def __init__(self, first, last, pay, prog_lang):
        super().__init__(first, last, pay)
        self.prog_lang = prog_lang
```


```python
class Manager(Employee):
    # list of employees that this manager supervises 
    def __init__(self, first, last, pay, employees=None):
        super().__init__(first, last, pay)

        if employees is None:
            self.employees = []
        else:
            self.employees = employees
    # not methods to populate that list or dict or manipulate it
    def add_emp(self, emp):
        if emp not in self.employees:
            self.employees.append(emp)

    def remove_emp(self, emp):
        if emp in self.employees:
            self.employees.remove(emp)

    def print_emps(self):
        for emp in self.employees:
            print(
                emp.fullname())           
```

###  Creating objects using an API response

For a previous project I needed to retreive some data from an API and then store it in a Pandas DataFrame. I will do the same here assuming that Employees data is stored somewhere on the company's website. To achieve this I created a class called `Account` which connects to the API using `requests` then fetches the Employees attributes (first name, last name, ...etc) then uses them to create Employee objects and finally, store them in a Panda's DataFrame for further work possibly. 

This will act as an alternative constructor to the Employee class objects, au lie de `__init__` the main constructors. In Python,`@classmethod` is usually used to do this. For the purpose of demonstrating I will use this generic user API [https://randomuser.me/api/](https://randomuser.me/api/) as employee information. I did explore the structure of the Json response from said API and based on that I wrote below code. 


```python
# by adding this method to class Employee
@classmethod    
def from_api(cls):
    """
    Retreives Employee profile from API
    Creates Employee Object 
    """   
    url = "https://randomuser.me/api/"
    response = requests.get(url)
    response = response.text
    parsed_response = json.loads(response)
    data = parsed_response["results"][0]
    name=data['name']
    first = name['first']
    last = name['last']
    pay = 60000 # default base salary
    return cls(first, last, pay)
```

### Storing Employees in Panda's DataFrame



```python
# Note how I used encapsulation to share information between the two classes
class Accounts:
    """
    The Accounts Object has the following attributes and methods:
    
    Attributes
    ======
    employees: list of employees to store in a Pandas Dataframe     
    """
    
    def __init__(self):
        """
        Instantiates list of employees
        """
        
        self.employees = []

    def create_employee(self):
        """
        adds a single employee to the list attribute 
        """
        
        self.employees.append(Employee.from_api())

    def create_multiple_employees(self, n_users):
        """
        adds multiple Employees to the list attribute
        """
        
        for _ in range(n_users):
            self.create_employee()

    def employees_to_pandas(self):
        """
        Returns a DataFrame of the objects found in the employees list
        """
        
        df = pd.DataFrame()
        for _ in range(len(self.employees)):
            df = pd.DataFrame([t.__dict__ for t in self.employees])
        return df
```


```python
# creating an account object, retrieve 20 employees and store them in a DataFrame
object_ = Accounts()
object_.create_multiple_employees(20)
object_.employees_to_pandas()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>first</th>
      <th>last</th>
      <th>pay</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Jen</td>
      <td>Ward</td>
      <td>60000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Rigo</td>
      <td>Manthey</td>
      <td>60000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Brad</td>
      <td>Shelton</td>
      <td>60000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Line</td>
      <td>Laurent</td>
      <td>60000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Jeanine</td>
      <td>Thomas</td>
      <td>60000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Axel</td>
      <td>Henry</td>
      <td>60000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Nalan</td>
      <td>Velioğlu</td>
      <td>60000</td>
    </tr>
    <tr>
      <th>7</th>
      <td>آدرین</td>
      <td>نجاتی</td>
      <td>60000</td>
    </tr>
    <tr>
      <th>8</th>
      <td>آراد</td>
      <td>قاسمی</td>
      <td>60000</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Jo</td>
      <td>Garza</td>
      <td>60000</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Afşar</td>
      <td>Erçetin</td>
      <td>60000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Eden</td>
      <td>Talma</td>
      <td>60000</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Begüm</td>
      <td>Balcı</td>
      <td>60000</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Joakim</td>
      <td>Daae</td>
      <td>60000</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Larry</td>
      <td>Knight</td>
      <td>60000</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Théo</td>
      <td>Da Silva</td>
      <td>60000</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Cristobal</td>
      <td>Velasco</td>
      <td>60000</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Thea</td>
      <td>Martin</td>
      <td>60000</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Oscar</td>
      <td>Andersen</td>
      <td>60000</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Kathy</td>
      <td>Thomas</td>
      <td>60000</td>
    </tr>
  </tbody>
</table>
</div>



# References
1. [Python3.8 Documentations](https://docs.python.org/3/)
2. [Python Data Model](https://docs.python.org/3/reference/datamodel.html)
3. [Requests: HTTP for Humans](https://requests.readthedocs.io/en/master/)
4. [C. Schafer OOP Tutorial](https://docs.python.org/3/reference/datamodel.html)
5. [J. Powell: PyData 2017](https://www.youtube.com/watch?v=cKPlPJyQrt4)


