---
layout: page
title: Golden Harvest
description: 농수산물 유통 플랫폼의 재고 관리 서비스
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
</div>

---

<h3 style="font-size: 1.2rem; font-weight: 700; margin-bottom: 1rem;">📌 프로젝트 개요</h3>

<table style="width: 100%; border-collapse: collapse; margin-bottom: 2rem;">
  <tbody>
    <tr style="border-bottom: 1px solid #eee;">
      <td style="padding: 8px; font-weight: 600; width: 120px;">기간</td>
      <td style="padding: 8px;">2025.08 ~ 2026.02</td>
    </tr>
    <tr style="border-bottom: 1px solid #eee;">
      <td style="padding: 8px; font-weight: 600;">팀 구성</td>
      <td style="padding: 8px;">5인 팀 프로젝트 (재고 도메인 전담)</td>
    </tr>
    <tr style="border-bottom: 1px solid #eee;">
      <td style="padding: 8px; font-weight: 600;">GitHub</td>
      <td style="padding: 8px;">
        <a href="https://github.com/Gold-Team-Project/golden-harvest-integrated" target="_blank">
          바로가기
        </a>
      </td>
    </tr>
  </tbody>
</table>

---

<h3 style="font-size: 1.2rem; font-weight: 700; margin-bottom: 1rem;">💡 핵심 구현</h3>

<div style="display: flex; flex-direction: column; gap: 1rem; margin-bottom: 2rem;">

  <div style="border-left: 3px solid #6DB33F; padding-left: 1rem;">
    <span style="font-weight: 700;">CQRS 패턴 적용</span>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.3rem;">
      쓰기는 JPA, 읽기는 MyBatis로 분리하여 각 목적에 맞는 기술을 선택했습니다.
    </p>
  </div>

  <div style="border-left: 3px solid #6DB33F; padding-left: 1rem;">
    <span style="font-weight: 700;">이벤트 기반 비동기 처리</span>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.3rem;">
      Kafka Consumer → Spring ApplicationEvent → EventListener 구조로
      서비스 간 결합도를 낮추고 보상 트랜잭션을 구현했습니다.
    </p>
  </div>

  <div style="border-left: 3px solid #6DB33F; padding-left: 1rem;">
    <span style="font-weight: 700;">동시성 제어</span>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.3rem;">
      재고 차감 시 비관적 락, 상태 변경 시 낙관적 락을 상황에 맞게 구분 적용했습니다.
    </p>
  </div>

  <div style="border-left: 3px solid #6DB33F; padding-left: 1rem;">
    <span style="font-weight: 700;">멱등성 보장</span>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.3rem;">
      Bloom Filter 1차 필터링 + DB 2차 검증으로 Kafka 중복 메시지를 방어했습니다.
    </p>
  </div>

  <div style="border-left: 3px solid #6DB33F; padding-left: 1rem;">
    <span style="font-weight: 700;">FIFO 재고 소진 로직</span>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.3rem;">
      농수산물 특성을 반영해 입고일 기준 오래된 재고부터 순차 소진하는 로직을 구현했습니다.
    </p>
  </div>

</div>

---

<h3 style="font-size: 1.2rem; font-weight: 700; margin-bottom: 1rem;">🤔 기술적 의사결정</h3>

<table style="width: 100%; border-collapse: collapse; margin-bottom: 2rem;">
  <thead>
    <tr style="border-bottom: 2px solid #ddd;">
      <th style="text-align: left; padding: 8px;">결정 사항</th>
      <th style="text-align: left; padding: 8px;">선택</th>
      <th style="text-align: left; padding: 8px;">이유</th>
    </tr>
  </thead>
  <tbody>
    <tr style="border-bottom: 1px solid #eee;">
      <td style="padding: 8px;">읽기 기술</td>
      <td style="padding: 8px;">MyBatis</td>
      <td style="padding: 8px;">복잡한 다중 JOIN 쿼리 직접 제어 필요</td>
    </tr>
    <tr style="border-bottom: 1px solid #eee;">
      <td style="padding: 8px;">재고 차감 락</td>
      <td style="padding: 8px;">Pessimistic Lock</td>
      <td style="padding: 8px;">충돌 시 재고 오차 발생 위험이 높음</td>
    </tr>
    <tr style="border-bottom: 1px solid #eee;">
      <td style="padding: 8px;">중복 방지</td>
      <td style="padding: 8px;">Bloom Filter + DB</td>
      <td style="padding: 8px;">성능과 정확성의 균형</td>
    </tr>
    <tr style="border-bottom: 1px solid #eee;">
      <td style="padding: 8px;">이벤트 relay</td>
      <td style="padding: 8px;">Kafka → Spring Event</td>
      <td style="padding: 8px;">트랜잭션 경계 분리 및 테스트 용이성</td>
    </tr>
  </tbody>
</table>