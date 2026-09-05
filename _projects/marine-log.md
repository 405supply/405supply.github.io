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
      <td style="padding: 8px; font-weight: 600; width: 120px;">기간</td>
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
      <td style="padding: 8px; font-weight: 600;">본인 역할</td>
      <td style="padding: 8px;">위치 기반 로직(PostGIS 연동, 좌표-행정구역 변환), 대시보드 통계 API, CSV 내보내기, S3 사진 업로드</td>
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

---

<h3 style="font-size: 1.2rem; font-weight: 700; margin-bottom: 1rem;">🌊 서비스 소개</h3>

<p style="font-size: 0.95rem; line-height: 1.8; margin-bottom: 2rem;">
  해안 정화 활동은 대부분 개별 단체가 수기(엑셀, 카카오톡)로 기록해와, 활동 데이터가 정량적으로 축적되지 못하고
  단체 간 협업 기회도 놓치는 문제가 있었습니다. Marine Log는 정화 활동의 <b>제보 → 검증(답사) → 참여자 모집 →
  실행 → 완료</b> 흐름을 시스템화하고, 지리 정보 기반으로 활동 데이터를 시각화해 이 문제를 해결하고자 했습니다.
</p>

---

<h3 style="font-size: 1.2rem; font-weight: 700; margin-bottom: 1rem;">💡 핵심 구현</h3>

<div style="display: flex; flex-direction: column; gap: 1rem; margin-bottom: 2rem;">

  <div style="border-left: 3px solid #4169E1; padding-left: 1rem;">
    <span style="font-weight: 700;">PostGIS 기반 공간 데이터 처리</span>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.3rem;">
      단순 위경도 컬럼 대신 위경도를 Point 객체로 변환해 PostGIS geometry 타입으로 저장하고, GIST 공간 인덱스를
      적용해 지도 영역(Bounding Box) 기반 조회 성능을 확보했습니다.
    </p>
  </div>

  <div style="border-left: 3px solid #4169E1; padding-left: 1rem;">
    <span style="font-weight: 700;">히트맵을 위한 공간 격자 집계 쿼리</span>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.3rem;">
      활동 밀집도 히트맵을 위해, 원시 좌표를 애플리케이션이 아닌 SQL(CTE + ST_SnapToGrid) 레벨에서 격자 단위로
      집계하고 Min-Max 정규화까지 수행하는 쿼리를 직접 설계해, 대량 위치 데이터의 전송량과 클라이언트 연산
      부담을 줄였습니다.
    </p>
  </div>

  <div style="border-left: 3px solid #4169E1; padding-left: 1rem;">
    <span style="font-weight: 700;">카카오 좌표-행정구역 변환 연동</span>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.3rem;">
      사용자가 지도에서 지정한 좌표를 카카오 지오코딩 API로 "시/도-시/군/구-읍/면/동" 행정구역 정보로 변환해
      저장했습니다. 이를 통해 실시간 API 호출 없이 로컬 데이터만으로 전국/광역 단위 집계 통계를 빠르게 제공할
      수 있는 구조를 만들었습니다.
    </p>
  </div>

  <div style="border-left: 3px solid #4169E1; padding-left: 1rem;">
    <span style="font-weight: 700;">대시보드 통계 & CSV 내보내기</span>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.3rem;">
      행정구역별/기간별 활동 집계, Bounding Box 조건 조회, 히트맵 데이터 등 위치 기반 통계 API 3종을 설계했고,
      활동 데이터를 CSV로 내보내는 기능을 구현해 단체가 자체적으로 활동 실적을 정리하거나 외부(후원처, 지자체)에
      증빙할 수 있도록 지원했습니다.
    </p>
  </div>

  <div style="border-left: 3px solid #4169E1; padding-left: 1rem;">
    <span style="font-weight: 700;">다중 소셜 로그인 + JWT/Redis 인증 체계</span>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.3rem;">
      Google/Naver/Kakao 3개 Provider를 지원하는 OAuth2 로그인을 구축했고, Access/Refresh Token을 이원화해
      Refresh Token은 Redis에 저장함으로써 로그아웃 시 즉시 세션을 무효화할 수 있도록 설계했습니다.
      Refresh Token은 httpOnly 쿠키로 전달해 XSS 탈취 위험을 줄였습니다.
    </p>
  </div>

  <div style="border-left: 3px solid #4169E1; padding-left: 1rem;">
    <span style="font-weight: 700;">S3 기반 사진 업로드</span>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.3rem;">
      활동 인증 및 수거 쓰레기 기록에 필요한 사진을 AWS S3에 업로드·관리하는 기능을 구현해, 활동의 신뢰도 있는
      기록·검증이 가능하도록 지원했습니다.
    </p>
  </div>

</div>

---

<h3 style="font-size: 1.2rem; font-weight: 700; margin-bottom: 1rem;">🔧 트러블슈팅</h3>

<div style="display: flex; flex-direction: column; gap: 1rem; margin-bottom: 2rem;">

  <div style="border-left: 3px solid #E67E22; padding-left: 1rem;">
    <span style="font-weight: 700;">외부 API(카카오 지오코딩) 응답 지연으로 인한 활동 등록 지연</span>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.3rem;">
      활동 등록 시 간헐적으로 응답이 눈에 띄게 느려지는 현상이 발생했습니다. 대부분은 정상 속도였지만, 특정
      요청에서만 등록 완료까지 2~3초 이상 소요되는 경우가 있었습니다.
    </p>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.5rem;">
      활동 등록 로직 내부에서 좌표를 행정구역 정보로 변환하기 위해 외부 API(카카오)를 호출하고 있었고, 이 호출이
      트랜잭션 범위 안에 동기 방식으로 포함되어 있었습니다. 외부 API인 만큼 응답 속도를 통제할 수 없었고, 이
      지연이 그대로 활동 등록 API의 응답 지연과 DB 커넥션 점유 시간 증가로 이어졌습니다.
    </p>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.5rem;">
      다음 두 가지 개선 방안을 검토했습니다.
    </p>
    <ul style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.3rem;">
      <li>외부 지오코딩 API 의존성을 없애고, 행정구역 경계 데이터를 확보해 직접 내부 로직으로 처리</li>
      <li>활동 등록은 지오코딩 API 응답을 기다리지 않고 선제적으로 완료한 뒤, 지오코딩 처리를 비동기로 분리</li>
    </ul>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.5rem;">
      다만 두 방안 모두 실제로 적용하지는 못했습니다. 직접 지오코딩 구현은 행정구역 경계를 다루는 데 매우 높은
      수준의 기하학적 계산이 요구되어 제한된 공모전 개발 기간 내 해결이 현실적으로 어렵다고 판단했고, 비동기
      분리 역시 호출 실패 시 재요청 처리 등 섬세한 로직이 필요해 시간상 구현하지 못했습니다. 활동 등록 자체가
      빈번한 액션이 아니고 지연이 발생해도 2~3초 수준으로 크지 않다고 판단해, 이번 프로젝트 범위에서는
      우선순위를 낮추고 원인과 해결 방향을 명확히 정리해두었습니다.
    </p>
  </div>

</div>

---

<h3 style="font-size: 1.2rem; font-weight: 700; margin-bottom: 1rem;">💭 회고</h3>

<p style="font-size: 0.95rem; line-height: 1.8; margin-bottom: 1rem;">
  좌표와 같은 공간 데이터에 대한 최적화를 고민하다가 PostGIS라는 공간 데이터 인덱싱 기법을 처음 알게 되었는데,
  일반 관계형 데이터와 다른 인덱싱·쿼리 방식이 필요하다는 것을 실제 성능 차이로 경험했습니다. 새로운 영역의
  데이터 처리를 직접 활용해보면서 폭넓은 시야와 의미 있는 경험을 쌓을 수 있었습니다.
</p>

<p style="font-size: 0.95rem; line-height: 1.8; margin-bottom: 2rem;">
  공모전이라는 시간 제약 속에서 완벽한 리팩터링과 기능 구현 우선순위 사이의 트레이드오프를 팀과 논의하며
  의사결정을 하는 경험을 하면서, 개발 역량의 하드 스킬만큼 서로 소통하고 조율하는 소프트 스킬의 중요성을
  체감했습니다.
</p>
