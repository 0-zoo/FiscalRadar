# 📊 재무 건전성 분석 시스템

**MySQL 파티셔닝 기반의 대용량 재무 데이터 분석 프로젝트**

---

## 🧾 프로젝트 개요
본 프로젝트는 금융감독원에서 제공하는 **대규모 재무제표 데이터**를 기반으로, **기업의 재무 건전성**을 다각도로 분석하는 데이터 엔지니어링 <br>프로젝트입니다.

**MySQL의 파티셔닝 기능**을 활용해 **연도별·기업별** 데이터 처리 성능을 극대화하고, SQL 기반의 지표 계산을 통해 **위험 기업 탐지, 산업군 비교 분석, <br>성장성 평가** 등 다양한 정량적 인사이트를 도출합니다.

---

## 👤 팀원 소개
<table>
  <tbody>
    <tr>
      <td align="center"><a href="https://github.com/moonstone0514"><img src="https://github.com/moonstone0514.png" width="100px;" alt=""/><br /><sub><b>@moonstone0514</b></sub></a><br /></td>
      <td align="center"><a href="https://github.com/yeomyeoung"><img src="https://github.com/yeomyeoung.png" width="100px;" alt=""/><br /><sub><b>@yeomyeoung</b></sub></a><br /></td>
      <td align="center"><a href="https://github.com/0-zoo"><img src="https://github.com/0-zoo.png" width="100px;" alt=""/><br /><sub><b>@0-zoo</b></sub></a><br /></td>
      <td align="center"><a href="https://github.com/Jsumin07"><img src="https://github.com/Jsumin07.png" width="100px;" alt=""/><br /><sub><b>@Jsumin07</b></sub></a><br /></td>

  </tbody>
</table>

<br>

본 프로젝트에 대한 자세한 정리는 Notion에서 확인할 수 있습니다.  
👉 [📘 노션 문서 바로가기](https://www.notion.so/FISA-2-22a345bedb798036b02ef2342873da8a?source=copy_link)  

---

## 🎯 프로젝트 목표

- 📥 금융감독원의 **재무제표 CSV 데이터** 정제 및 DB 적재
- 🗃 **MySQL 파티셔닝**을 통한 대용량 데이터 효율적 관리
- 📈 **재무 지표 분석**: 안정성, 수익성, 성장성 지표 계산
- 🔍 **이상치 탐지 및 위험 기업 자동 분류**
- 🏷 업종·기업 간 **정량적 비교 분석**
- 📊 Kibana 시각화를 통한 **데이터 인사이트 시각적 전달**

---

## 📦 데이터셋 출처

* 금융감독원 재무제표 데이터셋 (https://www.data.go.kr/data/15034611/fileData.do)

* 주요 항목: 금융감독원_재무정보조회_단일회사 재무제표 정보

* 주요 컬럼: 부채비율, 영업이익, ROE, ROA 등


## 🗃 financial 테이블 스키마

| 컬럼명 | 타입 | 설명 |
|--------|------|------|
| 재무제표종류 | VARCHAR(100) | 연결재무제표 등 |
| 종목코드 | VARCHAR(20) | 기업 고유 코드 |
| 회사명 | VARCHAR(100) | 기업명 |
| 시장구분 | VARCHAR(100) | 코스피/코스닥 |
| 업종 | INT | 업종 코드 |
| 업종명 | VARCHAR(100) | 업종 이름 |
| 결산월 | INT | 결산 월 |
| 결산기준일 | DATE | 기준 일자 |
| 보고서종류 | VARCHAR(100) | 사업/반기보고서 등 |
| 통화 | VARCHAR(10) | 예: KRW |
| 항목코드 | TEXT | 재무 항목 코드 |
| 항목명 | VARCHAR(200) | 재무 항목명 |
| 인덱스 | INT | 항목 정렬용 인덱스 |
| 금액 | VARCHAR(50) | 금액 (계산용 정수값) |


## 🧹 데이터 전처리

| 단계 | 주요 작업 | 세부 설명 |
|------|-----------|-----------|
| 1. 테이블 구조 생성 | `financial` 테이블 생성 | CSV 스키마 기반 테이블 구조 정의 |
| 2. CSV 데이터 적재 | `LOAD DATA INFILE` 사용 | utf8mb4 인코딩, 따옴표 처리, 날짜 형식 변환 |
| 3. 컬럼 정리 | 불필요 컬럼 제거 | 분석에 불필요한 컬럼 DROP (업종, 결산월, 통화, 항목명) |

    
## 🧩 파티셔닝 전략

**MySQL 8.0**의 `RANGE + HASH` 파티셔닝 구조를 적용하여 **연도별 분산 처리 + 기업 단위 서브 파티션**을 구현하였습니다.

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
SUBPARTITION BY HASH (CRC32(회사명))
SUBPARTITIONS 3 (
    PARTITION p2022 VALUES LESS THAN (2022),
    PARTITION p2023 VALUES LESS THAN (2023),
    PARTITION p2024 VALUES LESS THAN (2024),
    PARTITION pmax VALUES LESS THAN MAXVALUE
);
🔍 주의: CAST(회사명 AS UNSIGNED)는 오류 가능. 대신 CRC32() 활용.
```
✔ 성능 최적화를 위해 연도 기반 Range 파티셔닝

✔ 서브 파티션에서 기업명 해시를 사용해 병렬성 강화

<img width="760" height="200" alt="image" src="https://github.com/user-attachments/assets/93792f1d-012a-48e9-87e8-861c96ccde24" />

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
## 🔎 재무 지표 분석 로직

### 주요 지표

| 지표 유형 | 지표명 | 항목명 조합 (항목명 값 기준) |
| --- | --- | --- |
| 안정성 | 부채비율 | 총부채 / 자본총계 |
|  | 이자보상배율 | 영업이익 / 이자비용 |
| 수익성 | ROE | 당기순이익 / 자본총계 |
|  | ROA | 당기순이익 / 총자산 |
| 성장성 | 영업이익 증가율 | (당기 - 전기) / 전기 |
|  | 순이익 증가율 | 당기순이익(당기) - 전기 / 전기 |

---

## 📉 사례 기반 리스크 탐지

- **미래에셋증권**: 부채비율 급등 (800% → 1100%)
  - https://v.daum.net/v/20240522191424944
- **우리금융지주**: ROE 하락, 순이익 -25% 감소
    - https://www.ibtomato.com/mobile/mView.aspx?no=14916#:~:text=,%EB%8B%A4%EA%B0%81%ED%99%94%EC%99%80%20%EC%9E%90%EA%B8%88%20%EC%A1%B0%EB%8B%AC%EC%9D%B4%20%EC%A0%88%EC%8B%A4%ED%95%B4%EC%A1%8C%EB%8B%A4%EB%8A%94%20%ED%8F%89%EA%B0%80%EB%8B%A4
- **NH농협손해보험**: 지급여력비율 급락 (316% → 175%)
    - https://kpinews.kr/newsView/1065600605746042

---

## ✅ 주요 기능 구현 요약 및 핵심 SQL 분석 결과

### 📊 재무 지표 분석 SQL

- **부채비율 계산**  
  `총부채 / 자본총계 * 100`  
  (연도별 · 기업별)
  
  ```sql
  WITH debt AS (
    SELECT 회사명, 종목코드, 결산기준일, 금액 AS 총부채
    FROM finance_data_partitioned
    WHERE 항목코드 LIKE '%Liabilit%'
  ),
  equity AS (
    SELECT 회사명, 종목코드, 결산기준일, 금액 AS 자본총계
    FROM finance_data_partitioned
    WHERE 항목코드 LIKE '%Equity%'
  )
  SELECT d.회사명, d.종목코드, d.결산기준일,
         ROUND(d.총부채 / NULLIF(e.자본총계, 0) * 100, 2) AS 부채비율
  FROM debt d
  JOIN equity e
    ON d.회사명 = e.회사명 AND d.결산기준일 = e.결산기준일;
  ```

<br>

- **자본잠식 기업 탐지**  
  `자본총계 <= 0` 조건

  ```sql
  SELECT 회사명, 종목코드, 결산기준일, SUM(금액) AS 자본총계
  FROM finance_data_partitioned
  WHERE 항목코드 LIKE '%Equity%'
    AND YEAR(결산기준일) = 2023
  GROUP BY 회사명, 종목코드, 결산기준일
  HAVING 자본총계 <= 0;
  ```
<br>

- **ROE (자기자본이익률)**  
  `당기순이익 / 자본총계 * 100` (2022~2024)

  ```sql
  WITH profit AS (
    SELECT 회사명, 종목코드, 결산기준일, SUM(금액) AS 당기순이익
    FROM finance_data_partitioned
    WHERE 항목코드 LIKE '%ProfitLoss%' AND YEAR(결산기준일) = 2022
    GROUP BY 회사명, 종목코드, 결산기준일
  ),
  equity AS (
    SELECT 회사명, 종목코드, 결산기준일, SUM(금액) AS 자본총계
    FROM finance_data_partitioned
    WHERE 항목코드 LIKE '%Equity%' AND YEAR(결산기준일) = 2022
    GROUP BY 회사명, 종목코드, 결산기준일
  )
  SELECT p.회사명, p.종목코드, p.결산기준일,
         ROUND(p.당기순이익 / NULLIF(e.자본총계, 0) * 100, 2) AS ROE
  FROM profit p
  JOIN equity e
    ON p.회사명 = e.회사명 AND p.결산기준일 = e.결산기준일
  WHERE e.자본총계 > 0;
  ```

  <br>

- **영업이익 증가율 분석**  
  전년도 대비 증감률 계산

  ```sql
  WITH prev AS (
    SELECT 회사명, 종목코드, SUM(금액) AS 영업이익_전기
    FROM finance_data_partitioned
    WHERE 항목코드 LIKE '%ifrs%' AND 항목코드 LIKE '%OperatingIncome%' AND 결산기준일 = '2022-12-31'
    GROUP BY 회사명, 종목코드
  ),
  curr AS (
    SELECT 회사명, 종목코드, SUM(금액) AS 영업이익_당기
    FROM finance_data_partitioned
    WHERE 항목코드 LIKE '%ifrs%' AND 항목코드 LIKE '%OperatingIncome%' AND 결산기준일 = '2023-12-31'
    GROUP BY 회사명, 종목코드
  )
  SELECT 
    c.회사명,
    c.종목코드,
    c.영업이익_당기,
    p.영업이익_전기,
    ROUND((c.영업이익_당기 - p.영업이익_전기) / NULLIF(p.영업이익_전기, 0) * 100, 2) AS 증가율
  FROM curr c
  JOIN prev p 
    ON c.회사명 = p.회사명 AND c.종목코드 = p.종목코드
  WHERE 
    p.영업이익_전기 IS NOT NULL
    AND c.영업이익_당기 IS NOT NULL
    AND p.영업이익_전기 != 0
    AND ROUND((c.영업이익_당기 - p.영업이익_전기) / NULLIF(p.영업이익_전기, 0) * 100, 2) BETWEEN -200 AND 500;
  ```

<br>

- **위험 기업 수 집계**  
  `부채비율 > 200%` 또는 `자본총계 ≤ 0` 조건 포함
  
  ```sql
  WITH debt AS (
    SELECT 회사명, 종목코드, 결산기준일, SUM(금액) AS 총부채
    FROM finance_data_partitioned
    WHERE 항목코드 LIKE '%Liabilit%'
    GROUP BY 회사명, 종목코드, 결산기준일
  ),
  equity AS (
    SELECT 회사명, 종목코드, 결산기준일, SUM(금액) AS 자본총계
    FROM finance_data_partitioned
    WHERE 항목코드 LIKE '%Equity%'
    GROUP BY 회사명, 종목코드, 결산기준일
  ),
  joined AS (
    SELECT d.회사명, d.종목코드, d.결산기준일,
           d.총부채, e.자본총계,
           ROUND(d.총부채 / NULLIF(e.자본총계, 0) * 100, 2) AS 부채비율
    FROM debt d
    JOIN equity e ON d.회사명 = e.회사명 AND d.결산기준일 = e.결산기준일
  )
  SELECT 
    YEAR(결산기준일) AS 연도,
    COUNT(*) AS 위험기업수
  FROM joined
  WHERE 
    (부채비율 > 200 AND 자본총계 > 0)  -- 레버리지 과다
    OR 자본총계 <= 0                   -- 자본잠식
  GROUP BY YEAR(결산기준일)
  ORDER BY 연도;
  ```

---

### 🔍 주요 분석 결과

- **부채비율 > 200% 기업**  
  유안타증권, DB금융투자, 한국투자증권 등 다수 존재

- **자본잠식 기업 (2023년 기준)**  
  BNK금융지주, TS인베스트먼트, 삼성카드, 현대해상 등 약 28개사

- **ROE 최고치 기업 (2023)**  
  한국스탠다드차타드은행, 신한라이프생명보험, NH투자증권 등

- **연도별 위험 기업 수**
  - 2022년: `14개사`
  - 2023년: `14개사`
  - 2024년: `16개사`
 
    <img width="1492" height="778" alt="image (2)" src="https://github.com/user-attachments/assets/b57cd900-c572-4a25-8b03-c264bad262fc" />


---

## 🚀 향후 확장 가능성
📅 1. 데이터 연도 확장 및 자동 업데이트

2024년 데이터에 국한된 현재 범위를 2020~2023년까지 확장하여 시계열 분석을 강화하고,
DART API를 활용한 자동 수집·갱신 시스템을 구축하여 매년 최신 데이터를 반영할 수 있도록 합니다.
(현재 제공 데이터 수: 539,578건)

🔍 2. Elasticsearch 연동 및 검색 성능 개선

- MySQL 분석 결과를 Elasticsearch에 인덱싱하여,
기업명 + 조건 검색 기반의 고속 필터링 및 Kibana/OpenSearch 시각화 기능 지원 가능

  예: 현대해상 AND 부채비율 > 300% AND 자본잠식 = True

📈 3. Tableau 연동 및 시각화 고도화

- Kibana를 넘어 Tableau 또는 Power BI를 연동하여
산업별 매출·ROE 비교, 위험도 Heatmap 등 인터랙티브 시각화 대시보드로 확장

🧠 4. 머신러닝 기반 위험 예측

- ROE·부채비율 등 지표를 기반으로 기업 위험도 분류 모델(XGBoost 등) 구축,
이상치 탐지(Isolation Forest)와 함께 위험·주의·건전 기업으로 자동 분류 가능

🧩 5. 업종별 지표 커스터마이징

- 업종별 중요 지표(예: 보험업 = 지급여력비율, 은행 = BIS 비율 등)를 반영해
맞춤형 분석 지표 체계를 구축하고, 업종 간 상대적 평가 기능 제공

---

## 마무리 및 고찰

이번 프로젝트는 단순한 재무 데이터 분석을 넘어, **실무 환경에서 활용 가능한 데이터 처리 파이프라인과 재무 건전성 평가 시스템**을 직접 설계하고 구현한 의미 있는 경험이었습니다.

MySQL의 파티셔닝 기능을 활용하여 연도별·기업별로 데이터를 구조화하고, SQL 기반의 재무 지표 분석 로직을 통해 위험 기업 탐지, 산업군 비교, 수익성/성장성 평가 등 다양한 인사이트를 도출하였습니다.

하지만 본 프로젝트는 **재무제표 데이터에 대한 구조 이해와 분석 프로세스 구축 자체에 초점을 맞췄기 때문에**, 실제 데이터의 양은 상대적으로 제한된 상태였다. 이에 따라, 파티셔닝이 **쿼리 성능 향상에 미치는 실질적 차이**를 통계적으로 입증하기에는 어려움이 있었습니다.

### 향후 보완 방향

- `2020~2024년`까지의 전체 데이터를 통합하여 **진정한 대용량 환경 구성**
- DART API를 활용한 **데이터 자동 수집 및 갱신 시스템 구축**
- 파티셔닝 유무에 따른 쿼리 성능 실험 및 **벤치마크 분석 추가**
- Elasticsearch 또는 ClickHouse 등 **고속 분석 DB 연계 검토**

이를 통해 **정량적 재무 분석 시스템의 실용성과 확장성**을 더욱 강화하고, 향후에는 머신러닝 기반의 위험 예측 시스템이나 실시간 모니터링 대시보드까지 연계 가능한 통합 플랫폼으로 발전시킬 수 있을 것입니다.

---


## 🧯 트러블슈팅

<details>
<summary>🔧 1. 외부 테이블을 이용하기 위한 파일 위치 선정</summary>

### 📝 Oracle에서 외부 테이블을 통해 CSV를 로드하려 했지만, 로그 파일 경로 문제로 실행 실패

```sql
SQL Error [29913] [99999]: ORA-29913: error in executing ODCIEXTTABLEOPEN callout
ORA-29400: data cartridge error
error opening file /ce5/02.sql/csv/TEST_EXTERNAL_1887.log
```

### 📄 상세정보  
- 외부 테이블 정의 자체는 문제 없었으나, 로그 파일 생성 시 OS 권한 문제로 실패  
- Oracle 프로세스는 /home 같은 상위 경로에 접근 권한이 없음

### ⚠️ 영향  
- 외부 테이블 기반의 CSV 조회 실패  
- 데이터 적재 자동화 단계에서 중단됨

### 🛠 처리한 단계  
1. Oracle에서 DIRECTORY 객체 생성: `CREATE DIRECTORY total_dir AS '/home/total';`  
2. 외부 테이블 생성 시 `DEFAULT DIRECTORY total_dir` 명시  
3. 해당 디렉토리에 파일 이동 및 권한 부여 (`chmod`, `chown`)

### ✅ 해결 방법  
- 로그 및 데이터 파일은 반드시 Oracle이 직접 접근 가능한 디렉토리 내에 위치해야 함 (`/home/total` 등)

</details>

<br>

<details>
<summary>🔧 2. Oracle SQL 테이블 컬럼명이 한글로 깨져서 출력됨</summary>

### 📝 `desc` 명령어 시 컬럼명이 `???`로 출력되는 문자 인코딩 문제 발생

```sql
SQL> desc total;
 ??????    VARCHAR2(100)
 ???       VARCHAR2(100)
```

### 📄 상세정보  
- DB 자체 문자셋은 `AL32UTF8`로 설정되어 문제 없음  
- 클라이언트 환경변수 `NLS_LANG`이 미설정 → 한글 출력 불가

### ⚠️ 영향  
- SQL*Plus 또는 CLI 환경에서 컬럼 구조 파악이 어려움  
- 쿼리 작성 시 오타 발생 가능성 증가

### 🛠 처리한 단계  

```bash
export NLS_LANG=KOREAN_KOREA.AL32UTF8
```

### ✅ 해결 방법  
- 환경변수 적용 후 SQL*Plus 재접속 → 컬럼명 정상 출력 확인됨

</details>

<br>

<details>
<summary>🔧 3. Oracle에서 파티셔닝 안됨</summary>

### 📝 Oracle XE는 파티셔닝 기능 미지원

### 📄 상세정보  
- Oracle XE 11g는 RANGE, HASH 등 파티셔닝 기능을 공식적으로 제공하지 않음

### ⚠️ 영향  
- 대용량 테이블 쿼리 성능 저하  
- 분석 속도 저하

### 🛠 처리한 단계  
- DBMS를 MySQL로 전환하고 RANGE + HASH 파티셔닝 구조로 재설계

### ✅ 해결 방법  
- MySQL 8.0 이상에서는 파티셔닝 기능 기본 제공 → 안정적으로 대체 완료

</details>

<br>

<details>
<summary>🔧 4. 문자열 길이가 컬럼 정의보다 길기 때문에 발생하는 오류</summary>

### 📝 VARCHAR 길이보다 긴 문자열 입력 시 INSERT 실패

```sql
SQL Error [1406] [22001]: Data too long for column '항목코드' at row 26
```

### 📄 상세정보  
- 항목코드 VARCHAR(200) → 일부 데이터는 220자 이상

### ⚠️ 영향  
- 전체 데이터 적재 실패 또는 누락  
- 자동화 과정 중 중단

### 🛠 처리한 단계  

```sql
ALTER TABLE financial
MODIFY COLUMN 항목코드 VARCHAR(500);
```

### ✅ 해결 방법  
- VARCHAR 길이 상향 조정 → 모든 데이터 정상 입력 확인

</details>

<br>

<details>
<summary>🔧 5. RANGE 파티셔닝은 사용할 수 있는 컬럼 타입에 제한</summary>

### 📝 VARCHAR 컬럼에 RANGE 파티셔닝 시 오류 발생

### 📄 상세정보  
- MySQL RANGE 파티셔닝은 INT, DATE 등 제한된 타입만 허용  
- 종목코드는 문자열 형태(VARCHAR)로 저장되어 사용 불가

### ⚠️ 영향  
- 종목코드를 기준으로 파티셔닝 불가  
- 성능 개선 실패

### 🛠 처리한 단계  
- 종목코드를 `CAST(종목코드 AS UNSIGNED)` 또는 DATE 기반 파티셔닝으로 변경

### ✅ 다음 단계  
- VARCHAR 파티셔닝 필요 시 HASH 또는 LIST 방식 고려

</details>

<br>

<details>
<summary>🔧 6. Oracle - Tableau 연결 실패</summary>

### 📝 Docker 기반 Oracle에 Tableau 연결 실패 (`ORA-12541`)

### 📄 상세정보  
- Oracle XE Docker 이미지 사용  
- NAT + 포트포워딩(1521) 구성  
- 리스너 정상 작동 확인 후에도 Tableau 연결 불가

### ⚠️ 영향  
- Tableau 시각화 불가 → Kibana로 대체 진행

### 🛠 처리한 단계  
- 리스너 상태 확인 (`lsnrctl status`)  
- Windows 방화벽 및 VirtualBox 포트포워딩 점검  
- 다양한 서비스 이름/계정 시도

### ✅ 다음 단계  
- Oracle을 Windows 로컬에 직접 설치하거나 브리지 네트워크 방식 고려

</details>

<br>
<details>
<summary>🔧 7. MySQL 내에서 [] 처리 안 되는 문제</summary>

### 📝 `[138930]` 형태의 종목코드 → 쿼리 인식 실패

### 📄 상세정보  
- Pandas에서 CSV 처리 시 `종목코드`가 리스트처럼 저장됨  
- `[` 및 `]` 문자로 인해 문자열 비교 실패

### ⚠️ 영향  
- WHERE 조건식 작동 안 함  
- 그룹화, 정렬 등 전처리 오류 발생

### 🛠 처리한 단계  

```python
df['종목코드'] = df['종목코드'].apply(lambda x: str(x).replace('[', '').replace(']', '').strip())
```

### ✅ 해결 방법  
- 전처리 후 재업로드 → SQL 쿼리 정상 작동

</details>

<br>

<details>
<summary>🔧 8. 항목코드 해석 문제</summary>

### 📝 항목명이 없이 `항목코드`만 존재 → 지표 계산 시 가독성 저하

### 📄 상세정보  
- 항목코드 예: `ifrs-full_ProfitLoss`, `dart_Liabilities`  
- 사람이 이해할 수 있는 "총부채", "자본총계" 등의 이름 없음

### ⚠️ 영향  
- 필터링 조건 불명확  
- 지표별 SQL 자동화 로직 구축 어려움

### 🛠 처리한 단계  
1. DART 기준 항목코드 목록 확보  
2. 지표별 키워드 기반 매핑 정의 (e.g., `ProfitLoss`, `Liabilit`)  
3. LIKE 검색 기반 항목 자동 추출

### ✅ 다음 단계  
- 항목코드 ↔ 항목명 매핑 테이블 구축  
- 업종별 커스터마이징 및 유지보수 자동화 고려

</details>

<br>

<details>
<summary>🔧 9. 부채비율 계산 쿼리 결과 이상 현상 분석</summary>

### 📝 실행한 쿼리 목적 요약  
부채비율 = 총부채 / 자본총계 × 100 을 계산하기 위해,  
같은 회사명 + 같은 결산기준일 조건으로 `JOIN`하여 계산

```sql
-- 총부채 / 자본총계 * 100 → 부채비율 계산
-- 같은 회사명 + 같은 결산기준일 조건으로 JOIN
WITH debt AS (
  SELECT 회사명, 종목코드, 결산기준일, SUM(금액) AS 총부채
  FROM finance_data_partitioned
  WHERE 항목코드 LIKE '%Liabilit%'
  GROUP BY 회사명, 종목코드, 결산기준일
),
equity AS (
  SELECT 회사명, 종목코드, 결산기준일, SUM(금액) AS 자본총계
  FROM finance_data_partitioned
  WHERE 항목코드 LIKE '%Equity%'
  GROUP BY 회사명, 종목코드, 결산기준일
)
SELECT d.회사명, d.종목코드, d.결산기준일,
       ROUND(d.총부채 / NULLIF(e.자본총계, 0) * 100, 2) AS 부채비율
FROM debt d
JOIN equity e
  ON d.회사명 = e.회사명 AND d.결산기준일 = e.결산기준일;
```

### 🔎 결과 해석 및 이상 징후 분석  

#### ✅ 1. 중복 결과 발생  
같은 회사명, 종목코드, 결산기준일이 여러 건 출력됨  
예: `BNK금융지주 / 138930 / 2022-12-31`이 3건 중복

➡️ **원인**: `LIKE '%Liabilit%'`, `%Equity%` 조건에  
유동부채, 비유동부채, 총부채 등 **여러 항목이 동시에 매칭되어 곱집합(M:N JOIN)** 발생

#### ✅ 2. 음수 또는 0 부채비율  
예: -7800.0, -135.71, 0.0, -7333.33, NaN 등

➡️ **원인**:
- 자본총계가 0 또는 음수 → 회계상 부실 기업
- 총부채가 음수 → 데이터 수집 또는 매핑 오류 가능

#### ✅ 3. 결측값 (NULL 또는 빈값)  
예: 미래에셋캐피탈, NH투자증권 등 일부 기업의 부채비율이 없음

➡️ **원인**:
- 해당 항목코드를 만족하는 데이터가 **하나라도 누락**되면 JOIN 불가
- 일부 기업은 총부채만 있고 자본총계 없음 (또는 그 반대)

### 🧠 개선 방향  

#### ✅ A. 항목코드 필터링 개선  
지금은 너무 광범위한 조건 (`LIKE '%Liabilit%'`) 사용 중  
→ 정확한 코드 지정으로 좁히기

```sql
WHERE 항목코드 IN ('ifrs-full_Liabilities', 'dart_Liabilities', ...)
```

➡️ **항목명이나 코드 정규 리스트**를 기반으로 필터링하면 중복 방지 및 신뢰도 향상

</details>

<br>

<details>
<summary>🔧 10. 부채비율 계산 결과 음수 발생 원인 분석</summary>

### 📝 부채비율 계산 시 일부 기업에서 음수값이 발생함

```sql
예시 결과:
BNK금융지주: -7800.0  
DB금융투자: -135.71  
이베스트투자증권: -7333.33  
미래에셋캐피탈: NULL  
```

### 📄 상세정보  
- 계산식: `총부채 / 자본총계 * 100`  
- 정상적인 경우 0~수백 % 수준이지만, 일부 기업에서 음수 또는 NaN 발생

### ⚠️ 영향  
- 회계적 해석 또는 시각화 시 부정확한 정보 제공  
- 위험 기업과 데이터 오류가 구분되지 않음

### 🛠 주요 원인 및 대응 방안

| 구분 | 원인 설명 | 해결 방법 |
|------|-----------|------------|
| ① | 자본총계가 음수 | 기업의 누적 적자가 심해 자본총계 < 0 (자본잠식 상태) | `자본총계 < 0` 기업은 별도 플래깅하거나 위험기업으로 분류 |
| ② | 총부채가 음수 | 항목 매핑 오류 or 잘못된 입력 | `총부채 < 0`인 경우, 원시데이터에서 항목코드와 금액 재검토 |
| ③ | 항목코드 필터링 문제 | `LIKE '%Liabilit%'` 조건이 너무 광범위하여 음수 항목 포함 | 신뢰할 수 있는 항목명 or 코드만 사용 (예: `총부채` 전용 코드) |
| ④ | 중복 계산 | M:N JOIN으로 인해 곱집합 발생 → 음수 누적 | `SUM + GROUP BY`를 선처리하여 해결 (현재 적용 중) |

### ✅ 개선된 쿼리 예시  
- 부채비율이 0 이상이고, 자본총계도 양수인 경우만 필터링

```sql
WITH debt AS (
  SELECT 회사명, 종목코드, 결산기준일, SUM(금액) AS 총부채
  FROM finance_data_partitioned
  WHERE 항목코드 LIKE '%Liabilit%'
  GROUP BY 회사명, 종목코드, 결산기준일
),
equity AS (
  SELECT 회사명, 종목코드, 결산기준일, SUM(금액) AS 자본총계
  FROM finance_data_partitioned
  WHERE 항목코드 LIKE '%Equity%'
  GROUP BY 회사명, 종목코드, 결산기준일
)
SELECT d.회사명, d.종목코드, d.결산기준일,
       ROUND(d.총부채 / e.자본총계 * 100, 2) AS 부채비율
FROM debt d
JOIN equity e
  ON d.회사명 = e.회사명 AND d.결산기준일 = e.결산기준일
WHERE d.총부채 >= 0 AND e.자본총계 > 0;
```

</details>

<br>
<details>
<summary>🔧 11. 종목코드 NULL 기입 </summary>

### 📝 종목코드에 NULL 기입

### 📄 상세정보  
- 종목코드에 NULL값이 기입된 상태라, 사용이 난감했다.

### ⚠️ 영향  
- 종목코드에 NULL값이 존재하는 한, 기준으로 사용하기 어려움.

### 🛠 처리한 단계  
종목코드를 INT -> VARCHAR(20)로 변경 후 사용.

### ✅ 해결 방법  
- 사용에는 이상이 없으나, 개발 과정 중 자료 분석을 위한 조사를 하다가 비상장회사는 종목코드가 존재하지 않을 수도 있다는 것을 알게 되었다.
- 중복코드를 다시 원복시키지는 않고 진행.
  

</details>
