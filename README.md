# 📊 재무 건전성 분석 시스템 (MySQL 파티셔닝 기반)

## 🧾 개요

본 프로젝트는 대용량 재무제표 데이터를 기반으로 **기업의 재무 건전성**을 분석하는 **데이터 엔지니어링 프로젝트**입니다.  
MySQL의 파티셔닝 기능을 적극 활용하여 **연도별·기업별 재무 분석의 성능을 극대화**하고, SQL 기반 지표 계산을 통해 다양한 **재무 위험 신호 및 성장 패턴을 정량적으로 추출**합니다.

---

## 🎯 프로젝트 목표

- 📥 대용량 CSV 재무 데이터를 MySQL RDB에 정제/적재
- 🔀 MySQL 파티셔닝을 통한 연도·기업 기준 데이터 분산 처리
- 📈 대표 재무 지표 산출: **안정성, 수익성, 성장성**
- 🔎 이상치 탐지 및 **위험 기업 식별**
- 📊 산업군·기업 간 재무 지표 비교 분석

---

## 🗂 데이터 구성

| 컬럼명 | 예시 | 설명 |
|--------|------|------|
| `재무제표종류` | 포괄손익계산서, 기타 - 연결 | 재무제표 유형 및 연결 여부 |
| `종목코드` | [138930] | 기업 고유 코드 (전처리 시 괄호 제거 및 정수형 변환 필요) |
| `회사명` | BNK금융지주 | 기업 이름 |
| `시장구분` | 유가증권시장상장법인 | 상장 시장 구분 |
| `업종명` | 기타 금융업 | 산업 분류 |
| `결산기준일` | 2024-12-31 | 재무 데이터 기준 일자 |
| `항목코드` | dart_XXXX / ifrs-full_XXXX | 항목 식별 코드 |
| `항목명` | 영업이익, 자본총계 등 | 재무 항목 설명 |
| `금액` | 53,000,000,000 | 금액 |

※ 일부 항목은 같은 의미라도 항목명이 달라질 수 있으므로 **동의어 처리 매핑 테이블** 적용 필요.

---

## 📐 분석 지표 정의

### 🛡 안정성

| 지표명 | 계산식 | 의미 |
|--------|--------|------|
| **부채비율** | 총부채 / 자본총계 × 100 | 낮을수록 안전 |
| **이자보상배율** | 영업이익 / 이자비용 | 1 미만일 경우 이자비용 부담 초과 위험 |

### 💰 수익성

| 지표명 | 계산식 | 의미 |
|--------|--------|------|
| **ROE** | 당기순이익 / 자본총계 × 100 | 주주 자본 대비 수익률 |
| **ROA** | 당기순이익 / 총자산 × 100 | 전체 자산 대비 수익률 |

### 📈 성장성

| 지표명 | 계산식 | 의미 |
|--------|--------|------|
| **영업이익 증가율** | (당기 - 전기) / 전기 × 100 | 본업 수익 성장률 |
| **당기순이익 증가율** | (당기 - 전기) / 전기 × 100 | 최종 수익 성장률 |

---

## 🧩 파티셔닝 전략

대용량 재무 데이터의 **성능 최적화** 및 **쿼리 효율성**을 위해 **다단계 파티셔닝**을 적용합니다:

```sql
CREATE TABLE finance_data_partitioned (
    회사명 VARCHAR(100),
    종목코드 VARCHAR(20),
    항목코드 VARCHAR(200),
    항목명 VARCHAR(150),
    결산기준일 DATE,
    금액 BIGINT
)
PARTITION BY RANGE (YEAR(결산기준일))
SUBPARTITION BY HASH (CAST(회사명 AS UNSIGNED))
SUBPARTITIONS 3 (
    PARTITION p2022 VALUES LESS THAN (2022),
    PARTITION p2023 VALUES LESS THAN (2023),
    PARTITION p2024 VALUES LESS THAN (2024),
    PARTITION pmax VALUES LESS THAN MAXVALUE
);
```
---

## ✅ 파티셔닝 효과
- 📆 연도별 전체 조회 쿼리 최적화

- 🏢 기업별 병렬 처리 분산

- 🧮 지표 계산 시 I/O 최소화 및 응답 시간 단축

🧪 예시 결과 활용
- ✅ ROE 상위 10대 금융사 도출

- ⚠️ 이자보상배율 1 미만 기업 탐지 → 고위험 기업

- 📉 순이익 급감 기업 추적 (우리금융지주 등 실사례 반영)

- 📊 업종별 평균 ROE 비교 (보험업 vs 증권업 등)
---

## 🛠 기술 스택
| 범주  | 도구                              |
| --- | ------------------------------- |
| 언어  | Python 3.11                     |
| 분석  | Pandas, SQLAlchemy              |
| DB  | MySQL 8.0 (파티셔닝 기능)             |
| 시각화 | Kibana (추후 Tableau 연계 가능)       |
| 환경  | Colab, VSCode, DBeaver (쿼리 테스트) |

---

🗞 참고 사례 기반 분석 (실제 기업 적용)
▶ 미래에셋증권
부채비율: 800% → 1100%

이자보상배율: 업계 평균 대비 급락 → 위험 감지

▶ 우리금융지주
순이익 감소: 전년 대비 -25%

ROE 하락, 비용 증가 → 건전성 경고

▶ NH농협손해보험
지급여력비율: 316% → 175%

이자비용 상승, 자본 확충 → 수익성 압박

---

## 🧯 트러블슈팅

<details>
<summary> 1. 외부 테이블을 이용하기 위한 파일 위치 선정</summary>

## 요약

SQL 쿼리구문 오류


🔴 에러 내용 분석

java
복사
편집
SQL Error [29913] [99999]: ORA-29913: error in executing ODCIEXTTABLEOPEN callout  
ORA-29400: data cartridge error  
error opening file /ce5/02.sql/csv/TEST_EXTERNAL_1887.log
상세정보
외부 테이블 정의에는 문제가 없지만, Oracle이 로그 파일을 생성하려고 할 때 실패한 경우

영향
외부테이블을 이용한 .csv파일 사용이 어려움

처리한 단계
Oracle에서 DIRECTORY 객체 만들기 (정확한 경로 지정)

외부 테이블 정의 시 DEFAULT DIRECTORY 및 LOCATION 사용

서버 접속 권한이 있다면 위처럼 파일 확인

해결 방법
Oracle 외부 테이블은 OS의 디렉토리를 참조하기 위해 DIRECTORY 객체를 사용함
→ /home에는 Oracle 프로세스가 직접 접근할 권한이 없으므로, 하위 디렉토리(/home/total)를 따로 만들고 파일 이동

/home/total 디렉터리에 읽기 및 쓰기 권한 부여

total.csv 파일은 /home/total 디렉토리에 있어야 함

</details> <details> <summary> 2. Oracle SQL 테이블 컬럼명이 한글로 깨져서 출력됨</summary>
요약
Oracle DB에서 desc total 명령어를 실행했을 때 컬럼명이 한글로 표시되지 않고 ???로 깨져 출력되는 문제 발생

SQL> desc total;
 Name                                      Null?    Type
 ----------------------------------------- -------- ----------------------------
 ??????                                             VARCHAR2(100)
 ????                                               VARCHAR2(20)
 ???                                                VARCHAR2(100)
 ...
영향
테이블 구조 확인이 불가하여 쿼리 작성이나 스키마 점검에 불편

CLI 가독성 저하

처리한 단계
DB 문자셋 확인 → AL32UTF8

클라이언트 환경변수 설정

bash
복사
편집
export NLS_LANG=KOREAN_KOREA.AL32UTF8
SQL*Plus 재접속 후 desc total 실행 → 한글 컬럼명 정상 출력

</details> <details> <summary> 3. Oracle에서 파티셔닝 안됨</summary>
요약
데이터베이스 파티셔닝 기능을 공식적으로 지원하지 않아서 MySQL로 변경

상세정보
Oracle XE 11g의 제한으로 인해 파티셔닝 기능 부재

처리한 단계
MySQL로 전환하여 파티셔닝 기능 구현

</details> <details> <summary> 4. 문자열 길이가 컬럼 정의보다 길기 때문에 발생하는 오류</summary>
요약
항목코드 컬럼에 입력하려는 문자열이 VARCHAR 정의보다 길어서 발생한 오류

상세정보
java
복사
편집
SQL Error [1406] [22001]: Data truncation: Data too long for column '항목코드' at row 26
영향
지정한 VARCHAR 길이보다 긴 데이터가 들어올 경우 삽입 실패

처리한 단계
sql
복사
편집
ALTER TABLE financial
MODIFY COLUMN 항목코드 VARCHAR(500);
</details> <details> <summary> 5. RANGE 파티셔닝은 사용할 수 있는 컬럼 타입에 제한</summary>
요약
MySQL의 RANGE 파티셔닝은 정수, 날짜형 타입만 가능

상세정보
TEXT, BLOB, FLOAT 등의 타입은 사용 불가

종목코드가 VARCHAR로 되어 있어 RANGE 파티셔닝 실패

처리한 단계
종목코드를 INT로 전처리하거나 DATE 컬럼으로 파티셔닝 적용

</details> <details> <summary> 6. Oracle - Tableau 연결 실패</summary>
요약
Tableau를 통한 Oracle Docker 컨테이너 연결 실패

상세정보
Oracle 11g XE Docker 이미지 사용

VirtualBox + NAT + 포트포워딩 구성

Tableau 연결 시 ORA-12541 발생

영향
Tableau 시각화 불가 → Kibana로 대체

처리한 단계
리스너 상태, 방화벽, 포트포워딩 점검

다양한 연결 방식 반복 시도

다음 단계
Windows에 Oracle 직접 설치 또는 VirtualBox 네트워크 모드 변경

</details> <details> <summary> 7. MySQL 내에서 [] 처리 안 되는 문제</summary>
요약
[138930] 형태의 문자열에서 대괄호 제거 전처리 누락

상세정보
Pandas → MySQL 업로드 과정에서 대괄호 미처리

WHERE 종목코드 = 138930 비교 실패

처리한 단계
python
복사
편집
df['종목코드'] = df['종목코드'].apply(lambda x: str(x).replace('[', '').replace(']', '').strip())
fillna, to_csv() 후 재업로드

</details> <details> <summary> 8. 항목코드 해석 문제</summary>
요약
지표 계산에 필요한 항목명을 찾기 어려움 → 항목코드만 존재

상세정보
항목코드만 존재 (ifrs-full_, dart_)

사람이 읽을 수 있는 항목명이 없어 분석 어려움

영향
SQL 지표 필터링 정확도 저하

지표 자동화 어려움

처리한 단계
DART 기준 714개 코드 확보

키워드 기반 LIKE 검색 적용

항목코드 → 지표 매핑 자동화

다음 단계
항목코드 ↔ 항목명 매핑 테이블 구축

업종별 특수 항목 반영

추후 ML 분류 모델 고려

</details> ```


