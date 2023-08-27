# SqlTuning
업무에 바로 쓰는 SQL튜닝

* 1장. Mysql 과 MariaDB 개요   
* 2장. sql 튜닝 용어를 직관적으로 이해하기   
* 3장. sql 튜닝의 실행계획 파헤치기   
* 4장. 악성 sql 튜닝으로 초보자 탈출하기   
* 5장. 악성 sql 튜닝으로 전문가 되기   

### 1장. Mysql 과 MariaDB 개요 

1. MYSQL과 오라클의 구조적 차이
   
   - 오라클은 서버이중화 시 각각의 DB서버가 하나의 스토리지를 공유한다.
   - MYSQL은 서버이중화 시 각각의 DB서버가 각각의 스토리지를 사용한다.
   - 그래서 MYSQL은 이중화 시 주로 마스터(주)-슬레이브(종)구조를 사용한다.
   - 마스터노드는 쓰기,읽기 모두 가능. 슬레이브 노드는 읽기만 가능하다.
   - 보통 마스터에서는 INSERT, UPDATE, DELETE 슬레이브에서는 SELECT 만 처리하는 식으로 이중화한다.   
   ```
   * 쿼리 오프로딩
   
   - DB서버의 트랜잭션에서 쓰기와 읽기를 각각분리하여 DB 처리량을 늘리는 성능향상 기법
   - 쓰기 트랜잭션 : UPDATE, INSERT, DELETE
   - 읽기 트랜잭션 : SELECT 
   ```
2. MYSQL과 오라클의 지원 기능의 차이

   - 오라클과 MYSQL은 제공하는 조인알고리즘 기능에 차이가 있다.
   - MYSQL 은 대부분 중첩루프조인 방식을 제공
   - 오라클은 그외에 정렬병합조인, 해시조인 방식도 제공한다.
   - (오라클의 조인성능이 더 뛰어나다고 볼 수 있는듯)
   - MYSQL은 오라클과 달리 데이터를 저장하는 스토리지 엔진개념을 포함하므로
   - 오픈소스 DBMS를 바로 꽂아서 사용할 수 있는 확장성이 특징이다(?)
   - MYSQL은 오라클 대비 메모리 사용률이 상대적으로 낮아서 비교적 낮은 사양에서도 가능하다.
   - MYSQL은 1MB 메모리 환경에서도 가능하나, 오라클은 최소 수백MB의 환경이 제공되어야 설치가능.

3. MYSQL과 오라클L의 SQL구문의 차이

   - INNULL / NVL
   - LIMIT 5 / ROWNUM <= 5
   - NOW() / SYSDATE
   - IF / DECODE
   - DATE_FORMAT, TO_CHAR
   - SUBSTRING / SUBSTR 등
  
4. Mysql SWOT 분석

   - S : 오픈소스(무료), 경량
   - W : 조인알고리즘 부족
   - O : 스토리지 엔진 확장성
   - T :  Mysql 라이선스 정책 (오라클에 인수되었으므로 유료화가능성) / MARIADB는 괜찮 ^^
  
### 2장. SQL 튜닝용어를 직관적으로 이해하기

1. 물리엔진과 오브젝트 용어

   - parser 파서 : SQL문 분석. 문법, 구문검사
   - optimizer 옵티마이저 : 사용자가 요청한 데이터를 빠르고 효율적으로 찾아가는 전략 계획 수립
   - 스토리지 엔진 (innoDB, MyISAM, MEMORY등) : 디스크나 메모리에서 데이터를 가져오는 역할.
   - innoDB엔진을 주로 사용함. 대량의 쓰기 트랜잭션이 발생하면 MyISAM엔진, 메모리 데이터를 빠르게 읽으려면 MEMORY 엔진을 사용하는 식으로 응용가능.

2. 옵티마이저 

   - 옵티마이저는 mysql의 핵심엔진중 하나로 DBMS의 두뇌.
   - 어떤순서로 테이블접근할지, 인덱스 사용여부, 어떤인덱스를 사용할지, 임시테이블을 사용할지 등 전략 수립
   - 실행계획으로 도출할수 있는 경우의 수가 너무 많을때는 시간이 오래걸리므로, 모든 실행계획을 다 판단하지는 않음.
   - 즉 옵티마니저가 선택한 전략이 최상이 실행계획은 아닐 수 도 있음.

3. DB오브젝트 용어

   - 기본키(PK) : mysql에서 기본키는 클러스터형 인덱스로 작동한다. 기본키의 구성열 순서를 기준으로 물리적인
   - 스토리지에 데이터가 쌓이므로 비슷한 기본 키값들은 근거리에 저장된다. 기본키를 활용하여 인덱스 스캔을 수행하면
   - 테이블 데이터에 더 빠르게 접근 할 수 있다.

     ```
     * 기본키 인덱스 주의사항.

     기본키와 똑같은 인덱스를 생성하면 인덱스가 저장되는 물리적 공간 낭비와 인덱스 정렬의 오버헤드가 발생함.
     아래 예시 처럼 기본키와 똑같은 인덱스는 생성하지 말자.

     CREATE TABLE MEMBER (
        ID INT(11),
        NAME VARCHAR(14) NOT NULL
        PRIMARY KEY (ID),
        INDEX_I_ID (ID)
     )

     ```

     - 인덱스 : 모든 데이터를 처음부터 끝까지 전부 차례로 검색하는 비효율적인 방식을 개선하기위해 인덱스가 필요.
     - 책의 목차와 비슷하다고 볼 수 있음. 인덱스를 사용하면 원하는 페이지를 빠르게 찾을 수 있다. 
     - 
     - 
