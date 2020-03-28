# Partition
<br/>

## MySQL 샘플 데이터 추가하기

아래 깃헙에서 ZIP 파일을 다운로드 받아서 압축을 푼 후 해당 디렉토리로 이동한다. <br/>
그리고 아래와 같이 import 한다.<br/>
<pre>
$ mysql -u root -p < employees.sql
</pre>
추가 후 employees 데이터베이스가 생성된 것을 확인할 수 있다.<br/>
<br/>
https://github.com/datacharmer/test_db <br/>
https://futurists.tistory.com/19 <br/>
<br/><br/>

## 파티셔닝 하기

파티셔닝 하려는 employees 테이블이 외래키가 없는데도 불구하고 외래키 때문에 파티셔닝을 할 수 없다는 에러가 뜬다.<br/>
그래서 아래와 같이 employees_2 테이블을 만들어서 데이터만 복제했다.<br/>
<pre>
create table employees_2
as select * from employees;
</pre>
그리고 연도를 기준으로 파티셔닝을 했다.<br/>
<pre>
alter table employees_2
partition by range(year(hire_date)) (
	partition p1985 values less than (1986),
	partition p1986 values less than (1987),
	partition p1987 values less than (1988),
	partition p1988 values less than (1989),
	partition p1989 values less than (1990),
	partition p1990 values less than (1991),
	partition p1991 values less than (1992),
	partition p1992 values less than (1993),
	partition p1993 values less than (1994),
	partition p1994 values less than (1995),
	partition p1995 values less than (1996),
	partition p1996 values less than (1997),
	partition p1997 values less than (1998),
	partition p1998 values less than (1999),
	partition p1999 values less than (2000),
	partition p2000 values less than maxvalue
);
</pre>
아래 쿼리로 employees_2 테이블에 생성된 파티션 목록을 확인할 수 있다.<br/>
<pre>
select * from information_schema.partitions
where table_name = 'employees_2';
</pre>
특정 파티션을 조건으로 테이블 데이터 조회하기.<br/>
<pre>
select * from employees_2 partition(p1986) e;
</pre>
조인해서 데이터를 조회할 경우에도 조인할 범위가 줄어들어서 이전보다 더 빠르게 조회할 수 있다.<br/>
<pre>
select * from employees_2 partition(p1986) e, titles t, salaries s
where e.emp_no = t.emp_no and e.emp_no = s.emp_no;
</pre>
<br/><br/>

## 파티션 프루닝 (* pruning: 가지치기)

파티션 프루닝은 다음과 같은 WHERE 구문을 사용하여 데이터를 추출하는 경우 관계된 파티션만 접근하게 하는 기술을 말한다.<br/>
<pre>
partition_column = constant
partition_column IN (constant1, constant2, ..., constantN)
</pre>
만약 아래와 같은 예제가 있다고 가정하자.<br/>
<pre>
CREATE TABLE t2 (
   fname VARCHAR(50) NOT NULL,
   lname VARCHAR(50) NOT NULL,
   region_code TINYINT UNSIGNED NOT NULL,
   dob DATE NOT NULL
)
PARTITION BY RANGE(YEAR(dob)) (
   PARTITION d0 VALUES LESS THAN (1970),
   ...
);
</pre>
위의 예제의 경우 다음과 같은 쿼리들은 프루닝을 사용할 수 있다.<br/>
<pre>
SELECT * FROM t2 WHERE dob = '1982-06-23';

UPDATE t2 SET region_code = 8 WHERE dob BETWEEN '1991-02-15' AND '1997-04-25';
</pre>
https://mysqldba.tistory.com/130<br/>
<br/><br/>

### 파티셔닝 단점
- 데이터를 입력 받으면 어디에 넣어야 하는가?
- 큰 테이블을 Join 하는 것이 힘들다.

### 파티셔닝 장점
- 테이블을 작게 나누어 입력에 강점을 보인다.

### 파티셔닝 제약사항
- 모든 파티션은 동일한 스토리지 엔진 사용.
- 테이블과 인덱스를 별도로 파티션 할 수 없다. 테이블과 인덱스를 같이 파티셔닝 해야 한다.
- 파티션 된 테이블은 외래키(foreign key)를 지원하지 않는다.
- 파티션 된 테이블은 FullText index를 지원하지 않는다.
- 파티션 된 테이블은 Geometry 컬럼 타입을 지원하지 않는다.
- 한 테이블당 최대 파티션 개수는 1,024개이다.

<br/>
http://tnsdogfoot.blogspot.com/2014/09/2014-9-25.html <br/>

<br/><br/>






