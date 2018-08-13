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