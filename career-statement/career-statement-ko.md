# 경력 기술서

## 요약

저는 **코드 품질, 아키텍처 건전성, 운영 안정성**을 중심으로 결제 시스템을 설계·구축해온 백엔드 엔지니어입니다.

단순한 기능 구현을 넘어, 장기적으로 유지보수 가능하고 확장 가능한 시스템을 만드는 것을 목표로 개발해왔습니다.

월 200 ~ 250만 건 규모의 결제 시스템 구축에 참여했으며, 특히 실시간 대사 시스템을 2.5년간 전담하여 전체 코드의 84%를 기여했습니다. 3계층 정합성 검증 시스템을 통해 99.9% 이상의 최종 정합성을 달성하며 일 6 ~ 7만 건의 자동 대사를 처리하도록 시스템을 설계했습니다. 검증 프로세스를 개선하여 소요 시간을 83% 단축했고(30 ~ 40분 → 5분), Hash 기반 파티셔닝과 Worker 레벨 캐싱을 통해 PG API 호출을 약 30% 감소시켰습니다.

외부 PG의 불안정성으로 인해 거래 상태가 초기에는 불완전할 수 있지만, 실시간 조회 → 비동기 재처리 → 배치 대사로 이어지는 다층 검증 구조를 통해 최종적으로 데이터가 점진적 일관성 수렴(Progressive Consistency Convergence) 하도록 설계했습니다.

**핵심 기술 성과:**

**분산 결제 시스템 설계 및 구현**
- 월 200 ~ 250만 건 규모 결제 시스템 구축 및 운영 (3개 주요 프로젝트, **770건 커밋**)
- Payment-Transaction 도메인 분리를 통한 부분 환불 트랜잭션 모델 설계
- 멀티 PG 통합 프레임워크 설계 및 운영 (Strategy Pattern 기반 확장 구조, 10개 PG 통합)
- Remote Partitioning 기반 분산 배치 시스템 구축 → 일 6 ~ 7만 건 자동 대사 처리

**점진적 일관성 수렴(Progressive Consistency Convergence) 아키텍처**
- 실시간 대사 시스템 2.5년 전담 → **전체 코드의 84% 기여**, 아키텍처 및 운영 전략 주도
- 실시간 조회 → 비동기 재시도 → 배치 대사로 이어지는 3계층 정합성 시스템 설계 → **99.9%+ 최종 정합성 달성**
- Kafka 기반 자동 재시도 파이프라인 구축 → **월 600건 이상 수동 처리 제거**, CS 비용 3% 절감
- Hash 기반 파티셔닝 및 Worker 레벨 캐싱으로 **PG API 호출 약 30% 감소**

**개발 프로세스 및 운영 효율화**
- 멀티 PG 검증 프로세스 자동화 구축 → **검증 시간 83% 단축** (30 ~ 40분 → 5분)
- PG 장애 자동 대응 시스템 구축 (Circuit Breaker, 표준화된 오류 처리)

---

## 핵심 기술 역량

### 기술 스택

| 카테고리 | 기술 | 경험 |
|---------|------|------|
| **언어/프레임워크** | Kotlin, Spring Boot, Spring Batch, Kotlin Coroutines, JPA/Hibernate | 3년+ 프로덕션 |
| **메시징/이벤트** | Kafka, Kafka Streams, AWS SQS, Spring Integration | 이벤트 기반 대사 시스템 |
| **데이터베이스** | MySQL, Redis, Spring Data JPA | 트랜잭션 DB, 분산 캐싱/락 |
| **클라우드/인프라** | AWS (MSK, SQS, IAM, Secrets Manager), Docker, Terraform, Datadog, Grafana | IaC 기반 운영 |

---

## 경력

### SOCAR - 백엔드 엔지니어

**결제개발팀 | 2022년 4월 ~ 현재 (3년 9개월)**

SOCAR는 한국 최대 규모의 카셰어링 플랫폼입니다. 결제개발팀에서 **월 200 ~ 250만 건(일 6 ~ 7만 건)** 규모의 거래를 처리하는 결제 인프라를 설계·운영하고 있습니다.

외부 PG의 불안정성과 분산 시스템 환경에서 발생하는 거래 상태 불일치 및 정합성 문제를 해결하며, 실시간·비동기·배치 처리 구조를 기반으로 일관성 있는 결제 시스템을 구축하는 데 집중하고 있습니다.

---

## 주요 프로젝트

### 프로젝트 1. 거래 일관성 시스템 - 3계층 점진적 일관성 아키텍처 (2023.09 ~ 2026.03, 2.5년)

**개요**
- **역할:** 아키텍처 설계, 핵심 구현, 운영 배포 및 개선 (단독 백엔드 엔지니어)
- **기여도:** 273 커밋, +14,721줄, 전체 코드의 84% 기여 (Main Contributor)

PG 네트워크 타임아웃 및 응답 지연으로 인해 거래 상태가 불확정적인 상황이 빈번하게 발생했습니다. 이를 해결하기 위해 3계층으로 구성된 독립적 정합성 시스템을 설계했습니다.

```
계층 1: 실시간 거래 상태 조회 (< 30초, CS 즉시 대응)
계층 2: Kafka 기반 승인취소 재처리 (초 ~ 분, 월 600건+ 자동 복구)
계층 3: Spring Batch 일일 대사 (익일, 일 6 ~ 7만 건, 최종 상태 수렴)
```

**주요 성과**

| 지표 | 결과 |
|------|------|
| 최종 정합성 달성률 | 99.9%+ |
| 일일 자동 대사 | 6 ~ 7만 건 |
| 월 자동 복구 | 600건+ |
| CS 비용 절감 | 약 3% |
| PG API 호출 감소 | 약 30% |

---

#### Phase 3. Spring Batch 분산 대사 시스템 (2026.01 ~ 2026.03)

실시간 조회와 비동기 재처리만으로는 모든 거래 상태를 완벽하게 검증할 수 없었습니다. PG 응답 지연, 네트워크 불안정성, 일시적 장애 등으로 인해 일부 거래는 실시간으로 파악하기 어려웠고, 최종적인 정합성 보장을 위해서는 익일 일괄 검증이 필요했습니다.

일 6 ~ 7만 건 규모를 고려했을 때 단일 노드 방식은 SPOF(단일 장애 지점)로 인한 안정성 리스크가 있었고, 동일 결제 건(승인 → 취소 → 부분취소)에 대한 중복 PG 조회가 발생할 수 있었습니다.

이러한 문제를 사전에 고려하여 Remote Partitioning 패턴과 Hash 기반 파티셔닝, Worker 레벨 캐싱을 적용한 분산 처리 시스템으로 설계했습니다. 동일 주문의 모든 거래를 같은 Worker에서 처리하도록 파티셔닝하여 캐시 효율을 극대화했고, PG API 호출을 약 30% 감소시켰습니다.

3개월간 진행했으며, 130KB 이상의 문서화를 통해 시퀀스 다이어그램을 포함한 상세한 설계 문서를 작성했습니다.

**Remote Partitioning 패턴 아키텍처**
```
마스터 노드 (작업 시작, 파티션 분배, 재시도 조정, 고아 레코드 복구)
    ↓
워커 노드 1~N (PG 조회, 일관성 검증, 상태 업데이트)
    ↓
batch_target 테이블 (거래별 상태 추적: READY → RUNNING → SUCCESS)
```

**주요 기술 설계 결정**

**1) batch_target 상태 관리 테이블**
```sql
CREATE TABLE batch_target (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    batch_type VARCHAR(50) NOT NULL,
    batch_target_date VARCHAR(20) NOT NULL,
    batch_group INT NOT NULL,
    partition_key VARCHAR(200) NOT NULL,
    business_key VARCHAR(200) NOT NULL,
    status VARCHAR(20) NOT NULL,  -- READY, RUNNING, SUCCESS, FAILED
    step_execution_id BIGINT,
    worker_hostname VARCHAR(100),
    version BIGINT NOT NULL DEFAULT 0,
    UNIQUE KEY batch_target_uk01 (batch_type, batch_target_date, business_key),
    INDEX batch_target_ix01 (batch_target_date, batch_group, status)
);
```

상태를 관리하여 배치 재시작 시 중복 처리를 방지하고(SUCCESS 거래 자동 스킵), 워커 장애 시 RUNNING 레코드를 감지하여 복구할 수 있도록 했으며, Step 실행 정보나 파티션 할당 등의 데이터를 추적할 수 있도록 했습니다.

---

**2) clientRequestKey 기반 해시 파티셔닝**

`hash(clientRequestKey) % workerCount` 방식으로 파티션을 할당하여, 동일 결제 플로우(승인 → 취소 → 부분취소)가 항상 동일 워커에서 처리되도록 보장했습니다.

이를 통해 워커 내 PG 조회 결과를 캐싱하여 중복 PG 호출을 방지하고, 병렬 환경에서도 동일 주문의 거래 처리 순서를 보장했습니다.

---

**3) Append-Only 이력 + 스냅샷 분리**

`payment_provider_reconciliation_log` 테이블에 PG 조회 결과를 Append-only 방식으로 저장하여 불변 감사 로그를 유지하고, `payment_provider_reconciliation` 테이블에는 Payment 단위 최종 상태를 저장하여 보고용으로 활용합니다. 이를 통해 규정 준수를 위한 불변 감사 로그를 유지하면서도 효율적인 쿼리 성능을 확보했습니다.

---

**4) PG 조회 최소화 - Worker 레벨 In-Memory 캐싱**

**문제 상황**

하나의 결제 주문에 여러 거래(승인 → 취소 → 부분취소)가 존재하여 모든 거래가 동일한 `paymentKey`로 PG 조회를 요청하면서 중복 API 호출이 발생했습니다. 이로 인해 PG API 호출 빈도가 과다하여 외부 API 비용이 증가하고, PG 응답 지연이 누적되어 배치 처리 시간이 증가했습니다.

**해결 방법: Hash Partitioning + Worker 레벨 캐싱**

**1. Hash Partitioning으로 동일 주문 → 동일 Worker 보장**

```kotlin
// Batch Group 계산 (clientRequestKey 해시 기반)
val batchGroup = Math.abs(clientRequestKey.hashCode()) % GRID_SIZE

BatchTargetEntity.forReconciliation(
    batchGroup = batchGroup,              // 0 ~ 5 중 하나
    partitionKey = clientRequestKey,      // "socar:order-12345"
    businessKey = transactionKey,         // PG 거래 고유 키
    ...
)
```

동일한 `clientRequestKey`(주문 식별자)는 항상 동일한 `batchGroup`에 배정되므로, 해당 주문의 모든 거래(승인/취소/부분취소)가 동일한 Worker에서 처리됩니다.

**2. Worker 레벨 메모리 캐시**

```kotlin
// Worker 프로세스 스코프 캐시 (Step 종료 시 자동 해제)
val pgInquiryCache = mutableMapOf<String, ReconciliationResult>()

// 대사 수행 시 캐시 전달
val result = reconciliationService.reconcile(
    paymentKey = paymentKey,
    transactionKey = target.businessKey,
    cache = pgInquiryCache,
)
```

**3. 캐싱 효과 예시**

```
시간순 처리 (Worker 2가 batchGroup 2 담당):

1) "order-12345" 승인 거래 처리
   → PG 조회 (paymentKey="PAY-12345")
   → 캐시 저장: {"PAY-12345": ReconciliationResult}

2) "order-12345" 취소 거래 처리
   → 동일 paymentKey="PAY-12345"
   → 캐시 히트! (PG 조회 안 함)

3) "order-12345" 부분취소 거래 처리
   → 동일 paymentKey="PAY-12345"
   → 캐시 히트! (PG 조회 안 함)

결과: 3번 조회 → 1번 조회로 감소 (66% 절감)
```

**안전성 보장**
- Worker 간 격리된 메모리 공간 → 동시성 이슈 방지
- Step 종료 시 자동 소멸 → Stale Data 우려 없음
- **Unknown/Failed는 캐싱 안 함** (재시도 대상)

**결과**
- PG API 호출 약 30% 감소 (취소/환불 건에 대한 캐시 히트)
- 배치 처리 효율 향상 및 PG 서버 부하 감소

---

**5) 트러블슈팅 경험**

분산 배치 시스템 구축 과정에서 두 가지 핵심 문제를 해결했습니다.

**문제 1. Kafka 이벤트 중복 처리**

Worker 처리 시간이 Kafka 폴링 주기(`max.poll.interval.ms`)를 초과하면 Consumer가 dead로 간주되어 커밋되지 않은 이벤트가 재발행되는 문제가 발생했습니다. 이로 인해 동일한 작업 그룹을 여러 Worker가 동시 처리하여 중복 처리 및 PG 조회 중복 호출이 발생했습니다.

**해결 방법: 3계층 방어 설계**

1. **DB 레벨 (Unique Constraint)**
   - `(batchType, batchTargetDate, businessKey)` 조합으로 중복 생성 원천 차단

2. **애플리케이션 레벨 (Optimistic Locking)**
   ```kotlin
   @Version
   @Column(name = "version", nullable = false)
   var version: Long = 0
   ```
   - JPA Optimistic Lock으로 동시 UPDATE 시 하나만 성공
   - 충돌 시 `OptimisticLockException` 발생 → 재시도

3. **비즈니스 레벨 (중복 실행 방지)**
   ```kotlin
   // SUCCESS가 아닌 건만 처리 (READY, FAILED만 대상)
   val targets = batchTargetRepository.findByBatchTargetDateAndBatchGroupAndStatusNotAndIdGreaterThan(
       status = BatchTargetStatus.SUCCESS,
       ...
   )
   ```
   - 이미 처리 완료(`SUCCESS`)된 건은 자동 스킵

**결과:**
- 중복 처리 효과적 방지: Unique Constraint + Version Lock으로 데이터 정합성 강력하게 보장
- PG 중복 조회 제거: 중복 실행 방지로 재시도 시에도 안전
- 안정적인 재시작: Spring Batch Restart 시에도 정확히 미완료 건만 재처리

---

**문제 2. Kafka Partition 분산 불균형**

초기에는 `sequenceNumber`만 Kafka message key로 사용했습니다. 6개 작업 그룹이 있을 때 6개 고정 key(`"0"`, `"1"`, ..., `"5"`)만 생성되어 Kafka의 hash 기반 파티셔닝 특성상 특정 파티션에만 메시지가 집중되고 나머지 파티션은 유휴 상태가 되었습니다.

**문제점:**
- 특정 Kafka Partition에 메시지 쏠림 현상
- 특정 Consumer(Worker)에 작업 부하 집중
- 병렬 처리 효율 저하 → 총 처리 시간 증가 (예: Worker A는 2개 그룹, Worker B는 0개)

**해결 방법: Kafka Message Key 개선**

```kotlin
// 개선 전: sequenceNumber만 사용
setMessageKeyExpression("headers['sequenceNumber']")
// 문제: "0", "1", ..., "5" → 6개 고정 key

// 개선 후: batchTargetDate + sequenceNumber 조합
setMessageKeyExpression(
    "(headers['batchTargetDate'] ?: 'unknown') + ':' + (headers['sequenceNumber']?.toString() ?: 'unknown')"
)
// 예시: "2026-01-15:0", "2026-01-15:1", ..., "2026-01-16:0", ...
```

**효과:**
- **날짜별로 다른 Key 생성** → Kafka의 hash 분산 효율 극대화
- **파티션 균등 분배** → 모든 Consumer에 작업 분산
- **순서 보장 유지** → 동일 날짜 + 동일 번호는 항상 같은 파티션으로 라우팅

**결과:**
- Kafka Partition 균등 분산: 6개 고정 key → 날짜별 고유 key로 확장
- Worker 부하 분산 개선: 특정 Worker 과부하 해소
- 총 처리 시간 단축: 병렬성 향상으로 전체 배치 시간 감소 (추정 20-30%)

#### Phase 2. Kafka 기반 승인취소 재처리 시스템 (2025.04 ~ 06)

월 600건 이상의 승인취소 실패 건이 수동 재처리되고 있었습니다. 카셰어링 특성상 재고 확보를 위해 승인취소가 실패해도 예약이 우선 취소되기 때문에, 정상적으로 승인취소 되지 못한 거래건은 수기처리되고 있었습니다.

수동 개입을 줄여 휴먼에러를 줄이고 운영리소스 낭비를 막기 위해 이벤트 기반 비동기 재처리 파이프라인을 구축했습니다. Kafka를 통해 실패 건을 자동으로 재시도하며, 무한 재시도 방지를 위해 `maxRetryCount` 기반 제한을 설정했고, 멱등성 보장을 위해 `requestId`를 `idempotencyKey`로 사용했습니다. 재시도 이력(`RetryAttemptLog`)에 모든 시도를 기록하여 감사 추적이 가능하도록 했으며, 실패 시 Slack 알림으로 즉시 파악할 수 있도록 했습니다.

정책 기반 재시도 로직을 통해 월 600건 이상의 승인취소 실패 거래를 자동 재처리하고, CS 운영 시간을 약 3% 감소시켰습니다.

---

#### Phase 1. 실시간 거래 상태 조회 API (2023.09 ~ 12)

네트워크 타임아웃으로 응답을 받지 못했으나 실제로는 결제된 건이 빈번히 발생했습니다. 고객 문의 시 CS팀이 즉각적으로 거래 상태를 확인할 수 있는 시스템이 필요했습니다.

Strategy 패턴을 기반으로 멀티 PG 통합 조회 API를 구축하여 CS팀이 거래 발생 후 30초 이내에 즉시 응답할 수 있도록 했고, 10개 PG를 지원합니다. 각 PG의 이질적인 조회 API 호출을 처리하고, 공통 `PgInquiryResult`로 정규화했습니다. `@PgInquiryHistoryLogging` 어노테이션을 통해 조회 이력을 자동 저장하여 감사 추적이 가능하도록 했습니다.

---

3계층 시스템이 통합되면서 각 계층은 서로 다른 시간대와 범위에서 독립적으로 동작하면서도 최종적으로 거래 일관성을 보장합니다.

| 계층 | 지연 | 범위 | 자동 복구 | 목적 |
| --- | --- | --- | --- | --- |
| 실시간 조회 | < 30초 | 온디맨드 | 아니오 | CS 즉시 응답 |
| 비동기 재시도 (Kafka) | 초 ~ 분 | 월 600건+ | 예 | 자동 취소 재시도 |
| 일일 배치 (Spring Batch) | 익일 | 일 6 ~ 7만 건 | 예 | 최종 상태 수렴 |

3계층 독립 검증 시스템을 통해 99.9% 이상의 최종 정합성을 유지하며 일일 6 ~ 7만 건의 자동 대사를 처리하고 있으며, 월 600건 이상의 승인취소 실패 거래를 자동으로 재시도합니다. 이를 통해 CS 운영 비용을 약 3% 절감했습니다.

---

### 프로젝트 2. 결제 코어 플랫폼 재설계 (2025.08 ~ 진행중, 6개월)

**개요**
- **역할:** 핵심 도메인 설계 및 구현 (팀: 백엔드 엔지니어 3명)
- **기여도:** 333 커밋, +176,948줄
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

이를 통해 고객 환불 정책을 유연화했고(전액 환불만 가능 → 부분 환불 가능), 여러 번에 걸친 부분 취소 내역을 추적할 수 있게 되었습니다. 또한 금융 규정 준수를 위한 감사 추적이 가능해졌습니다.

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

InquiryResolver 패턴으로 전환하면서 중복 코드 852줄을 제거했고, Spring Boot AutoConfiguration을 활용해 신규 PG 클라이언트 모듈을 의존성에 추가하는 것만으로 자동 빈 등록이 되도록 했습니다(`@AutoConfiguration` + `@ComponentScan`). 최종적으로 NHN KCP, Toss, Kakao Pay, Naver Pay, Naver Simple Pay, Nice Pay, NiceEPay(card, account), EasyPay, Bluewalnut 등 10개 PG 통합을 완료했습니다.

---

#### 3. 테스트 자동화로 검증 시간 83% 단축

10개 PG 환경 테스트에 수작업으로 30 ~ 40분이 소요되었습니다. 각 PG마다 승인 → 취소 → 조회를 개별 실행하고 스프레드시트에 수동 기록하는 방식이었습니다.

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

변경 전에는 각 PG사별로 수동으로 생성→승인→취소를 반복 테스트했지만, 변경 후에는 7개 PG사 × 3개 시나리오를 한 번의 API 호출로 자동 실행할 수 있게 되어 검증 시간을 30 ~ 40분에서 5분으로 단축했습니다. 회귀 테스트 자동화로 PG 통합 품질도 보장할 수 있게 되었습니다.

---

#### 4. PG 장애 자동 대응 시스템

PG 장애를 로그를 통해 사후에만 감지하고 있어서 사전 대응이 불가능했습니다. 오류 표준화 프레임워크와 다층 데코레이터 패턴을 적용하여 자동 대응 체계를 구축했습니다.

**1) PG 오류 표준화 프레임워크**
- **10개 PG사 오류 코드 체계화:** 이질적인 오류 코드를 14개 표준 카테고리로 통일 (8001 ~ 8999)
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
