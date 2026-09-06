---
layout: page
title: Golden Harvest
description: 농산물 유통 플랫폼의 재고 관리 서비스
img: # 나중에 추가
importance: 1
category: Team Project
github: https://github.com/Gold-Team-Project/golden-harvest-integrated
---

<div style="margin-bottom: 2rem;">
  <img src="https://img.shields.io/badge/Java-007396?style=flat-square&logo=openjdk&logoColor=white"/>
  <img src="https://img.shields.io/badge/Spring Boot-6DB33F?style=flat-square&logo=springboot&logoColor=white"/>
  <img src="https://img.shields.io/badge/Apache Kafka-231F20?style=flat-square&logo=apachekafka&logoColor=white"/>
  <img src="https://img.shields.io/badge/JPA-59666C?style=flat-square&logo=hibernate&logoColor=white"/>
  <img src="https://img.shields.io/badge/MyBatis-000000?style=flat-square"/>
  <img src="https://img.shields.io/badge/Redis-FF4438?style=flat-square&logo=redis&logoColor=white"/>
  <img src="https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white"/>
  <img src="https://img.shields.io/badge/AWS-232F3E?style=flat-square&logo=amazonwebservices&logoColor=white"/>
</div>

---

<h3 style="font-size: 1.2rem; font-weight: 700; margin-bottom: 1rem;">📌 프로젝트 개요</h3>

<table style="width: 100%; border-collapse: collapse; margin-bottom: 2rem;">
  <tbody>
    <tr style="border-bottom: 1px solid #eee;">
      <td style="padding: 8px; font-weight: 600; width: 120px;">기간</td>
      <td style="padding: 8px;">2025.12 ~ 2026.02 (9주)</td>
    </tr>
    <tr style="border-bottom: 1px solid #eee;">
      <td style="padding: 8px; font-weight: 600;">유형</td>
      <td style="padding: 8px;">한화시스템 Beyond SW Camp 최종 프로젝트</td>
    </tr>
    <tr style="border-bottom: 1px solid #eee;">
      <td style="padding: 8px; font-weight: 600;">팀 구성</td>
      <td style="padding: 8px;">5인 (백엔드 3명, 프론트엔드 1명, 팀장/PM 1명) — 재고 도메인 전담</td>
    </tr>
    <tr style="border-bottom: 1px solid #eee;">
      <td style="padding: 8px; font-weight: 600;">배포</td>
      <td style="padding: 8px;">AWS ECS</td>
    </tr>
    <tr style="border-bottom: 1px solid #eee;">
      <td style="padding: 8px; font-weight: 600;">GitHub</td>
      <td style="padding: 8px;">
        <a href="https://github.com/Gold-Team-Project/golden-harvest-integrated" target="_blank">통합 리포지토리</a>
        &nbsp;·&nbsp;
        <a href="https://github.com/Gold-Team-Project/golden-harvest-inventory" target="_blank">담당 서비스(재고)</a>
      </td>
    </tr>
  </tbody>
</table>

**한 줄 요약**: 여러 개의 독립된 서비스(상품, 구매주문, 판매주문, 재고 등)로 구성된 MSA 환경에서, 재고 서비스의 입출고·폐기·가격정책 도메인을 설계하고, Kafka 이벤트를 매개로 한 서비스 간 데이터 정합성 전략(Event Carried State Transfer)과 동시성 제어, CQRS 아키텍처를 구현했습니다.

**해결하고자 한 문제**
- 농산물은 신선도·유통기한이 중요한 만큼, 입고일 기준 **선입선출(FIFO)** 방식의 정확한 재고 소진이 필요함
- 여러 서비스(상품 서비스, 주문 서비스, 재고 서비스)가 분리된 MSA 구조에서, **서비스 간 강결합 없이** 정합성을 유지해야 함
- 대량 동시 주문 상황에서 **재고 초과 판매(Overselling)를 방지**해야 함
- 관리자가 폐기 손실, 재고 현황을 **정량적으로 모니터링**할 수 있어야 함

**담당 역할**: 재고 서비스 Command/Query 아키텍처 전체 설계 · Kafka Consumer/Producer 및 이벤트 기반 입고 처리 파이프라인 · Lot 기반 FIFO 출고 로직 및 동시성 제어 · 폐기(Discard) 도메인 및 통계 API · Bloom Filter 기반 중복 처리 방지

---

<h3 style="font-size: 1.2rem; font-weight: 700; margin-bottom: 1rem;">🏗️ 시스템 아키텍처</h3>

```
┌─────────────────┐        Kafka Topic         ┌──────────────────────────┐
│  구매주문 서비스   │  purchase.order.created   │                          │
│ (Purchase Order) │ ─────────────────────────▶ │                          │
└─────────────────┘                             │                          │
                                                 │                          │
┌─────────────────┐        Kafka Topic          │      재고(Inventory)      │
│  판매주문 서비스   │   sales.order.created      │         서비스            │
│  (Sales Order)   │ ─────────────────────────▶ │                          │
└─────────────────┘                             │  ┌────────────────────┐  │
                                                 │  │   Command 영역      │  │
┌─────────────────┐   item.master.updated /     │  │  (JPA, 도메인 로직)  │  │
│   상품 서비스     │   item.origin.price.updated │  └────────────────────┘  │
│  (Item Service)  │ ─────────────────────────▶ │  ┌────────────────────┐  │
└─────────────────┘                             │  │   Query 영역        │  │
                                                 │  │  (MyBatis, 집계쿼리) │  │
                     Kafka Topic                │  └────────────────────┘  │
                 purchase.order.result           │                          │
              ◀───────────────────────────────  │                          │
                  (성공/실패 결과 회신)             └──────────────────────────┘
```

**패키지 구조 (DDD + CQRS 스타일)**
```
com.teamgold.goldenharvest.domain.inventory
├── command                              # 쓰기 모델 (JPA)
│   ├── application
│   │   ├── controller                   # Command 전용 컨트롤러 (폐기, 가격정책 등)
│   │   ├── service                      # 도메인 서비스 (InboundService, LotService, OutboundService, DiscardService, PricePolicyService)
│   │   ├── event                        # Kafka Consumer / Spring EventListener / 이벤트 DTO
│   │   └── dto
│   ├── domain
│   │   ├── lot                          # Lot, Inbound, Outbound, PricePolicy 애그리거트
│   │   ├── discard                      # Discard, DiscardStatus
│   │   └── mirror                       # ItemMasterMirror (타 서비스 데이터 미러)
│   └── infrastructure                   # Repository, IdGenerator 등
└── query                                # 읽기 모델 (MyBatis)
    ├── controller                       # 조회/통계 전용 컨트롤러
    ├── service
    ├── mapper                           # @Mapper 기반 SQL 매퍼
    └── dto
```

---

<h3 style="font-size: 1.2rem; font-weight: 700; margin-bottom: 1rem;">💡 핵심 기술 포인트</h3>

#### 1. MSA 이벤트 기반 통신 & Event Carried State Transfer

**문제 상황**: 상품 정보(상품명, 등급, 원가 등)는 별도의 "상품 서비스"가 소유합니다. 재고 서비스가 조회 API마다 상품 서비스에 동기(REST) 호출을 하면 상품 서비스 장애 시 재고 서비스도 연쇄 장애가 발생하고, 네트워크 왕복 비용으로 조회 성능이 저하되며, 서비스 간 강결합이 발생합니다.

**해결**: 상품 서비스에서 데이터가 변경될 때 이벤트를 발행하고, 재고 서비스는 이를 구독해 필요한 데이터만 로컬 DB에 복제(Mirror)해 둡니다. 이후 조회는 전부 로컬 DB에서 처리합니다.

```java
@Component
@RequiredArgsConstructor
@Slf4j
public class MasterDataUpdateEventListener {

    private final ItemMasterMirrorService itemMasterMirrorService;

    // Master Data의 update를 listen하여 snapshot을 저장하는 event 기반 처리 메소드이다
    @Async
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void updateItemMasterMirror(ItemMasterUpdatedEvent itemMasterUpdatedEvent) {
        log.info("마스터데이터 업데이트 이벤트 수신 완료.");
        itemMasterMirrorService.updateItemMasterMirror(itemMasterUpdatedEvent);
        log.info("마스터데이터 mirror 업데이트 완료");
    }

    @Async
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void updateItemPrice(ItemOriginPriceUpdatedEvent itemOriginPriceUpdateEvent) {
        log.info("원가 업데이트 이벤트 수신 완료.");
        itemMasterMirrorService.updateOriginPrice(itemOriginPriceUpdateEvent);
        log.info("원가 업데이트 이벤트 처리 완료");
    }
}
```

```java
@Service
@RequiredArgsConstructor
public class ItemMasterMirrorService {

    private final ItemMasterMirrorRepository itemMasterMirrorRepository;

    // 상품 마스터 정보 변경 이벤트 수신 → 로컬 미러 테이블 갱신
    public void updateItemMasterMirror(ItemMasterUpdatedEvent itemMasterUpdatedEvent) {
        ItemMasterMirror itemMasterMirror = ItemMasterMirror.builder()
                .skuNo(itemMasterUpdatedEvent.skuNo())
                .itemName(itemMasterUpdatedEvent.itemName())
                .gradeName(itemMasterUpdatedEvent.gradeName())
                .varietyName(itemMasterUpdatedEvent.varietyName())
                .baseUnit(itemMasterUpdatedEvent.baseUnit())
                .isActive(itemMasterUpdatedEvent.isActive())
                .fileUrl(itemMasterUpdatedEvent.fileUrl())
                .build();

        itemMasterMirrorRepository.save(itemMasterMirror);
    }

    // 원가 변경 이벤트 수신 → 원가만 별도 갱신
    public void updateOriginPrice(ItemOriginPriceUpdatedEvent itemOriginPriceUpdateEvent) {
        List<ItemMasterMirror> itemMasterMirrors = itemMasterMirrorRepository.findBySkuNo(
            itemOriginPriceUpdateEvent.skuNo());

        if (itemMasterMirrors.size() != 1) {
            throw new BusinessException(ErrorCode.INVALID_REQUEST);
        }

        ItemMasterMirror itemMasterMirror = itemMasterMirrors.getFirst();
        itemMasterMirror.updatePrice(itemOriginPriceUpdateEvent.originPrice());
    }
}
```

> 각 마이크로서비스가 자신의 도메인 데이터에만 쓰기 권한(Single Source of Truth)을 갖고, 타 서비스 데이터가 필요할 때는 이벤트로 전달받아 로컬에 읽기 전용으로 복제하는 전략을 적용해, 서비스 간 결합도를 낮추고 조회 성능을 확보했습니다.

<br>

#### 2. CQRS — Command(JPA) / Query(MyBatis) 분리

Command(쓰기)와 Query(읽기)를 패키지 레벨부터 물리적으로 분리했습니다.

| 구분 | 기술 | 이유 |
|---|---|---|
| Command | Spring Data JPA + 도메인 엔티티 | 재고 차감, 상태 전이 등 비즈니스 규칙을 엔티티에 캡슐화하고 트랜잭션 일관성 보장 |
| Query | MyBatis (`@Mapper` + 네이티브 SQL) | 다중 테이블 조인, 동적 필터링, 집계 등 복잡한 조회를 SQL 레벨에서 직접 최적화 |

**Query 쪽 — 판매가 계산까지 포함한 재고 조회 쿼리**
```java
@Mapper
public interface LotMapper {

    @Select("""
        SELECT
            l.sku_no AS skuNo,
            SUM(l.quantity) AS quantity,
            i.item_name AS itemName,
            i.grade_name AS gradeName,
            i.variety_name AS varietyName,
            i.base_unit AS baseUnit,
            ROUND(i.current_origin_price * COALESCE(1 + p.margin_rate, 1.2), 0) AS customerPrice,
            i.file_url AS fileUrl
        FROM tb_lot l
        JOIN tb_item_master_mirror i ON l.sku_no = i.sku_no
        LEFT JOIN tb_price_policy p ON l.sku_no = p.sku_no
        WHERE l.lot_status = 'AVAILABLE'
          AND (l.sku_no = #{skuNo} OR #{skuNo} IS NULL)
          AND (i.item_name LIKE CONCAT('%', #{itemName}, '%') OR #{itemName} IS NULL)
        GROUP BY l.sku_no, i.item_name, i.grade_name, i.variety_name,
                 i.base_unit, i.current_origin_price, p.margin_rate
        ORDER BY i.item_name, l.sku_no
        LIMIT #{limit} OFFSET #{offset}
    """)
    List<AvailableItemResponse> findAllAvailableItems(
            @Param("limit") int limit, @Param("offset") int offset,
            @Param("skuNo") String skuNo, @Param("itemName") String itemName);
}
```
원가에 마진율(margin_rate)을 적용해 고객 판매가를 쿼리 단계에서 즉시 계산하고, 가격 정책이 없는 상품은 기본 마진율(1.2배)을 적용하는 `COALESCE` 처리까지 SQL에서 해결했습니다.

**Command 쪽 — 도메인 규칙이 캡슐화된 애그리거트**
```java
@Entity
@Table(name = "tb_lot")
public class Lot {
    // ...
    @Version
    private Long version;   // 낙관적 락

    public enum LotStatus { AVAILABLE, ALLOCATED, DEPLETED, DISCARDED }

    public Integer consumeQuantity(Integer quantity) {
        int actualConsume = Math.min(this.quantity, quantity);
        this.quantity -= actualConsume;

        if (this.quantity < 0) {
            throw new BusinessException(ErrorCode.INSUFFICIENT_STOCK);
        }
        if (this.quantity == 0) {
            this.lotStatus = LotStatus.DEPLETED;
        }
        return actualConsume; // 실제로 이 Lot에서 차감된 수량 반환
    }
}
```

> 쓰기 로직은 JPA 엔티티에 도메인 규칙(재고 차감 시 상태 자동 전이, 수량 검증)을 캡슐화해 응집도를 높였고, 조회 로직은 MyBatis로 복잡한 조인·동적 조건·계산식을 SQL 레벨에서 직접 다뤄 성능과 가독성을 모두 확보하는 CQRS 아키텍처를 적용했습니다.

<br>

#### 3. Kafka 멱등성 처리 — Bloom Filter + DB 이중 방어

**문제 상황**: Kafka는 "적어도 한 번(At-Least-Once)" 전달을 보장하므로, 네트워크 재시도·리밸런싱 등으로 동일 이벤트가 중복 수신될 수 있습니다. 구매 주문 이벤트가 중복 처리되면 재고가 실제보다 많이 입고되는 심각한 데이터 오류가 발생합니다.

```java
@Component
@Slf4j
@RequiredArgsConstructor
public class PurchaseOrderConsumer {

    private final ApplicationEventPublisher eventPublisher;
    private final InboundRepository inboundRepository;
    private final BloomFilterManager filter;

    @Transactional
    @KafkaListener(topics = "purchase.order.created", groupId = "golden.harvest.inventory.processor")
    public void consume(PurchaseOrderCreatedEvent event) {
        log.info("purchase.order.created event consuming");

        // 1차: Bloom Filter로 빠르게 "처음 보는 요청인지" 확률적으로 판별
        if (!filter.isFirstRequest("purchase.order.created", event.purchaseOrderId())) {
            // 2차: Bloom Filter가 "중복일 수 있음"이라 판단한 경우에만 DB 조회로 확정 검증
            if (Objects.nonNull(inboundRepository.findByPurchaseOrderItemId(event.purchaseOrderId()))) {
                log.info("구매 주문 중복 감지됨");
                return;
            }
        }

        eventPublisher.publishEvent(event);
    }
}
```

**설계 의도**
- Bloom Filter는 False Positive는 있어도 False Negative는 없는 확률적 자료구조 → "처음 요청"이라 판단되면 반드시 처음이 맞고, "중복일 수 있다"고 판단되면 그때만 비용이 더 드는 DB 조회로 재확인
- 매 요청마다 DB를 조회하는 대신, 대부분의 정상(비중복) 트래픽은 Bloom Filter만으로 빠르게 통과시켜 DB 부하를 줄이는 최적화
- `InboundService`에서도 한 번 더 `purchaseOrderItemId` 유니크 검증을 하는 다층 방어(Defense in Depth) 구조

```java
@Transactional
public String processInbound(PurchaseOrderCreatedEvent purchaseOrderEvent) {
    if (inboundRepository.findByPurchaseOrderItemId(purchaseOrderEvent.purchaseOrderId()).isPresent()) {
        throw new BusinessException(ErrorCode.DUPLICATE_REQUEST);
    }
    // ...
}
```

> Kafka의 At-Least-Once 전달 특성으로 인한 메시지 중복 처리 문제를 해결하기 위해, Bloom Filter로 대부분의 트래픽을 저비용으로 필터링하고, 의심스러운 케이스만 DB 조회로 재확인하는 2단계 멱등성 처리 전략을 설계했습니다.

<br>

#### 4. 실패 처리 전략 — 재시도 가능 여부 분기 & 보상 이벤트

**문제 상황**: Kafka Consumer가 이벤트 처리 중 예외를 만났을 때, 일시적 오류(DB 커넥션 순단 등)와 영구적 오류(유효성 검증 실패 등)를 구분하지 않고 무조건 재시도하면, 영구적 오류 건이 무한 재시도되며 리소스를 낭비합니다.

```java
@Component
@RequiredArgsConstructor
@Slf4j
public class PurchaseOrderEventListener {

    private final InboundService inboundService;
    private final KafkaProducerHelper producer;

    /* PurchaseOrderConsumer에서 trigger한 이벤트를 받아 처리한다
     * 실패시 retry 및 보상 트랜잭션/이벤트 발행을 구현한다
     */
    @EventListener
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void handlePurchaseOrder(@Valid PurchaseOrderCreatedEvent purchaseOrderCreatedEvent) {
        log.info("구매 주문 이벤트 수신 완료. 객체: {}", purchaseOrderCreatedEvent);

        try {
            String createdLotNo = inboundService.processInbound(purchaseOrderCreatedEvent);

            producer.send(
                    "purchase.order.result",
                    purchaseOrderCreatedEvent.purchaseOrderId(),
                    PurchaseOrderResultEvent.success(purchaseOrderCreatedEvent.purchaseOrderId()),
                    null
            );

            log.info("구매 주문 이벤트 처리 성공. Lot 번호: {}", createdLotNo);

        }
        catch (Exception e) {
            log.error("구매 주문 이벤트 처리 중 오류 발생: {}", e.getMessage());

            if (RetryableExceptions.isRetryable(e)) {
                log.warn("Retryable 오류 발생, kafka retry trigger.");
                throw e;
            }
            else {
                log.error("Retry 불가능 오류 발생, 보상 이벤트 발행 및 롤백 시작.");

                producer.send(
                        "purchase.order.result",
                        purchaseOrderCreatedEvent.purchaseOrderId(),
                        PurchaseOrderResultEvent.fail(purchaseOrderCreatedEvent.purchaseOrderId()),
                        null
                );

                TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();

                log.warn("구매 주문 이벤트 처리 실패 트랜잭션 롤백 및 이벤트 발행 완료.");
            }
        }
    }
}
```

```java
public record PurchaseOrderResultEvent(String purchaseOrderId, String status) {
    public static PurchaseOrderResultEvent success(String purchaseOrderId) {
        return new PurchaseOrderResultEvent(purchaseOrderId, "success");
    }
    public static PurchaseOrderResultEvent fail(String purchaseOrderId) {
        return new PurchaseOrderResultEvent(purchaseOrderId, "fail");
    }
}
```

**설계 의도 — 분산 트랜잭션을 2PC 없이 해결**: 서로 다른 서비스(구매주문 서비스 ↔ 재고 서비스)에 걸친 트랜잭션을 하나로 묶는 대신, 각 서비스가 자기 트랜잭션만 책임지고 결과를 이벤트로 상대에게 알려주는 방식(Saga 패턴의 축소 구현)을 택했습니다. 재시도 가능한 오류는 Kafka Consumer의 재시도 메커니즘에 위임(가용성 우선)하고, 재시도 불가능한 오류는 즉시 실패를 알리는 보상 이벤트를 발행해 구매주문 서비스가 주문 상태를 롤백하거나 사용자에게 안내할 수 있도록 했습니다.

<br>

#### 5. 동시성 제어 — 비관적 락 & 낙관적 락 차등 적용

**문제 상황**: 농산물 특성상 인기 상품은 짧은 시간에 여러 주문이 몰릴 수 있습니다. 동시에 여러 트랜잭션이 같은 SKU의 재고를 조회·차감하면 Race Condition으로 실제 재고보다 많이 판매되는 초과 판매(Overselling)가 발생할 수 있습니다.

```java
@Repository
public interface LotRepository extends JpaRepository<Lot, String> {

    // 여러 주문이 동시에 같은 SKU를 소진하려 할 때 -> 비관적 쓰기 락으로 행 잠금
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    List<Lot> findBySkuNoAndLotStatusOrderByInboundDateAsc(String skuNo, Lot.LotStatus lotStatus);

    // 단건 조회(폐기 처리 등) -> 낙관적 락으로 가벼운 충돌 감지
    @Lock(LockModeType.OPTIMISTIC)
    List<Lot> findByLotNo(String lotNo);
}
```

| 시나리오 | 충돌 빈도 | 적용 락 | 이유 |
|---|---|---|---|
| 여러 주문이 동시에 같은 SKU 재고를 FIFO로 소진 | 높음 | `PESSIMISTIC_WRITE` | 조회 시점에 행을 잠가, 대기 중인 다른 트랜잭션이 갱신되기 전 값을 읽지 못하게 원천 차단 |
| 관리자가 개별 Lot을 폐기 처리 | 낮음 | `OPTIMISTIC` (`@Version`) | 충돌이 드문 단건 작업에 굳이 잠금 비용을 들이지 않고, 충돌 시에만 예외로 감지 |

> 재고 차감처럼 쓰기 경합이 잦은 로직에는 비관적 락(PESSIMISTIC_WRITE)으로 동시 접근을 원천 차단하고, 상대적으로 충돌 빈도가 낮은 단건 조회·수정에는 낙관적 락(@Version)을 적용해, 동시성 제어 비용과 안전성 사이의 균형을 맞췄습니다.

<br>

#### 6. FIFO 기반 재고 소진 로직

농산물은 신선도가 중요하므로, 먼저 입고된 재고부터 먼저 출고되어야 합니다.

```java
@Service
@RequiredArgsConstructor
public class LotService {

    private final LotRepository lotRepository;
    private final OutboundService outboundService;

    @Transactional
    public void consumeLot(SalesOrderCreatedEvent salesOrderEvent) {
        String skuNo = salesOrderEvent.skuNo();
        int quantity = salesOrderEvent.quantity();

        // 입고일 오름차순 정렬 -> 가장 오래된 재고부터 조회 (FIFO)
        List<Lot> availableItems = lotRepository.findBySkuNoAndLotStatusOrderByInboundDateAsc(
            skuNo, Lot.LotStatus.AVAILABLE);

        // 사전에 재고 총량을 검증해 부족하면 즉시 실패
        if (availableItems.stream().mapToInt(Lot::getQuantity).sum() < quantity) {
            throw new BusinessException(ErrorCode.INSUFFICIENT_STOCK);
        }

        int remainingQuantity = quantity;
        for (Lot item : availableItems) {
            if (remainingQuantity <= 0) break;

            // 한 Lot으로 부족하면 다음 오래된 Lot으로 넘어가며 분할 소진
            int consumed = item.consumeQuantity(remainingQuantity);
            remainingQuantity -= consumed;

            // 소진된 만큼 개별 출고 레코드 생성 -> 재고 이동 이력 추적 가능
            outboundService.processOutbound(item, consumed, salesOrderEvent);
        }
    }
}
```

- 단일 Lot의 재고가 부족하면 여러 Lot에 걸쳐 자동으로 분할 소진
- 소진된 각 Lot마다 별도의 `Outbound`(출고) 레코드를 생성해, 어느 Lot에서 얼마나 나갔는지 완전한 이력 추적이 가능
- 재고 총량 사전 검증으로 불필요한 부분 처리 후 롤백을 방지

> 신선식품 유통에서 중요한 선입선출(FIFO) 원칙을 도메인 로직으로 구현했습니다. 하나의 주문이 여러 Lot에 걸쳐 분할 소진될 수 있는 구조를 설계하고, 각 소진 단위마다 출고 이력을 남겨 재고 흐름을 완전히 추적할 수 있도록 했습니다.

<br>

#### 7. 도메인 모델 캡슐화 — 폐기(Discard) 처리

여러 애그리거트를 조합해 하나의 비즈니스 트랜잭션을 완성하는 예시입니다.

```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class DiscardService {

    private final DiscardRepository discardRepository;
    private final DiscardStatusRepository discardStatusRepository;
    private final LotRepository lotRepository;
    private final ItemMasterMirrorRepository itemMasterMirrorRepository;

    @Transactional
    public String discardItem(DiscardItemRequest discardItemRequest, Jwt jwt) {
        Lot lot = lotRepository.findByLotNo(discardItemRequest.getLotNo()).getFirst();
        if (Objects.isNull(lot)) {
            throw new BusinessException(ErrorCode.LOT_NOT_FOUND);
        }

        lot.consumeQuantity(discardItemRequest.getQuantity());  // Lot 애그리거트 스스로 상태 변경

        String discardId = IdGenerator.createId("DIS");
        DiscardStatus status = discardStatusRepository.findByDiscardStatus(discardItemRequest.getDiscardStatus())
            .orElseThrow(() -> new BusinessException(ErrorCode.INVALID_DISCARD_STATUS));

        ItemMasterMirror itemMaster = itemMasterMirrorRepository.findById(lot.getSkuNo())
            .orElseThrow(() -> new BusinessException(ErrorCode.NO_SUCH_SKU));

        Discard discard = Discard.builder()
            .discardId(discardId)
            .lotNo(lot.getLotNo())
            .discardStatus(status)
            .quantity(discardItemRequest.getQuantity())
            .discardedAt(LocalDateTime.now())
            .approvedBy(jwt.getSubject())      // JWT에서 승인자 정보 자동 추출
            .discardRate(BigDecimal.valueOf(discardItemRequest.getQuantity() / lot.getQuantity()))
            .totalPrice(itemMaster.getCurrentOriginPrice()
                .multiply(BigDecimal.valueOf(discardItemRequest.getQuantity())))
            .build();

        return discardRepository.save(discard).getLotNo();
    }
}
```

**ID 생성 전략 — 도메인 접두사 + 날짜 + 랜덤 시퀀스**
```java
public class IdGenerator {
    public static String createId(String type) {
        String sequenceStr = UUID.randomUUID().toString().replace("-", "").substring(0, 6).toUpperCase();
        String formattedType = type.substring(0, Math.min(3, type.length())).toUpperCase();
        String today = LocalDate.now().format(DateTimeFormatter.ofPattern("yyyyMMdd"));
        return formattedType + "_" + today + "_" + sequenceStr;  // 예: LOT_20250115_A1B2C3
    }
}
```
Lot, Inbound, Outbound, Discard 등 여러 애그리거트가 공통으로 사용하는 가독성 있는 비즈니스 ID 체계(자동 증가 PK 대신 타입+날짜+식별자를 조합)를 설계했습니다.

<br>

#### 8. 권한 기반 API 설계 & 감사 추적

OAuth2 Resource Server 기반 JWT 인증 + `@PreAuthorize`로 세밀한 권한 제어를 적용했습니다.

```java
@RestController
@RequestMapping("/api")
@RequiredArgsConstructor
public class DiscardController {

    private final DiscardService discardService;

    @PostMapping("/discard")
    @PreAuthorize("hasRole('ADMIN')")   // 폐기 승인은 관리자만 가능
    public ResponseEntity<ApiResponse<?>> discardItem(
            @RequestBody @Validated DiscardItemRequest discardItemRequest,
            @AuthenticationPrincipal Jwt jwt) {
        return ResponseEntity.ok(ApiResponse.success(discardService.discardItem(discardItemRequest, jwt)));
    }
}
```

일반 사용자는 `/api/items`(판매 가능 재고만), 관리자는 `/api/admin/items`(전체 이력 포함)로 API 레벨에서 노출 범위를 분리했습니다. 폐기 승인 시에는 `jwt.getSubject()`로 승인자를 자동 기록(`approvedBy`)해 감사 추적(Audit Trail)이 가능하도록 설계했습니다.

> OAuth2 Resource Server 기반 JWT 인증과 Role 기반 API 접근 제어(RBAC)를 일관되게 적용했고, 폐기 승인처럼 민감한 액션에는 요청자 정보를 토큰에서 추출해 자동으로 기록함으로써 감사 추적성을 확보했습니다.

<br>

#### 9. 통계/대시보드용 집계 쿼리

단순 CRUD를 넘어, 관리자가 재고 손실을 정량적으로 모니터링할 수 있는 통계 API를 제공합니다.

```java
@Select("""
    SELECT
        COALESCE(SUM(CASE WHEN discarded_at BETWEEN #{thisMonthStart} AND #{now}
                          THEN quantity ELSE 0 END), 0) AS currentQuantity,
        COALESCE(SUM(CASE WHEN discarded_at BETWEEN #{lastMonthStart} AND #{lastMonthUntilNow}
                          THEN quantity ELSE 0 END), 0) AS lastQuantity
    FROM tb_discard
    WHERE discarded_at BETWEEN #{lastMonthStart} AND #{now}
""")
DiscardVolumeResponse findDiscardVolume(
    @Param("lastMonthStart") LocalDateTime lastMonthStart,
    @Param("lastMonthUntilNow") LocalDateTime lastMonthUntilNow,
    @Param("thisMonthStart") LocalDateTime thisMonthStart,
    @Param("now") LocalDateTime now
);
```
이번 달 vs 지난 달 폐기량을 한 번의 쿼리로 비교 집계했습니다(CASE WHEN을 활용한 조건부 집계). 이 외에도 상품별 폐기율, 폐기 손실 금액 등을 SQL 집계 쿼리로 직접 설계해 재고 손실을 정량적으로 파악할 수 있도록 했습니다.

---

<h3 style="font-size: 1.2rem; font-weight: 700; margin-bottom: 1rem;">📋 API 명세 요약</h3>

| 구분 | Method | Endpoint | 권한 | 설명 |
|---|---|---|---|---|
| 재고 조회 | GET | `/api/items` | USER | 판매 가능 재고 목록 (SKU/상품명 필터) |
| 재고 조회 | GET | `/api/admin/items` | ADMIN | 과거 이력 포함 전체 재고 조회 |
| 재고 조회 | GET | `/api/items/metrics` | - | Lot 관련 핵심 지표 |
| 입출고 이력 | GET | `/api/inbound` | ADMIN | 입고 이력 조회 |
| 입출고 이력 | GET | `/api/outbound` | ADMIN | 출고 이력 조회 |
| 폐기 | POST | `/api/discard` | ADMIN | 재고 폐기 처리 |
| 폐기 | GET | `/api/discard/list` | ADMIN | 폐기 이력 목록 |
| 폐기 | GET | `/api/discard/volume` | ADMIN | 월별 폐기량 비교 |
| 폐기 | GET | `/api/discard/loss` | ADMIN | 폐기 손실 금액 |
| 폐기 | GET | `/api/discard/ratio-by-item` | ADMIN | 상품별 폐기율 |
| 가격 정책 | POST | `/api/price-policy` | ADMIN | 마진율 기반 가격 정책 등록 |
| 가격 정책 | PATCH | `/api/price-policy` | ADMIN | 가격 정책 수정 |
| 가격 정책 | GET | `/api/price-policy` | ADMIN | 가격 정책 목록 |

---

<h3 style="font-size: 1.2rem; font-weight: 700; margin-bottom: 1rem;">🔧 트러블슈팅</h3>

<div style="display: flex; flex-direction: column; gap: 1rem; margin-bottom: 2rem;">

  <div style="border-left: 3px solid #E67E22; padding-left: 1rem;">
    <span style="font-weight: 700;">Kafka 메시지 중복 소비로 인한 재고 이중 입고</span>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.3rem;">
      동일한 이벤트가 재처리되어 같은 구매 건에 대해 Lot이 중복 생성되는 현상이 있었습니다. Kafka Consumer의 재시도 과정에서
      커밋되지 않은 오프셋이 재전달되어 같은 요청이 중복 처리되는 것이 원인(At-Least-Once 특성)이었고, Bloom Filter로
      1차 필터링 후 의심 케이스만 <code>purchaseOrderItemId</code> 유니크 검증으로 DB 단계에서 2차 확인하는 순차적
      이중 방어 구조를 채택해 중복 이벤트로 인한 재고 정합성 오류를 방지했습니다.
    </p>
  </div>

  <div style="border-left: 3px solid #E67E22; padding-left: 1rem;">
    <span style="font-weight: 700;">LOT 재고 관리 내부 로직 오류</span>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.3rem;">
      특정 재고에 대한 상세 정보 불러오기가 가끔 오류를 일으키는 현상(<code>NoSuchElementException</code>)을 발견했습니다.
      <code>LotRepository</code>의 <code>findByLotNo</code>가 적절한 반환값을 가지지 않아, <code>DiscardService</code>에서
      빈 리스트에 <code>getFirst()</code>를 호출하는 버그였고, 빈 리스트를 선행적으로 확인하는 로직을 추가해 해결했습니다.
      조금 더 정교한 테스트 코드 작성의 필요성과, 눈으로 보기에 문제없어 보이던 코드도 언제든지 버그를 일으킬 수 있다는
      경각심을 다시 한번 갖게 된 경험이었습니다.
    </p>
  </div>

</div>

---

<h3 style="font-size: 1.2rem; font-weight: 700; margin-bottom: 1rem;">🤔 기술적 고민과 트레이드오프</h3>

<table style="width: 100%; border-collapse: collapse; margin-bottom: 2rem;">
  <thead>
    <tr style="border-bottom: 2px solid #333;">
      <th style="padding: 8px; text-align: left;">고민</th>
      <th style="padding: 8px; text-align: left;">선택지 A</th>
      <th style="padding: 8px; text-align: left;">선택지 B</th>
      <th style="padding: 8px; text-align: left;">최종 선택 & 이유</th>
    </tr>
  </thead>
  <tbody>
    <tr style="border-bottom: 1px solid #eee;">
      <td style="padding: 8px;">서비스 간 데이터 참조</td>
      <td style="padding: 8px;">매 요청마다 REST 동기 호출</td>
      <td style="padding: 8px;">이벤트 기반 로컬 미러링</td>
      <td style="padding: 8px;"><b>B</b> — 장애 전파 차단 및 조회 성능 확보. 대신 최종 일관성을 감수</td>
    </tr>
    <tr style="border-bottom: 1px solid #eee;">
      <td style="padding: 8px;">분산 트랜잭션 처리</td>
      <td style="padding: 8px;">2PC / 분산락 기반 강한 일관성</td>
      <td style="padding: 8px;">이벤트 기반 보상 트랜잭션</td>
      <td style="padding: 8px;"><b>B</b> — 강결합·가용성 저하를 피하고 최종 일관성을 택함</td>
    </tr>
    <tr style="border-bottom: 1px solid #eee;">
      <td style="padding: 8px;">동시성 제어</td>
      <td style="padding: 8px;">전체를 비관적 락으로 통일</td>
      <td style="padding: 8px;">시나리오별 락 전략 차등 적용</td>
      <td style="padding: 8px;"><b>B</b> — 불필요한 대기 비용 방지. 다만 락 전략 혼재로 인한 복잡도는 트레이드오프</td>
    </tr>
    <tr style="border-bottom: 1px solid #eee;">
      <td style="padding: 8px;">조회 성능 최적화</td>
      <td style="padding: 8px;">JPA만으로 모든 조회 구현</td>
      <td style="padding: 8px;">CQRS(JPA+MyBatis 이원화)</td>
      <td style="padding: 8px;"><b>B</b> — 통계/조인 쿼리는 네이티브 SQL 제어가 유리. 대신 두 기술 유지보수 비용 발생</td>
    </tr>
  </tbody>
</table>

---

<h3 style="font-size: 1.2rem; font-weight: 700; margin-bottom: 1rem;">💭 배운 점 & 회고</h3>

<p style="font-size: 0.95rem; line-height: 1.8; margin-bottom: 1rem;">
  MSA 환경에서 데이터의 주인을 정의하는 것과 각 서비스 사이의 영역을 구분짓는 것에 대한 중요성을 체감했습니다.
  Kafka 기반 이벤트 통신에서 멱등성·순서 보장·실패 처리를 고려하지 않으면 데이터 정합성 문제로 바로 이어지므로
  고려해야 할 사항이 많음을 느꼈고, 동시성 문제는 이론과 실제로 락 전략을 선택하고 근거를 설명하는 것 사이에
  큰 차이가 있음을 경험했습니다.
</p>

<p style="font-size: 0.95rem; line-height: 1.8; margin-bottom: 2rem;">
  프로젝트 아키텍처로 모놀리식 혹은 모듈러 모놀리스 구조도 고려할 수 있었지만, 굳이 MSA를 채택한 것은 더 깊은
  기술적 과제와 고민을 해보기 위함이었습니다. 실제로 여러 마이크로서비스로 쪼개는 것이 생각했던 것보다도 더
  높은 난이도와 고려해야 할 사항이 많았기 때문에, 저와 팀원 모두 일반적인 프로젝트보다 더 많은 시간과 노력이
  필요했습니다. 그럼에도 불구하고 본래 목적이었던 더 심도 있는 기술적 고민을 해보고 더 정교한 구조를 직접
  설계해보는 경험을 달성하여서, 더 뜻깊었던 프로젝트였습니다.
</p>
