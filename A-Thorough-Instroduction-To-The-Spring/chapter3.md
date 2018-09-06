# 3장 데이터 접근(JDBC, Tx)

## 3.1 스프링 프레임워크와 데이터 소스

### 3.1.1. 데이터 소스 개요
데이터 소스는 애플리케이션이 데티어베이스에 접근하기 위한 추상화된 연결 방식, 즉 커넥션을 제공하는 역할을 한다. 그리고 스프링 프레임워크가 제공하는 데이터 소스에는 크게 세 가지 종류가 있다.

#### 애플리케이션 모듈이 제공하는 데이터 소스
Commons DBCP나 Tomcat JDBC Connection Pool 과같이 서드파티가 제공하는 데이터 소스나 DriverManagerDataSource같이 스프링 프레임워크가 테스트 용도로 제공하는 데이터 소스를 빈으로 등록해서 사용하는 방식

#### 애플리케이션 서버가 제공하는 데이터 소스
애플리케이션 서버가 정의한 데이터 소스를 JNDI(Java Naming and Directory Interface)를 통해 가져와서 사용하는 방식

#### 내장형 데이터베이스를 사용하는 데이터 소스
HSQLDB, H2, Apache Derby 같은 내장형 데이터베이스에 접속하는 데이터 소스

### 3.1.2 데이터 소스 설정
spring-jdbc를 의존관계 정의 

ex) pom.xml
```
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
    </dependency>
```

#### 애플리케이션 모듈이 제공하는 데이터 소스
EX)<br>
DB : PostgreSQL<br>
DataSource : DBCP<br>
설정값은 jdbc.properties기재 되어있다고 가정.

자바기반설정
```
@Configuration
@propertySource("classpath:jdbc.properties")
public class PoolingDataSourceConfig{

    /* Commons DBCP가 제공하는 데이터 소스 객체를 스프링워크에서 사용하기위해 빈으로 정의한다. 애플리케이션을 종료할 때 
     * 데이터 소스와 관련된 리소스가 해제될 수 있도록 @Bean 애너테이션의 destoryMethod 속성에 BasicDataSource 클래스의 
     * close 메서드를 지정한다. 
     */
    @Bean(destoryMethod = "close") 
    public DataSource dataSource(
        @Value("{database.driverClassName}") String driverClassName,
        @Value("{database.url}") String url,
        @Value("{database.username}") Striong name,
        @Value("{database.password}") String password,
        @Value("{cp.maxTotal}") int maxTotal,
        @Value("{cp.maxIdle}") int maxIdle,
        @Value("{cp.minIdle}") int minIdle,
        @Value("{cp.maxWaitMillis}") long maxWaitMillis){ // 데이터베이스와 관련된 접속 정보나 커넥션 풀과 관련된 설정 정보를 프로퍼티 파일에서 읽어 메서드의 파라메터를 통해 전달한다. 
            BasicDataSource dataSource = new BasiceDataSource(); // 데이터 소스를 생성하기 위해 BasicDataSource 클래스를 사용한다. 
            dataSource.setDriverClassName(driverClassName);
            dataSource.setUrl(url);
            dataSource.setUsername(username);
            dataSource.setPassword(password);
            dataSource.setDefaultAutoCommit(false);
            dataSource.setMaxTotal(maxTotal);
            dataSource.setMaxIdle(maxIdle);
            dataSource.setMinIdle(minIdle);
            dataSource.setMaxWaitMillis(maxWaitMillis);
            return dataSource;
        }
    )
}
```

데이터베이스 관련 설정 정보
```
database.url=jdbc:postgresql://localhost/sample
database.username=postgres
database.password=postgres
database.driverClassName=org.postgresql.Driver
cp.maxTotal=96
cp.maxIdle=16
cp.minIdle=0
cp.maxWaitMillis=60000
```

#### 애플리케이션 서버가 제공하는 데이터 소스
애플리케이션 서버가 제공하는 데이터 소스를 활용하는 방식, 애플리케이션 서버에 'jdbc/mydb'라는 JNDI명으로 커넥션 풀이 만들어져 있다고 가정한다. 

```
@Configuration
public JndiDatasourceConfig(){
    @Bean // DataSource 타입의 빈을 정의한다.
    public DataSoruce dataSource(){
        //애플리케이션 서버에 있는 리소스를 JNDI를 통해 룩업하기 위해 JndiTemplate클래스의 인스턴스를 생성
        JndiTemplate jndiTemplate = new JndiTemplate(); 

        // lookup 메서드를 통해 JNDI명이 'java:com/env/jdbc/mydb'인 리소스(데이터소스)를 찾아온다.
        return jndiTemplate.lookupo("java:com/env/jdbc/mydb", DataSource.class)
    }
}
```


#### 내장형 데이터베이스를 사용하는 데이터 소스
내장형 데이터베이스를 사용할 때는 애플리케이션을 기동할 때마다 데이터베이스가 새로 구축되기 때문에 테이블과 같은 기본 구조를 만들기 위한 DDL(Data Define Language)과 초기 데이터 적재를 위한 DML(Data Manipulation Language)를 준빈한다. 
EX) <br>
내장형 데이터베이스 : H2<br>
초기 데이터 적재를 위한 DML : insert-init-data.sql<br>

```
@Configuration
public class DatasourceEmbeddedConfig{
    @Bean // DataSouce 타입의 빈
    public DataSource dataSource(){
        return new EmbeddedDatabaseBuilder()   // 데이터 소스를 생성하기 위해 EmbeddedDatabaseBuilder 클래스를 사용
                    .setType(EmbeddedDatabaseType.H2) // SetType 메서드로 사용할 내장형 데이터베이스 지정
                    .setScriptEncoding("UTF-8") // setScriptEncoding 메서드로 SQL 파일의 파일 이코딩 방식 시정
                     // addScript 메서드로 애플리케이션 기동시 실행할 SQL파일을 지정한다.
                    .addScript("META-INF/sql/schema.sql", "META-INF/sql/insert-init-data.sql") 
                    .build(); // build 메서드로 이제까지 설정한 내용으로 데이터 소스를 생성한다.
    }
}
```

## 3.2. 스프링 JDBC

### 3.2.1. 스프링 JDBC 개요 
스프링 JDBC는 SQL의 실제 내용과 상관없이 공통적이면서도 반복적으로 수행되는 JDBC 처리를 개발자가 직접 구현하는 대신 프레임워크가 대행하는 기능을 제공한다.

프레임워크가 대신 처리(공통적이면서도 반복적인)
 - 커넥션 연결과 종료
 - SQL 문의 실행
 - SQL 문 실행 결과 행에 대한 반복 처리
 - 예외 처리

 개발자가 구현할 부분
 - SQL 문의 정의
 - 파라메터 설정
 - ResultSet에서 결과를 가져온 후, 각 레코드별로 필요한 처리

### 3.2.2. JdbcTemplate 클래스르 활용한 CRUD
  SQL 만 가지고도 데이터베이스를 쉽게 다룰 수 있게 도와주는 클래스를 제공한다. 특히 NamedParameterJdbcTemplate 클래스는 사실상 데이터를 조작하는 처리를 JdbcTemplate클래스에 위임하게되는데 이 둘의 차이는 JdbcTemplate 클래스가 데이터 바인딩시 ? 문자를 플레이스홀더로 사용하는반면, NamedParameterJdbcTemplate 클래스는 데이터 바인딩시 파라메터 이름을 사용할 수 있어서 ?를 사용할때보다 좀 더 직관적으로 데이터를 다룰 수 있게 해준다는 것이다. <br>
  JdbcTemplate 클래스를 애플리케이션에서 사용 할 때는 개발자가 직접 JdbcTemplate를 만들어 쓰기보다는 DI 컨테이너가 만든 JdbcTemplate을 @Autowired 애너테이션으로 주입받아쓰는것이 일반적이다.

#### JdbcTemplate 클래스가 제공하는 주요 메서드
 
 |메서드|설명|
 |----|----|
 |queryForObject|하나의 결과 레코드중에서 하나의 컬럼 값을 가져올 때 사용함. RowMapper와 함께 사용하면 하나의 레코드 정보를 객체에 맵핑할 수있음|
 |queryForMap|하나의 결과 레코드 정보를 Map 형태로 맵핑 할 수있음|
 |queryForList|여러 개의 결과 레코드를 다룰 수 있음, List의 한 요소가 한 레코드에 해당, 한 레코드의 정보는 queryForObject나 queryForMap을 사용할 때와 같음|
 |query|ResultSetExtractor, RowCallbackHandler와 함께 조회할 때 사용함|
 |update|데이터를 변경하는 SQL(예: INSERT, DELETE, UPDATE)을 실행할 때 사용함|

 
#### 3.2.3. SQL 질의 결과를 POJO로 반환
스프링 JDBC는 처리 결과 값을 자바가 기보넞긍로 제공하는 데이터 타입이나, Map, List 같은 컬렉션 타입으로 반환한다. 보통 애플리케이션을 개발할 때는 자바에서 제공하는 타입이 아니라 해당 비지니스에 마는 데이터 타입을 POJO 형태로 만들어 쓰는 경향이 있기 때문에 반환값을 가공해야 할 수있다. 스프링 JDBC에서는 기본저긍로 반환된 결과값을 원하는 형태로 변환하기 쉽도록 다음과같은 3가지 인터페이스를 제공한다. 

 |메서드|설명|
 |----|----|
 |RowMapper|RowMapper 인터페이스는 JDBC의 ResultSet을 순차적으로 읽으면서 원하는 POJO 형태로 맵핑하고 싶을 때 사용한다. 그래서 이 인터페이스를 사용하면, ResultSet의 한 행(Row)를 읽어 하나의 POJO 객체로 변환 할 수 있다. 두 ResultSetExtractor와의 차이점은 ResultSet의 한 행이 하나의 POJO 객체로 변환디ㅗ고, ResultSet이 다음 행으로 넘어가는 커서 제어를 개발자가 직접하지 않고 스프링 프레임워크가 대신해준다는 점이다.|
 |ResultSetExtractor| ResultSetExtractor 인터페이스는 JDBC의 ResultSet을 자유롭게 제어하면서 원하는 POJO 형태로 맵핑하고 싶을때 사용한다. RowMapper와의 차이점은 ResultSet의 여러 행 사이를 자유롭게 이동할 수 있다는 점이다. RowMapper는 ResultSet에서 한 행씩만 참조할 수 있어서 다음 행으로 커서를 이동하는 next같은 메서드를 제공하지 않는다 그래서 ResultSetExtractor를 사용하면 ResultSet의 여러 행에서 필요한 값을 꺼내서 하나의 POJO객체에 채워넣을 수 있따.
|RowCallbackHandler|RowCallbackHandler 인터페이스의 메서드는 반환값이 없다 그래서 JDBC의 ResultSet 정보 처리 결과를 반환하기 위해 사용하는 것 이 아니라 별도의 다른 처리를 하고 싶을 때 사용한다. 예를 들면, ResultSet에서 읽은 데이터를 파일 형태로 출력하거나, 조회된 데이터를 검증하는 용도로 활용할 수 있다. 

#### 3.2.4. 데이터 일괄처리
- 배치처리
    - NamedParameterJdbcTemplate 클래스에 batchUpdate 메서드 활용
- 저장 프로시저 호출
    - JdbcTemplate 클래스의 call 메서드와 execute메서드로 실행
        - StoredProcedure 클래스와 SimpleJdbcCall 클래스등 항솽에 맞게 적절한 것 을 골라 쓸 것.

## 3.3. 트랜젝션관리

### 3.3.1. 트랜젝션 관리자
스프링 트랜잭션 처리에 중심이 되는 인터페이스 PlatformTransactionManager
- PlatformTransactionManager를 직접호출하지 않고 스프링이 제공하는 API를 사용하는 사례가 많아짐

PlatformTransactionManager의 대표적인 구현 클래스
 |클래스명|설명|
 |----|----|
 |DataSourceTransactionManager|JDBC 및 마이바티스등 JDBC 기반 라이브러리 데이터베이스에 접근하는 경우 이용한다.|
 |HibernateTransactionManager|하이버네이트를 이용해 데이터베이스에 접근하는 경우 이용한다.|
 |JpaTransactionManager|JPA로 데이터베이스에 접근하는 경우 이용한다.|
 |JtaTransactionManager|JTA에서 트랜잭션을 관리하는 경우 이용한다.|
 |WebLogicJtaTransactionManager|애플리케이션 서버인 웹로직의 JTA에서 트랜잭션을 관리하는 경우 이용한다.|
 |WebSphereUowJtaTransactionManager|애플리케이션 서버인 웹스피어의 JTA에서 트랜잭션을 관리하는 경우 이용한다.|

트랜잭션 관리자의 정의 
 - PlatformTransactionManager의 빈의 정의한다.
 - 트랜잭션을 관리하는 메서드를 정의한다.

글로벌 트랜잭션의 경우 JTA Transaction Manager를 사용, 다만 애플리케이션 서버 제품이 따라 조금씩 동작방식이 다를수있어 서브클래스를 제공 되기도한다.


### 3.3.2. 선언적 트랜젝션

#### @Transaction을 이용한 트랜잭션
@Transaction 애너테이션을 빈의 public 메서드에 추가하는 것으로 대상 메서드의 지작 종료에 맞춰 트랜잭션을 시작, 커밋 할 수있다, 기본상태에서는  unchecked Expception이 발생하면 자동으로 롤백된다.

#### 트랜젝션 제어에 필요한 정보
 |속성|설명|
 |----|----|
 |value|여러 트랜잭션 관리자를 이용하는 경우 트랜잭션 관리자의 qualifier를 지정한다. 기본트랜잭션 관리자를 사용하면 생략할 수 있다.
 |transactionManager|value의 별칭(스프링 4.2 버전부터 추가)
 |propagation|트랜잭션의 전파 방식을 지정한다.|
 |isolation|트랜잭션의 격리 수준을 지정한다.|
 |timeout|트랜잭션 제한 시간(초)을 지정한다. 기본값은 -1(사용하는 데이터베이싀 사양이나 설정에 따라 다름)|
 |readOnly|트랜잭션의 읽기 전용 플래그를 지정한다. 기본값은 false(읽기전용이 아님)|
 |rollbackFor|여기에 지정한 예외가 발생하면 트랜잭션을 롤백시킨다. 예외 클래스명을 여러개 나열할 수 있으며, ','으로 구분한다. 따로 지정해주지 않았다면 RuntimeException과 같은 비검사 예외가 발생할 때 트랜잭션이 롤백 된다.|
 |rollbackForClassName| 여기에 지정한 예외바 발생하면 트랜잭션을 롤백시킨다. 예외 이름을 여러 개 나열할 수있으며, ','로 구분한다.|
 |noRollbackFor|예기에 지정한 예외가 발생하더라도 트랜잭션을 롤백시키지 않는다. 예외 클래스명을 여러개 나열할 수있으며, ','로 구분한다.|
 |noRollbackForClassName| 여기에 지정한 예외가 발생하더라도 트랜잭션을 롤백시키지 않는다. 예외 이름을 여러개 나열할 수 있으며, ','로 구분한다.|

### 3.3.3. 명시적 트랜잭션 

#### PlatformTransactionManager를 이용한 명시적 트랜잭션 제어
TransactionDefinition 및 TransactionStatus를 이용해 트랜잭션의 시작과 커밋, 그리고 롤백을 명시적으로 처리

#### TransactionTemplate을 활용한 명시적 트랜잭션 제어 
PlatfromTransactionManager보다도 구조적으로 트랜잭션 제어를 기술 할 수 있다. TransactionCallback 인터페이스가 제공하는 메서드에 구현하고 TransactionTemplate의 execute 메서드에 인수로 전달 한다.

### 3.3.4 트랜잭션 격리 수준과 전파 방식

#### 트랜잭션 격리수준
@Transactional의 isaolation 속성, TransactionDefinition과 TransacltionTemplate의 setIsolationLevel 메서드에 지정 할 수 있다.
 |트랜잭션 격리 수준|설명|
 |----|----|
 |DEFAULT|사용하는 데티터베이스의 기본 격리 수준을 이용한다.|
 |READ_UNCOMMITTED|더티 리드(Dirty Read), 반복되지 않은 읽기(Unrepeatable Read), 팬텀 읽기(Phantom Read)가 발생한다. 이 격리 수준은 커밋되지 않은 변경 데이터를 다른 트랜잭션에 참조하는 것을 허용한다. 만약 변경 데이터가 롤백 된경우 다음 트랜잭션에 무효한 데이터를 조회하게된다.|
 |READ_COMMITED|더티 리드를 방지하지만 반복되지 않은 읽기, 팬텀 읽기는 발생한다. 이 격리 수준은 커밋되지 않은 변경 데이터를 다른 트랜잭션에 참조하는 것을 금지한다.|
 |REPATABLE_READ| 더티 리드, 반복되지 않는 읽기를 방지하지만 팬텀 읽기는 발생한다.|
 |SERIALIZABLE|더티 리드, 반복되지 않은 읽기, 팬텀 읽기를 방지한다.|

 #### 트랜잭션 전파 방식
 |전파방식|설명|
 |----|----|
 |REQUIRED|이미 만들어진 트랜잭션이 존재한다면 해당 트랜잭션 관리 범위 안에 함께 들어간다. 만약 이미 만들어진 트랜잭션이 존재하지 않는다면 새로운 트랜잭션을 만든다.|
 |REQUIRES_NEW|이미 만드러진 트랜잭션 범위안에 들어가지 않고 반드시 새로운 트랜잭션을 만든다. 만약 이미 만들ㅇ저ㅣㄴ 트랜잭션이 아직 종료되지 않았다면 새로운 트랜잭션은 보류 상태가 되어 이전 트랜잭션이 끝나는 것을 기다려야한다.|
 |MANDATORY| 이미 만들어진 트랜잭션 범위 안에 들어가야 한다 만약 기존 만들어진 트랜잭션이 없다면 예외가 발생한다|
 |NEVER|트랜잭션 고나리를 하지 않는다. 만약 이미 만들어진 트랜잭션이 있다면 예외가 발생한다.|
 |NOT_SUPPORTED|트랜잭션을 관리하지 않는다. 만약 이미만들저진 트랜잭션이 있다면 이전 트랜잭션이 끝나는 것을 기다려한다.|
 |SUPPORTS|이미 만들어진 트랜잭션이 있다면 그 범위 안에 들어가고, 만약 트랜잭션이 없다면 트랜잭션 관리를 하지 않는다.|
 |NESTED|REQUIRED와 마찬가지로 현재 트랜잭션이 존재하지 않으면 새로운 트랜잭션을 만들고 이미 존재하는 경우에는 이미 만드러진 것을 계속 이용하지만, NESTED가 적용 구간은 중첩된 트랜잭션처리처럼 취급한다. NESTED 구간 안에서 롤백이 발생한 경우 NESTED 구간 안의 처리내용은 모두 롤백되지만 NESTED 구간 밖에서 실행된 처리내용은 롤백되지 않는다. 단, 부모 트랜잭션에서 롤백되면 NESTED 구간의 트랜잭션은 모두 롤백된다.|


## 3.4 데이터 접근시 예외처리
### 3.4.1 스프링 프레임워크에서 지공하는 데이터 접근관련 예외
#### DataAccessException을 부모 클래스로 하는데이터 접근 예외 계층구조
스프링 프레임워크에서는 데이터접근과 관련된 모든 예외가 DataAcessException을 부모 클래스로 하는 계층구조로 만들어져있다.

#### 비검사 예외를 활용한 DataAccessException 구현
계층 구조의 부모 클래스인 DataAcessException은 RuntimeException의 자식클래스로 구현이 돼있다. 이로써 DataAcessException은 비검사 예외가 된다. 비검사 예외는 try~ catch나 throws에 의해 예외처리가 강제 되지 않기때문에 예외를 굳치 처리할 필요가 없다면 예외처리를 생략해도 된다.

#### 구현을 은폐한 데이터 접근 예외
각기 다른 데이터베이스의 차이에서 발생하는 데이터 접근 예외는 모두 각자 정의한 모양을 가지고 있다. 스프링의 데이터 접근 기능에는 각 제품 별로 정의된 서로 다른 예외 클래스를 데이터 접근 방법과 특정 제품에 종속되지 않는 공통적인 예외 클래스로 변환하는 기능이 있다. 이 기능을 사용하면 DA 구현이 바뀌더라도 예외 처리 코드에 영향을 주지 않는다. 

### 3.4.2 데이터 접근 관련 예외처리
데이터 접근 예외는 비검사 예외로 구현돼 있다. 따로 예외처리가 필요하다면 발생할만한곳에 try~ catch로 원하는 예외를 잡아서 처리해야 한다.

###3.4.3. 데이터 접근 관련 예외의 변환 규칙 커스터마이징
각 데이터베이스 오류 코드와 데이터 접근 예외에 관련된 내용은 spring-jdbc-xxx.jar에 포함된 sql-error-codes.xml을 사용한다. 만약 클래스패스 바로 아래에 sql-error-codes.xml을 두고 그 내용을 재정의하면 기본 설정된 내용을 커스터마이징 할 수 있다. 

