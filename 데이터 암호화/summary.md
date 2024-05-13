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

### 대칭 키 암호 알고리즘
  - 암복호화 알고리즘에 사용하는 암복호화 두 키가 동일하면 '대칭 키 알고리즘'이라고 한다.
  - DES, 3DES, AES, ARIA, SEED, LEA 등으로 다양하다.
  - 암호화의 강도를 위해 128비트 이상의 알고리즘 사용을 권장하는데, DES(64비트), 3DES(112비트) 등은 암호화 강도가 약하므로 가급적 사용하지 않는다.
  - AES(128/192/256), ARIA(128/192/256), SEED(128/256), LEA(128/192/256) 비트로 암호화 지원
    ```java
    import java.security.MessageDigest;
    import java.security.SecureRandom;
    import java.util.Arrays;
    import java.util.Base64;
    import javax.crypto.Chiper;
    import javax.crypto.spec.IvParameterSpec;
    import javax.crypto.spec.SecretKeySpec;

    public class AESEncryption {

      private SecretKeySpec secretKey;
      private byte[] initVetor;

      // 대칭키 암호화 메서드
      public String encrypt(String plainText, String secretString) throws Exception {

        // 암호화에 사용할 변수 선언
        byte[] sha256 = null;
        byte[] cipherText = null;

        // secretString에 대한 hash value 생성
        sha256 = MessageDigest.getInstance("SHA-256").digest(secretString.getBytes("UTF-8"));
        // 해시 값을 16바이트(128비트) 배열에 나누어 저장
        sha256 = Arrays.copyOf(sha256, 16);
        // 저장된 해시 값을 AES 알고리즘을 이용하여 secret key 생성
        secretKey = new SecretKeySpec(sha256, "AES");

        // 첫 번째 블록 암호화를 위해 난수(Random) 값을 이용하여 초기화 벡터 생성
        initVector = new byte[16];
        SecureRandom secureRandom = new SecureRandom();
        secureRandom.nextBytes(initvector);

        /* 암호화 객체 생성
        * 암호화 알고리즘: AES
        * 블록 암호화 운영 코드: CBC
        * 패딩: PKCS5Padding
        */
        Cipher cipher = Cipher.getInstance("AES/CBC/PKCS7Padding");

        // 생성된 암호화 객체 초기화 - 암호화 모드, 대칭 키, 초기화 벡터 값 사용
        cipher.init(Cipher.ENCRYPT_MODE, secretKey, new IvParameterSpec(initVector));

        // 원본 데이터 최종 암호화
        cipherText = cipher.doFinal(plainText, getBytes("UTF_8"));
        // Base64로 Encoding하여 암호문 반환
        return Base64.getEncoder().encodeToString(cipherText);
      }

      // 대칭키 복호화 메서드
      public String decrypt(String cypherText, SecretKeySpec secretKey) throws Exception {
        byte[] plainText = null;

        // 암호화 객체 생성
        Cipher cipher = Cipher.getInstance("AES/CBC/PKCS7Padding");
          
        /*
        * 암호화 모드, 대칭키, 초기화 벡터 값을 사용하여 암호화 객체 초기화
        * secretKey(대칭키)와 initVector(초기화 벡터)는 encrypt에 사용한 값과 동일하게 설정
        */
        cipher.init(Cipher.DECRYPT_MODE, secretKey, new IvParameterSpec(initVector));

        // 암호화 데이터 최종 복호화
        plainText = cipher.doFinal(Base64.getDecoder(), decode(cipherText));

        // byte > String 으로 형식 변환 후 반환
        return new String(plainText);
      }

      // 대칭 키 반환 메서드
      public SecretKeySpec getSecretKey() {
        return secretKey;
      }
    
    }
    ```
    - secretKey를 이용하여 평문을 암호문을 변환하는 전체 로직
      1. 암호화에 사용할 비밀키를 만든다.
      2. 비밀키, 암호 알고리즘 AES.. 등을 매개변수로 하여 secretKey 객체를 생성한다.
      3. 암호화 객체를 생성한다.(암호화 알고리즘/운용 모드/패딩 방식)
          - 운용 모드(ECB, CBC 등)
            - ECB(Electronic Code Book): 데이터를 블록으로 나눈 후 각각 암호화
            - CBC(Cipher Block-Chaining): 데이터를 블록으로 나누고 블록 간 체인을 만들어 암호화. 이전 블록의 암호화 결과를 다음 블록 암호화에 사용하므로 '체인'이라 한다.
          - 패딩 방식: 블록 암호화 방식을 사용할 때 각 블록의 크기가 모두 동일하게 하기 위해 사용. 16바이트 블록의 경우, 모자라는 블록이 있으면 크기를 맞추기 위해 임의의 값을
            추가하는데 이를 패딩이라 한다.
            - PKCS#5: 패딩 값을 최대 8바이트까지 채운다.
            - PKCS#7: 패딩 값을 최대 255바이트까지 채운다. 근래 대부분의 대칭키 암호 알고리즘 블록 크기는 16바이트 이상이므로 이를 많이 사용한다.
      4. 최종 암호화 한다. plainText의 문자열을 바이트 값으로 변환하여 최종 암호화하고, Base64 형식으로 인코딩하여 최종 반환한다.
        > Base64 Encoding (이진 데이터 > 문자열 데이터)
        > 이진 데이터는 네트워크 통신 과정에서 신호 왜곡, 비트 오류 등으로 손실될 위험이 있고 데이터베이스 관점에서 문자열 데이터보다 처리가 어렵다. 그렇기 때문에 최종 암호화
          과정에서 Base64 기반 인코딩을 한번 더 해준다.
    - 복호화는 암호화 과정과 동일하게 복호화를 위한 옵션들을 사용한 뒤, 최종적으로 'return new String(plainText)'를 통해 복호화 문자열을 반환한다.

### 비대칭 키 암호 알고리즘
  - 암호화에 사용하는 키와 복호화에 사용하는 키가 서로 다르다.
  - 공개키와 비밀키는 하나의 쌍으로, 공개키로 암호화하면 복호화는 반드시 쌍을 이루는 비밀키로 해야 한다. 다른 쌍의 비밀키로는 불가능하다.
  - 암호화 복호화에 공개키, 비밀키가 모두 쓰일 수 있으나 쌍이 되는 키로 암복호화가 가능하다는 것을 알아야 한다.
  - A <-> B 간에 데이터 암호화 통신을 가정한다.
    1. B는 A에게 자신의 공개키를 먼저 전달한다.(B는 자신만의 공개키-비밀키 쌍을 갖고 있다.)
    2. A는 전달받은 B의 공개키를 통해 평문을 암호문으로 암호화한다.
    3. A가 B로 데이터를 전달한다.
    4. B는 전달받은 암호문을 자신의 비밀키로 복호화한다.

#### 대칭키와 비대칭키 암호 알고리즘의 차이
  - 대칭키 방식이 비대칭키 방식에 비해 연산 속도가 빠르지만, 하나의 키를 상대방에게 전달하기 어렵다는 단점이 있다.
  - 대칭키 방식에서 보안이 안되는 하나의 키를 암호화해서 보내는 방식을 비대칭키 암호화 방식이라고 생각하면 된다.
  - 기존 하나의 보안키를 공개키로 암호화해서 비밀키의 형태로 상대에게 전달하고, 암호화하는데 사용한 공개키를 함께 전달하여 받은 사람이 공개키로 비밀키를 복호화할 수 있도록 한다.
  - 처음 비밀 키를 교환하는데 비대칭키 암호 알고리즘을 사용하고, 비밀키 교환이 성공적으로 이루어지면 그 비밀키를 대칭키로 활용하여 이후에는 빠른 속도를 보장하는 대칭키 암호
    알고리즘으로 문서를 암복호화한다.

|구분|대칭 키|비대칭 키|
|:---:|:---:|:---:|
|키 갯수|1개(암복호화 동일키 사용)|2개(서로 다른 공개키, 비밀키 사용)|
|키 길이|192/256/512비트|2048/4096비트|
|암호화 속도|빠름|느림|
|암호 알고리즘|AES/ARIA/SEED|RSA|
|키 교환|어려움|쉬움|
|사용 목적|데이터 암호화|전자서명, 키 교환, 인증|

#### 비대칭 키 암호 알고리즘 자바 코드
  ```java
  import javax.crypto.Cipher;
  import java.security.KeyPair;
  import java.security.KeyPairGenerator;
  import java.security.NoSuchAlgorithmException;
  import java.security.PrivateKey;
  import java.security.PublicKey;
  import java.util.Base64;

  public class RSAEncryption {

    private PrivateKey privateKey = null;
    private PublicKey publicKey = null;

    public RSAEncryption() {
      this.makeKey();
    }

    // 비대칭 키 생성 메서드
    public KeyPair makeKey() {

      KeyPairGenerator keyPairGenerator = null;
      KeyPair keyPair = null;

      try {
        // 1. 비대칭 키 생성을 위한 객체 생성
        keyPairGenerator = KeyPairGenerator.getInstance("RSA");

        // 2. RSA 알고리즘에서 사용할 비트수 결정
        keyPairGenerator.initialize(2048);

        // 3. 비대칭 키 한 쌍(공개 키, 비밀 키) 생성
        keyPair = keyPairGenerator.generateKeyPair();

        // 4. 비밀 키 변수 할당
        privateKey = keyPair.getPrivate();
        // 5. 공개 키 변수 할당
        publicKey = keyPair.getPublic();

        } catch (NoSuchAlgorithmException e) {
          e.printStackTrace();
      }
      return KeyPair;
    }

    // 암호화 함수
    public String encrypt(String plainText) throws Exception {
      byte[] cipherText = null;

      // 1. 암호화 객체 생성
      Cipher cipher = Cipher.getInstance("RSA");

      // 2. 암호화 모드, 공개 키를 입력하여 암호화 객체를 초기화
      cipher.init(Cipher.ENCRYPT_MODE, publicKey);

      // 3. 데이터 암호화
      cipherText = cipher.doFinal(plainText.getBytes());

      // 4. Base64 인코딩 및 최종 암호문 반환
      return Base64.getEncoder().encodeToString(cipherText);
    }

    // 복호화 함수
    public String decrypt(String cipherText) throws Exception {
      byte[] plainText = null;

      // 1. 암호화 객체 생성
      Cipher cipher = Cipher.getInstance("RSA");

      // 2. 복호화 모드, 비밀 키를 통한 암호화 객체 초기화
      cipher.init(Cipher.DECRYPT_MODE, privateKey);

      // 3. 데이터 복호화
      plainText = new String(cipher.doFinal(Base64.getEncoder().encodeToString(cipherText)));

      // 4. 최종 평문 반환
      return plainText;
    }

  }
  ```

### 클라우드 보안
  - 온 프레미스(On-Premise): 회사 내부에 물리 서버를 구축하여 서비스하는 레거시 방식
    - 하드웨어: 서버, 네트워크
    - 소프트웨어: 리눅스, 윈도우와 같은 운영체제
    - 기타: 웹 서버, WAS(Web Applicaion Server), DBMS
  - 클라우드: 몇 번의 버튼 클릭, 아마존 AWS, 마이크로소프트 Azure, Naver Cloud, KT Cloud 등이 존재
    - 클라우드 컴퓨팅: 서버, 스토리지, 네트워크 등 다양한 컴퓨팅 자원을 풀 pool 형태로 인터넷을 통해 제공
    - 클라우드 서비스: 클라우드 컴퓨팅 기술을 사용하여 자원을 서비스 형태로 제공하는 것(일종의 렌탈 서비스)
    - SaaS(Software as a Service): 애플리케이션을 대여
    - PaaS(Platform as a Service): 플랫폼을 대여
    - IaaS(Infra as a Service): 인프라를 대여

### 클라우드 서비스를 사용할 때의 공유 책임 모델
  <img width="500" src="https://github.com/FutureMaker0/designPattern_singleton/assets/120623320/f0484b56-d23e-4648-a56b-67105284ca8e">
  <출저 - aws.amazon.com>







