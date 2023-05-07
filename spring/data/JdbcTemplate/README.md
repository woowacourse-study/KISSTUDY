# 발표자료

# 목차

1. JDBCTemplate의 사용이유
    - DriverManager과 DataSource
        - 공통점과 차이점
        - 초기 설정과 사용 방법
2. **JdbcTemplate 사용법**
    1. 스프링 부트 사용하여 **JdbcTemplate 사용하기**
    2. JdbcTemplate 생성시, DataSource 세팅해줘야 하는데, 어떻게 자동 등록될까?
3. JdbcTemplate 쿼리 사용하기
    - RowMapper
4. JdbcTemplate 업데이트 사용하기
    - keyHolder
5. JdbcTemplate execute() 메서드
6. NamedParameterJdbcTemplate 사용하기
    1. SqlParameterSource
    2. MapSqlParameterSource 
    3. BeanPropertySqlParameterSource
        - 자바 빈 클래스
7. SimpleJdbcInsert 사용하기

## JDBCTemplate의 사용이유

:  jdbc API를 직접 사용할 때 반복되는 작업을 대신 처리해 준다

1. 커넥션 획득
2. statement를 준비하고 실행
3. 결과를 반복하도록 루프를 실행
4. 커넥션 종료, statement 및 resultset 종료
5. 트랜잭션을 다루기 위한 커넥션 동기화
6. 예외 발생 시 스프링 예외 변환기 실행

jdbc api의 동작 흐름

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/567f2cd1-d9f0-4374-b3bf-9ed0e8febf41/Untitled.png)

### # DriverManager과 DataSource

- 공통점 : DriverManager과 DataSource는 모두 Java의 JDBC API의 일부분이며, Java에서 데이터베이스와 연결하기 위해 사용된다.

DriverManager은 JDBC 드라이버를 직접 로드하고, DriverManager.getConnection() 메소드를 사용하여 데이터베이스에 연결한다

- 매번 Connection을 생성할 때마다 JDBC 드라이버를 로드하고 Connection 객체를 생성합니다. 이는 애플리케이션의 성능을 저하시키고, 여러 사용자가 동시에 액세스하는 경우 Connection 경합이 발생하여 애플리케이션의 안정성에도 영향을 줄 수 있습니다
    
    * **Connection 경합이란?**
    
     여러 스레드 또는 사용자가 동시에 데이터베이스 연결을 요청할 때 발생하는 문제입니다. 이 때, 각각의 스레드 또는 사용자가 Connection을 요청하면서 드라이버는 Connection 객체를 생성하고 반환합니다. 이때, 드라이버는 동시에 여러 스레드 또는 사용자 요청을 처리하게 됩니다. 그러나 드라이버가 동시에 여러 개의 Connection 객체를 생성하는 것은 불가능하므로, 스레드 또는 사용자가 동시에 Connection 객체를 생성하려고 할 때 경합이 발생합니다. 이 경우, Connection 객체가 충분하지 않을 때는 애플리케이션의 성능 저하 및 오류 발생 등의 문제가 발생할 수 있습니다. 
    
- DriverManager는 Connection pooling을 지원하지 않으므로, 매번 Connection 객체를 생성하는 것은 비용이 많이든다.

DataSource는 데이터베이스 연결을 위해 JDBC 드라이버를 사용하지만, 데이터베이스 연결을 관리하는 더 많은 기능을 제공합니다. 예를 들어, DataSource는 커넥션 풀링, 트랜잭션 관리, 보안 구성 등의 기능을 제공

- 데이터베이스 연결 정보를 저장하는 Connection 객체를 관리하는데, Connection을 효율적으로 관리하기 위해 **커넥션 풀링을 사용**합니다. 즉, DataSource는 Connection 객체를 미리 생성하여 풀에 저장하고, 필요할 때마다 이를 재사용하여 데이터베이스와의 연결을 유지한다
- 트랜잭션 관리를 위한 메소드를 제공합니다. 이를 사용하여 데이터베이스 작업을 하나의 트랜잭션으로 묶어서 처리할 수 있습니다. 이는 여러 개의 데이터베이스 작업을 하나의 원자적인 작업으로 처리하고, 작업이 실패할 경우 이전 상태로 롤백할 수 있도록
- DataSource는 데이터베이스 연결에 대한 보안 구성을 제공합니다. 이를 사용하여 연결에 대한 암호화, 인증, 권한 부여 등을 설정

```java
//db.properties 파일: 접속정보 저장
#mysql DB properties
MYSQL_DB_DRIVER_CLASS=com.mysql.jdbc.Driver
MYSQL_DB_URL=jdbc:mysql://localhost:3306/UserDB
MYSQL_DB_USERNAME=pankaj
MYSQL_DB_PASSWORD=pankaj123

****// MySql 데이터 소스 가져오기 : properties파일 읽어서, 처리하는 방식****
public static DataSource getMySQLDataSource() {
        Properties props = new Properties();
        FileInputStream fis = null;
        MysqlDataSource mysqlDS = null;
        try {
            fis = new FileInputStream("db.properties");
            props.load(fis);
            mysqlDS = new MysqlDataSource();
            mysqlDS.setURL(props.getProperty("MYSQL_DB_URL"));
            mysqlDS.setUser(props.getProperty("MYSQL_DB_USERNAME"));
            mysqlDS.setPassword(props.getProperty("MYSQL_DB_PASSWORD"));
        } catch (IOException e) {
            e.printStackTrace();
        }
        return mysqlDS;

//****MySql 데이터 소스 사용하기 : 클라이언트 클래스****
private static void testJDBCPDataSource() {
		DataSource ds = DBCPDataSourceFactory.getMySQLDataSource();
		try{
			Connectioncon = ds.getConnection();
			Statementstmt = con.createStatement();
			ResultSet rs = stmt.executeQuery("select id, name from User");
			while(rs.next()){
			}
		} 
		catch () {}
		finally { //close하기 }
}
```

* DataSource의 초기 설정과 사용 방법:

1. 커넥션 풀링 라이브러리를 추가한다. 대표적인 라이브러리로는 Apache Commons DBCP, HikariCP 등.
2. 커넥션 풀의 설정을 정의한다. 주요 설정 항목으로는 최대 커넥션 수, 커넥션 유지 시간, 커넥션 풀링의 유형 등.
3. DataSource 객체를 생성한다. 이때, 커넥션 풀링 라이브러리에서 제공하는 DataSource 구현체를 사용하거나, 직접 DataSource 구현체를 작성한다.
    
    예) MysqlDataSource 객체 사용하여, 데이터 베이스 연결정보(db.properties)를 저장한다.
    
4. DataSource를 사용하여 Connection 객체를 얻는다. 이때, DataSource.getConnection() 메서드를 호출하여 Connection 객체를 얻는다.

* DriverManager의 초기 설정과 사용 방법:

1. JDBC 드라이버를 다운로드하고 클래스패스에 추가한다.
2. 드라이버를 로드합니다. DriverManager의 Class.forName() 메서드를 사용하여 드라이버 클래스를 로드하고, 드라이버 클래스가 정적 블록에서 DriverManager에 등록되도록 한다.
    
    드라이버를 로드하고 등록하는 다른 방법은 Service Provider Interface (SPI)를 사용하는 것
    
3. 데이터베이스 연결 정보를 정의한다. 주요 정보로는 데이터베이스 URL, 사용자 이름, 암호 등.
4. DriverManager.getConnection() 메서드를 호출하여 Connection 객체를 얻는다. 이때, 데이터베이스 연결 정보를 매개변수로 전달하여 Connection 객체를 생성한다.
5. Connection 객체를 사용하여 데이터베이스 연산을 수행한다. 이때, Statement, PreparedStatement, CallableStatement 등을 사용하여 SQL 문장을 실행.
6. Connection 객체를 close() 메서드를 호출하여 연결을 종료. 이때, Connection 객체가 사용 중인 경우는 close() 메서드를 호출하지 않는 것이 좋다.

## **JdbcTemplate 사용법**

1. jdbc 라이브러리를 프로젝트에 추가
    
    (build.gradle에 의존성 추가)
    
    ```java
    dependencies {
    	**implementation 'org.springframework.boot:spring-boot-starter-jdbc'**
    }
    ```
    
2. DataSource 설정
    
    JdbcTemplate은 DataSource를 사용하여 데이터베이스에 연결하기 때문에, 먼저 DataSource를 설정해야 한다.
    
3. JdbcTemplate 빈 설정
    
    예) JavaConfig 파일에 DataSource 설정과 JdbcTemplate 빈 설정 
    
    ```java
    @Configuration
    public class AppConfig {
        
        @Bean
        public DataSource dataSource() {
            BasicDataSource dataSource = new BasicDataSource();
            dataSource.setDriverClassName("com.mysql.jdbc.Driver");
            dataSource.setUrl("jdbc:mysql://localhost:3306/mydb");
            dataSource.setUsername("root");
            dataSource.setPassword("root");
            return dataSource;
        }
    
    		@Bean
    		    public JdbcTemplate jdbcTemplate() {
    		        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource());
    		        return jdbcTemplate;
    		    }
    }
    ```
    
4. JdbcTemplate을 사용
    
    :Spring의 DI (의존성 주입) 기능을 이용하여 JdbcTemplate 빈을 주입받아 사용한다
    

### 스프링 부트 사용하여 **JdbcTemplate 사용하기**

- application.properties 설정을 통해 DataSource를 설정한다(데이터 소스 안에 드라이버 설정하고, url 설정, 사용자, 비밀번호 설정 등이 들어있다)
    
    ```java
    spring.datasource.url=jdbc:h2:mem:test;MODE=MySQL
    spring.datasource.driver-class-name=org.h2.Driver
    ```
    
1. 스프링 빈으로 등록
    - application.properties 설정을 통해 DataSource를 등록한다.
        - 스프링 부트는 기본적으로 HikariCP를 커넥션 풀로 사용한다.
        - Connection Pool 설정을 application.properties에서 처리할 수 있다.
        - DataSourceURL이 지정되지 않는 경우, 내장 DB(메모리 DB)를 생성하려고 시도한다.
    - url 과, mysql 서버 아이디, mysql 서버 비밀번호
        
        ```java
        // 드라이버 설정, url 설정
        spring.datasource.driver-class-name=org.h2.Driver
        spring.datasource.url=jdbc:h2:~/data/mydatabase;MODE=MYSQL
        //사용자, 비밀번호 등록안하면 사용하는 데이터베이스의 기본값으로 설정됨
        spring.datasource.username=sa
        spring.datasource.password=
        //h2 콘솔 경로와 사용여부 설정
        spring.h2.console.enabled=true
        spring.h2.console.path=/h2-console
        
        // h2를 인메모리가 아니라, 디스크 방식으로 사용하는 경우
        spring.datasource.initialization-mode=always
        spring.datasource.schema=classpath:schema-mysql.sql
        
        ```
        

### JdbcTemplate 생성시, DataSource 세팅해줘야 하는데, 어떻게 자동 등록될까?

```java
public JdbcTemplate(DataSource dataSource) {   
setDataSource(dataSource);   
afterPropertiesSet();
}
```

## JdbcTemplate 쿼리 사용하기

```java
// queryForObject 메서드 
queryForObject(String sql, Class<T> requiredType)
queryForObject(String sql, Class<T> requiredType, @Nullable Object... args)
queryForObject(String sql, RowMapper<T> rowMapper, @Nullable Object... args)

// query 메서드 : 위와 동일
```

### RowMapper란?

: Spring JDBC를 사용할 때 데이터베이스의 행(row)을 객체로 매핑(mapping)할 때 사용한다.

즉, 데이터베이스에서 검색한 결과세트에서(ResultSet)에서 각 행(row)을 매핑하여 Java객체로 변환한다

- queryForObject와 함께 사용하면, 객체 1개가 반환되고
- query와 함께 사용하면, 객체 리스트가 반환된다

## JdbcTemplate 업데이트 사용하기

INSERT, UPDATE, DELETE 등 데이터를 변경하고 싶을 때는 update() 메서드를 사용한다

```java
public int *update(String sql, @Nullable Object... args)
내부적으로, 
-> update(String sql, @Nullable PreparedStatementSetter pss) //sql, newArgPreparedStatementSetter(args)
-> update(final PreparedStatementCreator psc, @Nullable final PreparedStatementSetter pss) // new SimplePreparedStatementCreator(sql), pss
호출함

public int update(final PreparedStatementCreator psc, final KeyHolder generatedKeyHolder)*

//보통 쿼리 진행될 때 최종적으로 호출되는, update메서드
logger.debug("Executing prepared SQL update");
return *updateCount*(
	//ps는 PreparedStatement임
	execute(psc, ps -> {
			try {
				if (pss != null) {
					pss.setValues(ps);
				}
				int rows = ps.executeUpdate();
				~~if (logger.isTraceEnabled()) {
					logger.trace("SQL update affected " + rows + " rows");
				} 있는건데, 없다고 생각하고 코드보기~~
				return rows;
			}
			finally {
				if (pss instanceof ParameterDisposer) {
					((ParameterDisposer) pss).cleanupParameters();
				}
			}
		}, true)
);
```

update() 메서드가 반환하는, int는 SQl 실행결과에 영향을 받은 행(row)의 수 이다

`return *updateCount*(~)`

### KeyHolder란?

: 자동으로 생성된 키 값을 구해준다

사용법

```java
*int update(final PreparedStatementCreator psc, final KeyHolder generatedKeyHolder)*

public void insert(final Member member) {
    KeyHolder keyHolder = new GeneratedKeyHolder();

    jdbcTemplate.update((Connection con) -> {
				//PreparedStatement 객체를 생성하는데 두 번째 파라미터로 String 배열인 ["id"]를 주었다.
				// 이 **두번째 파라미터는 자동 생성되는 키 칼럼 목록을 지정할 때 사용**
        PreparedStatement pstmt = con.prepareStatement(
            "insert into MEMBER (..)" + "values (?, ?, ?, ?)",
            new String[]{"id"}
				);

        // setString 을 통해 ? 에 값을 할당한다.
        pstmt.setString(1, ..);
        return pstmt;
    }, keyHolder);

//KeyHolder에 보관된 값은 getKey() 메서드를 이용해 가져온다
Number keyValue = keyHolder.getKey();// 자동
}
```

## JdbcTemplate execute() 메서드

:이외에 임의의 SQL을 실행할 때는 execute() 메서드를 사용할 수 있다. 

테이블을 생성하는 DDL 등에 사용한다

```java
execute(PreparedStatementCreator psc, PreparedStatementCallback<T> action, boolean closeResources) {
	/ ... /

	Connection con = DataSourceUtils.getConnection(obtainDataSource());
			PreparedStatement ps = null;
			try {
				ps = psc.createPreparedStatement(con); **//sql기반으로 PreparedStatement만들고**
				applyStatementSettings(ps);
				T result = **action.doInPreparedStatement(ps);  //action에 있는 pss(?인자들)을 적용**
				handleWarnings(ps);
				return result; // result가 행의 숫자임
			}
			catch (SQLException ex) {
				/ ... /
			}
			finally { // closeStatement 닫고, releaseConnection도 닫아?줌
				if (closeResources) {
					if (psc instanceof ParameterDisposer) {
						((ParameterDisposer) psc).cleanupParameters();
					}
					JdbcUtils.closeStatement(ps);
					DataSourceUtils.releaseConnection(con, getDataSource());
			}
		}

//
PreparedStatementCallback<T> action =
(ps) -> {
			try {
				if (pss != null) { 
					pss.setValues(ps);**//sql의 ?에 인자들 넣어주는 것**
				}
				int rows = ps.executeUpdate();
				return rows;
			}

```

## NamedParameterJdbcTemplate 사용하기

### 사용이유

(문제 상황)

JdbcTemplate은 파라미터를 순서대로 바인딩한다.

순서에 맞춰 파라미터를 바인딩 하기는 쉽지 않을 것이고, 만약 실수로 파라미터가 잘못 들어가게 되면 버그를 고치기가 매우 어렵다.

(해결방법)

NamedParameterJdbcTemplate는 이름을 지정해서 파라미터를 바인딩할 수 있다.

### 사용법

1. sql문에 ":파라미터 이름” 
    
    ⇒ 파라미터 이름에 해당하는 파라미터를 넣어준다
    
2. Map 혹은 SqlParameterSource을 사용해 파라미터를 전달한다
    1. Map구조 사용하기
        
        ```java
        Map<String, Object> param = Map.of("id", id);
        Game game = namedTemplate.queryForObject(sql, param, gameRowMapper());
        ```
        
    2. SqlParameterSource 인터페이스의 구현체 사용하기
        
        아래, **SqlParameterSource란?** 참고하기
        
    

예시)

```java
public int useNamedParameterJdbcTemplate(String firstName) {
        String sql = "select count(*) from customers where first_name = :first_name";
				SqlParameterSource paramSource = new MapSqlParameterSource("first_name", firstName);
        return namedParameterJdbcTemplate.queryForObject(sql, paramSource, Integer.class);
}

// 반환타입 지정하는 경우
public <T> T queryForObject(String sql, SqlParameterSource paramSource, Class<T> requiredType)
public <T> T queryForObject(String sql, Map<String, ?> paramMap, Class<T> requiredType)
// RowMapper 이용하는 경우
public <T> T queryForObject(String sql, SqlParameterSource paramSource, RowMapper<T> rowMapper)
public <T> T queryForObject(String sql, Map<String, ?> paramMap, RowMapper<T>rowMapper)

// NamedParameterJdbcTemplate 클래스
public <T> T queryForObject(String sql, SqlParameterSource paramSource, RowMapper<T> rowMapper) {
//query(PreparedStatementCreator psc, RowMapper<T> rowMapper)
		List<T> results = getJdbcOperations().query(getPreparedStatementCreator(sql, paramSource), rowMapper);
		return DataAccessUtils.nullableSingleResult(results);
}

protected PreparedStatementCreator getPreparedStatementCreator(String sql, SqlParameterSource paramSource) {
		return getPreparedStatementCreator(sql, paramSource, null);
	}	

protected PreparedStatementCreator getPreparedStatementCreator(String sql, SqlParameterSource paramSource,
			@Nullable Consumer<PreparedStatementCreatorFactory> customizer) {

		ParsedSql parsedSql = getParsedSql(sql); //sql 파싱하고
		//파싱된 sql과 sqlParam소스로 statement  객체 생성
		PreparedStatementCreatorFactory pscf = getPreparedStatementCreatorFactory(parsedSql, paramSource);
		~~if (customizer != null) {
			customizer.accept(pscf);
		}~~
		Object[] params = NamedParameterUtils.buildValueArray(parsedSql, paramSource, null);
		return pscf.newPreparedStatementCreator(params);
	}
```

### SqlParameterSource란?

MapSqlParameterSource 사용하기

```java
public class MapSqlParameterSource extends AbstractSqlParameterSource {

	private final Map<String, Object> values = new LinkedHashMap<>();

	public MapSqlParameterSource(String paramName, @Nullable Object value){
			addValue(paramName, value);
		}
	public MapSqlParameterSource(@Nullable Map<String, ?> values) {
			addValues(values);
		}
}

//생성시, 값 저장하기
new MapSqlParameterSource("first_name", firstName);

//메서드 체인 이용하기
new MapSqlParameterSource()
.addValue("itemName", itemName )
.addValue("price", price)
.addValue("quantity", quantity)
.addValue("id", itemId); 
template.update(sql, param);
```

BeanPropertySqlParameterSource 사용하기

자바빈 프로퍼티 규약을 통해 자동으로 파라미터 객체를 생성한다.

```java
public class BeanPropertySqlParameterSource extends AbstractSqlParameterSource {

	private final BeanWrapper beanWrapper;

	public BeanPropertySqlParameterSource(Object object) {
		this.beanWrapper = PropertyAccessorFactory.forBeanPropertyAccess(object);
}
```

- 주어진 JavaBean 객체의 bean 속성에서 매개변수 값을 얻는다.
- Bean 속성의 이름은 매개변수 이름과 일치해야 합니다.

### 자바빈이란?

: 자바빈 규약 또는 자바빈 관례에 따라 만들어진 클래스

### 자바빈 규약이란?

1. **자바빈은 기본(default)패키지가 아닌, 특정 패키지에 속해 있어야 한다.**
2. **기본 생성자가 존재해야 한다.**
3. **멤버변수의 접근제어자는 private로 선언되어야 한다.**
4. **멤버변수에 접근 가능한 getter 와 setter 메서드가 존재해야 한다.**
    - 메서드 작성시, **get멤버변수이름,**  **set멤버변수이름 이어야 한다.**
5. **getter 와 setter는 접근자가 public으로 선언되어야 한다.**
6. **직렬화 되어 있어야 한다. (선택사항)**
    
    * 직렬화란?
    
    : 객체를 입출력에 사용할 수 있도록 객체의 멤버들을 바이트 형태로 변환시키는 것
    
    java.io.Serializable 인터페이스를 상속하여 직렬화할 수 있다.
    

```java
package nextstep.helloworld.jdbc; **//1번**

public class Customer implements Serializable { //6번
		**//3번**
    private long id; 
    private String firstName, lastName;

		**//2번**
    public Customer() {
    }

    public Customer(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    public Customer(long id, String firstName, String lastName) {
        this.id = id;
        this.firstName = firstName;
        this.lastName = lastName;
    }

		// 4-5번
    public long getId() {
        return id;
    }

    public String getFirstName() {
        return firstName;
    }

    public String getLastName() {
        return lastName;
    }
}
```

## SimpleJdbcInsert 사용하기

### 사용 이유

: JdbcTemplate를 사용하면서 데이터베이스에 값을 직접 삽입하다보면, 삽입 직후 조회(READ) 과정없이 곧장 삽입된 데이터의 primary key 값을 얻고 싶을 때 사용한다.

### 사용방법

: SimpleJdbcInsert는 생성 시점에 데이터베이스 테이블의 메타 데이터를 조회한다. 따라서, 테이블에 어떤 칼럼이있는지 확인할 수 있으므로 usingColumns을 생략할 수 있다. 만약 특정 칼럼만 지정해서 저장하고싶다면 usingColumns를 사용하면 된다.

초기 설정

- **.withTableName("테이블 이름")**
    
    : 데이터 저장할 테이블 명 지정
    
- .usingColums(String… columnNames)
    
    : INSERT SQL에 사용할 칼럼을 지정한다. 특정 값만 저장하고 싶을 때 사용한다. 생략할수 있다.
    
- **.usingGeneratedKeyColumns("테이블의 기본키 column명");**

사용법
```java
//NamedParameterJdbcTemplate와 같이 SqlParameterSource 구현체나 Map 자료구조로 파라미터 값을 입력한다

executeAndReturnKey(Map<String, ?> args)
executeAndReturnKey(SqlParameterSource parameterSource)

//사용 예시
public void saveAll(List<WinnerEntity> winners) {
        SqlParameterSource[] sqlParameterSource = SqlParameterSourceUtils.createBatch(winners);
        Arrays.stream(simpleJdbcInsert.executeBatch(sqlParameterSource));
    }

public int[] executeBatch(SqlParameterSource... batch) {
		return doExecuteBatch(batch);
	}
```
