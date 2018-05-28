# springdata-demo

Przykład realizowany w ramach zajęć seminaryjnych z przedmiotu Zaawansowane Technologie Internetowe na Wydziale Fizyki i Informatyki Stosowanej. 30.05.2018

## Krok 1 - Utworzenie projektu Maven'owego
W **Eclipse IDE** w menu `File > New > Other ...` z folderu `Maven` należy wybrać `Maven Project` i kliknąć `Next >`. Następnie w oknie dialogowym zaznaczyć opcję `Create a simple project`

![Image - Create a simple project option](https://image.ibb.co/cHVdvy/1.png)

Kolejnym krokiem jest podanie informacji na temat swojego ***Maven'owego*** projektu. Pole `groupId` oznacza najczęściej nazwę aplikacji, a `artifactId` jest unikalną nazwą modułu. W naszym przykładzie nie jest to jednak zbyt istotne, dlatego wypełniamy pola według poniższego wzoru:

![Image - GroupId ArtifactId window](https://image.ibb.co/h8sBhd/2.png)

## Krok 2 - Uruchomienie aplikacji Spring-Boot'owej
W hierarchii utworzonego projektu wyszukujemy plik `pom.xml`. Otwieramy jego podgląd w formacie `xml` i uzupełniamy plik o poniższy kod:

```
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.0.2.RELEASE</version>
</parent>

<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
</dependencies>

<properties>
	<java.version>1.8</java.version>
</properties>

<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
		</plugin>
	</plugins>
</build>
```

Do uruchomienia prostej ***Spring-Boot'owej*** aplikacji brakuje jeszcze klasy zawierającej metodę `main`. W katalogu `src/main/java` tworzymy nowy pakiet `com/wfiis/app`, a w nim następnie klasę `Application.java`.

![Image - Application.java class and package](https://image.ibb.co/b51j5y/3.png)

Klasa ta powinna mieć następującą zawartość:
```
package com.wfiis.app;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

Teraz możemy sprawdzić, czy nasza aplikacja ***Spring-Boot'owa*** została skonfigurowana poprawnie. W tym celu w oknie ze strukturą projektu klikamy prawym przyciskiem myszy na projekt i wybieramy z menu kontekstowego `6 Maven install`. Sprawdzamy czy w konsoli zostało zalogowane poprawne zbudowanie naszego jara:

```
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```
## Krok 3 - Utworzenie konta w mlab.com
Rejstracja w serwisie https://mlab.com/welcome/ umożliwia utworzenie darmowej instancji nierelacyjnej bazy danych. Serwis przy wyborze wersji SANDBOX udostępnia 500 MB przestrzeni na dane. 

Po udanej rejestracji (potwierdzeniu otrzymanym na maila) należy utworzyć nową bazę `sample-database`, a w niej kolekcję `employeeCollection`.

## Krok 4 - Konfiguracja Spring-Data
Spring-Data posiada moduły dla wielu znanych baz danych. Jego możliwości zostaną przedstawione na przykładzie modułu dedykowanemu do jednej z znanych ***No-SQL'owych*** baz danych jaką jest ***MongoDB***. Pierwszym krokiem jest uzupełnienie pliku `pom.xml` o dodatkową dependencję odpowiedzialną za integrację naszej aplikacji z bazą ***MongoDB***:

```
<dependency>
	<groupId>org.springframework.data</groupId>
	<artifactId>spring-data-mongodb</artifactId>
</dependency>
```

Następnym krokiem jest utworzenie pliku z danymi potrzebnymi do połączenia z bazą danych. Na nasze potrzeby skorzystamy automatycznej konfiguracji dostarczonej przez ***Spring-Data***, której do połączenia z bazą danych wystarczy wskazać potrzebne do nawiązania połączenia dane. W tym celu w katalogu `src/main/resources` tworzymy plik `application.properties`,

![Image - application properties path](https://image.ibb.co/dz0Uky/5.png)

w którym umieszczamy następujące dane z dokładnością do hostu, portu i nazwy bazy danych utworzonej w serwisie https://mlab.com/:
```
spring.data.mongodb.host=<<host>>
spring.data.mongodb.port=<<port>>
spring.data.mongodb.database=<<database>>
spring.data.mongodb.username=<<username>>
spring.data.mongodb.password=<<password>>
```
W pola `<<username>>` oraz `<<password>>` wpisać dane logowania użytkownika, którego należy utworzyć w bazie danych `sample-database` w serwisie https://mlab.com/.

## Krok 5 - Stworzenie modelu
Kolejnym krokiem w ramach laboratorium będzie stworzenie klasy modelu, na bazie którego rekordy będą przetrzymywane w bazie danych. W tym celu w katalogu `src/main/java` tworzymy nowy pakiet `com/wfiis/model` w którym dodajemy klasę `Employee.java` o następującej zawartości:

```
package com.wfiis.model;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "employeeCollection")
public class Employee {

	@Id
	private String id;
	private String firstName;
	private String lastName;
	private String description;

	private Employee() {
	}

	public Employee(String firstName, String lastName, String description) {
		this.setFirstName(firstName);
		this.setLastName(lastName);
		this.setDescription(description);
	}

	public String getFirstName() {
		return firstName;
	}

	public void setFirstName(String firstName) {
		this.firstName = firstName;
	}

	public String getLastName() {
		return lastName;
	}

	public void setLastName(String lastName) {
		this.lastName = lastName;
	}

	public String getDescription() {
		return description;
	}

	public void setDescription(String description) {
		this.description = description;
	}
}
```

## Krok 6 - Przykładowe operacje na bazie danych
W tym kroku przedstawione zostaną przykładowe operacje na bazie danych z wykorzystaniem `MongoRepository`.

Aby wykonać operację na bazie za pomocą `MongoRepository` należy stworzyć interfejs rozszerzający interfejs `MongoRepository` dostarczony przez moduł `spring-data-monngodb`. Tworzymy pakiet `com\wfiis\dao` a w nim następujący interfejs:
```
package com.wfiis.dao;

import java.util.List;

import org.springframework.data.mongodb.repository.MongoRepository;

import com.wfiis.model.Employee;

public interface EmployeeDao extends MongoRepository<Employee, String> {

	List<Employee> findByFirstName(String firstName);

	List<Employee> findByFirstNameNot(String firstName);

}
```

Dodatkowo w celu przetestowania funkcjonalności utworzymy kontroler `EmployeeController` w nowym pakiecie `com/wfiis/controllers` korzystający z stworzonego przez nas interfejsu.
```
package com.wfiis.controller;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.ResponseBody;

import com.wfiis.dao.EmployeeDao;
import com.wfiis.model.Employee;

@Controller
public class EmployeeController {
	
	@Autowired
	private EmployeeDao employeeDao;
	
	@GetMapping(value = "/employee")
    public @ResponseBody Employee addStudent(Employee employee) {        	        
        return employeeDao.save(employee);
    }
	
	@GetMapping(value = "/employees")
	public @ResponseBody List<Employee> getAll() {
		return employeeDao.findAll();
	}
	
	@GetMapping(value = "/employees/findByFirstName/{firstName}")
	public @ResponseBody List<Employee> getByFirstName(@PathVariable String firstName) {
		return employeeDao.findByFirstName(firstName);
	}
	
	@GetMapping(value = "/employees/findNotByFirstName/{firstName}")
	public @ResponseBody List<Employee> getNotByFirstName(@PathVariable String firstName) {
		return employeeDao.findByFirstNameNot(firstName);
	}
	
}
```

Należy również wskazać pakiety w których znajdują się kontrolery naszej aplikacji oraz interfejsy rozszerzające `MongoRepository`. W tym celu w klasie `Application.java` dodajemy następujące adnotacje:

```
@ComponentScan(basePackages = { "com.wfiis.controller"} )
@EnableMongoRepositories(basePackages = "com.wfiis.dao")
```

Teraz możemy już uruchomić naszą aplikację. Klikając prawym przyciskiem myszy na nazwę naszego projektu w hierarchii plików wybieramy `Run As > 2 Maven build` i w wyświetlonym oknie w polu `Goals` wpisujemy `spring-boot:run` w celu uruchomienia naszej ***Spring-Boot'owej*** aplikacji. A następnie zatwierdzamy przyciskiem `Run`.

![Image - maven run, spring boot goal](https://image.ibb.co/mR2ACd/5a.png)

W tym momencie w oknie konsoli powinny wyświetlać się kolejno informacje na temat startującej aplikacji ***Spring-Boot'owej*** a wśród nich logi dotyczące portu na którym została uruchomiona aplikacja (w naszym przypadku domyślnego) czy baza danych do której uzyskaliśmy połączenie.

```
...
2018-05-28 21:24:12.192  INFO 14904 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
...
2018-05-28 21:34:57.369  INFO 10028 --- [.mlab.com:37650] org.mongodb.driver.connection            : Opened connection [connectionId{localValue:2, serverValue:19790}] to ds137650-a.mlab.com:37650
...
```
