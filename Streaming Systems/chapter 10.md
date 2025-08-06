# 개요
<img width="867" height="397" alt="image" src="https://github.com/user-attachments/assets/84ea6808-e0ed-4921-89bd-ba03ef71f234" />  

- 발표구성 및 내용 간략히 소개
  - 유래 및 연도
  - 특징
  - 장점 및 단점
  - 다른 시스템과 비교하여을 때의 강점은?
  - 그 외 기술적 특징

## MillWheel
### 1. 유래 및 연도
- Google 내부에서 2008년부터 개발이 시작되어 2013년 VLDB에 논문으로 공개됨.
  <img width="600" height="395" alt="image" src="https://github.com/user-attachments/assets/ec7d4692-919e-431f-85e2-29ce4d1ca296" />
- FlumeJava, MapReduce 등의 한계를 극복하기 위해 지속적인 스트리밍 처리와 정확한 시간 기반 처리를 목적으로 설계되었음.
- 이후 Google Cloud Dataflow와 Apache Beam의 시간 모델 및 상태 모델의 기반이 됨.
### 2. 특징
- 무한 스트리밍 데이터 처리를 위한 분산 스트림 처리 프레임워크.
- 정확히 한 번 처리(exactly-once semantics)를 실현하는 최초의 대규모 시스템 중 하나.
- 이벤트타임 기반 윈도우 처리, 워터마크, 사용자 타이머(timer) 기능을 통합적으로 지원.
- 연산자 간 DAG(Directed Acyclic Graph) 구성, 연산자별 상태(state) 및 메시지 기반 checkpoint 관리.
- 내부적으로 외부 상태 저장소(예: Bigtable 등)를 사용하여 durable state를 유지함.
### 3. 장점 및 단점
#### 3.1. 장점
- 트랜잭셔널 메시지 처리와 원자적 상태 업데이트로 Exactly-Once 처리 보장.
- 늦게 도착한 이벤트(out-of-order data)에 대한 처리를 위한 워터마크 개념 도입.
- 상태 저장의 내구성(durable state) 확보로 장애 복구 시 강력한 일관성 유지 가능.
- 고성능 실시간 처리에 적합하며, 타이머를 통해 복잡한 비동기 논리 구현 가능.
#### 3.2. 단점
- 내부 전용 시스템으로 오픈소스가 아니며, 외부 접근 불가.
- 상태 접근은 RPC 호출을 통한 외부 저장소 의존 → 연산 오버헤드 및 latency 증가.
- 글로벌 체크포인트가 존재하지 않으며 연산자 단위 상태로 구성 → 파이프라인 관리 복잡.
- 프로그래밍 모델이 매우 저수준으로, 추상화가 부족하여 생산성 측면에서 불리함.
### 4. 다른 시스템과 비교하였을 때의 강점은?
- MillWheel은 당시 기준으로는 매우 획기적인 시간 모델을 제안함:  
이벤트타임 처리, 워터마크 전파, 타이머 관리 등 현재 Flink/Beam에서도 채택된 개념을 대부분 최초 구현.
- Storm보다 latency는 유사하지만 일관성(consistency) 및 중복제거(idempotency) 측면에서 압도적인 차이.
- Kafka Streams나 Spark Structured Streaming처럼 micro-batch 기반이 아닌 True Stream Processor로 분류됨.
- Cloud Dataflow 및 Beam 모델의 핵심 설계 사상(특히 시간과 상태에 대한 모델링)은 MillWheel에서 직접 계승됨.
### 5. 그 외 기술적 특징
- 상태 관리 (State Management):  
각 연산자는 상태를 key-value 형태로 관리하며, 외부 durable 저장소에 분산 저장됨. 상태 접근은 반드시 RPC를 통해 이루어지며, atomic read-modify-write 연산을 제공.
- 워터마크 (Watermark):  
시스템의 입력이 특정 시점까지 도달했음을 나타내는 시계열 신호로, 연산자의 시간 결정, 타이머 동작, 윈도우 폐쇄 등에 핵심적으로 사용됨.
- 타이머 및 트리거 (Timers & Triggers):  
사용자는 임의의 이벤트타임 또는 처리시간에 타이머를 등록 가능하며, 지정된 시간에 트리거가 실행되어 상태 갱신, 출력, 알람 등을 유도.
- 정확히 한 번 처리 (Exactly-Once Semantics):  
상태 업데이트와 출력의 원자성 확보를 위해 메시지 기반 체크포인트와 상태 커밋이 연산자 단위로 tightly coupled 되어 있음.  
재처리 시에도 중복 없이 결과를 복원 가능하며, 이것은 후속 시스템인 Cloud Dataflow의 정확성 보장과 동일한 방식.
- 장애 복구 및 내구성 (Fault Tolerance):  
각 연산자의 상태는 지속적으로 외부 스토리지에 flush되며, 장애 복구 시 상태 및 메시지 커서(offset) 복원으로 정확한 재시작 가능.  
단, 전역적인 스냅샷이 존재하지 않아 장애 시 연산자간 타이밍 불일치가 발생할 수 있음.

## Apache Kafka
### 1. 유래 및 연도
- 2011년 LinkedIn에서 내부 로그 처리용으로 개발 시작.  
- 이후 Apache 프로젝트로 전환되었으며, 2025년 기준 최신 버전은 Kafka 4.x로 KRaft 기반 메타데이터 관리 도입.
### 2. 특징
- 고가용성 분산 로그 기반 메시징 시스템.
- 메시지의 내구성, 순서 보장, 파티셔닝을 통한 확장성 제공.
- Kafka Streams, ksqlDB 등을 통한 내장 스트림 처리 기능 보유.
- Confluent Platform, Kafka Connect, Schema Registry 등 풍부한 생태계 지원.
### 3. 장점 및 단점
#### 3.1. 장점
- 내구성 있는 로그 구조로 장애 시에도 메시지 유실 없음.
- 트랜잭션 및 idempotent 프로듀서로 Exactly-Once 처리 구현 가능.
- 대규모 스트림 처리 시스템에서 사실상 표준화된 메시지 전송 계층.
- Kafka Streams는 클러스터 없이 애플리케이션 내에서 경량 실행 가능.
#### 3.2. 단점
- 복잡한 상태 기반 연산, 윈도우 처리 등은 Flink 대비 제한적.
- Kafka Streams는 상태 저장 시 RocksDB와 체인지로그 토픽을 사용하여 I/O 부하 존재.
- 워터마크 및 트리거 기능 부재로 이벤트타임 기반 정밀 제어 어려움.
### 4. 다른 시스템과 비교하였을 때의 강점은?
- Kafka는 스트림 전송 계층으로 가장 널리 사용되며, Flink, Spark, Beam 등과 높은 호환성 보유.
- Kafka Streams는 별도 클러스터 없이 JVM 애플리케이션 내에서 실행 가능하며, 마이크로서비스 아키텍처에 적합.
- End-to-End Exactly-Once 보장 기능을 Kafka 내부에서 자체적으로 처리 가능하다는 점에서 외부 상태 스토리지에 의존하는 시스템보다 단순화된 정확성 보장 제공.
### 5. 그 외 기술적 특징
- 메시지 저장은 append-only log 구조로, 파티션별 순서 보장.
- 프로듀서는 enable.idempotence=true 설정으로 중복 없는 전송 보장.
- 트랜잭션 API를 활용하면 다중 파티션에 대한 원자적 쓰기 및 소비자 오프셋 커밋 가능.
- Kafka Streams는 상태를 로컬 RocksDB에 저장하고, Kafka 토픽에 체인지로그로 복제.
- 이벤트타임이 아닌 처리시간 기반 윈도우 및 grace period로 지연 이벤트 처리.
- 체크포인트는 오프셋 커밋 기반이며, 트랜잭션 처리 시 출력과 오프셋 커밋이 함께 수행되어 Exactly-Once semantics 실현.

## Cloud Dataflow
### 1. 유래 및 연도
- 2015년 Google Cloud Platform에서 공개한 완전 관리형 스트리밍 및 배치 처리 서비스.
  <img width="600" height="527" alt="image" src="https://github.com/user-attachments/assets/05b5b1ad-c4cf-4bfb-93dd-544adde55b0d" />
- Apache Beam 모델의 참조 구현체로 기능하며, Google 내부 기술(MillWheel, FlumeJava 등)을 기반으로 설계됨.
### 2. 특징
- 스트리밍 + 배치 통합 프로그래밍 모델을 지원하는 서버리스 플랫폼.
- 자동 스케일링, 장애 복구, 모니터링 등 운영 자동화 기능 탑재.
- Windmill 실행 엔진 기반으로 정확성, 지연 최적화 기능을 제공.
- GCP 서비스와 긴밀하게 통합되어 BigQuery, Pub/Sub, Bigtable 등과의 연결이 용이함.
### 3. 장점 및 단점
#### 3.1. 장점
- 완전 관리형으로 클러스터 운영 부담이 없고, 작업 스케일 아웃/스케일 인 자동화.
- Beam API 기반으로 고급 윈도우, 트리거, 세션 처리 기능 제공.
- 정확히 한 번 처리 보장, 이벤트타임 기반 처리 가능.
- GCP의 모니터링, 로깅, 보안 정책과 완벽하게 통합됨.
#### 3.2. 단점
- GCP 종속 서비스로 다른 클라우드나 온프레미스에서는 사용 불가.
- Flink 등과 달리 시스템 레벨 튜닝 옵션 제한.
- 낮은 지연이 핵심인 초실시간 처리에는 다소 부적합.
#### 4. 다른 시스템과 비교하였을 때의 강점은?
- Apache Beam의 공식 러너로, Beam 모델의 모든 기능을 완전하게 구현한 참조 시스템.
- 운영 자동화 수준이 높아 실무에서 배포, 확장, 모니터링이 압도적으로 간편함.
- GCP 네이티브 서비스들과의 통합성이 뛰어나, 엔드투엔드 분석 파이프라인 구성에 적합.
#### 5. 그 외 기술적 특징
- 상태는 Windmill 엔진이 관리하며, Bigtable 기반의 분산 저장소에 상태를 영속화.
- 이벤트타임 워터마크는 소스→연산자 간 전파되며, 각 연산자는 입력 워터마크의 최소값으로 현재 위치 판단.
- 트리거는 AfterWatermark, AfterProcessingTime 등 복합 조건 기반으로 설정 가능.
- 지연 데이터는 Allowed Lateness 설정을 통해 수용 범위 지정 가능.
- Exactly-Once 처리는 메시지별 ID를 통한 중복 감지 + 상태 업데이트 및 출력의 원자적 커밋으로 구현.
- 장애 발생 시 저장된 상태와 메시지 ID 기반 재처리, 중복 제거를 수행함.

## Apache Flink
### 1. 유래 및 연도
- 독일 TU Berlin의 Stratosphere 프로젝트에서 출발.
- 2015년 Apache Flink로 공식 릴리스.
- 현재는 Apache Software Foundation에서 관리 중이며, 2025년 기준 Flink 2.0 릴리스됨.
### 2. 특징
- 스트리밍 우선(Stream-first) 철학 기반의 분산 데이터 처리 프레임워크.
- 상태 기반 이벤트 처리, 이벤트타임, 세밀한 윈도우/트리거/타이머 제어 지원.
- 배치 처리도 스트리밍의 특수 케이스로 다루는 통합 실행 모델 채택.
- 다양한 커넥터 및 Table API, SQL 인터페이스 지원.
### 3. 장점 및 단점
#### 3.1. 장점
- 고성능: Operator chaining, pipelined execution, off-heap state 등으로 높은 처리량.
  <img width="600" height="657" alt="image" src="https://github.com/user-attachments/assets/70a8d858-6a68-45a6-b4d3-44ab321261c2" />
- 강력한 상태 관리: RocksDB 또는 인메모리 backend, Keyed State, Queryable State 등 지원.
- 정확히 한 번 처리 보장: Chandy-Lamport 기반 체크포인트 + savepoint.
  <img width="600" height="525" alt="image" src="https://github.com/user-attachments/assets/1775ba8f-3925-41d7-ae88-c42bf3091b24" />
  <img width="600" height="630" alt="image" src="https://github.com/user-attachments/assets/8d438177-b75a-4f18-a7f3-acae8af44184" />
- Event-time 중심의 처리 모델과 유연한 트리거 API.
#### 3.2. 단점
- 클러스터 구성 및 운영 복잡: JobManager, TaskManager, checkpoint 디렉토리 등 설정 필요.
- 시스템 수준 튜닝(예: checkpoint 주기, 병렬도, backpressure 조절 등)에 대한 깊은 이해 필요.
- 장애 복구/savepoint 이관에는 체크포인트 동기화와 파티션 정합성 고려 필요.
### 4. 다른 시스템과 비교하였을 때의 강점은?
- Flink는 상태 기반 복잡한 실시간 처리에 가장 적합하며, CEP, 세션 집계, 스트리밍 조인에 유리함.
- Kafka Streams는 단순 구조에서 유리하지만 상태 규모, 커스텀 처리 측면에서 Flink보다 제한됨.
- Beam보다 먼저 savepoint 기능을 제공했고, 실행 계획 최적화 및 제어 능력 우수.
- Cloud Dataflow보다 낮은 레벨에서 세밀한 설정이 가능하며, 성능/지연 튜닝 폭이 넓음.
### 5. 그 외 기술적 특징
- 상태 관리: RocksDB(디스크 기반) 또는 heap state(메모리 기반) 선택 가능.
  <img width="600" height="535" alt="image" src="https://github.com/user-attachments/assets/c97e6de9-f6b0-467b-b974-be6463ec5a0a" />
- 상태는 Keyed State로 분리되어 각 파티션에 분산 저장됨.
- 체크포인트/스냅샷: Barrier-aligned checkpoint를 통해 글로벌 스냅샷 생성.
- Savepoint는 사용자 요청 시 생성되며, 파이프라인 재시작 및 업그레이드에 활용.
- 워터마크: 이벤트타임 기반 처리의 핵심으로, 소스 및 연산자 간 전파됨.
- 주기적 또는 수동으로 워터마크를 생성할 수 있으며, 허용 지연(lateness) 설정 가능.

## Apache Beam
### 1. 유래 및 연도
- Google Cloud Dataflow의 오픈소스화 프로젝트로, 2016년 Apache 프로젝트로 전환됨.
- "Batch + Streaming" 통합 프로그래밍 모델 제공을 목표로 설계됨.
### 2. 특징
- 실행 엔진에 독립적인 범용 프로그래밍 모델.
- 하나의 파이프라인 정의로 Flink, Spark, Dataflow 등 다양한 실행 환경에서 실행 가능.
- 트리거, 윈도우, 누산 모드, 지연 이벤트 허용 등 정교한 시간 모델 지원.
- SDK: Java, Python, Go / DSL: Scio(Scala), SQL.
### 3. 장점 및 단점
#### 3.1. 장점
- 실행 플랫폼에 종속되지 않는 이식성 (portability).
- 배치/스트리밍 API 통합으로 동일 파이프라인을 유연하게 작성 가능.
- 시간 추론 및 상태 타이머 등 MillWheel 기반 모델링 계승.
#### 3.2. 단점
- Beam 모델이 복잡해 러너에 따라 구현 수준 차이 발생.
- 실행 엔진에 따라 성능, 기능 제약 (예: Flink는 고성능, Spark는 일부 제한).
- cross-language 지원 미완성 상태 (예: Python SDK + Flink runner 조합).
### 4. 다른 시스템과 비교하였을 때의 강점은?
- 하나의 코드로 다양한 환경에 배포 가능 (예: 개발은 로컬 Spark, 운영은 Dataflow).
- 모델 수준에서 이벤트타임, 트리거, 세션 등 스트리밍의 핵심 개념을 완비함.
- 실행 환경 변화에 대응 가능한 유연성 확보.
### 5. 그 외 기술적 특징
- 상태 관리: Beam은 사용자 정의 상태(StateSpec)를 제공하며, SDK에서 정의된 상태는 러너에 의해 구현됨.
- 워터마크: 데이터 소스에서의 이벤트타임 추론을 통해 시스템 내 워터마크 전파.
- 트리거: AfterWatermark, AfterProcessingTime, AfterCount, composite trigger 등을 지원.
- 윈도우: Fixed, Sliding, Session 윈도우 외에도 custom window 구현 가능.
- Exactly-Once: 구현은 러너에 따라 다름. Flink, Dataflow는 지원, Spark는 일부 제한.
- Cross-language 실행: 향후 목표로 Python SDK → Java 러너 등 완전 이식성을 목표로 함.
- 트리거 및 윈도우: 처리시간, 이벤트시간, count 기반 트리거 조합 가능.  
Tumbling, Sliding, Session 등 다양한 윈도우 종류와 trigger/pane 처리 모델 제공.
- 정확히 한 번 처리: 체크포인트 타이밍에만 외부 시스템에 결과 출력 → side-effect 격리.  
Sink 연산에서 Two-phase commit 구현 시 외부 시스템에도 exactly-once 보장 가능.
