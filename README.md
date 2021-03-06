# PROJECT LOMBOK - WYBÓR ZAGROŻEŃ I CIEKAWYCH FUNKCJI

## 1. Funkcjonalności

### 1.1 @AllArgsConstructor, @NoArgsConstructor, @RequiredArgsConstructor
Generują konstruktory. RequiredArgsConstructor zawiera tylko pola finalne. Umożliwiają dodanie adnotacji nad konstruktorem. Przykład:
```java
@AllArgsConstructor(onConstructor=@__(@Autowired))
public class SomeSpringController {
    private final CarRepository;
    private final EmployeeRepository;
    private final AnotherRepository;
}
```
Powyższy przykład mocno redukuje kod użyty do wstrzyknięcia komponentów Springa.

### 1.2 @Getter, @Setter
Generuje gettery/settery. Można używać nad nazwą klasy, lub przed konkretnym polem.<br>
Umożliwia określenie modyfikatora dostępu:
```java
@Setter(AccessLevel.PROTECTED) private String name;
```
Umożliwiają dodanie adnotacji do generowanego gettera/settera przy metodzie lub parametrze. Przykład:
```java
@Setter(onParam_=@Max(10000))
private long id;
```
<strong>Zagrożenia: 2.1, 2.4</strong>

### 1.3 @Builder
Dodaje budowniczego do klasy. Ułatwia budowanie instancji, kiedy np. konstruktor ma wiele pól.
```java
@Builder
@Data
public class EmployeeEntity {
    private int id;
    private String name;
    private String surname;
    private int carId;
    private String textInfo;

    public static void main(String[] args) {
        EmployeeEntity employee = EmployeeEntity.builder()
            .id(100)
            .name("Janek")
            .surname("Kowalski")
            .carId(123)
            .textInfo("additional text")
            .build();
    }
}
```

### 1.4 @Data
> All together now: A shortcut for @ToString, @EqualsAndHashCode, @Getter on all fields, @Setter on all non-final fields, and @RequiredArgsConstructor!

Z Lombokiem:
```java
@Data
public class UserValue {
    String name;
    int age;
    final String finalProperty;
}
```
Bez Lomboka:
```java
public class UserValue {
    String name;
    int age;
    final String finalProperty;

    public UserValue(String finalProperty) {
        this.finalProperty = finalProperty;
    }
    //GETTERY
    //SETTERY BEZ PÓL FINALNYCH
    //EQUALS
    //HASHCODE
    //TOSTRING
}
```

<strong>Zagrożenia: 2.6</strong>

### 1.5 @Value
Niemutowalny odpowiednik @Data. Ustawia wszystkie pola jako ``` private final ```, dodaje gettery, konstruktor, toString, hashCode, equals.
<br>Z Lombokiem:
```java
@Value
public class UserValue {
    String name;
    int age;
}
```
Bez Lomboka:
```java
public final class UserValue {
    private final String name;
    private final int age;

    public UserValue(String name, int age) {
        this.name = name;
        this.age = age;
    }

    //GETTERY
    //HASHCODE
    //EQUALS
    //TOSTRING
}
```
<strong>Zagrożenia: 2.6</strong>

### 1.6 @Accessors(fluent = true)
- zmienia getProperty() na property()
- zmienia setProperty(Property property) na property(Property property)
- settery zwracają this, co umożliwia ich łańcuchowe wykonanie

Adnotacja przydatna w funkcyjnych kawałkach kodu.
Przykład z typowymi akcesorami:

```java
List<User> users = credentialsList.stream()
            .map(credential -> {
                    User user = new User();
                    user.setLogin(credential.getLogin);
                    user.setPassword(credential.setPassword);
                    return user;
                }).collect(toList()); 
```

Przykład z @Accessors(fluent = true):
```java
List<User> users = credentialsList.stream()
            .map(credential -> 
                new User()
                .login(credential.login())
                .password(credential.password))
            .collect(toList());
```

### 1.7 @SneakyThrows
W przypadku checked exception opakowuje je w runtime exception. W taki sposób nie trzeba pisać ``` throws XYZException ``` przy deklaracji metody ani używać klauzuli try-catch.

Powiązane z zagrożeniami 2.7.

Przykład kodu z Lombokiem:
```java
public class SneakyThrowsExample implements Runnable {
  @SneakyThrows(UnsupportedEncodingException.class)
  public String utf8ToString(byte[] bytes) {
    return new String(bytes, "UTF-8");
  }
  
  @SneakyThrows
  public void run() {
    throw new Throwable();
  }
}
```
Bez Lomboka:
```java
public class SneakyThrowsExample implements Runnable {
  public String utf8ToString(byte[] bytes) {
    try {
      return new String(bytes, "UTF-8");
    } catch (UnsupportedEncodingException e) {
      throw Lombok.sneakyThrow(e);
    }
  }
  
  public void run() {
    try {
      throw new Throwable();
    } catch (Throwable t) {
      throw Lombok.sneakyThrow(t);
    }
  }
}
```
<strong>Zagrożenia: 2.7</strong>
### 1.8 @Cleanup
Java 7 umożliwiła klauzulę try-with-resources, która samodzielnie zamyka użyte zasoby. Użycie adnotacji przy konkretnej zmiennej
(np. ``` @Cleanup InputStream in = new FileInputStream(); ```) wygeneruje podobną funkcjonalność.

@Cleanup umożliwia określenie metody "sprzątającej" - ma przez to przewagę nad try-with-resources, które działa tylko na klasach implementujących interfejs AutoCloseable. Przykład:
```java
@Cleanup("myDisposeMethod") MyDisposeableClass myClass = new MyDisposeableClass();
```

### 1.9 Wsparcie dla niektórych Loggerów
Adnotacje typu @Slf4j, @Log, @CommonsLog, @Log4j, @Log4j2, @XSlf4j dodają logger do klasy i umożliwiają dostęp do niego poprzez np. ``` log.debug() ```

### 1.10 @NonNull
Można użyć tej adnotacji przy konkretnym parametrze konstruktora - wewnątrz generowany jest kod sprawdzający czy obiekt nie jest nullem, ewentualnie rzucający NullPointerException.
```java
public class NonNullExample extends Something {
  private String name;
  
  public NonNullExample(@NonNull Person person) {
    super("Hello");
    this.name = person.getName();
  }
}
```

### 1.11 @Getter(lazy=true)
Umożliwia leniwe zaciąganie pól do obiektu. Używa java.util.concurrent.AtomicReference, dopiero przy użyciu getX() realnie otrzymuje wartość.

### 1.12 @UtilityClass
Dodaje prywatny konstruktor rzucający wyjątkiem i dołącza "static" do metod i pól. Do zastosowania w bezstanowych klasach pomocniczych, które mają tylko statyczne metody i nie powinny mieć instancji tworzonych przez konstruktor.
Z Lombokiem:
```java
@UtilityClass
public class SuperCalculatorUtils {
    private final int TWO = 2;
    public int addTwo(int x) {
        return x+TWO;
    }
}
```
Bez Lomboka:
```java
public class SuperCalculatorUtils {
    private static final int TWO = 2;
    public static int addTwo(int x) {
        return x+TWO;
    }
}
```

### 1.12 Configuration system
https://projectlombok.org/features/configuration <br>
Umożliwia zablokowanie określonych funkcjonalności określonych przez zespół za niebezpieczne.

### 1.13 Delombok
Umożliwia podejrzenie kodu generowanego przez Lombok. Użycie: PPM -> Refactor -> Delombok -> All lombok annotations (lub określenie konkretnych)

## 2. Zagrożenia

### 2.1 Rename na getterze/setterze zmienia nazwę pola.

```java
@Data
class Employee {
    int id;
    String name;
}

class EmployeeUtil {
    public void welcomeEmployee(Employee employee) {
        System.out.println("Witaj, " + employee.getName());
    }
}
```

Kiedy używajemy narzędzi IDE do zmiany nazwy metody (np. SHIFT+F6 w IntelliJ Idea) na employee.getName(), zmieni się także pole "name" klasy Employee.

### 2.2 Zależności cykliczne przy @ToString

```java
class Employee {
    @ToString
    Order order;
}

class Order {
    @ToString
    Employee employee;
}
```
Jeśli pracownik będzie przechowywał referencję do zamówienia, a zamówienie do pracownika (lub wystąpi inna forma zależności cyklicznej) wywołanie Employee#toString lub Order#toString spowoduje StackOverflowException.

POTENCJALNE ROZWIĄZANIA (**DO USTALENIA PRZEZ ZESPÓŁ!**):

##### 2.2.1 Zablokowanie adnotacji @ToString, własna implementacja toString()
Zalety:
- mniej adnotacji przy nazwie klasy i pól
- brak możliwości popełnienia błędu 2.2

Wady:
- konieczność wpisywania za każdym razem metody ręcznie
- przy każdym dopisaniu pola do klasy konieczność ręcznego dopisania go do toString()

##### 2.2.2 @ToString(onlyExplicitlyIncluded = true) + @ToString.Include
```java
@ToString(onlyExplicitlyIncluded = true)
class Employee {

    @ToString.Include
    String name;

    @ToString.Include
    String surname;

    @ToString.Include
    Date creationDate;

    Order order;
    List<Employee> friends;    
}
```
Zalety:
- częściowa automatyzacja generowania metody toString()
- dodanie do klasy kolejnego pola nie spowoduje samodzielnie zależności cyklicznych

Wady:
- mniej czytelny kod w sekcji pól wewnątrz klasy
- konieczność ręcznego wpisywania @ToString.Include przy każdym polu, które chcemy mieć załączone w toString()

##### 2.2.3 @ToString.Exclude
```java
@ToString
class Employee {
    String name;
    String surname;
    Date creationDate;
    @ToString.Exclude
    Order order;
    @ToString.Exclude
    List<Employee> friends;
}
```
Zalety:
- automatyzacja generowania metody toString()
- przy dodawaniu pól do klasy nie trzeba pamiętać o dodawaniu ich do metody toString()

Wady:
- mniej czytelny kod w sekcji pól wewnątrz klasy
- jeśli ktoś przy dodawaniu pola powodującego zależność cykliczną zapomni o dodaniu adnotacji, naraża na StackOverflowException

### 2.3 Stosowanie z innymi preprocesorami
https://stackoverflow.com/a/38554197 <br>
Jeśli inne preprocesory używają w jakiś sposób kodu generowanego przez Lombok, mogą nie działać prawidłowo.

### 2.4 Łamanie enkapsulacji
Dopisanie kolejnego pola do klasy z adnotacją @Getter/@Setter od razu dodaje dostęp do jej wartości/modyfikacji


### 2.5 Niepoprawne stosowanie @EqualsAndHashcode
##### 2.5.1 Problem porównywania dwóch obiektów
Lombok domyślnie używa wszystkich pól do metody equals. Czasami jest to nieporządane - np. jeśli w encji "Employee" zmienimy "phone" nie powoduje to od razu, że mówimy o zupełnie innym obiekcie
##### 2.5.2 Problem zależności cyklicznych
Analogiczny do 2.2


### 2.6 Stosowanie @Data lub @Value bez znajomości konsekwencji
Adnotacja @Data łączy funkcjonalność kilku innych:
- @ToString
- @EqualsAndHashCode
- @Getter
- @Setter
- @RequiredArgsConstructor

Należy pamiętać o 2.2, 2.4, 2.5
@Value powoduje podobne komplikacje.

### 2.7 Leniwe ładowanie kolekcji + @ToString/@EqualsAndHashCode

```java
@Entity
@ToString
@Getter
@Setter
public class University {
   @Id
    private String id;
    private String name;
    private String address;

    @OneToMany(fetch = FetchType.LAZY)
    private List<Student> students;
}
```
Wywołanie University#toString spowoduje zaciągnięcie wszystkich studentów z bazy danych - może znacząco spowolnić działanie programu.

### 2.8 Nieobsłużenie błędu przy @SneakyThrows.
Jeśli metoda jest oznaczona @SneakyThrows, IDE ani kompilator nie poinformują nas o tym, że jakaś linijka może powodować checked exception. W ten sposób możemy nie obsłużyć istotnego błędu. Możemy też nie dodać logów, co utrudni analizowanie kłopotliwej sytuacji.

### 2.9 Wskazanie linijki występowanego błędu wygenerowanego kodu
Jeśli wygenerowany kod rzuca wyjątkiem, w logach zostanie wyświetlona linijka z adnotacją. Nie będziemy mieli wglądu do konkretnej linijki wygenerowanego kodu.
Przykład 1:
```java
public class LombokException {
    private String name;

    public LombokException(@NonNull String name) {
        this.name = name;
    }

    public static void main(String[] args) {
        var lombokEx = new LombokException(null);
    }
}
```
Logi:
```text
Exception in thread "main" java.lang.NullPointerException: name is marked non-null but is null
	at pl.insert.infrastructure.infoserwis.entity.LombokException.<init>(LombokException.java:8)
	at pl.insert.infrastructure.infoserwis.entity.LombokException.main(LombokException.java:13)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.base/java.lang.reflect.Method.invoke(Method.java:564)
	at com.intellij.rt.execution.application.AppMainV2.main(AppMainV2.java:131)
```

Przykład 2:
```java
public class LombokException2 {
    private static class Disposeable {
        public void dispose() {
            throw new RuntimeException();
        }
    }

    public static void main(String[] args) {
        @Cleanup("dispose") var disposeable = new Disposeable();
    }
}
```
Logi:
```text
Exception in thread "main" java.lang.RuntimeException
	at pl.insert.infrastructure.infoserwis.entity.LombokException2$Disposeable.dispose(LombokException2.java:8)
	at pl.insert.infrastructure.infoserwis.entity.LombokException2.main(LombokException2.java:13)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.base/java.lang.reflect.Method.invoke(Method.java:564)
	at com.intellij.rt.execution.application.AppMainV2.main(AppMainV2.java:131)
```
