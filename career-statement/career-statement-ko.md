# 경력 기술서

## 요약

저는 **코드 품질, 아키텍처 건전성, 운영 안정성**을 중심으로 결제 시스템을 설계·구축해온 백엔드 엔지니어입니다.

단순한 기능 구현을 넘어, 장기적으로 유지보수 가능하고 확장 가능한 시스템을 만드는 것을 목표로 개발해왔습니다.

월 200~250만 건 규모의 결제 시스템 구축에 참여했으며, 특히 실시간 대사 시스템을 2.5년간 전담하여 전체 코드의 84%를 기여했습니다. 거래 일관성을 거의 100%에 가깝게 유지하면서 일 6~7만 건의 자동 대사를 처리하도록 시스템을 설계했습니다. 검증 프로세스를 개선하여 소요 시간을 83% 단축했고(30~40분 → 5분), PG API 호출을 50~70% 감소시켜 배치 처리 시간을 30~50% 단축했습니다.

외부 PG의 불안정성으로 인해 거래 상태가 초기에는 불완전할 수 있지만, 실시간 조회 → 비동기 재처리 → 배치 대사로 이어지는 다층 검증 구조를 통해 최종적으로 데이터가 점진적 일관성 수렴(Progressive Consistency Convergence) 하도록 설계했습니다.

---

## 핵심 기술 역량

### 기술 스택

| 카테고리 | 기술 | 경험 |
|---------|------|------|
| **언어/프레임워크** | Kotlin, Spring Boot, Spring Batch, Kotlin Coroutines, JPA/Hibernate | 3년+ 프로덕션 |
| **메시징/이벤트** | Kafka, Kafka Streams, AWS SQS, Spring Integration | 이벤트 기반 대사 시스템 |
| **데이터베이스** | MySQL, Redis, Spring Data JPA | 트랜잭션 DB, 분산 캐싱/락 |
| **클라우드/인프라** | AWS (MSK, SQS, IAM, Secrets Manager), Docker, Terraform, Datadog, Grafana | IaC 기반 운영 |

### 설계 역량

**분산 시스템**
- 이벤트 기반 아키텍처를 활용한 비동기 처리 구조 설계
- Spring Batch Remote Partitioning 기반 마스터-워커 분산 처리 시스템 구축
- 멱등성 및 낙관적 락을 활용한 장애 허용·재시도 전략 수립

**아키텍처 패턴**
- CQRS 기반 읽기/쓰기 책임 분리
- Strategy Pattern을 활용한 멀티 PG 추상화 구조 설계
- Domain-Driven Design 기반 경계 컨텍스트 분리 및 모듈화

**결제 시스템 도메인**
- 멀티 PG 통합 프레임워크 설계 및 운영
- 실시간 → 비동기 → 배치로 이어지는 거래 일관성 수렴 구조 설계
- 부분 취소 및 다중 취소를 지원하는 트랜잭션 모델 설계

---

## 경력

### SOCAR - 백엔드 엔지니어

**결제개발팀 | 2022년 4월 ~ 현재 (3년 9개월)**

SOCAR는 한국 최대 규모의 카셰어링 플랫폼입니다. 결제개발팀에서 **월 200~250만 건(일 6~7만 건)** 규모의 거래를 처리하는 결제 인프라를 설계·운영하고 있습니다.

외부 PG의 불안정성과 분산 시스템 환경에서 발생하는 거래 상태 불일치 및 정합성 문제를 해결하며, 실시간·비동기·배치 처리 구조를 기반으로 일관성 있는 결제 시스템을 구축하는 데 집중하고 있습니다.

---

## 주요 프로젝트

### 프로젝트 1. 결제 코어 플랫폼 재설계 (2025.08 ~ 진행중, 6개월)

**개요**
- **역할:** 핵심 도메인 설계 및 구현 (팀: 백엔드 엔지니어 3명)
- **기여도:** 245 커밋, +159,674줄
- **핵심 문제:** 부분 환불 불가능, PG 확장성 부족, 수동 검증 프로세스
- **프로젝트 현황:** 핵심 아키텍처 설계·구현·테스트 자동화 완료, **기존 서비스에서 단계적 마이그레이션 진행 중**

---

#### 1. Payment-Transaction 도메인 분리 설계

기존 시스템은 결제를 이진 상태(승인/취소)로 취급하여 부분 환불이 불가능했습니다. 데이터베이스 스키마가 "10,000원 승인, 3,000원 취소, 7,000원 잔액" 같은 부분 환불 이력을 표현할 수 없는 구조였습니다.

**기술 구조**
- **Aggregate Root 분리:** Payment (1) : Transaction (N) 관계 설계
- Payment: 결제 생명주기 관리 (CREATED → APPROVING → APPROVED → PARTIAL_CANCELED → CANCELED)
- Transaction: 개별 거래 단위 추적 (APPROVAL, PARTIAL_CANCEL, FULL_CANCEL)
- **취소 금액 음수 저장:** 취소 트랜잭션을 음수로 저장하여 승인/취소 내역 단일 테이블 관리

**핵심 코드**
```kotlin
fun calculateBalance(): BigDecimal {
    return transactions
        .filter { it.type in listOf(APPROVAL, PARTIAL_CANCEL, FULL_CANCEL) }
        .sumOf { it.calculateSignedAmount() }
}
// 예: approved=10,000원, canceled=-3,000원 → 잔액=7,000원

fun canPartialCancel(amount: BigDecimal): Boolean {
    return calculateBalance() >= amount // 취소 가능 금액 음수 방지
}
```

이를 통해 고객 환불 정책을 유연화했고(전액 환불만 가능 → 부분 환불 가능), 여러 번에 걸친 부분 취소 내역을 완전히 추적할 수 있게 되었습니다. 또한 금융 규정 준수를 위한 완전한 감사 추적이 가능해졌습니다.

---

#### 2. 멀티 PG 통합 프레임워크 설계

각 PG 연동마다 표준화 가능한 로직을 재구현하는 문제가 있었고, 신규 PG 추가 시 수 주가 소요되었습니다. Module-based Plugin Architecture와 Strategy Pattern, Template Method Pattern을 조합하여 확장 가능한 구조를 만들었습니다.

**아키텍처 패턴**
- **provider-client-base 공통 모듈:**
  - `InquiryResolver<T>` 인터페이스: PG별 조회 응답 해석 전략 추상화
  - `PaymentFailureCategorizer`: 메시지 기반 에러 분류 엔진
  - `ExternalCallLogger`: 템플릿 메서드 패턴 기반 공통 로깅/에러처리 파이프라인
  - Kotlin Extension Functions: 횡단 관심사 분리

**공통 인터페이스**
```kotlin
interface PaymentGatewayClient {
    suspend fun approve(request: CommonApprovalRequest): CommonApprovalResponse
    suspend fun cancel(request: CommonCancellationRequest): CommonCancellationResponse
    suspend fun partialCancel(request: CommonPartialCancelRequest): CommonCancellationResponse
    suspend fun inquire(request: CommonInquiryRequest): CommonInquiryResponse
}
```

InquiryResolver 패턴으로 전환하면서 중복 코드 852줄을 제거했고, Spring AutoConfiguration을 활용해 신규 PG 클라이언트 모듈 추가 시 코어 로직 변경 없이 자동 통합되도록 했습니다. 최종적으로 NHN KCP, Toss, Kakao Pay, Naver Pay, Naver Simple Pay, Nice Pay, NiceEPay(card, account), EasyPay, Bluewalnut 등 10개 PG 통합을 완료했습니다.

---

#### 3. 테스트 자동화로 검증 시간 83% 단축

10개 PG 환경 테스트에 수작업으로 30~40분이 소요되었습니다. 각 PG마다 승인 → 취소 → 조회를 개별 실행하고 스프레드시트에 수동 기록하는 방식이었습니다.

**기술 구현**
- **TestScenarioService:** 복잡한 결제 플로우 단계별 자동 실행
  - 3가지 시나리오: `FULL_FLOW`, `PARTIAL_CANCEL`, `ONLY_CREATE_APPROVE`
  - 각 단계별 결과 추적 및 실패 지점 명확화
- **멀티 PG 병렬 테스트:** 여러 PG사에 동일 시나리오 자동 실행 및 결과 비교
- **정산팀 제공 파일 자동 생성:** 테스트 결과를 CSV로 자동 생성

**3계층 아키텍처**
```
테스트 레이어: API POST /test/scenario/{scenarioName}
    ↓ 예: "partial_cancel_twice" → 10,000원 승인 → 3,000원 부분 취소 → 7,000원 부분 취소
어댑터 레이어: 공통 요청을 PG별 형식으로 변환
    ↓ 반환: {pg_raw_response, common_response}
PG Client 서비스: 실제 PG API 호출 실행
```

변경 전에는 각 PG사별로 수동으로 생성→승인→취소를 반복 테스트했지만, 변경 후에는 7개 PG사 × 3개 시나리오를 한 번의 API 호출로 자동 실행할 수 있게 되어 검증 시간을 30~40분에서 5분으로 단축했습니다. 회귀 테스트 자동화로 PG 통합 품질도 보장할 수 있게 되었습니다.

---

#### 4. PG 장애 자동 대응 시스템

PG 장애를 로그를 통해 사후에만 감지하고 있어서 사전 대응이 불가능했습니다. 오류 표준화 프레임워크와 다층 데코레이터 패턴을 적용하여 자동 대응 체계를 구축했습니다.

**1) PG 오류 표준화 프레임워크**
- **10개 PG사 오류 코드 체계화:** 이질적인 오류 코드를 14개 표준 카테고리로 통일 (8001~8999)
- **예외 타입 3단계 구조:**

```kotlin
sealed class ProviderException {
    data class ProviderResponseException(
        val type: BusinessFailure | SystemFailure | UnexpectedResponse,
    )
    data class ProviderNoResponseException(
        val type: NetworkFailure | TimeoutFailure,
    )
}
```

**2) 실행 파이프라인 통합**
- **다층 데코레이터 패턴 (PaymentExecutionExtensions):**
  - `withLogging`: 모든 성공/실패 구조화 로깅
  - `withCircuitBreaker`: Circuit Breaker 적용
  - `withPaymentExceptionResolver`: 예외 표준화
  - `withExternalCallLogIfEnabled`: 외부 호출 로그 수집

이를 통해 PG 장애 시 자동 트래픽 차단으로 연쇄 장애를 방지하고, 표준화된 오류 메시지로 고객 경험을 개선할 수 있는 기반을 마련했습니다. 장애 탐지 시간도 Slack 즉시 알림을 통해 단축했습니다.

---

### 프로젝트 2. 거래 일관성 시스템 - 3계층 점진적 일관성 아키텍처 (2023.09 ~ 2026.02, 2.5년)

**개요**
- **역할:** 아키텍처 설계, 핵심 구현, 운영 배포 및 개선 (단독 백엔드 엔지니어)
- **기여도:** 273 커밋, +14,721줄, 전체 코드의 84% 기여 (Main Contributor)

PG 네트워크 타임아웃 및 응답 지연으로 인해 거래 상태가 불확정적인 상황이 빈번하게 발생했습니다. 이를 해결하기 위해 3계층으로 구성된 독립적 정합성 시스템을 설계했습니다.

```
계층 1: 실시간 거래 상태 조회 (< 30초, CS 즉시 대응)
계층 2: Kafka 기반 승인취소 재처리 (초~분, 월 600건+ 자동 복구)
계층 3: Spring Batch 일일 대사 (익일, 일 6~7만 건, 최종 상태 수렴)
```

**주요 성과**

| 지표 | 결과 |
|------|------|
| 거래 조회 신뢰성 | ~100% |
| 일일 자동 대사 | 6~7만 건 |
| 월 자동 복구 | 600건+ |
| CS 비용 절감 | ~3% |
| PG API 호출 감소 | 50~70% |
| 배치 처리 시간 단축 | 30~50% |

---

#### Phase 3. Spring Batch 분산 대사 시스템 (2026.01 ~ 2026.02)

실시간 조회와 비동기 재처리만으로는 모든 거래 상태를 완벽하게 검증할 수 없었습니다. PG 응답 지연, 네트워크 불안정성, 일시적 장애 등으로 인해 일부 거래는 실시간으로 파악하기 어려웠고, 최종적인 정합성 보장을 위해서는 익일 일괄 검증이 필요했습니다.

초기에는 단일 노드로 배치를 운영했으나, 일 6~7만 건 대사에 60분이 소요되면서 여러 문제가 드러났습니다. 단일 장애 지점(SPOF)으로 인한 안정성 리스크와 처리 시간 증가, 그리고 동일 결제 건(승인 → 취소 → 부분취소)에 대한 중복 PG 조회가 발생하고 있었습니다.

이를 해결하기 위해 Remote Partitioning 패턴과 Hash 기반 파티셔닝, Worker 레벨 캐싱을 적용하여 분산 처리 시스템을 구축했습니다. PG API 호출을 50~70% 감소시키고 배치 처리 시간을 30~50% 단축했으며, 워커 노드 장애가 전체 시스템에 영향을 미치지 않도록 격리했습니다.

2개월간 진행했으며, 130KB 이상의 문서화를 통해 시퀀스 다이어그램을 포함한 상세한 설계 문서를 작성했습니다.

**Remote Partitioning 패턴 아키텍처**
```
마스터 노드 (작업 시작, 파티션 분배, 재시도 조정, 고아 레코드 복구)
    ↓ (Spring Integration MessageChannel 기반 작업 분배)
워커 노드 1~N (PG 조회, 일관성 검증, 상태 업데이트)
    ↓
batch_target 테이블 (거래별 상태 추적: READY → RUNNING → COMPLETED)
```

**주요 기술 설계 결정**

**1) batch_target 상태 관리 테이블**
```sql
CREATE TABLE batch_target (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    batch_date DATE NOT NULL,
    client_request_key VARCHAR(255) NOT NULL,
    status VARCHAR(20) NOT NULL,  -- READY, RUNNING, COMPLETED, FAILED
    job_execution_id BIGINT,
    worker_id VARCHAR(50),
    partition_id INT,
    UNIQUE KEY uk_batch_target (batch_date, client_request_key),
    INDEX idx_batch_date_status (batch_date, status)
);
```

상태를 관리하여 배치 재시작 시 중복 처리를 방지하고(COMPLETED 거래 자동 스킵), 워커 장애 시 RUNNING 레코드를 감지하여 복구할 수 있도록 했으며, Job 실행 정보나 파티션 할당 등의 데이터를 추적할 수 있도록 했습니다.

---

**2) clientRequestKey 기반 해시 파티셔닝**

`hash(clientRequestKey) % workerCount` 방식으로 파티션을 할당하여, 동일 결제 플로우(승인 → 취소 → 부분취소)가 항상 동일 워커에서 처리되도록 보장했습니다.

이를 통해 워커 내 PG 조회 결과를 캐싱하여 중복 PG 호출을 방지하고, 병렬 환경에서도 최종 상태 계산 순서를 보장하며, 파티션 레벨 격리를 통해 워커 장애가 다른 파티션에 영향을 미치지 않도록 했습니다.

---

**3) Append-Only 이력 + 스냅샷 분리**

`payment_reconciliation_log` 테이블에 PG 조회 결과를 Append-only 방식으로 저장하여 불변 감사 로그를 유지하고, `payment_final` 테이블에는 거래일 기준 확정 상태를 저장하여 보고용으로 활용합니다. 이를 통해 규정 준수를 위한 불변 감사 로그를 유지하면서도 효율적인 쿼리 성능을 확보했습니다.

---

**4) 재시작 기반 복구 API** -> TODO 조금 더 보완 필요

워커 노드 장애 시 파티션이 RUNNING 상태로 무기한 남는 문제를 해결하기 위해, 재시작 API(`/api/v1/batch/reconciliation/restart?jobExecutionId=X`)를 통해 batch_target을 RUNNING에서 READY 상태로 재설정하고 데이터 손실 없이 안전하게 재처리할 수 있도록 했습니다.

또한 고아 레코드 복구 API(`/api/v1/batch/reconciliation/recovery/orphan-records?batchDate=YYYY-MM-DD`)를 통해 충돌한 워커의 RUNNING 레코드를 자동 감지하고 수동 개입 없이 자동 복구할 수 있도록 했습니다. 건너뛰기가 아닌 재시작 기반 접근을 선택하여 데이터 일관성을 보장했습니다.

---

**5) PG 조회 최소화 - Worker 레벨 In-Memory 캐싱**

동일 결제 건(승인 → 취소 → 부분취소)에 대해 중복 PG 조회가 발생하고 있었습니다. DB 레벨에서는 확정 상태(CANCELED/REFUNDED) 거래와 정합성 일치 확인된 거래, 처리 완료(SUCCESS) 건을 사전에 필터링했고, Worker 레벨에서는 In-Memory Cache를 도입했습니다.

```kotlin
class WorkerLocalCache {
    private val cache = ConcurrentHashMap<String, PgInquiryResult>()

    fun getOrQuery(paymentKey: String): PgInquiryResult {
        return cache.getOrPut(paymentKey) {
            pgClient.inquire(paymentKey)
        }
    }

    fun shouldCache(result: PgInquiryResult): Boolean {
        return result.status !in listOf(UNKNOWN, FAILED)
    }
}
```

Hash 기반 파티셔닝을 통해 동일 결제 거래는 동일 Worker에 배정되도록 보장하고, Worker 격리된 메모리 공간으로 동시성 이슈를 방지하며, 배치 종료 시 자동 소멸하여 Stale Data 우려를 제거했습니다.

이를 통해 PG API 호출을 10,000회/일에서 3,000~5,000회/일로 50~70% 감소시키고, 배치 처리 시간을 60분에서 30~42분으로 30~50% 단축했으며, PG 서버 부하를 감소시켜 API Rate Limit 여유를 확보했습니다.

---

#### Phase 2. Kafka 기반 승인취소 재처리 시스템 (2025.04 ~ 06)

월 600건 이상의 승인취소 실패 건이 수동 재처리되고 있었습니다. 카셰어링 특성상 재고 확보를 위해 승인취소가 실패해도 예약이 우선 취소되기 때문에, 정상적으로 승인취소 되지 못한 거래건은 수기처리되고 있었습니다.

수동 개입을 줄여 휴먼에러를 줄이고 운영리소스 낭비를 막기 위해 이벤트 기반 비동기 재처리 파이프라인을 구축했습니다. 정책 기반 재시도 로직을 통해 월 600건 이상의 승인취소 실패 거래를 자동 재처리하고, CS 운영 시간을 약 3% 감소시켰습니다.

**이벤트 기반 비동기 재처리 파이프라인**
```kotlin
@KafkaListener(topics = [OUTPUT_TOPIC], groupId = "payment-reconciliation-retry-events")
fun consume(message: String) {
    val scheduledEvent = ScheduledEvent.fromJson(message)
    retryService.retryCancelApproval(scheduledEvent)
}

fun retryCancelApproval(scheduledEvent: ScheduledEvent) {
    val retryAttemptLog = retryAttemptLogService.createRetryAttemptLog(scheduledEvent)

    val response = pgClient.refund(
        socarTransactionId = scheduledEvent.originSocarTxnId,
        pgTransactionId = scheduledEvent.pgTxnId,
    )

    val status = if (response.isSuccess || response.isAlreadyRefunded()) {
        RetryStatus.SUCCESS
    } else {
        RetryStatus.FAILED
    }

    retryAttemptLogService.update(retryAttemptLog, status = status)
}
```

**주요 기술적 해결 과제**

무한 재시도 방지를 위해 `maxRetryCount` 기반 제한을 설정했고, 실패 시 Slack 알림으로 즉시 파악할 수 있도록 했습니다. 멱등성 보장을 위해 `requestId`를 `idempotencyKey`로 사용하여 Kafka Consumer에서 동일 `requestId`는 한 번만 처리하도록 했으며, 재시도 이력(`RetryAttemptLog`)에 모든 시도를 기록했습니다.

---

#### Phase 1. 실시간 거래 상태 조회 API (2023.09 ~ 12)

네트워크 타임아웃으로 응답을 받지 못했으나 실제로는 결제된 건이 빈번히 발생했습니다. 고객 문의 시 CS팀이 즉각적으로 거래 상태를 확인할 수 있는 시스템이 필요했습니다.

Strategy 패턴을 기반으로 멀티 PG 통합 조회 API를 구축하여 CS팀이 거래 발생 후 30초 이내에 즉시 응답할 수 있도록 했고, 10개 PG를 지원합니다.

```kotlin
fun getTargetPgService(pgName: String): PgService {
    return when (pgName) {
        Paygate.NICE.name -> niceService
        Paygate.TOSS.name -> tossService
        Paygate.NHN_KCP.name -> nhnKcpService
        // ... 10개 PG 지원
        else -> error("Unsupported pgName: $pgName")
    }
}

suspend fun getTransaction(request: PaymentAuditMessage): PgInquiryResult {
    val response = getTargetPgService(request.pgName)
        .getTransaction(pgEventLogContext)

    return PgInquiryResult(
        pgState = response.mapToTransactionState(),
        pgTransactionId = response.mapToPgTransactionId(),
    )
}
```

각 PG의 이질적인 조회 API 호출을 처리하고, 공통 `PgInquiryResult`로 정규화했습니다. `@PgInquiryHistoryLogging` 어노테이션을 통해 조회 이력을 자동 저장하여 감사 추적이 가능하도록 했습니다.

---

3계층 시스템이 통합되면서 각 계층은 서로 다른 시간대와 범위에서 독립적으로 동작하면서도 최종적으로 거래 일관성을 보장합니다.

| 계층 | 지연 | 범위 | 자동 복구 | 목적 |
| --- | --- | --- | --- | --- |
| 실시간 조회 | < 1초 | 온디맨드 | 아니오 | CS 즉시 응답 |
| 비동기 재시도 (Kafka) | 초~분 | 월 600건+ | 예 | 자동 취소 재시도 |
| 일일 배치 (Spring Batch) | 익일 | 일 6~7만 건 | 예 | 최종 상태 수렴 |

거래 상태 조회 신뢰성을 거의 100%에 가깝게 유지하면서 일일 6~7만 건의 자동 대사를 처리하고 있으며, 월 600건 이상의 승인취소 실패 거래를 자동으로 재시도합니다. 이를 통해 CS 운영 비용을 약 3% 절감했습니다.
