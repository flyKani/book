# 1장. 스프링 프레임워크

## 1.1. 스프링 프레임워크 개요

 - 아파치 라이선스를 따르는 오픈소스 프로젝트 
 - J2EE 기반의 복잡한 개발 방식에 대한 반작용으로 안티테제(antithese) 역할
 - 개발하고 사용하기 쉬운 경량 프레임워크 보급하는데 큰 몫
 - 10년이상 긴 개발을 거치면서 거대한 생태계를 만듦
 - 가장 많이 활용되는 자바 소프트웨어이자 가장성공한 오픈소스 소프트웨어

## 1.2. 스프링 프레임워크의 역사
 
 - 스프링의 기원은 로드 존슨의 "Export One-on-One: J2EE Design and Development"
 - 겨울 뒤에 봄이 온다는 의미로 얀 카로프가 프레임워크 이름을 "Spring Framework"로 제안
 - 스프링 프레임워크 1.x
    - IoC(Inversion of Control) 컨테이너(DI 컨테이너)
    - AOP(Aspect Oriented Programming)
    - XML 기반의 빈 정의
    - 프레임워크 모듈간의 결합도 약화
    - 트랜젝션 관리
    - 데이터 엑세스
 - 스프링 프레임워크 2.0
    - 스프링 시큐리티 (Spring Security)
    - 스프링 웹플로우 (Spring Web Flow)
 - 스프링 프레임워크 2.5
    - 애너테이션(Annotation) 방식으로 설정가능
        - DI(Dependency Injection)과 MVC(Model View Controller)
    - 시스템 연계 
        - 스프링 인티그레이션(Spring Integration)
    - 배치 처리
        - 스프링 배치(Spring Batch)
 - 스프링 프레임워크 3.0
    - 자바 기반의 설정 방식(java-based-configuration) 지원
    - DI의 자바 사양(Spec) JSR-330지원
    - JPA(Java Persistence API) 2.0
    - 빈 검증(Bean Validation)
    - 스프링 MVC(Spring MVC)개선
        - RESTful 프레임워크 사용 가능
 - 스프링 프레임워크 4.0
    - 웹소켓(WebSocket)
    - 웹 메시징
    
## 1.3. 스프링 관련 프로젝트에 관해
현재 스프링 프레임워크를 중심으로 10가지 이상의 다양한 스프링 프로젝트가 존재

### 1.3.1. 스프링 MVC
웹 기반을 위한 프레임워크로, 아키텍처에는 MVC 패턴을 채택하고 있다.
 
 - 액션 기반 프레임워크
    - 구조가 단순, 이해하기 쉽고 확장성이 높다
 - POJO(Plain Old Java Object) 형태로 구현 하는 방식
 - 애너테이션 기반 설정
 - 서플릿 API 추상화
 - 스프링 DI 컨테이너와의 연계
 - 풍부한 확장 기능
 - 각종 서드파티 라이브러리와의 연계
    - JSON : 잭슨(Jackson)
    - 템플릿 엔진 : 타일즈(Apache Tiles), 프리마커(FreeMarker)
    - RSS & Feed : 롬(Rome)
    - 보고서 출력 : 제스퍼리포트(JasperReports)
    - 빈 검증 : 하이버네이트 밸리데이터(Hibernate Validator)
    - 날짜 시간정보 처리 : 조다 타임 (Joda-Time)
 - 라이브러리 자체가 스프링 지원 형태
    - 타임리프(Thymeleaf)
    - Mybatis

### 1.3.2. 스프링 시큐리티
애플리케이션의 인증(Authentication)이나 인가(Authorization)와 같은 기능을 쉽게 구현할 수 있도록 도와주는 프레임워크
 
 - 인증
    - 베이직 인증(Basic)
    - 다이제스트 인증(digest authentication)
    - X.509 클라이언트 인증서
    - LDAP(Lightweight Directory Access Protocol)
    - Open ID
    - 서드파티 제공 인증 방식과 연계 가능
    - 독자적인 인증 조합 가능
 - 보안 기능
    - CSRF(Cross Site Request Forgery)
    - 보엔 응답 헤더 출력(Security HTTP Response Headers)
    - 세션관리(Session Management)
    - 보안기능은 서븦릿 필터의 메커니즘을 응용해 구현되어 있음
    - 원하는 기능을 넣고 빼고, 독자적인 인증 방식을 추가하는 것은 쉽다.

### 1.3.3. 스프링 데이터
다양한 데이터 저장소의 데이터에 손쉽게 접근할 수 있게 해주는 프레임워크, 스프링 데이터는 일종의 엄브렐라(Umbrella Project)로서 그 안에는 다양한 스프링 프로젝트로 구성되어 있다.
    
 - Spring Data Commons
    - 스프링 데이터의 중심이 되는 프로젝트, 데이터 접근에 필요한 공통적인 인터페이스(Repository 인터페이스)를 제공한다, 그래서 다양한 데이터 저장소를 지원하는 모듈은 모두 Spring Data Commons에서 정의한 Repository 인터페이스를 구현한다.
 - Spring Data JPA
    - JPA(Java Persistence API)를 이용해 데이터에 접근하기 위한 모듈
 - ETC
    - Spring Data Mongo
    - Spring Data Redis
    - Spring Data Solr
    - 그 밖에도 No SQL, In memory, 검새 엔진 증에 접근하기 위한 다양한 접근 모듈이 있다.

### 1.3.4. 스프링 배치
배치(batch) 애플리케이션을 개발하기 위한 경량 프레임워크, 대량의 데이터를 처리하는데 필요한 공통 기능을 제공한다. 

### 1.3.5. 스프링 인티그레이션
EIP(Enterprise Integration Patterns)로 알려진 시스템 연계 아키텍처 패턴에 기초해서 연계 애플리케이션을 쉽게 개발할 수 있게 도와주는 프레임워크다. 시스템 간의 복잡한 연계를 풀어주기 위한 단순한 모델을 제공한다.

### 1.3.6. 스프링 클라우드
분산 환경에서 클라우드 환경에 좀더 최적화된 애플리케이션을 개발 하기 위한 프렘워크와 툴의 모음이며, 다수 관련 프로젝트로 구성 돼있다.
 - Spring Cloud Config
    - 프로파일이나 프로퍼티 같은 정보를 Git과 같은 외부 환경에서 관리하고 배포하는 구조 제공
 - Spring Cloud Netflix
    - Eureka, Hystrix, Zuul, Archaius 같은 넷플릭스가 제공하는 다양한 오픈소스 소프트웨어를 사용하는 구조를 제공
- Spring Cloud Bus
    - 분산환경의 노드사이를 AMQP 같은 경량 메시징 브로커에 연결하는 구조를 제공한다.
- Spring Cloud Connectors
    - 클라우드 파운드리나 히로쿠(Heroku) 같은 다양한 백엔드 환경에 접속 하기 위한 구조를 제공한다.

### 1.3.7. 스프링 툴 스위트
스프링 툴 스위트(STS: Spring Tool Suite)는 스프링 기반 애플리케이션을 개발하기에 최적화된 이클립스 기반 통합 개발 환경이다.
 
### 1.3.8. 스프링 IO 플랫폼
스프링 기반 애플리케이셔늘 개발하고 실행할 때 필요한 스프링 관련 라이브러리나 서드파티 라이브럴의 벌전을 결정하고, 의존 관계를 해결하기 위한 스프링 프로젝트
 - 스프링의 라이브러, 서드파티 라이브러리 의존 관계 관리 같은 작업에서 해방 -> 개발에 더 전념
 - 개발 단계에서의 버전, 의존 관계 관리 
 - 배포 단계애플리케이션 디플로이를 용이
 - 내부적으로 스프링 부트와 그레일즈(Grails) 실행환경(DSR: Domain Specific Runtime)으로 지원

### 1.3.9. 스프링 부트
최소한의 설정만으로도 프로덕션 레벨의 스프링 기반 애플리케이션을 쉽게 개발할 수 있게 도와주는 스프링 프로젝트
 - XML이나 자바 기반 설정을 이용한 빈의 정의나 서블릿 설정을 하지 않아도 되며, 별도의 애플리케이션 서버에 디플로이 할 필요도 없다.


## 1.4. Java EE와의 관계
 - 스프렝은 프레임워크는 J2EE 기반의 자바 애플리케이션 프레임워크
 - JAVA EE도 스프링 프레임워크의 장점을 받아들임
 - 스프링과 JAVA EE의 관계는 줄어드는 추세