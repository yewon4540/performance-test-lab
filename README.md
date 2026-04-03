# performance-test-lab

JMeter와 Locust를 사용해 웹 애플리케이션의 사용자 시나리오 기반 부하 테스트를 실습하고, 결과를 비교·해석하기 위한 랩 저장소입니다.

이 저장소는 단순히 툴 실행법을 나열하는 것이 아니라, 아래 질문에 답할 수 있도록 구성하는 것을 목표로 합니다.

- 어떤 사용자 흐름을 테스트했는가?
- 어떤 방식으로 부하를 증가시켰는가?
- 어떤 지표를 기준으로 병목을 해석했는가?
- JMeter와 Locust는 각각 어떤 장단점이 있었는가?

---

## 1. 프로젝트 개요

이 프로젝트는 수강신청 예제 웹 애플리케이션을 대상으로 로그인 후 강의 목록과 상세 페이지를 탐색하는 흐름을 부하 테스트하는 실습 프로젝트입니다.

테스트 대상 애플리케이션은 두 가지 구현체를 포함합니다.

- Flask 기반 예제 앱: [Course_registration_flask/app.py](Course_registration_flask/app.py)
- Spring Boot 기반 예제 앱: [Course_registration_java/src/main/java/com/example/demo/controller/WebController.java](Course_registration_java/src/main/java/com/example/demo/controller/WebController.java)

테스트 도구는 다음 두 가지입니다.

- JMeter 시나리오: [jmeter/scenario/overload.jmx](jmeter/scenario/overload.jmx)
- Locust 시나리오: [locust/scenario/locustfile.py](locust/scenario/locustfile.py)

---

## 2. 테스트 목적

이 프로젝트의 목적은 다음과 같습니다.

1. 로그인 기반 웹 서비스의 대표 사용자 흐름을 부하 상황에서 재현
2. 동시 사용자 증가에 따른 응답시간, 처리량, 오류율 변화를 관찰
3. JMeter와 Locust의 시나리오 구성 방식 및 결과 산출 방식 비교
4. 단순 실행 결과 나열이 아니라 “무엇을 검증했고 어떻게 해석했는지”를 설명 가능한 형태로 정리

---

## 3. 사용 도구

| 구분 | 도구 | 용도 |
| --- | --- | --- |
| 부하 테스트 | JMeter | GUI 기반 테스트 플랜 작성, 단계적 사용자 증가, 리포트 확인 |
| 부하 테스트 | Locust | Python 코드 기반 사용자 행동 정의, 헤드리스/웹 UI 실행 |
| 대상 애플리케이션 | Flask | 경량 예제 앱으로 빠른 시나리오 검증 |
| 대상 애플리케이션 | Spring Boot | 다른 런타임 환경에서 동일 흐름 테스트 |
| 데이터 | CSV | 테스트 계정 데이터 공급 |
| 결과 후처리 | pandas, matplotlib, seaborn | Locust CSV 결과를 시각화 |

### JMeter에서 사용한 주요 요소

- `CSV Data Set Config`
- `HTTP Request Defaults`
- `Stepping Thread Group` 플러그인 기반 단계적 사용자 증가
- `Weighted Switch Controller` 기반 시나리오 제어

### Locust에서 사용한 주요 요소

- `HttpUser`
- `on_start()` 기반 로그인 처리
- `@task` 기반 사용자 행동 가중치 설정
- 헤드리스 실행 및 CSV 결과 저장

---

## 4. 테스트 시나리오

현재 저장소의 핵심 시나리오는 “수강신청 서비스 탐색 흐름”입니다.

### 사용자 여정

1. 로그인 요청
2. 강의 목록 조회
3. 강의 상세 페이지 조회
4. 로그아웃

### 엔드포인트 기준 시나리오

| 단계 | 메서드 | 경로 | 설명 |
| --- | --- | --- | --- |
| 1 | POST | `/login` | 사용자 인증 |
| 2 | GET | `/course` | 강의 목록 조회 |
| 3 | GET | `/course/1` ~ `/course/4` | 강의 상세 페이지 반복 조회 |
| 4 | GET 또는 POST | `/logout` | 세션 종료 |

### 부하 모델

#### JMeter

- CSV 파일로 사용자 계정 로드
- 단계적으로 사용자를 증가시키는 형태의 스레드 그룹 사용
- 최대 100명 수준까지 점진 증가하도록 설정
- 대표 웹 흐름을 연속 호출하는 구조

#### Locust

- 사용자 시작 시 로그인 수행
- `view_courses`, `view_course_details`, `logout` 작업을 가중치 기반으로 반복
- 헤드리스 스크립트에서는 step-load 옵션으로 사용자 수를 점진 증가

---

## 5. 측정 지표

이 저장소에서 핵심적으로 봐야 하는 지표는 다음과 같습니다.

| 지표 | 의미 | 해석 포인트 |
| --- | --- | --- |
| TPS / Throughput / Requests per second | 초당 처리 요청 수 | 부하가 증가해도 처리량이 선형적으로 증가하는지 확인 |
| Average Response Time | 평균 응답시간 | 전체적인 체감 성능 수준 파악 |
| Median Response Time | 중앙값 응답시간 | 일부 이상치보다 일반적인 사용자 경험 파악에 유리 |
| P90 / P95 / P99 | 상위 백분위 응답시간 | 느린 요청이 얼마나 자주 발생하는지 확인 |
| Failure Rate / Error Count | 오류율, 실패 건수 | 임계 부하 구간 탐지 |
| Concurrent Users | 동시 사용자 수 | 어느 수준부터 성능 저하가 시작되는지 판단 |

특히 아래 세 가지 지표를 함께 보면 해석에 도움이 됩니다.

- 사용자 수 대비 TPS
- 사용자 수 대비 P95 응답시간
- 사용자 수 대비 오류율

---

## 6. 실행 방법

### 6-1. Flask 애플리케이션 실행

```bash
cd Course_registration_flask
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python app.py
```

기본 포트는 `5001`입니다.

### 6-2. Spring Boot 애플리케이션 실행

```bash
cd Course_registration_java
mvn spring-boot:run
```

기본 포트는 `8082`입니다.

### 6-3. JMeter 실행

#### GUI 실행

1. JMeter 설치
2. [jmeter/scenario/overload.jmx](jmeter/scenario/overload.jmx) 로드
3. `base_url`, `port`, CSV 경로 등 변수 값을 환경에 맞게 수정
4. 실행 후 Summary Report 또는 결과 리스너 확인

#### CLI 실행 예시

```bash
cd jmeter
jmeter -n -t scenario/overload.jmx -l result/result.jtl -e -o result/report
```

> 참고: 현재 JMeter 시나리오는 플러그인 의존 요소가 있으므로 `Custom Thread Groups`, `bzm - Weighted Switch Controller` 계열 플러그인이 필요할 수 있습니다.

### 6-4. Locust 실행

```bash
cd locust
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

#### 헤드리스 실행

```bash
cd locust
locust -f scenario/locustfile.py \
  --headless \
  --host=http://localhost:5001 \
  -u 20 -r 5 -t 1m \
  --csv=result/test
```

#### 스크립트 실행

```bash
cd locust
bash locust.sh
```

#### UI 모드 실행

```bash
cd locust
bash UI_mode_locust.sh
```

실행 후 브라우저에서 `http://localhost:8089`로 접속할 수 있습니다.

---

## 7. 결과 해석 방법

결과는 “수치가 낮다/높다”보다 “부하 증가에 따라 어떤 패턴이 나타나는가”를 중심으로 해석해야 합니다.

### 예시 해석 기준

#### 1) TPS는 증가하지만 P95 응답시간도 함께 급증하는 경우

- 서버가 요청을 받아들이고는 있지만 처리 지연이 누적되는 상태일 수 있습니다.
- 애플리케이션 로직, 세션 처리, 템플릿 렌더링, DB I/O 병목을 의심할 수 있습니다.

#### 2) 사용자 수 증가 구간에서 오류율이 급격히 상승하는 경우

- 서버 자원 한계, 세션 처리 문제, 타임아웃 설정 부족 가능성이 있습니다.
- 단순 평균 응답시간보다 먼저 오류율을 확인해야 합니다.

#### 3) 평균 응답시간은 안정적이지만 P95/P99가 높은 경우

- 일부 요청만 비정상적으로 느린 tail latency 문제일 수 있습니다.
- 평균값보다 백분위 지표를 함께 보는 편이 실제 지연 구간을 이해하는 데 유리합니다.

#### 4) 엔드포인트별 편차가 큰 경우

- `/course`보다 `/course/{id}`가 느리다면 상세 조회 로직 또는 템플릿/데이터 처리 비용이 더 크다는 의미일 수 있습니다.
- URL별 비교표를 만들어 병목 후보를 제시하면 문서 완성도가 높아집니다.

---

## 8. 결과 산출물

현재 저장소에는 아래와 같은 결과 파일 예시가 포함되어 있습니다.

- Locust CSV 결과: [locust/result/test_stats.csv](locust/result/test_stats.csv)
- Locust 오류/실패 결과: [locust/result/test_exceptions.csv](locust/result/test_exceptions.csv), [locust/result/test_failures.csv](locust/result/test_failures.csv)
- Locust 그래프 이미지: [locust/result/locust_graph.png](locust/result/locust_graph.png)
- JMeter 결과 디렉터리: [jmeter/result](jmeter/result)

결과는 아래와 같은 형식으로 정리하면 비교와 해석이 쉬워집니다.

| 조건 | 최대 동시 사용자 | TPS | Avg 응답시간 | P95 | 오류율 | 해석 |
| --- | --- | --- | --- | --- | --- | --- |
| Baseline | 10 | - | - | - | - | 기준 성능 |
| Load | 50 | - | - | - | - | 안정 구간 여부 확인 |
| Stress | 100 | - | - | - | - | 한계 구간 탐색 |

---

## 9. 한계점 및 개선 방향

이 저장소는 학습 및 비교 목적의 랩 환경이므로, 실제 운영 환경 성능 검증으로 바로 일반화하기에는 한계가 있습니다.

### 현재 한계

- 테스트 대상 애플리케이션이 단순 템플릿 렌더링 중심이라 실제 서비스 병목을 충분히 재현하지 못함
- 데이터셋이 작고 사용자 수가 제한적이어서 캐시/세션 경합 상황 재현이 약함
- 시스템 리소스(CPU, 메모리, GC, DB 연결 수)와의 상관분석이 포함되어 있지 않음
- 결과 요약이 툴 기본 출력 중심이라 의사결정용 비교 자료로는 부족함

### 개선 방향

1. **시나리오 고도화**
   - 로그인 성공/실패 분리
   - 특정 페이지 집중 조회 시나리오
   - 읽기/쓰기 혼합 시나리오 추가

2. **관측성 강화**
   - 서버 CPU, 메모리, GC, 애플리케이션 로그와 테스트 결과를 함께 수집
   - 응답시간 급증 시점과 인프라 지표를 연결해 분석

3. **결과 비교 체계화**
   - JMeter vs Locust 비교표 작성
   - Flask vs Spring Boot 비교 실행
   - 부하 단계별 Baseline / Load / Stress 결과 표준화

4. **재현성 개선**
   - 환경 변수 기반 설정 통일
   - 결과 파일 네이밍 규칙 정리
   - 테스트 실행 전제조건과 플러그인 목록 명문화

---

## 10. 저장소 구조

```text
.
├── Course_registration_flask/   # Flask 기반 테스트 대상 앱
├── Course_registration_java/    # Spring Boot 기반 테스트 대상 앱
├── jmeter/                      # JMeter 시나리오, 데이터셋, 결과
├── locust/                      # Locust 시나리오, 실행 스크립트, 결과
├── git_upload/                  # README 이미지 리소스
└── README.md
```

### 구조 해설

- `Course_registration_flask/`: 빠르게 실행 가능한 경량 테스트 대상
- `Course_registration_java/`: 다른 런타임에서 동일 흐름 검증용 대상
- `jmeter/`: GUI 중심 성능 테스트 자산
- `locust/`: 코드 기반 성능 테스트 자산