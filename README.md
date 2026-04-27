<h1 align="center">
    <img src="docs/assets/imgs/CoreBridge-icon.png" alt="CoreBridge 아이콘" width="30" height="30">
    CoreBridge
</h1>

> 한화시스템 BEYOND 17기 팀 Halo 최종 프로젝트 <br>
> 개발 기간 : 2025.09 ~ 2025.11

---

# 프로젝트 주소

[프로젝트 바로가기 - www.core-bridge.co.kr](https://www.core-bridge.co.kr/jobs)

## 계정 별 접근 가능한 URL

- `https://www.core-bridge.co.kr/jobs~` : 모든 권한의 사용자
- `https://www.core-bridge.co.kr/admin/~` : 채용 담당자, 면접관, 관리자

# 테스트 계정

## 관리자

- ID : `admin01@core-bridge.co.kr`
- PW : `qwer1234`

## 채용 담당자

- ID : `recruiter01@core-bridge.co.kr`
- PW : `qwer1234`

## 면접관

- ID : `interviewer01@core-bridge.co.kr`
- PW : `qwer1234`

## 지원자

- ID : `lesw1216@gmail.com`
- PW : `qwer1234`

---

# CI / CD 계획서 & 통합 테스트 결과서

[CI / CD 계획서 & 통합 테스트 결과서 바로가기](https://github.com/lesw1216/Halo-CoreBridge-BE/wiki/4.-%EC%8B%9C%EC%8A%A4%ED%85%9C-%ED%86%B5%ED%95%A9)

---

# 프로젝트 발표 PPT

협업 과정, 핵심 기능 동작, 2차 고도화를 정리한 프로젝트 발표 자료는 아래의 링크 접속

[프로젝트 발표 PPT 바로가기](https://docs.google.com/presentation/d/e/2PACX-1vSZAKetn5fhtIXEs2gfxnLoQ5HVGFccZesbTQW8Q-WVkMxvx7QG5ZkT4CUqiH0t8udq9UgJEqe3Y5-4/pub?start=false&loop=false&delayms=3000)

# 📑 목차 (Table of Contents)

- [프로젝트 기획과 설계](#프로젝트-기획과-설계)
    - [1. 시스템 아키텍처](#-1-시스템-아키텍처)
    - [2. ERD](#-2-erd)
    - [3. 프로젝트 기획서](#-3-프로젝트-기획서)
    - [4. 요구사항 명세서](#-4-요구사항-명세서)
    - [5. WBS](#-5-wbs)

- [프로젝트 소개](#-프로젝트-소개)
    - [1. 개요](#1-개요)
    - [2. 핵심 기능](#2-핵심-기능)
        - [1. 면접 일정 관리](#1-면접-일정-관리)
        - [2. 대면 면접 시 이력서와 평가지 제공](#2-대면-면접-시-이력서와-평가지-제공)
        - [3. 유연한 채용 프로세스 체인](#3-유연한-채용-프로세스-체인)
        - [4. 채용 단계 대시보드 & 파이프라인 시각화](#4-채용-단계-대시보드--파이프라인-시각화)
        - [5. 채용 공고 별 지원자 현황 통계](#5-채용-공고-별-지원자-현황-통계)
        - [6. 검색 엔진 확장](#6-검색엔진-확장)

---

# 프로젝트 기획과 설계

## 🔧 1. 시스템 아키텍처

![시스템아키텍쳐.png](https://github.com/user-attachments/assets/fa562ef8-c613-4f0e-8a2c-3ace9a98cf95)

## 🔗 2. ERD

![ERD.png](./docs/ERD.png)

## 📑 3. 프로젝트 기획서

[프로젝트기획서 바로가기](./docs/프로젝트기획서.pdf)

## ✅ 4. 요구사항 명세서

[요구사항명세서 바로가기](./docs/요구사항명세서.pdf)

<details>
<summary>요구사항 명세서 상세보기</summary>
<div markdown="1">

![요구사항명세서.png](./docs/assets/imgs/요구사항명세서.png)

</div>
</details>

## 📅 5. WBS

[일정 관리를 위한 WBS 바로가기](./docs/WBS.pdf)

<details>
<summary>WBS 상세보기</summary>
<div markdown="1">

![WBS.png](./docs/assets/imgs/WBS.png)

</div>
</details>

<br><br>

# 📋 프로젝트 소개

## 1. 개요

오늘날 스타트업부터 중견·대기업에 이르기까지, 채용 프로세스는 빠르고 신뢰성 있게 관리하는 것이 필수 요소가 되었습니다.
그러나 많은 기업은 여전히 스프레드시트, 이메일, 메신저, 실제 서류로 지원자 상태를 관리하고 있어, 실시간 확인의 어려움, 정보 불일치, 일정 충돌 등의 문제가 반복되고 있습니다.

대기업은 자체 ATS 시스템을 구축하거나 고가의 솔루션을 사용하지만,
중소·중견기업은 도입 비용, 운영 부담, 복잡한 기능 때문에 어려움을 겪는 경우가 많습니다.

이에 CoreBridge는 가볍고 핵심 기능 중심으로 구성된 채용 플랫폼을 개발했습니다.
필요한 기능을 빠르게 도입할 수 있도록 하고, 기업의 성장에 따라 맞춤형으로 확장 가능한 구조를 핵심 가치로 삼고 있습니다.

---

## 2. 핵심 기능

### 1. 면접 일정 관리

- 캘린더 기반 UI를 제공하며, 팀 전체가 면접 일정을 시각적으로 공유할 수 있습니다.

### 2. 대면 면접 시 이력서와 평가지 제공

- 면접관 여러 명이 동시에 평가할 수 있는 평가지·이력서 뷰어를 제공합니다.

### 3. 유연한 채용 프로세스 체인

- 공고별로 단계(서류 → 1차 → 2차 → 과제 등) 를 자유롭게 구성할 수 있습니다.
- 조직 특성, 직무 요구사항에 따라 맞춤형 프로세스를 생성할 수 있습니다.

### 4. 채용 단계 대시보드 & 파이프라인 시각화

- 지원자가 어떤 단계에 있는지 칸반 형태로 시각화하여 전체 흐름을 쉽게 파악할 수 있습니다.

### 5. 채용 공고 별 지원자 현황 통계

- 공고 기준으로 지원자 목록과 진행 상태를 조회할 수 있습니다.

### 6. 검색엔진 확장

- 초기에는 SQL 기반 검색 엔진을 제공하여 빠른 기능을 제공합니다.
- 기업 규모 증가, 대량의 공고, 지원자 데이터 발생 시 고객사의 요청이 발생하면 Elasticsearch 기반 검색 엔진으로 전환이 가능하도록 설계되어 유연한 검색 성능 확장이 가능합니다.

---

<br>

# 화면 설계서 - 피그마

[화면 설계서 - 피그마 바로 가기](https://www.figma.com/design/lj1DLU2vuv1w3ZXXiXvGIx/CoreBridge?node-id=0-1&t=7ucfKIqZA5ZeHXSk-1)

# 기능 동작 테스트

<details>
  <summary> 회원 </summary>

## 이메일 인증

![이메일인증](/docs/gif/applicant/1.%20이메일인증.gif)

## 회원 가입

![회원가입](/docs/gif/applicant/2.%20회원가입.gif)

## 로그인

![로그인](/docs/gif/applicant/3.%20로그인.gif)

## 로그 아웃

![로그 아웃](/docs/gif/applicant/4.%20로그아웃.gif)

</details>

<details>
  <summary>계정 찾기</summary>

## 이메일 찾기

![이메일 찾기](/docs/gif/계정찾기/이메일%20찾기.gif)

## 비밀번호 찾기

![비밀번호 찾기](/docs/gif/계정찾기/비밀번호%20찾기.gif)

</details>

<details>
  <summary>지원자 </summary>

## 채용 공고 상세 조회

![설명 텍스트](/docs/gif/applicant/5.%20지원자용%20채용공고%20전체조회%20및%20상세조회.gif)

  </details>
<div>

<details>
  <summary> 채용 공고 등록 </summary>

프로세스 생성
![설명 텍스트](/docs/gif/recruiter/1.%20채용공고%20최초저장시%20프로세스%20설정.gif)

질문지 생성
![설명 텍스트](/docs/gif/recruiter/2.%20채용공고%20최초저장시%20질문지%20생성.gif)

채용공고 저장
![설명 텍스트](/docs/gif/recruiter/3.%20채용공고%20저장.gif)

</details>

<details>
  <summary> 채용 관리 - 상세 </summary>

## 채용공고 상세조회

![설명 텍스트](/docs/gif/recruiter/4.%20채용공고%20전체조회%20상세조회.gif)

## 지원자 채용 단계 수정

![설명 텍스트](/docs/gif/recruiter/5.%20채용관리%20파이프라인%20편집기능.gif)

## 채용 프로세스 추가

![설명 텍스트](/docs/gif/recruiter/6.%20채용프로세스%20추가.gif)

## 채용 프로세스 삭제

![설명 텍스트](/docs/gif/recruiter/7.%20채용프로세스%20삭제.gif)

## 채용 프로세스 순서편집

![설명 텍스트](/docs/gif/recruiter/8.%20채용프로세스%20순서편집.gif)

## 채용 프로세스 이름 및 색상 편집

![설명 텍스트](/docs/gif/recruiter/9.%20채용프로세스%20이름%20및%20색상%20수정.gif)

</details>

---
