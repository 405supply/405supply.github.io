---
layout: page
title: Marine Log
description: 해안 정화 활동 기록 및 통계 플랫폼
img: # 나중에 추가
importance: 2
category: Team Project
github: https://github.com/Marine-Log/Marine-Log-backend
---

<div style="margin-bottom: 2rem;">
  <img src="https://img.shields.io/badge/Java-007396?style=flat-square&logo=openjdk&logoColor=white"/>
  <img src="https://img.shields.io/badge/Spring Boot-6DB33F?style=flat-square&logo=springboot&logoColor=white"/>
  <img src="https://img.shields.io/badge/Spring Security-6DB33F?style=flat-square&logo=springsecurity&logoColor=white"/>
  <img src="https://img.shields.io/badge/Spring Statemachine-6DB33F?style=flat-square&logo=springboot&logoColor=white"/>
  <img src="https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white"/>
  <img src="https://img.shields.io/badge/PostGIS-4169E1?style=flat-square&logo=postgresql&logoColor=white"/>
  <img src="https://img.shields.io/badge/Redis-FF4438?style=flat-square&logo=redis&logoColor=white"/>
  <img src="https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white"/>
  <img src="https://img.shields.io/badge/AWS-232F3E?style=flat-square&logo=amazonwebservices&logoColor=white"/>
</div>

---

<h3 style="font-size: 1.2rem; font-weight: 700; margin-bottom: 1rem;">📌 프로젝트 개요</h3>

<table style="width: 100%; border-collapse: collapse; margin-bottom: 2rem;">
  <tbody>
    <tr style="border-bottom: 1px solid #eee;">
      <td style="padding: 8px; font-weight: 600; width: 140px;">기간</td>
      <td style="padding: 8px;">2025.09 ~ 2025.12 (3개월)</td>
    </tr>
    <tr style="border-bottom: 1px solid #eee;">
      <td style="padding: 8px; font-weight: 600;">유형</td>
      <td style="padding: 8px;">2025년 제15회 ICTCoC 피우다 프로젝트</td>
    </tr>
    <tr style="border-bottom: 1px solid #eee;">
      <td style="padding: 8px; font-weight: 600;">팀 구성</td>
      <td style="padding: 8px;">4인 (백엔드 2명, 프론트엔드 1명, 팀장/기획 1명)</td>
    </tr>
    <tr style="border-bottom: 1px solid #eee;">
      <td style="padding: 8px; font-weight: 600;">성과</td>
      <td style="padding: 8px;">🏆 최종 결선 15팀 선정</td>
    </tr>
    <tr style="border-bottom: 1px solid #eee;">
      <td style="padding: 8px; font-weight: 600;">GitHub</td>
      <td style="padding: 8px;">
        <a href="https://github.com/Marine-Log/Marine-Log-backend" target="_blank">바로가기</a>
      </td>
    </tr>
  </tbody>
</table>

**문제의식**: 해안 정화 활동은 대부분 개별 단체가 수기(엑셀, 종이, 카카오톡)로 기록해왔습니다. 이로 인해 어디서, 얼마나, 어떤 쓰레기가 수거되는지 정량적 데이터로 축적되지 않고, 여러 단체가 같은 지역에서 중복 활동을 하거나 서로 협업할 기회를 놓치며, 활동 실적을 대외적으로 증명(후원 유치, 지자체 협업 등)하기 어려운 문제가 있었습니다.

**한 줄 요약**: 정화 활동의 생명주기를 State Machine으로 모델링하고, PostGIS 기반 공간 데이터 처리로 활동 위치·밀집도를 시각화하며, 여러 단체가 협업하는 연합 활동 기능까지 지원하는 백엔드 시스템을 설계·구현했습니다. Marine Log는 정화 활동의 **제보 → 검증(답사) → 참여자 모집 → 실행 → 완료**라는 전체 흐름을 시스템화하고, 지리 정보 기반으로 활동 데이터를 시각화해 이 문제를 해결하고자 했습니다.

**담당 역할**: 위치 기반 로직(PostGIS 연동, 좌표-행정구역 변환) · 대시보드 통계 API(히트맵, 행정구역별 집계) · CSV 내보내기 · S3 사진 업로드 · OAuth2 소셜 로그인(Google/Naver/Kakao) 및 JWT/Redis 인증 체계

---

<h3 style="font-size: 1.2rem; font-weight: 700; margin-bottom: 1rem;">🏗️ 시스템 아키텍처</h3>

**활동(Activity) 상태 전이 다이어그램**
```
                    ┌───────────┐
        ┌──────────▶│  REJECTED │ (반려)
        │           └───────────┘
        │ REJECT
┌────────────┐  APPROVE   ┌───────────┐  START_SURVEY   ┌────────────┐
│  REPORTED  │───────────▶│  APPROVED │───────────────▶│  SURVEYING  │
│  (제보)     │            │  (승인)    │                 │  (답사중)    │
└────────────┘            └───────────┘                 └──────┬─────┘
                                                                │ FINISH_SURVEY
                                              REJECT_DURING_    │ (HOST 권한 필요)
                                              SURVEY            ▼
                                         ┌───────────┐   ┌─────────────┐
                                         │ REJECTED  │◀──│ RECRUITING  │
                                         └───────────┘   │  (모집중)     │
                                                          └──────┬──────┘
                                                                 │ START_ACTIVITY
                                                                 ▼
                                                          ┌─────────────┐
                                                          │ IN_PROGRESS │
                                                          │  (진행중)     │
                                                          └──────┬──────┘
                                                                 │ FINISH_ACTIVITY
                                                                 ▼
                                                          ┌─────────────┐
                                                          │    DONE     │
                                                          │  (완료)      │
                                                          └─────────────┘

   ※ CANCEL 이벤트는 모든 상태에서 CANCELED로 전이 가능 (Guard 항상 true)
   ※ COLLAB_REQUEST / COLLAB_APPROVE / COLLAB_REJECT: SURVEYING, RECRUITING 상태에서
      단체 간 연합 활동 요청 흐름을 병행 처리 (self-transition)
```

**패키지 구조**
```
domain.backend
├── activity                    # 활동(정화 기록) CRUD, Draft/Publish 워크플로우
│   ├── controller
│   ├── service
│   ├── domain (Activity, Waste, ActivityPicture)
│   └── dto
├── report                      # 활동 제보(ActivityReport) + State Machine
│   ├── statemachine
│   │   ├── config               # ActivityStateMachineConfig
│   │   ├── guard                # 전이 조건 검증 (Approve/StartSurvey/Collaboration 등)
│   │   └── action                # 전이 시 실행 로직
│   ├── domain
│   │   ├── entity (ActivityReport, Collaboration, ActivityStatusHistory)
│   │   └── enums (ActivityStatus, ActivityEvent, CollaborationStatus)
│   └── service (CollaborationService, ReadActivityReportListService)
├── location                    # PostGIS 기반 위치/공간 통계
│   ├── domain (Location - JTS Point)
│   ├── service
│   └── controller
├── organization                # 단체(Organization), 활동-단체 참여(HostedBy)
├── auth                        # 회원, 소셜 로그인, JWT, 관리자
│   └── domain.service (auth/user/admin)
├── notification                # SSE 기반 실시간 알림
└── common
    ├── config (SecurityConfig, SwaggerConfig)
    ├── security (JWT, OAuth2)
    └── exception (GlobalExceptionHandler)
```

---

<h3 style="font-size: 1.2rem; font-weight: 700; margin-bottom: 1rem;">💡 핵심 기술 포인트</h3>

#### 1. Spring Statemachine 기반 활동 생명주기 관리

**문제 상황**: 정화 활동은 "제보 → 관리자 승인 → 현장 답사 → 참여자 모집 → 실행 → 완료"라는 여러 단계를 거치며, 각 단계마다 누가(권한), 어떤 조건에서, 어떤 전이가 가능한지가 달라집니다. 이를 단순 if/switch 문으로 처리하면 상태·이벤트 조합이 늘어날수록 분기 폭발이 발생하고, 규칙이 서비스 코드 전반에 흩어집니다.

```java
public enum ActivityStatus {
    REPORTED, APPROVED, REJECTED, SURVEYING,
    RECRUITING, IN_PROGRESS, DONE, CANCELED
}

public enum ActivityEvent {
    APPROVE, REJECT, START_SURVEY, FINISH_SURVEY,
    START_RECRUIT, START_ACTIVITY, FINISH_ACTIVITY, CANCEL,
    REJECT_DURING_SURVEY,
    COLLAB_REQUEST, COLLAB_APPROVE, COLLAB_REJECT
}
```

```java
@Configuration
@EnableStateMachineFactory
@RequiredArgsConstructor
public class ActivityStateMachineConfig
        extends StateMachineConfigurerAdapter<ActivityStatus, ActivityEvent> {

    private final ApproveAction approveAction;
    private final RejectAction rejectAction;
    // ... 각 전이별 Action Bean 주입

    private final ApproveGuard approveGuard;
    private final StartSurveyGuard startSurveyGuard;
    // ... 각 전이별 Guard Bean 주입

    @Override
    public void configure(StateMachineTransitionConfigurer<ActivityStatus, ActivityEvent> t) throws Exception {
        t
            // ===== 제보 승인 =====
            .withExternal()
            .source(ActivityStatus.REPORTED)
            .target(ActivityStatus.APPROVED)
            .event(ActivityEvent.APPROVE)
            .guard(approveGuard)
            .action(approveAction)

            // ===== 반려 =====
            .and().withExternal()
            .source(ActivityStatus.REPORTED)
            .target(ActivityStatus.REJECTED)
            .event(ActivityEvent.REJECT)
            .action(rejectAction)

            // ===== 답사 시작 =====
            .and().withExternal()
            .source(ActivityStatus.APPROVED)
            .target(ActivityStatus.SURVEYING)
            .event(ActivityEvent.START_SURVEY)
            .guard(startSurveyGuard)
            .action(startSurveyAction)

            // ===== 답사 완료 → 모집 =====
            .and().withExternal()
            .source(ActivityStatus.SURVEYING)
            .target(ActivityStatus.RECRUITING)
            .event(ActivityEvent.FINISH_SURVEY)
            .guard(finishSurveyGuard)
            .action(finishSurveyAction)

            // ===== 모집 → 진행 =====
            .and().withExternal()
            .source(ActivityStatus.RECRUITING)
            .target(ActivityStatus.IN_PROGRESS)
            .event(ActivityEvent.START_ACTIVITY)
            .action(startActivityAction)

            // ===== 활동 → 완료 =====
            .and().withExternal()
            .source(ActivityStatus.IN_PROGRESS)
            .target(ActivityStatus.DONE)
            .event(ActivityEvent.FINISH_ACTIVITY)
            .action(finishActivityAction)

            // ===== 연합 요청 (SURVEYING 상태 내 self-transition) =====
            .and().withExternal()
            .source(ActivityStatus.SURVEYING)
            .target(ActivityStatus.SURVEYING)
            .event(ActivityEvent.COLLAB_REQUEST)
            .guard(collaborationRequestGuard)
            .action(collaborationRequestAction);
            // ... 이하 연합 승인/거절 전이 생략
    }
}
```

**Guard — 전이 가능 조건 검증 (단일 책임)**
```java
// 답사 시작은 APPROVED 상태이면서, 실제 report가 APPROVED 상태일 때만 허용
@Component
@RequiredArgsConstructor
public class StartSurveyGuard implements Guard<ActivityStatus, ActivityEvent> {

    private final ActivityReportRepository reportRepository;

    @Override
    public boolean evaluate(StateContext<ActivityStatus, ActivityEvent> context) {
        Long reportId = (Long) context.getMessageHeader("reportId");
        ActivityReport report = reportRepository.findById(reportId).orElseThrow();
        return report.getStatus() == ActivityStatus.APPROVED;
    }
}

// 상태 조건뿐 아니라, 요청자의 권한(Role)까지 함께 검증하는 복합 Guard
@Component
public class RejectDuringSurveyGuard implements Guard<ActivityStatus, ActivityEvent> {

    @Override
    public boolean evaluate(StateContext<ActivityStatus, ActivityEvent> context) {
        ActivityStatus state = context.getStateMachine().getState().getId();
        User actor = (User) context.getMessageHeader("user");

        if (state != ActivityStatus.SURVEYING) return false;
        return actor.getRole() == Role.HOST;   // HOST 권한만 답사 중 반려 가능
    }
}

// "어떤 상태에서든 취소 가능"이라는 예외 규칙도 Guard로 명시적 표현
@Component
public class CancelGuard implements Guard<ActivityStatus, ActivityEvent> {
    @Override
    public boolean evaluate(StateContext<ActivityStatus, ActivityEvent> context) {
        return true;  // 모든 상태에서 전이 허용
    }
}
```

**상태 전이 서비스 — 상태머신 로드/전송/영속화**
```java
@Service
@RequiredArgsConstructor
public class ActivityStateMachineService {

    private final StateMachineFactory<ActivityStatus, ActivityEvent> factory;
    private final StateMachinePersist<ActivityStatus, ActivityEvent, Long> persist;

    @Transactional
    public void sendEvent(
            Long reportId,
            ActivityEvent event,
            Object user,
            Consumer<MessageBuilder<ActivityEvent>> headerCustomizer
    ) throws Exception {

        // 1) 상태머신 가져오기
        StateMachine<ActivityStatus, ActivityEvent> sm = factory.getStateMachine(reportId.toString());
        sm.stop();

        // 2) DB의 기존 상태 불러오기
        StateMachineContext<ActivityStatus, ActivityEvent> context = persist.read(reportId);
        sm.getStateMachineAccessor().doWithAllRegions(access -> access.resetStateMachine(context));
        sm.start();

        // 3) Message 생성 (전이에 필요한 컨텍스트를 헤더로 전달)
        MessageBuilder<ActivityEvent> builder = MessageBuilder
                .withPayload(event)
                .setHeader("reportId", reportId)
                .setHeader("user", user);

        if (headerCustomizer != null) headerCustomizer.accept(builder);

        // 4) 이벤트를 동기 방식으로 전송
        boolean accepted = sm.sendEvent(builder.build());
        if (!accepted) {
            log.warn("Event rejected: {} (reportId={})", event, reportId);
        }

        // 5) 전이 후 상태를 DB에 저장
        ActivityStatus newState = sm.getState().getId();
        persist.write(new DefaultStateMachineContext<>(newState, null, null, null), reportId);
    }
}
```

**설계 의도**
- **Guard**: "이 전이가 지금 가능한가?"라는 조건만 담당 (상태 조건, 권한 조건, 도메인 조건 등)
- **Action**: 전이가 승인된 후 "무엇을 할 것인가?"를 담당 (실제 상태 변경, 연관 서비스 호출 등)
- 두 관심사를 분리함으로써 각 컴포넌트가 단일 책임을 가지며, 새로운 전이 규칙 추가 시 기존 코드를 건드리지 않고 새 Guard/Action만 추가하면 됨(개방-폐쇄 원칙)
- `context.getMessageHeader(...)`로 전이에 필요한 컨텍스트(reportId, 요청자 정보 등)를 주고받아, 상태값만으로 판단할 수 없는 복잡한 조건도 처리

> 수기로 관리되던 정화 활동의 진행 단계를 Spring Statemachine으로 모델링해, "어떤 상태에서 어떤 행동이 가능한가"라는 도메인 규칙을 Guard(조건 검증)와 Action(실행 로직)으로 분리하고 선언적 설정으로 관리했습니다. 이를 통해 상태·이벤트 조합이 늘어나도 if-else 분기 폭발 없이 규칙을 확장할 수 있는 구조를 확보했습니다.

<br>

#### 2. 단체 간 연합 활동(Collaboration) 도메인 설계

여러 정화 단체가 같은 지역/활동에 함께 참여하고 싶어도 이를 지원하는 시스템이 없다면 개별적으로 조율해야 합니다. Marine Log는 연합 활동 요청 → 승인/거절이라는 흐름을 공식 기능으로 제공합니다.

```java
@Entity
public class Collaboration extends BaseTimeEntity {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "activity_report_id", nullable = false)
    private ActivityReport activityReport;    // 어떤 제보에 대한 연합인지

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "host_org_id", nullable = false)
    private Organization hostOrg;             // 연합을 요청한 단체

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "target_org_id", nullable = false)
    private Organization targetOrg;           // 요청을 받은 단체

    @Enumerated(EnumType.STRING)
    private CollaborationStatus status;       // REQUESTED / APPROVED / REJECTED

    public void approve() { this.status = CollaborationStatus.APPROVED; }
    public void reject() { this.status = CollaborationStatus.REJECTED; }
}
```

**State Machine과의 통합** — 연합 요청은 답사(SURVEYING) 또는 모집(RECRUITING) 상태에서만 가능하도록 Guard로 제약합니다:
```java
@Component
public class CollaborationRequestGuard implements Guard<ActivityStatus, ActivityEvent> {
    @Override
    public boolean evaluate(StateContext<ActivityStatus, ActivityEvent> context) {
        ActivityStatus state = context.getStateMachine().getState().getId();
        return (state == ActivityStatus.SURVEYING || state == ActivityStatus.RECRUITING);
    }
}
```

> 단일 단체 활동을 넘어, 여러 단체가 협업하는 연합 활동 요청/승인/거절 기능을 별도 도메인(Collaboration)으로 설계했습니다. 활동의 특정 단계(답사·모집 중)에서만 연합 요청이 가능하도록 State Machine의 Guard로 제약해, 비즈니스 규칙과 상태 관리를 일관되게 통합했습니다.

<br>

#### 3. PostGIS + JTS 기반 공간 데이터 처리

**문제 상황**: 활동 위치를 단순히 위도/경도 숫자 컬럼으로 저장하면, "특정 지도 영역 안의 활동만 조회" 같은 공간 쿼리를 하기 위해 매번 애플리케이션 레벨에서 필터링해야 해 비효율적입니다.

```java
@Entity
@Table(name = "location")
public class Location extends BaseTimeEntity {

    @Id
    private Long activityId;

    @Column(columnDefinition = "geometry(Point, 4326)") // PostGIS 전용 geometry 데이터타입 (WGS84 좌표계)
    private Point geometry;

    private String region1depthName;   // 시/도
    private String region2depthName;   // 시/군/구
    private String region3depthName;   // 읍/면/동
    private String address;

    @OneToOne
    @MapsId
    @JoinColumn(name = "activity_id")
    private Activity activity;

    @Builder
    public Location(Activity activity, Double latitude, Double longitude,
                     String region1depthName, String region2depthName,
                     String region3depthName, String address) {
        if (latitude != null && longitude != null) {
            // JTS(Java Topology Suite)로 위경도를 Point 객체로 변환
            GeometryFactory factory = new GeometryFactory(new PrecisionModel(), 4326);
            this.geometry = factory.createPoint(new Coordinate(longitude, latitude));
        }
        this.activity = activity;
        this.region1depthName = region1depthName;
        this.region2depthName = region2depthName;
        this.region3depthName = region3depthName;
        this.address = address;
    }
}
```

```sql
-- schema.sql
CREATE INDEX IF NOT EXISTS idx_location_geom
ON location USING GIST (geometry);
```

**Bounding Box 쿼리 — 지도 화면 영역 내 활동만 조회**
```java
@Query(value = """
    SELECT l.*, a.*
    FROM location l
    JOIN activity a ON a.activity_id = l.activity_id
    WHERE ST_Within(l.geometry, ST_MakeEnvelope(:swLng, :swLat, :neLng, :neLat, 4326))
      AND a.activity_date BETWEEN :startDate AND :endDate
    """, nativeQuery = true)
List<LocationSpecificProjection> getSpecific(
        Double swLat, Double swLng, Double neLat, Double neLng,
        LocalDate startDate, LocalDate endDate, Long orgId);
```

**GIST 인덱스를 쓴 이유**: 일반 B-Tree 인덱스는 다차원 공간 데이터(좌표)의 "포함 관계", "근접 관계"를 효율적으로 인덱싱하지 못합니다. GIST(Generalized Search Tree)는 공간 데이터의 바운딩 박스 기반 트리 구조로, `ST_Within`, `ST_Contains` 같은 공간 연산의 조회 성능을 크게 개선합니다.

> 위치 데이터를 단순 좌표 컬럼이 아닌 PostGIS의 geometry 타입으로 저장하고 JTS로 다뤄, 지도 영역(Bounding Box) 기반 조회나 근접 검색 같은 공간 쿼리를 데이터베이스 레벨에서 효율적으로 처리할 수 있는 구조를 설계했습니다. GIST 공간 인덱스를 적용해 대량의 위치 데이터에서도 조회 성능을 확보했습니다.

<br>

#### 4. 히트맵을 위한 공간 격자 집계 쿼리

지도 위에 정화 활동 밀집도를 히트맵으로 시각화하기 위한 데이터가 필요했습니다.

```sql
-- 좌표를 0.1도 단위 격자로 스냅 후 수거 무게 합산 -> Min-Max 정규화로 1~10 가중치 산출
WITH filtered AS (
    SELECT l.geometry, a.weight_kg
    FROM location l
    JOIN activity a ON a.activity_id = l.activity_id
    WHERE a.activity_date BETWEEN :startDate AND :endDate
      AND a.weight_kg IS NOT NULL
),
grid AS (
    SELECT
        ST_SnapToGrid(geometry, 0.1) AS snapped,
        SUM(weight_kg) AS total_weight
    FROM filtered
    GROUP BY ST_SnapToGrid(geometry, 0.1)
),
stats AS (
    SELECT MIN(total_weight) AS min_w, MAX(total_weight) AS max_w FROM grid
)
SELECT
    ST_X(snapped) AS longitude,
    ST_Y(snapped) AS latitude,
    CASE
        WHEN stats.max_w = stats.min_w THEN 5
        ELSE 1 + 9 * (total_weight - stats.min_w) / (stats.max_w - stats.min_w)
    END AS weight
FROM grid, stats
```

**쿼리 설계 단계별 설명**
1. `filtered`: 기간 조건에 맞고 수거 무게 데이터가 있는 활동만 추출
2. `grid`: `ST_SnapToGrid(geometry, 0.1)`로 좌표를 0.1도(약 11km) 단위 격자에 스냅해, 개별 활동이 아닌 지역 단위로 뭉쳐서 수거 무게 합산 → 밀집도 단위 시각화 가능
3. `stats`: 전체 격자 중 최소/최대 수거량 산출
4. 최종 SELECT: Min-Max 정규화 공식(`1 + 9 * (x - min) / (max - min)`)으로 각 격자의 가중치를 1~10 스케일로 변환해, 프론트엔드 히트맵 라이브러리가 바로 사용할 수 있는 형태로 가공

**엣지 케이스 처리**: 모든 격자의 수거량이 동일한 경우(`max_w == min_w`) 0으로 나누기 오류를 방지하기 위해 `CASE WHEN`으로 기본값(5)을 반환합니다.

> 활동 밀집도 히트맵을 위해, 원시 좌표 데이터를 애플리케이션이 아닌 SQL(CTE + ST_SnapToGrid) 레벨에서 격자 단위로 집계하고 Min-Max 정규화까지 수행하는 쿼리를 직접 설계했습니다. 이를 통해 대량의 위치 데이터를 프론트엔드에 그대로 전달하지 않고, 시각화에 최적화된 형태로 미리 가공해 전송량과 클라이언트 연산 부담을 줄였습니다.

<br>

#### 5. 카카오 좌표-행정구역 변환 연동

사용자가 지도에서 클릭한 지점은 위경도 숫자일 뿐, "서울시 종로구 광화문" 같은 사람이 읽을 수 있는 행정구역 정보가 아닙니다.

```java
@Service
@Slf4j
@RequiredArgsConstructor
@Transactional
public class LocationService {

    private final LocationRepository locationRepository;
    private final RestClient restClient;
    private static final String KAKAO_GEOCODE_URL = "/v2/local/geo/coord2regioncode";

    @Transactional
    public Location saveForActivity(Activity activity, double lat, double lng) {
        if (Objects.isNull(activity)) {
            throw new IllegalArgumentException("Activity cannot be null");
        }

        // 카카오 API로 좌표 -> 행정구역 변환
        GeocodeResponse.Document regions = getRegion(lat, lng).getDocuments().getFirst();

        Location location = Location.builder()
                .activity(activity)
                .latitude(lat)
                .longitude(lng)
                .region1depthName(regions.getRegion1depthName())
                .region2depthName(regions.getRegion2depthName())
                .region3depthName(regions.getRegion3depthName())
                .address(regions.getAddressName())
                .build();

        return locationRepository.save(location);
    }

    public GeocodeResponse getRegion(double lat, double lng) {
        return restClient.get()
                .uri(uriBuilder -> uriBuilder
                        .path(KAKAO_GEOCODE_URL)
                        .queryParam("x", lng)
                        .queryParam("y", lat)
                        .build())
                .retrieve()
                .body(GeocodeResponse.class);
    }
}
```

이렇게 저장된 행정구역 정보(`region1/2/3depthName`)는 행정구역별 활동 집계(`getAggregatedNational`, `getAggregatedBroad`)에 바로 활용됩니다.

```java
@Transactional(readOnly = true)
public List<LocationAggregation> getAggregatedData(String bound) {
    List<LocationAggregationProjection> projections = switch (bound.toLowerCase()) {
        case "national" -> locationRepository.getAggregatedNational();  // 전국 단위 집계
        case "broad" -> locationRepository.getAggregatedBroad();        // 광역시/도 단위 집계
        default -> throw new IllegalArgumentException("Invalid bound");
    };

    return projections.stream()
            .map(projection -> LocationAggregation.builder()
                    .region(projection.getRegion())
                    .counts(projection.getCounts())
                    .latitude(projection.getLatitude())
                    .longitude(projection.getLongitude())
                    .build())
            .toList();
}
```

> 외부 지오코딩 API(Kakao)를 연동해 좌표 데이터를 행정구역 정보로 변환·저장함으로써, 이후 별도의 실시간 API 호출 없이 로컬 데이터만으로 행정구역 단위(전국/광역) 집계 통계를 빠르게 제공할 수 있는 구조를 만들었습니다.

<br>

#### 6. 다중 소셜 로그인 + JWT/Redis 인증 체계

Google, Naver, Kakao 3개 Provider를 동시 지원합니다.

```yaml
# application.yml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: ${GOOGLE_CLIENT_ID}
            client-secret: ${GOOGLE_CLIENT_SECRET}
            redirect-uri: "{baseUrl}/login/oauth2/code/google"
          naver:
            client-id: ${NAVER_CLIENT_ID}
            redirect-uri: "{baseUrl}/login/oauth2/code/naver"
            provider: naver
          kakao:
            client-id: ${KAKAO_CLIENT_ID}
            redirect-uri: "{baseUrl}/login/oauth2/code/kakao"
            provider: kakao
        provider:
          kakao:
            authorization-uri: https://kauth.kakao.com/oauth/authorize
            token-uri: https://kauth.kakao.com/oauth/token
            user-info-uri: https://kapi.kakao.com/v2/user/me
```

**JWT 발급 — Access/Refresh 이원화**
```java
@Component
@RequiredArgsConstructor
public class JwtTokenProvider {

    @Value("${jwt.secret}") private String secretBase64;
    @Value("${jwt.access-exp}") private long accessExpSeconds;
    @Value("${jwt.refresh-exp}") private long refreshExpSeconds;

    private Key key;

    @PostConstruct
    public void init() {
        this.key = Keys.hmacShaKeyFor(Decoders.BASE64.decode(secretBase64));
    }

    public String createAccessToken(String uuid, Map<String, Object> claims) {
        return createToken(uuid, claims, accessExpSeconds);
    }

    public String createRefreshToken(String uuid) {
        return createToken(uuid, Map.of("typ", "refresh"), refreshExpSeconds);
    }

    private String createToken(String subject, Map<String, Object> claims, long expSec) {
        Instant now = Instant.now();
        return Jwts.builder()
                .setClaims(claims)
                .setSubject(subject)
                .setIssuedAt(Date.from(now))
                .setExpiration(Date.from(now.plusSeconds(expSec)))
                .signWith(key, SignatureAlgorithm.HS256)
                .compact();
    }
}
```

**OAuth2 로그인 성공 후 처리 — Redis에 Refresh Token 저장 + httpOnly 쿠키**
```java
String accessToken = jwtTokenProvider.createAccessToken(uuid, Map.of(
        "email", email, "name", name, "role", role));
String refreshToken = jwtTokenProvider.createRefreshToken(uuid);

// Redis에 uuid 기준으로 refreshToken 저장 -> 서버 측에서 임의 무효화 가능
tokenStore.save(uuid, refreshToken, jwtTokenProvider.getRefreshExpSeconds());

// Refresh Token은 httpOnly 쿠키로 전달 -> XSS로 인한 탈취 방지
ResponseCookie refreshCookie = ResponseCookie.from("refreshToken", refreshToken)
        .httpOnly(true)
        .secure(false)  // 배포 환경에서는 true
        .sameSite("Lax")
        .path("/")
        .maxAge(jwtTokenProvider.getRefreshExpSeconds())
        .build();

response.addHeader(HttpHeaders.SET_COOKIE, refreshCookie.toString());
```

**권한 변경 시 토큰 즉시 재발급 — 세심한 UX 처리**
```java
@Transactional
public TokenResponse changeRole(UserChangeRoleRequest request) {
    User user = userRepository.findById(request.getUserId())
            .orElseThrow(() -> new IllegalArgumentException("유저 없음"));
    user.updateRole(request.getRole());

    Authentication auth = SecurityContextHolder.getContext().getAuthentication();
    boolean isSelfChange = auth != null &&
            auth.getPrincipal() instanceof UserPrincipal principal &&
            principal.userId().equals(user.getUserId());

    // 관리자가 "본인" 권한을 바꾼 경우, 재로그인 없이 새 권한이 즉시 반영되도록
    // SecurityContext와 JWT를 함께 재발급
    if (isSelfChange) {
        UserPrincipal newPrincipal = UserPrincipal.from(user);
        Authentication newAuth = new UsernamePasswordAuthenticationToken(
                newPrincipal, null, newPrincipal.getAuthorities());
        SecurityContextHolder.getContext().setAuthentication(newAuth);

        String newAccessToken = jwtTokenProvider.createAccessToken(
                String.valueOf(user.getUuid()), Map.of("role", user.getRole().name()));
        String newRefreshToken = jwtTokenProvider.createRefreshToken(String.valueOf(user.getUuid()));

        tokenStore.save(String.valueOf(user.getUserId()), newRefreshToken,
                jwtTokenProvider.getRefreshExpSeconds());

        return new TokenResponse(newAccessToken, newRefreshToken);
    }
    return null;  // 타인의 권한 변경은 토큰 재발급 불필요
}
```

> Google/Naver/Kakao 3개 소셜 Provider를 지원하는 OAuth2 로그인을 구축했고, Access/Refresh Token을 이원화해 Refresh Token은 Redis에 저장함으로써 로그아웃이나 관리자에 의한 강제 만료 시 즉시 세션을 무효화할 수 있는 구조를 만들었습니다. 특히 사용자의 권한(Role)이 변경되는 경우, 대상이 본인이면 SecurityContext와 토큰을 함께 재발급해 재로그인 없이 새 권한이 즉시 반영되도록 세심하게 처리했습니다.

<br>

#### 7. 활동 임시저장(Draft)/게시(Publish) 워크플로우

사용자가 활동 기록을 한 번에 완성하지 못하고 나눠서 작성할 수 있어야 하며, 완성 전(임시저장)과 완성 후(게시)의 데이터 검증 수준이 달라야 합니다.

```java
@Transactional
public ActivityDetailResponse updateActivity(Long id, ActivityUpdateRequest dto, Long userId) {
    Activity a = getOwned(id, userId);   // 소유자 검증

    if (a.getStatus() == Status.DELETED) {
        throw new IllegalArgumentException("삭제된 활동은 수정할 수 없습니다.");
    }

    applyPatch(a, dto);   // 부분 수정(PATCH) 적용

    if (dto.getLat() != null && dto.getLng() != null) {
        locationService.saveForActivity(a, dto.getLat(), dto.getLng());
    }

    // 상태별 차등 검증: PUBLISHED일 때만 필수값 검증, DRAFT는 자유롭게 저장 가능
    if (a.getStatus() == Status.PUBLISHED) {
        if (a.getName() == null || a.getName().isBlank()) {
            throw new IllegalArgumentException("활동명은 필수입니다.");
        }
        if (a.getActivityDate() == null) {
            throw new IllegalArgumentException("활동 날짜는 필수입니다.");
        }
    }

    return getOne(id, userId);
}

// 임시저장 -> 게시 전환
@Transactional
public ActivityDetailResponse publish(Long id, Long userId) {
    Activity a = getOwned(id, userId);

    if (a.getStatus() != Status.DRAFT) {
        throw new IllegalArgumentException("초안만 게시할 수 있습니다.");
    }
    if (a.getName() == null || a.getName().isBlank()) {
        throw new IllegalArgumentException("활동명은 필수입니다.");
    }
    if (a.getActivityDate() == null) {
        throw new IllegalArgumentException("활동 날짜는 필수입니다.");
    }

    a.publish();
    return getOne(id, userId);
}
```

- 활동 생성 시 즉시 게시(`createAndPublish`)와 임시저장(`createDraft`) 두 진입점을 모두 제공
- DRAFT ↔ PUBLISHED 상태에 따라 동일한 수정 API가 검증 강도를 다르게 적용 — 도메인 로직에서 "지금 이 활동이 어떤 상태인지"를 기준으로 분기
- REST API 설계상 `PATCH /activities/{id}`로 부분 수정을 지원해, 클라이언트가 변경된 필드만 전송 가능

> 사용자가 활동 기록을 한 번에 완성하지 않아도 되도록 임시저장/게시 이원화 워크플로우를 설계했고, 게시 여부에 따라 필수값 검증 강도를 다르게 적용해 초안 작성의 자유도와 게시물의 데이터 품질을 동시에 확보했습니다.

<br>

#### 8. 소프트 삭제 & 접근 제어

```java
@Transactional
public void delete(Long id, Long userId) {
    Activity a = getOwned(id, userId);
    a.softDelete();   // 물리 삭제가 아닌 상태 변경으로 이력 보존
}

public ActivityDetailResponse getOne(Long id, Long userIdOrNull) {
    Activity a = activityRepository.findByIdWithAllDetails(id)
            .orElseThrow(() -> new IllegalArgumentException("활동이 존재하지 않습니다. id=" + id));

    // 게시되지 않은(DRAFT) 활동은 작성자 본인만 조회 가능
    if (a.getStatus() != Status.PUBLISHED) {
        Long ownerId = a.getCreatedBy().getUserId();
        if (userIdOrNull == null || !ownerId.equals(userIdOrNull)) {
            throw new AccessDeniedException("게시되지 않은 활동에 접근할 권한이 없습니다.");
        }
    }
    return ActivityDetailResponse.from(a, a.getPictures());
}
```

> 삭제된 활동도 통계·이력 추적을 위해 물리 삭제 대신 소프트 삭제로 처리했고, 게시 여부에 따른 접근 제어(비공개 초안은 작성자만 조회 가능)를 서비스 레이어에서 명시적으로 검증해 데이터 노출 범위를 엄격히 관리했습니다.

<br>

#### 9. SSE 기반 실시간 알림

연합 요청, 활동 상태 변경 등의 이벤트를 사용자에게 실시간으로 push하기 위해 Server-Sent Events를 도입했습니다.

```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class NotificationService {

    private final NotificationRepository notificationRepository;
    private final SseEmitterService sseEmitterService;
    private final UserRepository userRepository;

    @Transactional
    public void send(User target, User sender, String message, String type) {
        Notification notification = Notification.builder()
                .targetUser(target)
                .sentUser(sender)
                .message(message)
                .type(type)
                .build();

        notificationRepository.save(notification);   // 알림 영속화 (읽지 않은 알림 조회 가능)

        // SSE로 실시간 push
        sseEmitterService.sendNotification(
                target.getUuid(), "notification", NotificationResponse.from(notification));
    }

    public long countUnread(Long userId) {
        User user = userRepository.findById(userId)
                .orElseThrow(() -> new IllegalArgumentException("유저 없음"));
        return notificationRepository.countByTargetUserAndIsReadFalse(user);
    }
}
```

- 알림을 DB에 영속화함과 동시에 SSE로 실시간 전송해, 접속 중이 아니었던 사용자도 나중에 읽지 않은 알림 목록을 조회 가능
- WebSocket 대신 SSE를 선택 → 알림처럼 서버→클라이언트 단방향 푸시만 필요한 경우, WebSocket보다 구현이 간단하고 HTTP 인프라와의 호환성이 좋음

> 연합 요청이나 활동 상태 변경 같은 이벤트를 사용자에게 실시간으로 전달하기 위해 SSE(Server-Sent Events)를 도입했습니다. 알림을 DB에 영속화해 오프라인 사용자도 추후 확인 가능하게 하면서, 접속 중인 사용자에게는 즉시 push하는 구조로 설계했습니다.

<br>

#### 10. N:M 관계 모델링 — 활동-단체 참여 및 기여도

여러 단체가 하나의 활동에 함께 참여할 수 있으므로, `Activity`와 `Organization`은 N:M 관계입니다. 이를 중간 엔티티(`HostedBy`)로 해소하면서, 각 단체의 기여도(contribution)라는 부가 속성까지 담았습니다.

```java
@Entity
@Table(name = "hosted_by",
        uniqueConstraints = @UniqueConstraint(
                name = "uk_hosted_by_pair",
                columnNames = {"activity_id", "org_id"}
        ))
public class HostedBy {

    @EmbeddedId
    private HostedById hostedById;   // 복합키 (activityId + orgId)

    @ManyToOne(fetch = FetchType.LAZY)
    @MapsId("orgId")
    @JoinColumn(name = "org_id", nullable = false,
            foreignKey = @ForeignKey(name = "fk_org_activity_id"))
    private Organization organization;

    @ManyToOne(fetch = FetchType.LAZY)
    @MapsId("activityId")
    @JoinColumn(name = "activity_id", nullable = false,
            foreignKey = @ForeignKey(name = "fk_activity_org_id"))
    private Activity activity;

    @Column(name = "contribution")
    private Double contribution;   // 각 단체의 기여도 (예: 0.33, 0.5 등)
}
```

- `@EmbeddedId` + `@MapsId`로 복합키(PFK, Primary Foreign Key)를 명시적으로 모델링해, JPA에서 흔히 발생하는 N:M 매핑 이슈에 대한 결정을 명확히 함
- 유니크 제약(`uk_hosted_by_pair`)으로 동일 활동-단체 조합의 중복 저장을 DB 레벨에서 방지
- `contribution` 필드는 향후 "단체별 실적 산정"과 같은 기능 확장을 염두에 둔 설계

> Activity와 Organization 간 N:M 관계를 중간 엔티티(HostedBy)와 복합키(@EmbeddedId + @MapsId)로 모델링했고, 각 단체의 활동 기여도를 부가 속성으로 저장해 향후 단체별 실적 집계 기능으로 확장 가능한 구조를 마련했습니다.

---

<h3 style="font-size: 1.2rem; font-weight: 700; margin-bottom: 1rem;">📋 API 명세 요약</h3>

| 구분 | Method | Endpoint | 권한 | 설명 |
|---|---|---|---|---|
| 활동 | POST | `/api/v1/activities` | HOST | 활동 즉시 게시 |
| 활동 | POST | `/api/v1/activities/drafts` | HOST | 활동 임시저장 |
| 활동 | POST | `/api/v1/activities/{id}/publish` | HOST | 임시저장 → 게시 전환 |
| 활동 | PATCH | `/api/v1/activities/{id}` | HOST | 활동 부분 수정 |
| 활동 | DELETE | `/api/v1/activities/{id}` | HOST | 활동 소프트 삭제 |
| 활동 | GET | `/api/v1/activities/{id}` | - (조건부) | 활동 상세 조회 (비공개는 본인만) |
| 활동 | GET | `/api/v1/activities` | - | 게시된 활동 목록(페이지네이션) |
| 위치 | GET | `/api/v1/location/aggregate` | - | 행정구역별 활동 집계 |
| 위치 | POST | `/api/v1/location/specific` | - | Bounding Box + 기간 조건 세부 위치 조회 |
| 위치 | POST | `/api/v1/location/weighted` | - | 히트맵용 가중치 격자 데이터 |
| 인증 | POST | `/auth/refresh` | - | Access Token 재발급 |
| 인증 | POST | `/auth/logout` | 인증 필요 | 로그아웃 (Redis 세션 삭제) |
| 알림 | GET | `/notifications` | 인증 필요 | 알림 목록 조회 |
| 알림 | GET | `/mypage/notifications/unread-count` | 인증 필요 | 읽지 않은 알림 수 |

---

<h3 style="font-size: 1.2rem; font-weight: 700; margin-bottom: 1rem;">🔧 트러블슈팅</h3>

<div style="display: flex; flex-direction: column; gap: 1rem; margin-bottom: 2rem;">

  <div style="border-left: 3px solid #E67E22; padding-left: 1rem;">
    <span style="font-weight: 700;">외부 API(카카오 지오코딩) 응답 지연으로 인한 활동 등록 지연</span>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.3rem;">
      활동 등록 시 간헐적으로 응답이 눈에 띄게 느려지는 현상이 발생했습니다. 대부분은 정상 속도였지만, 특정 요청에서만
      등록 완료까지 2~3초 이상 소요되는 경우가 있었습니다.
    </p>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.5rem;">
      활동 등록 로직 내부에서 좌표를 행정구역 정보로 변환하기 위해 외부 API(카카오)를 활용하고 있었고, 이 호출이
      트랜잭션 범위 안에 포함되어 있었습니다. API 자체의 응답 시간이 간헐적으로 튀는 구간이 있었는데, 외부 API인
      만큼 응답 속도를 통제할 수 없었고, 이 지연이 그대로 활동 등록 API의 응답 지연과 DB 커넥션 점유 시간 증가로
      이어졌습니다.
    </p>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.5rem;">
      해결을 위해 다음 두 가지 방안을 검토했습니다.
    </p>
    <ul style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.3rem;">
      <li>외부 지오코딩 API 의존성을 없애고, 행정구역 경계 데이터를 활용하여 직접 내부 로직으로 처리</li>
      <li>활동 등록을 지오코딩 API 응답을 기다리지 않고 선제적으로 등록 완료한 뒤, 비동기적으로 분리하여 사후 처리</li>
    </ul>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.5rem;">
      다만 위 두 방법 모두 개발 기간 대비 구현 난이도가 높았습니다. 직접 지오코딩의 경우 행정구역 경계를 처리하는
      로직에 매우 높은 수준의 기하학적 계산이 요구되어, 현실적으로 제한된 공모전 개발 기간 동안 해결하기 어렵다고
      판단했습니다. 비동기적 분리도 호출 실패 시 재요청을 하는 등의 섬세한 로직 구현이 필요했는데, 이 역시 시간상의
      문제로 미처 구현하지 못한 점이 아쉬움으로 남습니다.
    </p>
  </div>

</div>

---

<h3 style="font-size: 1.2rem; font-weight: 700; margin-bottom: 1rem;">💭 배운 점 & 회고</h3>

<p style="font-size: 0.95rem; line-height: 1.8; margin-bottom: 1rem;">
  좌표와 같은 공간 데이터에 대한 최적화를 고민하다가 PostGIS라는 공간 데이터 인덱싱 기법을 처음 알게 되었는데,
  일반 관계형 데이터와 다른 인덱싱·쿼리 방식이 필요하다는 것을 실제 성능 차이로 경험했습니다. 새로운 영역의 데이터
  처리를 실제로 활용해보면서 폭넓은 시야와 의미 있는 경험을 할 수 있었습니다.
</p>

<p style="font-size: 0.95rem; line-height: 1.8; margin-bottom: 2rem;">
  공모전이라는 시간 제약 속에서 완벽한 리팩터링과 기능 구현 우선순위 사이의 트레이드오프를 팀과 논의하며 의사결정을
  하는 경험을 하면서, 개발 역량의 하드 스킬만큼 서로 소통하고 조율하는 소프트 스킬의 중요성을 체감하게 되었습니다.
</p>
