# FULLTEXT(전문검색) Index 란?

## 배경

### Like 의 한계

어떤  어플리케이션에서 "스노우보드 잘 타는법'' 과  같은 특정 단어 혹은  문장을검색한다고 해보자.

<figure><img src="../../.gitbook/assets/image (5) (3).png" alt=""><figcaption></figcaption></figure>

DB 에 `like` 구문을 이용해서 검색을 진행할 것이다.&#x20;

하지만 `"스노우보드 잘 타는법"`  을 앞뒤로 포함하는 게시물을 검색해야 한다면?

INDEX 설정을 하였어도 % 위치에 따라 INDEX 가 정상적으로 작동하는 경우가 있지만 반대로 잘못 사용한 경우 **Full Scan 이 발생**할 수 있다.

그럼 어떻게 하면 효율적으로 검색을 할 수 있을까?

가장 널리 알려진 무료 검색엔진 elasticsearch 와 같은 ELK를 도입해서 사용한다면 제일 Best 이긴 하지만.

Mysql 5.7 이상부터 FullText N-gram 전문검색 기능을 지원을 해주고 이 또한 기존 like를 사용하는 것보다 성능이 훨씬우수하다고 검색엔진을 미리 구축하는 것보다 추 후에 데이터 양이 많이 쌓이면 그때 검색엔진을 도입하는게 더 나은 방안이라고 생각된다.

## FULLTEXT(전문검색) 전문검색 인덱스

먼저 전문검색 인덱스를 사용하는 방법에 대해 알아보자.

전문 검색 인덱스를 사용하려면 아래와 같이 반드시 테이블 생성 시 전문검색 인덱스를 생성하는 구문을 같이 넣어주어야 한다.

```sql
 CREATE TABLE IF NOT EXISTS board (
  first_name INT(11) UNSIGNED NOT NULL AUTO_INCREMENT,
  title VARCHAR(100) NOT NULL,
  user_id VARCHAR(350) NULL DEFAULT NULL,
  PRIMARY KEY (`first_name`), 
  FULLTEXT INDEX FT_TITLE (title)  WITH PARSER ngram // 전문 검색 인덱스 생성
)
```

전문 검색 인덱스 알고리즘은 다음 섹션의 알고리즘 종류 부분에 2가지로 나뉘는데

WITH PARSER `ngram` 을 추가 하지 않으면 Built-in parser로 fulltext index를 기본적으로 만든다.

위와 같이 전문 검색 인덱스를 생성하였으면 아래와 같이 사용이 가능하다.

```sql
SELECT * FROM board 
WHERE MATCH(title) AGAINST ('스노우보드');
```

즉, `board` 라는 테이블에 `title`  컬럼에 `스노우보드` 라는 문장을  검색할 때전문검색 인덱스를 이용하겠다 라는 뜻.

## 전문검색 인덱스 알고리즘

### 1. Built-in, Stop-word (구분자) parser

문장의 내용을 공백이나 Tab, 문장 기호, 또는 사용자가 정의한 문자열을 구분자로 등록한다. 테이블이 아래와 같이 있다고 가정해보자.

| id | title        |
| -- | ------------ |
| 1  | 스노우          |
| 2  | 스노우 보드       |
| 3  | 스노우 보드 잘 타는법 |

이제 `스노우` 가 포함된 문자열을 검색하고자 한다. 다음과 같은 구문을 사용하면 된다.

```
SELECT * FROM board WHERE MATCH (title) AGAINST ('스노우');
```

결과는 다음과 같다.

| id | title          |
| -- | -------------- |
| 1  | <p>스노우<br></p> |

id가 2인 row와 3인 row는 결과에 포함되지 않았다. 그 이유는 FullText Index 생성 시 공백을 구분자로 단어 단위로만 저장하게 된다. 따라서 완전히 일치한 단어만 검색 결과에 포함되게 된다.

기존 MySQL의 FullText Search 엔진은 Stopword 방식으로만 인덱싱이 가능했다.&#x20;

하지만 우리는 "스노우"라고 써도, "스노"라고 써도 table\_name의 모든 행을 결과에 포함시킬 수는 없을까?

이때 N-gram 을 사용한다.

### 2. N-gram <a href="#2-n-gram" id="2-n-gram"></a>

> N-gram은 통계학 기반의 언어 모델 중 하나이다. N-gram 언어 모델은 이처럼 다음 단어를 예측할 때 문장 내 모든 단어를 고려하지 않고 특정 단어의 개수`(N)`만 고려한다. 즉, N-gram은 `N`개의 연속적인 단어의 나열을 하나의 묶음(=token)으로 간주한다.

구분자 방식은 추출된 키워드의 일부(키워드의 뒷부분)만 검색하는 것은 불가능하다는 단점도 있다. 이러한 부분을 보완하기 위해 **지정된 규칙이 없는 전문도 분석 및 검색을 가능하게 하는 방법이 N-그램이라는 방식.**

**N-그램이란 본문을 무조건적으로 몇 글자씩 잘라서 인덱싱하는 방법이**다. 구분자에 의한 방법보다는 인덱싱 알고리즘이 복잡하고, 만들어진 인덱스의 크기도 상당히 큰 편**이다**.

N-그램에서 n은 인덱싱할 키워드의 최소 글자(또는 바이트) 수를 의미하는데, 일반적으로는 2글자 단위로 키워드를 쪼개서 인덱싱하는 2-Gram(또는 Bi-Gram이라고도 한다) 방식이 많이 사용한다. 아래는 n-gram 단위별 모델 이름을 나타낸 것이다.

| 모델명          |
| ------------ |
| Unigram(N=1) |
| Bigram(N=2)  |
| Trigram(N=3) |
| 4-gram(N=4)  |



ngram은 토큰 크기에 따라 단어를 잘라서 인덱싱하여 보관하고 있다. 글로벌 변수를 조회하면ngram\_token\_size 사이즈가 default 값으로 '2' 로 설정되어있는걸 볼 수 있다. -> mysql 설정파일에서 설정가능.

만약 토큰사이즈가 2 면 **"스노우 보드 잘 타는법"** 과 같이 제목을 저장한다면

* &#x20;스노
* 노우
* 우보
* 보드
* 드잘
* 잘타
* 타는
* 는법

위와같은 ngram word 인덱스가 생성이 된다. (기본적으로 공백은 무시됨)

그럼 해당 게시판은 위와같은 단어 인덱스에 포함 되어있으므로

실제 **'스노' 혹은 '우보'  '드잘'** 와 같은 단어를 검색한다면 해당 게시판이 검색 결과에 포함되어 진다.

#### **N-gram Full-text Index 생성 과정**

1\. 인덱스 대상 문서 n글자로 구성된 서브 그룹

2\. 백엔드 인덱스

3\. 인덱스 대상 서브 그룹

4\. 2-Gram 서브 그룹

5\. 프론트엔드 인덱스 : 생성



2-Gram 인덱싱 기법은 2글자 단위의 최소 키워드에 대한 키를 관리하는 프론트엔드(Front-end) 인덱스와 2글자 이상의 키워드 묶음(n-SubSequence Window)를 관리하는 백엔드(Back-end) 인덱스 2개로 구성된다.

****

1\) 인덱싱 대상 문서를 n-gram 의 n보다 큰 값의 토큰으로 서브 그룹을 나눕니다. 아래 그림에서는 4크기의 토큰으로 정하고 서브그룹을 구성.

<figure><img src="https://t1.daumcdn.net/cfile/tistory/270D0C50591EDE6D17" alt=""><figcaption></figcaption></figure>

\- 서브그룹 생성 방식은 해당 문서 데이터의 왼쪽을 시작으로 4글자 토큰을 만들고 토큰의 마지막 글자부터 다시 4글자 토큰을 만들어서 데이터의 끝까지 생성.

\- 각 문서에 대한 4글자 토큰 서브그룹을 생성.



2\) 위 1) 단계에서 추출된 서브그룹 중에서 중복되는 서브그룹별로 토큰 분류하고, 백엔드 인덱스를 만든다.

<figure><img src="https://t1.daumcdn.net/cfile/tistory/21696850591EEA2A08" alt=""><figcaption></figcaption></figure>

\- 중복되지 않는 4글자로 구성된 서브그룹 리스트를 만듭니다. 그래서 백엔드 인덱스를 만드는데에 사용.

\- 백엔드 인덱스에 중복되지 않는 4글자로 구성된 서브 그룹을 활용해서 백엔드 인덱스에 문서위치와 같이 생성.

\- 문서위치는 해당 서브그룹이 1)에서 위치하는 곳에 (문서번호,문서열번호)를 넣는니다.\




3\) 백엔드 인덱스에서 중복되지 않은 서브그룹에서 다시한번 n-gram용 서브그룹을 만든다.

<figure><img src="https://t1.daumcdn.net/cfile/tistory/23555F3F591F008513" alt=""><figcaption></figcaption></figure>

\- 각 서브그룹을 다시 한번 2-gram 방식으로 서브그룹을 2글자 토큰으로 나눈다.

\- n-gram 토크나이징 방식은 왼쪽부터 1글자씩 오른쪽으로 이동하면서 연속된 n글자로 토큰을 구성.

\- ex) 가나다라마 일때, n=2    =>    가나,나다,다라,라마

&#x20;   n=3    =>    가나다,나다라,다라마

&#x20;   n=4    =>    가나다라,나다라마

\


4\) 3)과정에서 생성된 2-Gram 토큰을 분류하고, 중복되지 않은 2-Gram 서브 그룹을 생성.

![](https://t1.daumcdn.net/cfile/tistory/2172683F591EFB9F10)

\


5\) 3), 4) 과정에서 n-gram 방식으로 토크나이징한 서브그룹 정보를 이용해서 프론트엔드 인덱스를 생성.

![](https://t1.daumcdn.net/cfile/tistory/2265453B591EFE1602)

\- 프론트엔드 인덱스에서 서브 그룹과 그룹 위치로 구성.

\- 그룹 위치는 3) 과정 인덱스 대상 서브그룹에서 (서브그룹번호,서브그룹열번호) 로 구성.

### **Stop-word parser와 N-gram parser의 비교**

#### 검색 속도

\- 검색어 바이트 수가 많아질수록, 쿼리 실행 시간이 Stop-word parser > **N-gram parser.**&#x20;

즉, N-gram 검색이 빠름.

#### 인덱스 용량

\- 저장 데이터 바이트 수가 많아질수록, 인덱스 스토리지 사용량은 Stop-word parser < **N-gram parser**.&#x20;

즉, N-gram이 더 많은 용량을 사용.



#### **검색 모드의 종류**

1. 자연어 검색(natural search)

\- 검색 문자열을 단어 단위로 분리한 후, 해당 단어 중 하나라도 포함되는 행을 찾는다.

ex) SELECT \* FROM “테이블명" WHERE MATCH (“검색컬럼명") AGAINST (‘맛집' IN NATURAL LANGUAGE MODE);\


&#x20; 2\. 불린 모드 검색(boolean mode search)

\- 검색 문자열을 단어 단위로 분리한 후, 해당 단어가 포함되는 행을 찾는 규칙을 추가적으로 적용하여 해당 규칙에 매칭되는 행을 찾는다.- 검색의 정확도에 따라 결과가 정렬되지 않는다.- 구문 검색이 가능하다.- 필수(+), 예외(-), 부분(\*), 구문(“ ") 연산자를 사용할 수 있다ex) SELECT \* FROM “테이블명" WHERE MATCH (“검색컬럼명") AGAINST (‘+대구\*+닭\*+맛집\*' IN BOOLEAN MODE);

ref: [https://m.blog.naver.com/jjdo1994/222348191751](https://m.blog.naver.com/jjdo1994/222348191751)

\
&#x20; 3\. 쿼리 확장 검색(query extension search)

\- 2단계에 걸쳐서 검색을 수행한다. 첫 단계에서는 자연어 검색을 수행한 후, 첫 번째 검색의 결과에 매칭된 행을 기반으로 검색 문자열을 재구성하여 두 번째 검색을 수행한다. 이는 1단계 검색에서 사용한 단어와 연관성이 있는 단어가 1단계 검색에 매칭된 결과에 나타난다는 가정을 전제로 한다.



[https://interconnection.tistory.com/95](https://interconnection.tistory.com/95)

[https://m.blog.naver.com/jjdo1994/222348191751](https://m.blog.naver.com/jjdo1994/222348191751)

[https://velog.io/@jduckling\_1024/MySQL-FullText-Search](https://velog.io/@jduckling\_1024/MySQL-FullText-Search)
