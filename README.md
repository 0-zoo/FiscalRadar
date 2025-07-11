<br>

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

<br>

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

____

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

____

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

____

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

____

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

____

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

____
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

____

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



