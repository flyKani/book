# 2장 스프링 코어(DI, AOP)

## 2.1. DI
 어떤 클래스가 필요로 하는 컴포넌트를 외부에서 생성한 후, 내부에서 사용 가능하게 만들어주는 과정을 '의존성을 주입(DI) 한다.' 또는 '인젝션(Injection) 한다'라고 말한다. <br>
 의존성 주입을 자동으로 처리하는 기반을 'DI 컨테이너'라고한다.
  - DI 장점
    - 의존성 해결
    - 싱글톤(Singleton) 및 프로토타입(Prototype)같은 인스턴스 스코프(Scope)관리를 DI가 대신함

### 2.1.1 DI 개요
 DI는 의존성 주입이라고도 하며, IoC라고하는 소프트웨어 디자인 패턴중 하나. 이때의 IoC는 인스턴스를 제어하는 주도권이 역전된다는 의미로 사용되는데, 컴퓨넌트를 구성하는 인스턴스의 생성과 의존 관계의 연결 처리를 해당 소스코드가 아닌 DI 컨테이너에서 대신해 주기 때문에 제어가 역전됐다고 보는 것
 - DI 컨테이터 관리 방식의 장점
    - 인스턴스의 스코프 제어
    - 인스턴스의 생명주기 제어
    - AOP 방식으로 공통 기능 넣을 수 있음
    - 의존하는 컴포넌트 간의 결합도를 낮춰 테스트하기 쉬움
 > 스프링 공식문서에는 스프링 프레임워크를 DI 컨테이너가 아니라 IoC 컨테이너라고 기재하고 있다.

### 2.1.2 ApplicationContext와 빈정의
 스프링 프레임워크에서는 Application Context가 DI 컨테이너 역할을 한다.
 - @Configuration과 @Bean 애너테이션을 사용해서 DI 컨테이너에 컴포넌트를 등록하면 애플리케이션은 DI 컨테이너에 있는 빈(Bean)을 ApplicationContext 인스턴스를 통해 가져올 수 있다. DI 컨테이너에 등록하는 컴포넌트를 빈(Bean)이라고 하고, 이 빈에 대한 설정을(Configuration) 정보를 '빈 정의(Bean Definition)'이라고 한다. DI 컨테이너에서 빈을 찾아오는 해위를 '룩업(Lookup)' 이라고 한다.

 - DI 컨테이너에서 빈 가져오기
    - XxxService xxxService = context.getBean(XxxService.class);
        - 가져오려는 빈 타입(Type)을 지정하는 방법. 지정한 타입에 해당하는 빈이 DI 컨테이너에 오직 하나만 있을 때 사용한다. 스프링 프레임워크 3.0.0 부터 사용 가능
    - XxxService xxxService = context.getBean("xxxService", XxxService.class);
        - 가져오려는 빈의 이름과 타임을 지정하는 방법이다. 지정한 타입에 해당하는 빈이 DI 컨테이너에 여러 개 있을 때 이름으로 구분하기 위해 사용한다. 스프링 프레임워크 2.5.6 버전까지는 반환값이 Object 타입이라서 원하느 빈의 타입으로 형변환 해야했지만 3.0.0 버전 이후부터는 형변환하지 않아도 된다.
    - XxxService xxxService = (XxxService) context.getBean("xxxService");
        - 가져오려는 빈의 이름을 지정하는 방법이다. 반환값이 Object 타입이라서 원하는 빈의 타입으로 형변환해야 한다. 스프링 프레임워크 1.0.0 부터 사용가능하다.

 - 빈을 설정하는 몇가지 방법
    - 자바기반 설정 방식(Java-based configuration)
        - 자바 클래스에 @Configuration 애너테이션을, 메서드에 @Bean 애너테이션을 사용해 빈을 정의하는 방법으로 스프링 프레임워크 3.0.0부터 사용할 수 있다. 최근에는 스프링 기반 애플리케이션 개발에 자주 사용되고 특히 스프링 부트에서 이 방식을 많이 활용한다.
    - XML 기반 설정 방식(XML-based configuration)
        - XML 파일을 사용하는 방법으로 &lt;bean&gt; 요소의 class 속성에 FQCN(Fully-Qualified Class Name)을 기술하면 빈이 정의된다. &lt;constructor-arg>나 &lt;property&gt; 요소를 사용해 의존성을 주입한다. 스프링 1.0.0부터 사용할 수 있다.
    - 애너테이션 기반설정 방식(Annotation-based configuration)
        - Component 같은 마커 애너테이션(Marker Annotation)이 부여된 클래스를 탐색해서(Component Scan) DI 컨테이너에 빈을 자동으로 등록하는 방법이다. 스프링 프레임워크 2.5부터 사용 할 수있다.
    
    - > 자바 기반 설정방식이나 XML 기반설정 방식만 사용해서 빈을 정의할 수도 있지만 대부분의경우 자가바 기반설정 방식과 애너테이션 기반설정 방식을 조합하거나 XML 기반 설정방식과 애너테이션 설정 방식의 조합을 많이 사용한다.


 - ApplicationContext을 생성하는 다양한 방법 
    - ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        - 자바 기반의 설정방식으로 AnnotationConfigApplicationContext의 생성자에 @configuration 애너테이션이 붙은 클래스를 인수로 전달한다.
    - ApplicationContext context = new AnnotationConfigApplicationContext("com.example.app");
        - 애너테이션 기반의 설정 방식으로 AnnotationConfigApplicationContext의 생성자에 패키지명을 인수로 전달한다. 지정된 패키지 이하의 경로에서 컴포넌트를 스캔한다.
    - ApplicationContext context = new ClassPathXmlApplicationContext("META-INF/spring/applicationContext.xml");
        - XML 기반의 설정 방식으로 ClassPathXmlApplicationContext의 생성자에 패키지명을 인수로 전달한다. 경로에 접두어(prefix)가 생략된 경우에는 클래스패스 안에서 상대 경로로 설정 파일을 탐색한다.
    - ApplicationContext context = new FileSystemXmlApplicationContext("./spring/applicationContext.xml");
        - XML 기반의 설정 방식으로 ClassPathXmlApplicationContext의 생성자에 패키지명을 인수로 전달한다. 경로에 접두어가 생략된 경우에는 JVM의 작업 디렉토리 안에서 상대 경로로 설정 파일을 탐색한다.
    
    - > ApplicationContext는 단독 애플리케이션에서 스프링 프레임워크를 사용하거나 JUnit으로 만든 테스트 케이스 안에서 스프링 프레임워크를 구동할 때 사용된다. 반면 웹 애플리케이션에서는 스프링 MVC를 활용하게 되는데, 이때는 ApplicationContext를 웹 환경에 맞게 확장한 WebApplicationContext를 사용한다.

### 2.1.3 빈설정
#### 자바기반의 설정 방식
 - 클래스에 @Configuration 애너네이션을 붙여 설정 클래스를 선언한다. 설정 클래스는 여러 개 정의 할 수 있다.
 - 메서드에 @Bean 애너테이션을 붙여 빈을 정의한다. 메서드명이 빈의 이름이 되고 그 빈의 인스턴스가 반환값이 된다. 반약 빈 이름을 다르게 명시하고 싶다면 @Bean(name="xxxx")와 같이 name 속성에 빈 이름을 재정의 하면된다.
  - 다른 컴포넌트를 참조해야 할 때는 해당 컴포넌트의 메서드를 호출한다. 의존성 주입이 프로그램 적인 방법으로 처리된다.

자바 기반 설정에서는 메서드 매개변수를 추가하는 방법으로 다른 컴포넌트에 의존성을 주입 할 수 있다. 단, 인수로 전달될 인스턴스에 대한 빈은 별도로 정의 돼 있어야한다.<br>
자바 기반 설정 방식만 사용해서 빈을 정의 할 때는 애플리케이션에서 사용 되는 모든 컴포넌트를 빈으로 정의해아한다.<br>
- 애너테이션 기반  설정 방식과 조압하면 설정 내용의 많은 부분을 줄일 수 있다.

#### XML 기반 설정 방식
XML 파일을 이용해 빈을 설정한다.
 - &lt;beans&gt; 요소 안에 빈 정의를 여러개 한다.
 - &lt;bean&gt; 요소에 빈을 정의한다. id 속성에 지정한 값의 빈의 이름이 되고, class 속송에 지정한 클래스가 해당 반의 구현 클래스다. 이때 class속성언 FQCN으로 패키지명부터 클래스명까지 정화하게 기재해야한다.
 - &lt;constructor-arg&gt; 요소에 생성자를 활용한 의존성 주입을한다. ref 속성에 주입할 빈의 이름을 기재한다.
    - 의존성 주입에 주입할 대상이 다른 빈이 아니라 특정 값인 경우. ref 속성을 사용하지 않고 value 속성을 사용한다.

XML만으로 설정하면 애플리케이션에서 사용하는 모든 컴포넌트의 빈을 정의해야하는 번거로움이 있따. 애너테이션 기반 설정과 조합하는 것이 일반적이다.

#### 애너테이션 기반 설정 방식
DI 컨테이너에 관리할 빈을 빈 설정에 정의하는 대신 빈을 정의하는 애너테이션을 빈의 클래스에 부여하는 방식
 - 애너테이션이 붙은 클래스를 탐석해서 DI 컨테이너에 자동등록
    - 컴포넌트스캔(Component Scan)
- 명시적 설정 X, 애너테이션이 붙었으면 DI 컨테이너가 자동으로 필요로 하는 의존 컴포넌트를 주입
    - 오토와이어링(Auto Wiring)
 
설정방법
 - 빈 클래스에 @Component 애너테이션을 붙여 컴포넌트 스캔이 되도록 만든다.
 - 생성자에 @Autowire 애너테이션을 부여해서 오토와이어링되도록 만든다. 오토이어링을 사용하면 기본적로 주입 대상과 같은 타입의 빈을 DI 컨테이너에서 찾아와 와이어링 대상에 주입하게 된다.

스캔 범위지정
 - 자바기반
    - 컴포넌트 스캔이 활성화되도록 클래스에 @ComponentScan 애너테이션을 부여, 애너테이션의 value 속성이나 basedPackages 속성에 컴포넌트 스캔을 패키지를 지정한다.
 - XML 기반
    - &lt;context:component-scan&gt; 요소의 base-packages 속성에 컴포넌트 스캔할 패키지를 지정한다.

DI 컨테이너에 등록되는 빈의 이름을 기본적으로 클래스명의 첫 글자를 소문제로 바꾼이름과 같다.


### 2.1.4 의존성 주입
 - 설정자 기반 의존성 주입 방식(setter-based dependency injection)
    - 메서드의 인수를 통해서 의존성을 주입하는 방식
        - 자바 기반
            - 자바 기반 설정 방식으로 세터 인젝션을 하는 경우, 마치 프로그램에서 인스턴스를 직접 생성하는 코드처럼 보이기 때문에, 빈을 정의한 설정인지 체감이 안될 수있다.
        - XML 기반
            - &lt;property&gt; 요소내에 name 속성에 주입할 대상의 이름을 지정
        - 애너테이션 기반 설정
            설정자 메서드에 @Autowired 애너테이션을 달아주기만 하면 된다.
 - 생성자 기반 의존성 주입 방식(constructor-based dependency injection)
    - 생성자의 인수를 사용해 의존성을 주입하는 방식
        - 자바기반 설정
            - 생성자에 의존 컴포넌트를 직접 설정
        - XML기반 설정
            - &lt;constructor-arg&gt; 요소에 참조하는 컴퍼넌트를 설정
        - 애너테이션 기반
            생성자에 @Autowired를 부여한다.
            - 스프링 4.3 부터 생성자가 하나만 존재하는 경우 생성자에 @Autowired를 지정하지 않아도 묵시적으로 컨스트럭터 인젝션이 수행된다.
    

 - 필드 기반 의존성 주입 방식(field-based injection)
    - 생성자나 설정자메서드를 쓰지않고 DI 컨테이너이너의 힘을 빌려 의종성을 주입하는 방식
    - 의종성을 주입하고 싶은 필드에 @Autowired 애너테이션을 달아주면된다.

### 2.1.5 오토와이어링
명시적으로 @Bean을 정의하지 않고도 DI 컨테이너에 빈을 자동으로 주입하는 방식이다.

- 타입을 사용한 방식(autowiring by type)
    - 기본적으로 의존성 주입이 반드시 성공한다고 가정
        - 실패시 : org.springframework.beans.factory.NosuchBeanDefinitionException Exceptio 발생
                - 의존성 주입실패
            - 필수조건 완화 방법 required 속성에 false (@Autowired(required = false)) ->  의존성 주입실패시 null 
        - 스프링4부터 Optional 사용가능 
            - @Autowired Optional&lt;XXXXX&gt; xxxx;
    - 같은타입의 빈이 여러 개 정의된 경우
        - @Qualifier 애너테이션을 추가해 빈 이름을 지정 
            - @Qualifier 수싱엑 구현 클래스의 이름이 포함된다거나 구현과 관련된 정보가 포함돼 있다면 빈의 명명 방법이 바람직하다고 볼수 없음.
                - 특정 구현체가 사용될 것으로 의식한 이름을 지정 (DI를 사용하는 의미 X)
                - 결합도가 높아짐
                - 요구사항과 취지에 맞춰 Qualifier역할을하는 애너테이션을 만들어 사용.
        - @Primary를 사용해 우선적으로 선택 될 빈지정
    

- 이름을 사용한 방식(autowiring by name)
    - 필드명이나 프로퍼티명이 일치 할 경우 빈 이름으로 필드 인젝션하는 방법이 존재 
        - @Resource 애너테이션
            - name 생략가능 : 필드이름과 같은 빈이 선택, 프로퍼티와 같은 이름의 빈이 선택 
            -  생성자 인젝션에서는 @Resouce 애너테이션을 사용하지 못한다.

- 컬렉션이나 맵 타입으로 오토와이어링
    - 스프링에서는 인터페이스를 구현한 빈을 컬렉션이나(Collection)이나 맵(Map) 타입에 담에서 가져오는 방법도 제공한다.

### 2.1.6 컴포넌트 스캔
클래스 로더(Class Loader)를 스캔하면서 특정 클래스를 찾아 DI컨테이너에 등록하는 방법
- 기본설정으러 컴포넌트 스캔하기
- 다음과 같은 애너테이션이 붙은 클래스가 탐생 대상이 되고 탐색된 컴포넌트는 DI 컨테이너에 등록
    ```
        - @Component (org.springframework.stereotype.Component)
        - @Controller (org.springframework.stereotype.Controller)
        - @Service (org.springframework.stereotype.Service)
        - @Repository (org.springframework.stereotype.Repository)
        - @Configuration (org.springframework.context.annotation.Configuration)
        - @RestController (org.springframework.web.bind.annotation.RestController)
        - @ControllerAdvice (org.springframework.web.bind.annotation.ControllerAdvice)
        - @ManagedBean (javax.annotation.ManagedBean)
        - @Bamed (javax.inject.Named)
    ```
- 자바 기반에서는 @ComponentScan 애너테이션 사용
- XML 기반에서는 &lt;context:component-scan&gt; 요소 사용
- 클래스 로더위에서 애너테이션이 붙은 클래스를 찾아야하기때문에 탐색 범위가 넓고 시간도 오래걸림
    - 애플리케이션 기동시간을 오래걸리게하는 원인


#### 스캔대상에서 가장 많이 활용되는 4가지
|애너테이션|설명|
|---------|----|
|@Controller|MVC 패턴에서의 C, 즉 컨틀롤러(Controller) 역할을 하는 컴포넌트에 붙이는 애너테이션, 클라이언트에서 오는 요청을 받고, 비지니스 로직의 처리 결과를 응답으로 돌려보내는 기능을 한다. 이때 실제 비지니스로직은 @Service가 붙은 컴포넌트에 처리하도록 위임한다.|
|@Service|비지니스 로직(service)을 처리하는 컴포넌트에 붙이는 애너테이션이다. 컨트롤러에서 받은 입력 데이터를 활용해 비지니스 로직을 실행하는 기능을 한다. 이때 영속적으로 보관해야 하는 데이터가 있따면 @Repository가 붙은 컴포넌트에서 처리하도록 위임한다.|
|@Repository|영속적인 데이터 처리를 수행하는 컴포넌트에 붙이는 애너테이션이다. ORM(Object-Relational Mapping) 관련 라이브러리를 활요앻 데이터의 CRUD를 처리하는 기능을한다.|
|@Component|위의 세 경우에 해당하지 않는 컴포넌트(유틸리티 클래스나 기타 자원 클래스 등)에 붙이는 애너네이션|



추가로 다른 컴포넌트를 더 포함하고 싶다면 필터를 적용하는 압버으로 스캔 범위를 커스터마이징 할 수 있다.
 - 애너테이션을 활용한 필터(ANNOTATION)
 - 할당 가능한 타입을 활용한 필터(ASSIGNABLE_TYPE)
 - 정규 표현식 패턴을 활용한 필터(REGEX)
 - AspectJ 패턴을 활용한 필터(ASPECTJ)

    
### 2.1.7 빈 스코프
 - DI 컨테이너는 빈의 의존관계뿐 아니라, 생존 기간(Bean scope)도 관리
 - DI 컨테이너가 관리하는 빈은 기본적으로 싱글톤으로 만들어짐

#### 스프링 프레임워크에서 사용 가능한 스코프 
|스코프|설명|
|-----|----|
|singleton|DI 컨테이너를 기동 할 때 인스턴스가 하나 만들어지고, 이후부터는 그 인스턴스를 공유하는 방식이다. 기본 스코프이기 때문에 별도로 스코프를 지정하지 않았다면 singleton으로 간주한다.|
|prototype|DI 컨테이너에 빈을 요청할 때마다 새로운 빈 인스턴스가 만들어진다. 멀티 스레드 환경에서 오동작이 발생하지 않아야하는(thread-safe) 빈이라면 singleton 스코프가 아닌 prototype을 사용해야 한다.
|request|HTTP 요청이 들어올 때마다 새로운 빈 인스턴스가 만들어진다. 웹 애플리케이션을 만들때만 사용할 수 있다.
|session|HTTP 세션이 만들어 질때마다 새로운 빈 인스턴스가 만들어진다. 웹 애플리케이션을 만들 때만 사용할 수 있다.
|global session|포틀릿(portlet) 환경에서 글로벌 HTTP 세션이 만들어질 때마다 새로운 빈 인스턴스가 만들어진다. 포틀릿을 사용한 웹 애플리케이션을 만들 때만 사용할 수있다.
|application|서블릿 컨텍스트(Servlet Context)가 만들어질 때마다 빈 인스턴스가 만들어진다. 웹 애플리케이션을 만들때만 사용할 수 있다.
|custom|스코프 이름을 직접 정할 수 있고 정의한 규칙에 따라 빈 인스턴스를 만들 수 있다.

- 스코프 설정
    - 자바 기반 : @Scope 애너테이션에 스코프 이름을 지정 ex) @Scope("prototype")
    - XML 기반 : &lt;bean&gt; 속성에 스코프 지정
- 다른스코프의 빈 주입
    - 스코프는 빈의 생존기간을 의미 => 빈간 스코프가 다르다. == 각 빈의 수명이 다르다.
    - 다른 스코프의 빈주입 사용시
        - @Lookup 애너테이션
            - DI컨테이너는 메서드를 바이트 코드로 형태로 만드는 기능이 있다. DI 컨테이너가 빈을 LookUP 하는 메서드를 만들다음, 그 메서드를 의존할 빈에게 주입하게되는데, 이 기능을 룩업 메서드 인젝션(Lookup Method Injection)이라고 한다. 
            - DI 컨테이너에게 룩업을 대행하게 하고 싶은 메서드에 @Lookup 애너테이션을 붙여줌
            - final, private으로 지정하거나, 매개변수 지정하면 X
            - @Lookup 애너테이션에는 value속성에 빈을 이름을 지정 할 수 있다. 지정하지 않으면 반환값 타입을 보고 룩업 대상 빈을 찾음
        - 스코프트 르락시
            - 기존 빈을 프락시로 감싼후, 이 프락시를 다른 빈에 주입하고, 주입받은 빈에서 이 프락시의 메서드를 호출하면 프락시 내부적으로 DI 컨테이너에서 빈을 룩업하고 룩업된 빈의 메서드를 실행하는 방식 
            - 보통 request 스코프나 session 스코프같이 수명이 짧은 빈을 singleton 스코프와 같은 사애적으로 수명이 긴 빈에 주입할때 사용
            -자바기반 설정시 @Scope 애너테이션을 붙인 다음 proxyMode 속성에 프록시를 만드는방법을 지정하면된다.
            - proxyMode 속성
                - ScopedProxyMode.INTERFACES
                    - JDK의 동적 프락시(java.lang.reflect.Proxy)를 사용해 인터페이스 기반의 프락시를 만든다.
                - ScopedProxyMode.TARGET_CLASS
                    - 스프링 프레임워크에 내장된 GCLIB을 사용해 서브클래스 기반의 프락시를 만든다.
            - XML 기반 설정시  &lt;aop:scoped-proxy&gt;요소를 사용한다.
            - 스프링프레임워크 공식문서에는 request, session, globalSession 스코프에서 스코프트 프락시를 사용하고, prototype 스코프에 대해서는 룩업 메서드 인젝션을 사용하도록 안내하고 있다.
            - prototype 스코프에서 스코프트 프락시를 사용하지 못하는 것은 아니지만, 주입된 필드에서 프락시 안에있는 메서드를 한번더 호출하기때문에 매번 새로운 인스턴스가 만들어질 때마다 가 각 프락시의 메서드가 반복해서 호출됨으로 효율성 측면에서 바람직하지 않다는 점을 감안해야한다.
 - 커스텀 스코프
    - 사용자가 직접 정의한 커스텀 스코프를 만들 수 있다.
    - Scope 인터페이스를 구현
    - CustomScopeConfigurer에 자신이 만든 스코프를 스코프명과 함께 설정

### 2.1.8 빈의 생명 주기
 1. initialzation (빈의 초기화 단계)
 2. activation (빈의 사용 단계)
 3. destruction (빈의 종료단계)

 #### 초기화 단계
 1. 빈 설정 읽기 및 보완
    1) 빈 설정 정보 읽어오기
    2) 빈 설정 정보 보완 처리
 2. 빈 생성 및 의존 관계 해결
    1) Contructor Injection 
    2) Field Injection
    3) Setter Injection
 3. 빈 생성 후 초기화 작업
    1) Bean Post Processor 전처리
    2) Bean Post Processor 초기화 처리
    3) Bean Post Processor 후처리


#### 빈 설정 정보 읽기 및 보완
1. 빈의 생성하는데 필요 한 정보를 수집(자바 Configuration, XML, 애너테이션 커포넌트 스캔중) 정보만 불러오는 단계로 빈을 생서하지 않는다.
2. Bean Factory Post Processor(BFPP)를 사용해 빈의 정보를 보완
    - org.springframework.beans.config.BeanFactoryPostProcessor를 구연한  클래스가 수행
    - Property placehoder는 이시점에 실제 프로퍼티 값으로 치환
#### 빈 생성 및 의존 관계 해결
빈의 정보를 읽고,빈 인스턴스를 성공적으로 생성했다면 의존 관계를 해결하기 위한 의존성 주입을 한다. 실제 수행되는순서는 다음과 같다.
1. 생성자 기반 의존성 주입
2. 필드 기반 의존성주입
3. 세터 기반 의존성 주입

#### 빈 생성 후 초기화 작업(bean initialization callback)
 빈 샌성 후의 초기화 작업(Post Construct)이 수행된다.
 크게 전처리, 초기화 처리, 후처리로 구분된다.
 초기화 부분의 처리 순서
 ```
  - 애너테이션 기반 : @PostConstruct애너테이션이 붙은 메서드
  - InitializingBean 인터페이스를 구현한 경우, afterPropertiesSet 메서드
  - 자바 기반의 설정의 경우 @Bean의 initMethod 속성에 지정한 메서드
  - XML 기반 설정을 사용하는 경우, <bean> 요소의 init-method 속성에 지정한 메서드
 ```
 빈이 생성이된 후에 이뤄지는 초기화는 빈을 생성할 때 해주는 초기화와 큰 차이가 있는데, 그것은 의존성 주입이 끝난 필드 값을 초기화에 활용 할 수있다는 점.
 전처리와 후처리는 BeanPostProcessor(BPP) 인터페이스(org.springframework.beans.factory.config.BeanPostProcessor)의 메서드를 통해 실행된다. 
```
public interface BeanPostProcessr {
    // 전처리
    Object postProcessBeforeInitialization(Object bean, String beanName);
    // 전처리
    Object postProcessAfterInitialization(Object bean, String beanName);
}
```
@PostConstruct 애너테이션을 활동하는 경우 반환 값이 void 이고 메서드 변수는 없어야한다. 같은 처리로는 InitializingBean 인터페이스를 구현하다음 afterPropertiesSet 메서드로 대체 할수도 있다. 또, 어떠한 이유로 @PostContruct 애너테이션이나, intializingBean인터페이스를 구현하지 못하는 경우 @Bean애너테이션에 initMehtod 속성에 Method명을 지정하거나, xml의경우 &lt;bean&gt;에 init-method속성에 메서드명을 지정하면된다.

### 종료 단계
스프링 프레임워크에서는 빈이 파괴되기 전에 전처리(PreDestory) 할 수 있는 방법을 제공한다.
#### 빈이 파괴되기 전에 전처리 수행
빈 생성후 초기화(PostConstruct)와 동작 방식이나 구조는 대칭, 동작은 반대로 
전처리 순서
 ```
  - 애너테이션 기반 : @PreDestroy애너테이션이 붙은 메서드
  - DisposableBean 인터페이스를 구현한 경우, destory 메서드
  - 자바 기반의 설정의 경우 @Bean의 destoryMethod 속성에 지정한 메서드
  - XML 기반 설정을 사용하는 경우, <bean> 요소의 destory-method 속성에 지정한 메서드
 ```
 빈 파괴의 전의 전처리(Pre Destory) 작업은 prototype 스코프의 빈에서는 동작하지 않는다.

 ### DI 컨테이너 종료
 ConfigurableApplicationContext 인터페이스 ApplicationContext 인터페이스를 확장한 서브 인터페이스로, 우리가 사용하는 DI 컨테이너가 바로 ConfigurableApplicationContext 인터페이스의 구현체다. 이 이넡페이스는 close라는 메서드가 있고 이것을 호출하면 DI 컨테이너가 종료된다.  


 
