# 안전하게 코드를 작성하는 법, 시큐어 코딩

## SQL Injection
- Injection(인젝션)은 우리 말로 '삽입'을 의미한다. 클라이언트가 웹 서버로 요청을 전달하는 과정에서 DB 데이터를 조회하는 SQL문에 공격 문자열이 추가 '삽입' 되는 것을 말한다.
- 공격 문자열을 웹 애플리케이션에서 검증을 통해 걸러내지 못하면 심각한 데이터의 유출로 이어질 수 있다.
- 웹 사이트는 서비스를 요청한 클라이언트가 선량한 고객인지, 악의적 목적을 가진 해커인지를 구분하지 않으며 정상 요청이라 판단되면 응답한다.

### 로그인 과정에서 사용되는 SQL 쿼리
> SELECT * FROM MEMBERS WHERE ID=USER_ID AND PASSWORD = USER_PW;
- user_id, user_pw 같은 변수가 바로 공격 문자열이 삽입될 수 있는 취약 포인트이다. 사용자가 입력한 값을 담는 공간이기 때문이다.
- user_id: kim, user_pw: 12345로 로그인을 시도한다 했을 때, 아래와 같은 SQL 쿼리가 실행된다.
  > select * from members where id='jin' and password='12345';
  - members 테이블에 where 절을 만족하는 데이터가 한 건 이상 있다면, 서버는 그 결과를 보고 사용자의 로그인을 허락한다.
  - 해커는 where 절에 사용된 변수에 공격 문자열을 삽입해 <b>where 절이 참이 되게 만들어<b> 서버로 접근한다.
    > select * from members where id='jin' and password=<b>'or 1=1--'<b>;
    - password에 담긴 'or 1=1--'과 같은 문자열이 바로 공격을 위한 값 조작이다.
    - or 조건식을 통해 1=1이 참으로 판단하며, --는 이하 뒷 문장을 주석 처리하여 SQL 쿼리문의 구조를 변경해버린다.
    - 이러한 상황에서 jin이라는 유저명을 가진 클라이언트가 실제 존재한다면, 해커는 로그인에 성공하게 되는 것이다.

- 사용자가 입력한 값이 DB 쿼리문에 포함된다면 바로 그 쿼리문이 SQL 인젝션의 공격을 받을 수 있는 포인트가 된다.
- 그렇기 때문에, 클라이언트가 누군지 모르는 상태에서 사용자의 입력값이 공격 문자열인지 검증하기 위한 secure coding이 매우 중요하다.
- 클라이언트 입력값이 필터링 과정 없이 그대로 SQL 쿼리문에 사용되는 코드는 언제나 취약하다. 공격 문자열이 SQL 문장에 삽입되어 문장 구조를 변경해버릴 수 있기 때문이다.

### 공격 문자열에 취약한 코드
```java
String username = request.getParameter("user");
String password = request.getParameter("password");

Statement sm = conn.createStatement();

ResultSet result = sm.executeQuery("select count(*) from members where user='"+user+"' and password='"+password+"'");
```

### 취약점을 보완한 시큐어 코딩 코드
```java
String username = request.getParameter("user");
String password = request.getParameter("password");

PreparedStatement psm = conn.createStatement(SecureQuery);
String SecureQuery = "select count(*) from members where user=? and password=?";

psm.setString(1, user);
psm.setString(2, password);

ResultSet rs = psm.executeQuery();
```
- 상기와 같이, PreparedStatement를 사용한 코드를 통해 안전성을 확보할 수 있다. 이를 사용하면, 쿼리문을 미리 컴파일하여 실행 계획을 캐시에 저장해놓기 때문에 이후 조작된
  공격 문자열을 통해 쿼리문의 구조를 바꿀 수 없게 된다. 다시 말해, 동일한 구조지만 공격 문자열이 탑재된 쿼리문이 들어왔을 때 다시 컴파일하는 것이 아니라 캐시에 저장된 실행 계획을
  사용하기 때문에 SQL 문장이 변경될 일이 없게 되는 것이다.
- 이와 같이, SQL Injection 공격에 안전한 시큐어 코딩은 매우 중요하다.

