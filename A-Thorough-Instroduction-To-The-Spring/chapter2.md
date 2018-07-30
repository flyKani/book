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
    - XML 파일을 사용하는 방법으로 <bean> 요소의 class 속성에 FQCN(Fully-Qualified Class Name)을 기술하면 빈이 정의된다. <constructor-arg>나 <property> 요소를 사용해 의존성을 주입한다. 스프링 1.0.0부터 사용할 수 있다.
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

