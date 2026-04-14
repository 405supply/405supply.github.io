---
layout: page
title: Marine Log
description: 해양 정화 활동 기록 및 통계 플랫폼
img: # 나중에 추가
importance: 2
category: Team Project
github: # 링크 알려주시면 추가할게요
---

<div style="margin-bottom: 2rem;">
  <img src="https://img.shields.io/badge/Java-007396?style=flat-square&logo=openjdk&logoColor=white"/>
  <img src="https://img.shields.io/badge/Spring Boot-6DB33F?style=flat-square&logo=springboot&logoColor=white"/>
  <img src="https://img.shields.io/badge/Spring Security-6DB33F?style=flat-square&logo=springsecurity&logoColor=white"/>
  <img src="https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white"/>
  <img src="https://img.shields.io/badge/PostGIS-4169E1?style=flat-square&logo=postgresql&logoColor=white"/>
  <img src="https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white"/>
  <img src="https://img.shields.io/badge/AWS-232F3E?style=flat-square&logo=amazonwebservices&logoColor=white"/>
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
      <td style="padding: 8px;">4인 팀 프로젝트</td>
    </tr>
    <tr style="border-bottom: 1px solid #eee;">
      <td style="padding: 8px; font-weight: 600;">본인 역할</td>
      <td style="padding: 8px;">지도 기반 위치 로직, 대시보드 통계, CSV 내보내기, 사진 업로드</td>
    </tr>
    <tr style="border-bottom: 1px solid #eee;">
      <td style="padding: 8px; font-weight: 600;">성과</td>
      <td style="padding: 8px;">🏆 공모전 본선 진출</td>
    </tr>
  </tbody>
</table>

---

<h3 style="font-size: 1.2rem; font-weight: 700; margin-bottom: 1rem;">🌊 서비스 소개</h3>

<p style="font-size: 0.95rem; line-height: 1.8; margin-bottom: 2rem;">
  해양 및 강 정화 활동을 기록하고 통계로 시각화하는 플랫폼입니다.<br>
  단체/개인이 정화 활동을 등록하고, 수거한 쓰레기 종류와 양을 기록하며,
  지도 기반으로 활동 현황을 확인할 수 있습니다.
</p>

---

<h3 style="font-size: 1.2rem; font-weight: 700; margin-bottom: 1rem;">💡 핵심 구현</h3>

<div style="display: flex; flex-direction: column; gap: 1rem; margin-bottom: 2rem;">

  <div style="border-left: 3px solid #4169E1; padding-left: 1rem;">
    <span style="font-weight: 700;">PostGIS 공간 쿼리 기반 지도 통계</span>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.3rem;">
      사용자가 지도에서 선택한 영역(경계 좌표)을 기반으로
      PostGIS의 공간 쿼리를 활용해 해당 범위 내 활동 데이터를 집계했습니다.
    </p>
  </div>

  <div style="border-left: 3px solid #4169E1; padding-left: 1rem;">
    <span style="font-weight: 700;">대시보드 통계</span>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.3rem;">
      단체별 수거량 순위, 재질별 쓰레기 분류 비율, 월별 수거 추이 등
      다양한 통계 데이터를 집계하여 대시보드로 제공했습니다.
    </p>
  </div>

  <div style="border-left: 3px solid #4169E1; padding-left: 1rem;">
    <span style="font-weight: 700;">CSV 내보내기</span>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.3rem;">
      활동 데이터를 CSV 파일로 내보내는 기능을 구현하여
      단체가 활동 기록을 외부에서 활용할 수 있도록 했습니다.
    </p>
  </div>

  <div style="border-left: 3px solid #4169E1; padding-left: 1rem;">
    <span style="font-weight: 700;">활동 사진 업로드</span>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.3rem;">
      AWS S3를 활용한 활동 사진 업로드 기능을 구현했습니다.
    </p>
  </div>

  <div style="border-left: 3px solid #4169E1; padding-left: 1rem;">
    <span style="font-weight: 700;">활동 상태 관리 (Draft / Published / Deleted)</span>
    <p style="font-size: 0.95rem; line-height: 1.7; margin-top: 0.3rem;">
      임시저장 → 게시 전환 흐름과 소프트 삭제를 구현하여
      데이터 유실 없이 활동 상태를 관리했습니다.
    </p>
  </div>

</div>