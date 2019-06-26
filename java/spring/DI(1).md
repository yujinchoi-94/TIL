# 2장 스프링 코어 (DI, AOP)

## 2.1 DI

하나의 처리를 구현하기 위해 여러 개의 컴포넌트를 통합할 때 DI(Dependency Injection, 의존성 주입)라는 접근 방식이 큰 힘을 발휘한다.

#### 예제

```java
public interface UserService {
    // 사용자 정보를 등록한다.
    void register(User user, String rawPassword);
}
```

```java
public interface PasswordEncoder {
    // 패스워드를 해시화한다.
    String encode(CharSequence rawPassword);
}
```

```java
public interface UserRepository {
    // 사용자 정보를 저장한다.
    User save(User user);
    // 사용자 계정명이 일치하는 사용자 수를 카운트한다.
    int countByUsername(String username);
}
```

```java
// 사용자 등록을 처리하는 구현 클래스
public class UserServiceImpl implements UserService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    
    public UserServiceImpl(javax.sql.DataSource dataSource) {
        this.userRepository = new JdbcUserRepository(dataSource);
        this.passwordEncoder = new BCryptPasswordEncoder();
    }
}
```

- `UserServiceImpl`을 개발하는 단계에서는 의존하는 컴포넌트의 클래스(`UserRepository`, `PasswordEncoder`)가 이미 완성돼 있어야 한다.
- 이처럼 필요한 컴포넌트를 생성자에서 직접 생성하는 방식은 일단 클래스가 생성되고 나면 이미 생성된 `UserRepository`나 `PasswordEncoder`의 구현 클래스를 교체하는 것이 사실상 어려울 수 있다. 이러한 클래스 간의 관계를 두고 *클래스 간의 결합도가 높다* 라고 말한다.

------

- 엔터프라이즈 애플리케이션을 개발할 때는 다양한 컴포넌트를 조합하는 것이 일반적이라고 했는데, 많은 컴포넌트에 의존해야 하는 클래스를 이 같은 방식으로 개발하는 것은 상당히 비효율적이다.

- 결합도 낮추기

  - 생성자 안에서 의존하는 컴포넌트 클래스의 구현 클래스를 직접 생성하는 대신, 생성자의 인수로 받아서 할당한다.

    ```java
    // 생성자를 활용한 의존 컴포넌트 초기화
    public UserServiceImpl(UserRepository userRepository,
                           PasswordEncoder passwordEncoder) {
    	this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }
    ```

    ```java
    // 애플리케이션에서 UserService를 사용
    UserRepository userRepository = new JdbcUserRepository(dataSource);
    PasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
    UserService userService = new UserServiceImpl(userRepository, passwordEncoder);
    ```

    - 이렇게 하면 `UserServiceImpl`의 생성자에서 `UserRepository`, `PasswordEncoder`의 구현 클래스에 관한 정보가 제거되어  `UserServiceImpl`의 외부에서  `UserRepository`, `PasswordEncoder`의 구현 클래스를 쉽게 변경할 수 있게 된다.

  - 구현 클래스가 결정 or 완성되지 않은 경우라면 어떻게 해야할까?

    ```java
    // 미완성 클래스를 더미로 대체
    UserRepository userRepository = new DummyUserRepository();
    PasswordEncoder passwordEncoder = new DummyPasswordEncoder();
    UserService userService = new UserServiceImpl(userRepository, passwordEncoder);
    ```

    - 구현 클래스의 더미 클래스를 임시로 만들어 대체하여 개발한다.
    - 하지만, 이 경우에도 개발자가 직접 의존성을 생성하여 주입 해야하기 때문에 재작업을 피할 수 없다.

- 이처럼 어떤 클래스가 필요로 하는 컴포넌트를 외부에서 생성한 후, 내부에서 사용 가능하게 만들어 주는 과정을 *의존성을 주입(DI)한다, 또은 인젝션(Injection)한다* 라고 말한다. 그리고 이러한 의존성 주입을 자동으로 처리하는 기반을 *DI 컨테이너*라고 한다.

- 스프링 프레임워크가 제공하는 기능 중 가장 중요한 것이 바로 이 DI 컨테이너의 기능이다. 

  - 스프링 프레임워크의 DI 컨테이너에 `UserService`이 의존 관계를 갖는 클래스의 구현 클래스를 알려주면 의존성이 자동으로 생성되어 주입된다.

### 2.1.1 DI 개요

> Dependency injection (DI) is a process whereby objects define their dependencies (that is, the other objects with which they work) only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method. 
>
> https://docs.spring.io/spring/docs/5.1.8.RELEASE/spring-framework-reference/core.html#beans-factory-collaborators

- 의존성 주입, IoC (Inversion of Control)이라고 하는 소프트웨어 디자인 패턴 중 하나
  - IoC : 인스턴스를 제어하는 주도권이 역전된다는 의미. 컴포넌트를 구성하는 인스턴스의 생성과 의존 관계 연결처리를 소스 코드가 아닌 DI 컨테이너에서 대신해주기 때문.
- DI 컨테이너를 활용하면 지금까지 인스턴스를 애플리케이션에서 직접 생성해서 쓰는 방법 대신 DI 컨테이너가 만들어주는 인스턴스를 가져오는 방법을 사용할 수 있다.
- DI 컨테이너에서 인스턴스를 관리하는 방식의 장점
  - 인스턴스의 스코프를 제어할 수 있다.
  - 인스턴스의 생명 주기를 제어할 수 있다.
  - AOP 방식으로 공통 기능을 집어넣을 수 있다.
  - 의존하는 컴포넌트 간의 결합도를 낮춰서 단위 테스트하기 쉽게 만든다.

### 2.1.2 ApplicationContext와 빈 정의

스프링에서는 `ApplicationContext`가 DI 컨테이너의 역할을 한다.

> The `org.springframework.context.ApplicationContext` interface represents the Spring IoC container and is responsible for instantiating, configuring, and assembling the beans. The container gets its instructions on what objects to instantiate, configure, and assemble by reading configuration metadata.

#### 예제

```java
@Configuration
public class AppConfig {
    @Bean
    UserRepository userRepository() {
        return new UserRepositoryImpl();
    }

    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Bean
    UserService userService() {
        return new UserServiceImpl(userRepository(), passwordEncoder())
    }
}
```

```java
// 1. 설정 클래스 (AppConfig)를 전달하고, DI 컨테이너 생성
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
// 2. DI 컨테이너에서 인스턴스를 가져옴
UserService userService = contect.getBean(UserService.class);
```

AppConfig

- DI 컨테이너에서 설정 파일의 역할
- 자바로 작성되어 있어 `Java Configuration Class`라고도 한다.
- `@Configuration`과 `@Bean` 애노테이션을 통해 DI 컨테이너에 컴포넌트를 등록하여 애플리케이션은 DI 컨테이너에 있는 Bean을 `ApplicationContext` 인스턴스를 통해 가져올 수 있다.

------

- 용어

  - Bean

    - DI 컨테이너에 등록하는 컴포넌트

      > In Spring, the bojects that form the  backbone of your application and that are managed by the Spring IoC Container called beans. A bean is an object that is instantiated, assembled, and otherwise managed by a Spring IoC container. Otherwise, a bean is simply one of many objects in your application. Beans, and the dependencies among them, are reflected in the configuration metadata used by a container.
      >
      > https://docs.spring.io/spring/docs/5.1.8.RELEASE/spring-framework-reference/core.html#beans-introduction

  - Bean Definition

    - 빈에 대한 설정 정보(Configuration)

  - Lookup

    - DI 컨테이너에서 빈을 찾아오는 행위

- DI 컨테이너에서 빈을 가져오는 방법

  - 빈의 타입을 지정
  - 빈의 타입과 이름을 지정
  - 빈의 이름을 지정

- 빈 설정 방법

  - 자바 기반 설정 방식
  - XML 기반 설정 방식
  - 애너테이션 기반 설정 방식

### 2.1.3 빈 설정

1. 자바 기반 설정 방식

   - 자바 코드로 빈을 설정한다. 이 때 사용되는 자바 클래스를 `Java Configuration Class`라고 한다.

   - `@Configuration`

     - 설정 클래스로 선언. 설정 클래스는 여러 개 정의할 수 있다.

   - `@Bean`

     - 빈을 정의. 메서드 명이 빈의 이름이 되고 그 빈의 인스턴스가 반환값이 된다.
     - 예시에서는 userRepository가 빈의 이름이고, 이름을 달리하고 싶다면 `@Bean(name="userRepo")`와 같이 속성에 빈의 이름을 지정한다.

   - 다른 컴포넌트를 참조해야 할 때는 해당 컴포넌트의 메서드를 호출한다. 의존성 주입이 *프로그램적인 방법*으로 처리된다.

     ```
     package com.yujinchoi.springstudy;
     
     public class Car {
         private CarEngine carEngine;
         private Wheel wheel;
     
         public Car() {
         }
     
         public Car(CarEngine carEngine, Wheel wheel) {
             this.carEngine = carEngine;
             this.wheel = wheel;
         }
     
         public CarEngine getCarEngine() {
             return carEngine;
         }
     
         public void setCarEngine(CarEngine carEngine) {
             this.carEngine = carEngine;
         }
     
         public Wheel getWheel() {
             return wheel;
         }
     
         public void setWheel(Wheel wheel) {
             this.wheel = wheel;
         }
     }
     
     ```

     ```java
     @Configuration
     public class Config {
         CarEngine carEngine() {
             return new CarEngine();
         }
     
         Wheel wheel() {
             return new Wheel();
         }
     
         @Bean
         Car car() {
             return new Car(carEngine(), wheel());
         }
     }
     ```

   - 메서드에 매개변수를 추가하는 방법으로 다른 컴포넌트의 의존성을 주입할 수 있다. *단, 인수로 전달된 인스턴스에 대한 빈은 별도로 정의되어 있어야 한다.*

     ```java
     @Configuration
     public class Config {
         @Bean
         CarEngine carEngine() {
             return new CarEngine();
         }
     
         @Bean
         Wheel wheel() {
             return new Wheel();
         }
         
         @Bean
         Car car(CarEngine carEngine, Wheel wheel) {
             return new Car(carEngine, wheel);
         }
     }
     
     ```

   - 자바 기반 설정 방식만 사용해서 빈을 설정할 때에는 애플리케이션에서 사용되는 모든 컴포넌트를 빈으로 정의해야 한다. 다만 뒤에서 설명할 애너테이션 기반 설정 방식과 조합하면 설정 내용의 많은 부분을 줄일 수도 있다.

2. XML 기반 설정 방식

   - XML 파일을 이용해 빈을 설정한다.

     ```xml
     <beans ...>
         <bean id="car" name="car" class="com.yujinchoi.springstudy.Car">
             <constructor-arg ref="carEngine"/>
             <constructor-arg ref="wheel"/>
         </bean>
         <bean id="engine" name="engine" class="com.yujinchoi.springstudy.Engine"/>
         <bean id="wheel" name="wheel" class="com.yujinchoi.springstudy.Wheel"/>
     </beans>
     ```

     - `<beans>` 태그 안에 여러 빈 정의를 한다.
     - `<bean>` 태그 안에 빈 정의를 한다. `construcotor-arg`를 통해 생성자를 이용한 주입할 의존성을 할 수 있다. `ref`에 의존성을 주입할 bean의 id를 적는다.
     - 의존성 주입을 할 대상이 다른 빈이 아니라 특정 값인 경우 `ref` 대신 `value` 를 사용한다.

3. 애너테이션 기반 설정 방식

   - DI 컨테이너에 관리할 빈을 빈 설정 파일에 정의하는 대신 빈을 정의하는 애너테이션(`@Component`)을 빈의 클래스에 부여하는 방식을 사용한다.

   - 이후 이 애너테이션이 붙은 클래스를 탐색해서 DI 컨테이너에 자동으로 등록하는데 이러한 탐색 과정을 컴포넌트 스캔이라고 한다.

   - 의존성 주입도 이제까지처럼 명시적으로 설정하는 것이 아니라 애너테이션(`@AutoWired`)이 붙어 있으면 DI 컨테이너가 자동으로 필요로 하는 의존 컴포넌트를 주입하게 한다. 이러한 주입 과정을 오토 와이어링(Auto Wiring)이라 한다.

   - 빈 클래스에 `@Component`를 붙여 컴포넌트 스캔이 되도록 만든다.

     ```java
     @Component
     public class UserSeriviceImpl implements UserService {
         @Autowired
         public UserSeriveImpl(UserRepository userRepository, PasswordEncoder passwordEncoder) {
             // 생략
         }
     }
     ```

   - 생성자에 `@Autowired` 애너테이션을 부여하여 오토와이어링 되도록 만든다. 오토와이어링을 사용하면 기본적으로 주입 대상과 같은 타입의 빈을 DI 컨테이너에서 찾아 와이어링 대상에 주입하게 된다.

   - 컴포넌트 스캔을 수행할 때에는 스캔할 범위를 지정해야 하는데 설정 방식으로는 자바 기반 설정 방식이나 XML 기반 설정 방식을 사용할 수 있다.

     - 자바 기반 설정 방식으로 컴포넌트 스캔 범위를 설정

       ```java
       @Configuration
       @ComponentScan("com.example.demo")
       public class AppConfig {
           
       }
       ```

       - 컴포넌트 스캔이 활성화되도록 클래스에 `@ComponentScan` 애너테이션을 부여한다. 애너테이션의 속성에 컴포넌트를 스캔할 패키지(`com.example.demo`)를 지정하였다. 이 속성을 생략할 경우 설정 클래스가 들어있는 패키지 이하를 스캔한다.

     - XML 기반 설정 방식으로 컴포넌트 스캔 범위를 설정

       ```xml
       ...
       	<context:component-scan base-packages="com.example.demo" />
       ```

   - DI 컨테이너에 등록되는 빈의 이름은 기본적으로 클래스명의 첫 글자를 소문자로 바꾼 이름과 같다. (`userServiceImpl`)

   - 명시적으로 빈의 이름을 지정하고 싶다면 애너테이션에 원하는 이름을 넣어주면 된다.
     `@Component("userService")`

### 2.1.4 의존성 주입

총 세가지 의존성 주입 방법을 사용할 수 있다. 1. Setter-based dependency injection 2. Construtor-based dependency injection 3. Field-based injection

1. Setter-based dependency injection

   - Setter 메소드의 인수를 통해 의존성을 주입하는 방식. Setter injection이라고 하자.

   > Setter-based DI is accomplished by the container calling setter methods on your beans after invoking a **no-argument constructor or a no-argument `static` factory method to instantiate your bean.**

   1. 자바 기반 설정 방식

      - Setter 메소드에 다른 컴포넌트의 참조 결과를 설정했다.

        ```java
        // UserServiceImpl에 setter 메소드 구현
        public class UserServiceImpl implements UserService {
            private UserRepository userRepository;
            private PasswordEncoder passwordEncoder;
            
            public void setUserRepository (UserRepository userRepository) {
                this.userRepository = userRepository;
            }
            public void setPasswordEncoder(PasswordEncoder passwordEncoder) {
                this.passwordEncoder = passwordEncoder;
            }
        }
        ```

        ```java
        @Bean
        UserService userService() {
            UserServiceImpl userService = new UserServiceImpl();
            userService.setUserRepository(userRepository());
            userService.setPasswordEncoder(passwordEncoder());
            return userService;
        }
        ```

      - 매개변수 형태로 의존 컴포넌트를 받게한 후, 그 값을 설정자 메서드를 통해 주입했다.

      - 자바 기반 설정 방식으로 setter injection을 하는 경우, <u>마치 프로그램에서 인스턴스를 직접 생성하는 코드처럼 보여 과연 이것이 빈을 정의한 설정인지 체감이 안 될 수 있다.</u>

        ```java
        @Bean
        UserService userService(UserRepository userRepository,
                                PasswordEncoder passwordEncoder) {
            UserServiceImpl userService = new UserServiceImpl();
            userService.setUserRepository(userRepository);
            userService.setPasswordEncoder(passwordEncoder);
            return userService;
        }
        ```

        - 매개변수 형태로 의존 컴포넌트를 받게한 후, 그 값을 설정자 메서드를 통해 주입했다.

        - 자바 기반 설정 방식으로 setter injection을 하는 경우, <u>마치 프로그램에서 인스턴스를 직접 생성하는 코드처럼 보여 과연 이것이 빈을 정의한 설정인지 체감이 안 될 수 있다.</u>

   1.  XML 기반 설정 방식

      - 마찬가지로 `UserServiceImpl`에 `setter` 메소드 들이 구현되어 있어야 한다.

        ```xml
        <bean id="userService" clas="com.example.demo.UserServiceImpl">
            <property name="userRepository" ref="userRepository" />
            <property name="passwordEncoder" ref="passwordEncoder" />
        </bean>
        ```

      - 주입할 대상을 `property`에 기술한다. `name`에 설정된 값에 따라 setter 메소드의 이름을 정하게 된다.

        - e.g `<property name="test" value="test" />` -> `setTest()`가 setter메소드의 이름이 된다.

2. 애너테이션 기반 설정 방식

   ```java
   @Component
   public class UserServiceImpl implements UserService {
       private UserRepository userRepository;
       private PasswordEncoder passwordEncoder;
       
       @Autowired
       public void setUserRepository (UserRepository userRepository) {
           this.userRepository = userRepository;
       }
       
       @Autowired
       public void setPasswordEncoder(PasswordEncoder passwordEncoder) {
           this.passwordEncoder = passwordEncoder;
       }
   }
   ```

   - setter 메소드에 `@Autowired` 애너테이션을 달아주기만 하면 된다. 애너테이션 방식을 사용하면 별도의 설정 파일을 둘 필요가 없다.

3. 
