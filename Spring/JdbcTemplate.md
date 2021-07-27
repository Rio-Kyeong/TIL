# JdbcTemplate
```
JdbcTemplate를 사용함으로써 불필요한 중복 코드들을 줄여 줍니다.
```
## JdbcTemplate 환경설정
<pre>
JdbcTemplate를 사용하기 위해서 필요한 Library를 <b>pom.xml 파일</b>에 설정한다.
Maven Repository  : <a href="https://mvnrepository.com/">https://mvnrepository.com/</a>
</pre>
### pom.xml
**spring-jdbc** 또는 **c3p0**를 사용한다.
```xml
<!-- 
    ojdbc :
    Oracle과 Java가 연동할 수 있는 API를 제공한다.
    Connection, PreparedStatement, ResultSet 등이 포함되어 있다.

    MyBatis 사용 시 Import 해준다.
-->
<dependency>
    <groupId>com.oracle</groupId>
    <artifactId>ojdbc6</artifactId>
    <version>12.1.0.2</version>
</dependency>

<!-- 
    Spring :
    JDBC API를 쓰기 편리하게 Spring에서 추상화 시켜놓은 모듈 
    MyBatis가 이 묘듈을 사용한다.    
-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>4.1.6.RELEASE</version>
</dependency>

<!-- 
    c3p0 :
    Connection Pool을 쉽게 생성할 수 있도록 추상화시킨 라이브러리
    com.mchange.v2.c3p0.ComboPooledDataSource
    클래스를 통해서 Connection Pool을 설정한다.
    
    빠른 처리/대응을 위해서 여러 개의 Connection을 미리 만들어 둔다.
    기본 설정으로는 최대 15개의 Connection을 생성한다.
-->
<dependency>
    <groupId>com.mchange</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.5</version>
</dependency>
```
### Repository
ojdbc를 사용하기 위해 Maven Repository를 따로 등록한다.
```xml
<!--
    JDBC : Java DataBase Connector

    Oracle JDBC (OJDBC)는 Maven Global Repository에 등록되어 있지 않음.
    ojdbc를 사용하기 위해서는 Maven Repository를 따로 등록해주어야 한다.
    등록을 통해서 ojdbc 라이브러리를 import할 수 있다.

    <dependencies> 태그 위에 기술한다.
-->
<repositories>
    <repository>
        <id>oracle</id>
        <name>ORACLE JDBC Repository</name>
        <url>http://maven.jahia.org/maven2</url>
    </repository>
</repositories>
```
## DAO 설정
### 커넥션 풀 사용하기 
```
DB Connection을 맺는 과정이 부하가 많이 걸리는 작업이라 동시에 많은 사람들이 DB 커넥션을 요구한다면 
최악의 경우 서버가 죽어버리는 문제가 발생할 수도 있기 때문에 커넥션 풀(DBCP)를 사용한다.

커넥션 풀(DBCP)이란 웹 컨테이너(WAS)가 실행되면서 DB와 미리 connection(연결)을 해놓은 객체들을 pool에 저장해두었다가.
클라이언트 요청이 오면 connection을 빌려주고, 처리가 끝나면 다시 connection을 반납받아 pool에 저장하는 방식이다.

JDBC 커넥션 풀을 지원하는 대표적인 오픈소스 중에 아파치 DBCP(dbcp.BasicDataSource)와 C3P0가 있다. 
이들은 Spring, Hibernate 등과 통합되어 DB 커넥션 풀을 제공하는 DataSource를 구성하여 자주 쓰인다.

아래 코드에서는 c3p0의 ComboPooledDataSource를 사용 하였다.
```
### DAO
```java
@Repository
public class MemberDao implements IMemberDao {
    // JdbcTemplate 변수 선언
    private JdbcTemplate template;

    @Autowired
    public MemberDao(ComboPooledDataSource dataSource) {
        this.template = new JdbcTemplate(dataSource);
    }
    // template에 자동 주입받은 dataSource를 받아서 넣어준다.
    // ComboPooledDataSource는 JDBC 커넥션 풀을 지원한다.

    @Override
    public int memberInsert(final Member member) {
        
        int result = 0;
        
        final String sql = "INSERT INTO member (memId, memPw, memMail) values (?,?,?)";
        
    //	1nd
    //  result = template.update(sql, member.getMemId(), member.getMemPw(), member.getMemMail());

    //	2nd : PreparedStatementCreator()를 이용한 방법
    	result = template.update(new PreparedStatementCreator() {
    			
    	    @Override
    		public PreparedStatement createPreparedStatement(Connection conn)throws SQLException {
    			PreparedStatement pstmt = conn.prepareStatement(sql);
    			pstmt.setString(1, member.getMemId());
    			pstmt.setString(2, member.getMemPw());
    			pstmt.setString(3, member.getMemMail());
    			
    			return pstmt;
    		}
    	});	
        return result;
    }

    @Override
    public Member memberSelect(final Member member) {
        
        List<Member> members = null;
        
        final String sql = "SELECT * FROM member WHERE memId = ? AND memPw = ?";
        
    //	1nd : PreparedStatementCreator()와 RowMapper<Generic>()를 이용한 방법
        members = template.query(new PreparedStatementCreator() {
            
            @Override
            public PreparedStatement createPreparedStatement(Connection conn)
                    throws SQLException {
                PreparedStatement pstmt = conn.prepareStatement(sql);
                pstmt.setString(1, member.getMemId());
                pstmt.setString(2, member.getMemPw());
                return pstmt;
            }
        }, new RowMapper<Member>() {

            @Override
            public Member mapRow(ResultSet rs, int rowNum) throws SQLException {
                Member mem = new Member();
                mem.setMemId(rs.getString("memId"));
                mem.setMemPw(rs.getString("memPw"));
                mem.setMemMail(rs.getString("memMail"));
                mem.setMemPurcNum(rs.getInt("memPurcNum"));
                return mem;
            }
        });

        if(members.isEmpty()) 
            return null;
        
        return members.get(0);
    }

    @Override
    public int memberUpdate(final Member member) {
        
        int result = 0;
        
        final String sql = "UPDATE member SET memPw = ?, memMail = ? WHERE memId = ?";
        
    //	1st
    //  result = template.update(sql, member.getMemPw(), member.getMemMail(),  member.getMemId());

    //	2nd : PreparedStatementCreator()를 이용한 방법
    	result = template.update(new PreparedStatementCreator() {
    			
            @Override
            public PreparedStatement createPreparedStatement(Connection conn)throws SQLException {
                PreparedStatement pstmt = conn.prepareStatement(sql);

                pstmt.setString(1, member.getMemPw());
                pstmt.setString(2, member.getMemMail());
                pstmt.setString(3, member.getMemId());
                
                return pstmt;
            }
        });
        return result;
    }

    @Override
    public int memberDelete(final Member member) {
        
        int result = 0;
        
        final String sql = "DELETE member WHERE memId = ? AND memPw = ?";
        
    //	1st
    //  result = template.update(sql, member.getMemId(), member.getMemPw());
        
    //	2nd : PreparedStatementCreator()를 이용한 방법
    	result = template.update(new PreparedStatementCreator() {
    		
    		@Override
    		public PreparedStatement createPreparedStatement(Connection conn)
    				throws SQLException {
    			PreparedStatement pstmt = conn.prepareStatement(sql);
    			pstmt.setString(1, member.getMemId());
    			pstmt.setString(2, member.getMemPw());
    			
    			return pstmt;
    		}
    	});
        return result;
    }
}//class
```
## 스프링 설정 파일을 이용한 DataSource설정
xml 파일 또는 java 파일로 설정할 수 있다. 
### servlet-context.xml
```xml
<!-- dataSource bean -->
<beans:bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <beans:property name="driverClass" value="oracle.jdbc.driver.OracleDriver" />
    <beans:property name="jdbcUrl" value="jdbc:oracle:thin:@localhost:1521:xe" />
    <beans:property name="user" value="scott" />
    <beans:property name="password" value="tiger" />
    <!-- 최대 풀 크기, 기본 값 : 15 -->
    <beans:property name="maxPoolSize" value="200" />
    <!-- 
        체크아웃시간 초과, 기본 값 : 0 
        풀이 소진되었을 때 getConnection()을 호출하는 클라이언트가 연결이 체크인되거나 획득될 때까지 대기하는 시간(밀리초)
    -->
    <beans:property name="checkoutTimeout" value="60000" />
    <!-- 최대 유휴 시간 -->
    <beans:property name="maxIdleTime" value="1800" />
    <!-- 유효 연결 테스트 기간 -->
    <beans:property name="idleConnectionTestPeriod" value="600" />
</beans:bean> 
```
### DBConfig.java
```java
@Configuration
public class DBConfig {

    @Bean
    public ComboPooledDataSource dataSource() throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        
        dataSource.setDriverClass("oracle.jdbc.driver.OracleDriver");
        dataSource.setJdbcUrl("jdbc:oracle:thin:@localhost:1521:xe");
        dataSource.setUser("scott");
        dataSource.setPassword("tiger");
        dataSource.setMaxPoolSize(200);
        dataSource.setCheckoutTimeout(60000);
        dataSource.setMaxIdleTime(1800);
        dataSource.setIdleConnectionTestPeriod(600);
        
        return dataSource;

    }	
}//class
```