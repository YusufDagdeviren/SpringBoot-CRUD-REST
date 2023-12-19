## Spring Boot DAO Add, Update, Delete

### Spring Boot REST: Get Single employee

**1.Adım Dao interface’imize metodu ekliyoruz**

- id’ye denk gelen değeri veritabanından getireceğiz.

```java
...
public interface EmployeeDAO {
    ...
    Employee findById(int id);
		...
}
```

**2. Adım Dao implemantasyonumuzu yapıyoruz**

- interface’imizdeki metodu implemente ediyoruz.
- id’ye denk gelen değeri veritabanından dönderiyoruz.

```java
...
@Repository
public class EmployeeDAOJpaImpl implements EmployeeDAO {
...
    @Override
    public Employee findById(int id) {
        Employee employee = entityManager.find(Employee.class, id);
        return employee;
    }
...
}
```

**3.Adım Service interface’imize findById metodunu ekliyoruz.**

- Buradaki adım DAO sınıfında yaptıklarımıza benzerdir.

```java
...
public interface EmployeeService {
...
    Employee findById(int id);
...

}
```

**4.Adım Service implematasyonunu yapıyoruz.**

- Direkt olarak Dao’ya yönlendiriyoruz.

```java
...
@Service
public class EmployeeServiceImpl implements EmployeeService{
    private EmployeeDAO employeeDAO;

    @Autowired
    public EmployeeServiceImpl(EmployeeDAO employeeDAO) {
        this.employeeDAO = employeeDAO;
    }
....
    @Override
    public Employee findById(int id) {
        return employeeDAO.findById(id);
    }
...
}
```

**5.Adım RestController’a id’ye göre get isteği atan endpoinitimizi ekliyoruz**

- Önce verilen id’ye göre veritabanında değer var mı onu kontrol ediyoruz.
- Eğer id’ye göre employee yoksa Exception fırlatıyoruz.
- Eğer id’ye göre employee dönerse return ediyoruz.

```java
...
@RestController
@RequestMapping("/api")
public class EmployeeRestController {
    private EmployeeService employeeService;

    @Autowired
    public EmployeeRestController(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }
    ...
    @GetMapping("/employees/{employeeId}")
    public Employee findById(@PathVariable int employeeId) {

        Employee employee = employeeService.findById(employeeId);
        // Eğer id veritabanından getirilmezse girecek
        if (employee == null) {
            throw new RuntimeException("Employee id not found - " + employeeId);
        }

        return employee;
    }
		...
}
```

### Spring Boot REST: Add Employee

**1.Adım Dao interface’imize metodu ekliyoruz**

- Veritabanından gelen değere göre ya veritabanına save yapacağız ya da update yapacağız.

```java
...
public interface EmployeeDAO {
		...
    Employee save(Employee employee);
		...
}
```

**2. Adım Dao implemantasyonumuzu yapıyoruz**

- entityManager.merge() metodu eğer employee id’si 0 ise veritabanına kaydeder ama eğer id’si varsa veritabanındaki değeri günceller.
- @Transactional işlemini servise bırakıyoruz. Bizim için bu işlemi servis yapacak.

```java
...
@Repository
public class EmployeeDAOJpaImpl implements EmployeeDAO {
    // define field for entitymanager

    private EntityManager entityManager;
    // set up constructor injector

    @Autowired
    public EmployeeDAOJpaImpl(EntityManager entityManager) {
        this.entityManager = entityManager;
    }
		...
    @Override
    public Employee save(Employee employee) {
        Employee dbEmployee = entityManager.merge(employee);
        return dbEmployee;
    }
		...
}
```

**3.Adım Service interface’imize save metodunu ekliyoruz.**

```java
...
public interface EmployeeService {
...
    Employee save(Employee employee);
...
}
```

**4.Adım Service implematasyonunu yapıyoruz.**

- @Transactional anotasyonunu burada kullanıyoruz.
- Ve direkt olarak dao sınıfının save metodunu çağırıyoruz.

```java
...
@Service
public class EmployeeServiceImpl implements EmployeeService{
    private EmployeeDAO employeeDAO;

    @Autowired
    public EmployeeServiceImpl(EmployeeDAO employeeDAO) {
        this.employeeDAO = employeeDAO;
    }
...
    @Override
    @Transactional
    public Employee save(Employee employee) {
        return employeeDAO.save(employee);
    }
...
}
```

**5.Adım RestController’a post isteği atan endpoinitimizi ekliyoruz.**

- @RequestBody sayesinde entity’mizin değerlerini json olarak body’den alabiliyoruz.
- employee’umuzun id değerini 0 yaparak save metodunu çağırdığımızda veritabanına save işlemini gerçekleştiriyoruz.

```java
...
@RestController
@RequestMapping("/api")
public class EmployeeRestController {

    private EmployeeService employeeService;

    // quick and dirty : inject employee dao (use constructor injection)
    @Autowired
    public EmployeeRestController(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }
...
    @PostMapping("/employees")
    public Employee save(@RequestBody Employee employee) {

        employee.setId(0);

        Employee dbEmployee = employeeService.save(employee);

        return dbEmployee;
    }
    ...
}
```


### Spring Boot REST: Update Employee

**1.Adım Dao interface’imize metodu ekliyoruz**

Bunu zaten bir önceki save işleminde yapmıştık. Update işleminde save ile aynı dao metodumuzu kullanacağız

**2. Adım Dao implemantasyonumuzu yapıyoruz**

Bunu zaten bir önceki save işleminde yapmıştık. Update işleminde save ile aynı dao metodumuzu kullanacağız

**3.Adım Service interface’imize save metodunu ekliyoruz.**

Bunu zaten bir önceki save işleminde yapmıştık. Update işleminde save ile aynı dao metodumuzu kullanacağız. Ekstradan servis metodu yazmamıza gerek yok.

**4.Adım Service implematasyonunu yapıyoruz.**

Bunu zaten bir önceki save işleminde yapmıştık. Update işleminde save ile aynı dao metodumuzu kullanacağız. Ekstradan servis metodu yazmamıza gerek yok.

**5.Adım RestController’a put isteği atan endpoinitimizi ekliyoruz.**

- Bu endpoint’imizde id’si girilmiş employee değeri gelecek bu yüzden direkt olarak ilgili id’nin değerini güncelleyebiliriz.

```java
...
@RestController
@RequestMapping("/api")
public class EmployeeRestController {

    private EmployeeService employeeService;

    // quick and dirty : inject employee dao (use constructor injection)
    @Autowired
    public EmployeeRestController(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }
  ....
    @PutMapping("/employees")
    public Employee update(@RequestBody Employee employee) {

        Employee dbEmployee = employeeService.save(employee);

        return dbEmployee;
    }
	....
}
```


### Spring Boot REST: Delete Employee

**1.Adım Dao interface’imize metodu ekliyoruz**

```java
...
public interface EmployeeDAO {
...
    void deleteById(int id);
}
```

**2. Adım Dao implemantasyonumuzu yapıyoruz**

- Yine burada @Transactional yapmıyoruz bu işlemi servisimize bırakıyoruz.

```java
...
@Repository
public class EmployeeDAOJpaImpl implements EmployeeDAO {
    // define field for entitymanager

    private EntityManager entityManager;
    // set up constructor injector

    @Autowired
    public EmployeeDAOJpaImpl(EntityManager entityManager) {
        this.entityManager = entityManager;
    }
...
    @Override
    public void deleteById(int id) {
        Employee employee = entityManager.find(Employee.class, id);
        entityManager.remove(employee);
    }
}
```

**3.Adım Service interface’imize deleteById metodunu ekliyoruz.**

```java
...

public interface EmployeeService {
...
    void deleteById(int id);

}
```

**4.Adım Service implematasyonunu yapıyoruz.**

@Transactional işlemini Servis’e bırakıyoruz.

```java
...
@Service
public class EmployeeServiceImpl implements EmployeeService{
    private EmployeeDAO employeeDAO;
    @Autowired
    public EmployeeServiceImpl(EmployeeDAO employeeDAO) {
        this.employeeDAO = employeeDAO;
    }
    ...
    @Override
    @Transactional
    public void deleteById(int id) {
        employeeDAO.deleteById(id);
    }
}
```

**5.Adım RestController’a delete isteği atan endpoinitimizi ekliyoruz.**

```java
...
@RestController
@RequestMapping("/api")
public class EmployeeRestController {

    private EmployeeService employeeService;

    // quick and dirty : inject employee dao (use constructor injection)
    @Autowired
    public EmployeeRestController(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }
  ...
    @DeleteMapping("/employees/{employeeId}")
    public String delete(@PathVariable int employeeId) {

        Employee employee = employeeService.findById(employeeId);

        if (employee == null) {
            throw new RuntimeException("Employee id not found - " + employeeId);
        }

        employeeService.deleteById(employeeId);

        return "Deleted employee id - " + employeeId;
    }
}
```