# Scanly 문서

## 프로젝트 소개

QR 코드 기반 디지털 명함 서비스

- 한 번의 스캔으로 명함 교환
- 양방향 명함 전송
- 실시간 정보 업데이트

## 기술 스택

| 구분 | 기술 |
|------|------|
| Frontend | React Native |
| Backend | Spring Boot |
| Database | PostgreSQL |
| Infrastructure | Railway (Backend) |

## 문서 구조

| 문서 | 설명           |
|------|--------------|
| [기획안](./기획안.md) | 서비스 전체 기획    |
| [서비스개요](./planning/00-서비스개요.md) | 핵심 가치, 용어 정의 |
| [회원](./planning/01-회원.md) | 회원 도메인 설계    |
| [명함](./planning/02-명함.md) | 명함 도메인 설계    |
| [명함첩](./planning/03-명함첩.md) | 명함첩 도메인 설계   |
| [미팅](./planning/04-미팅.md) | 미팅 도메인 설계    |
| [아키텍처](./architecture/architecture.md) | 시스템 아키텍처     |
| [알림](./planning/05-알림.md) | 알림 도메인 설계    |

## 주요 기능

- 회원가입/로그인
- 디지털 명함 생성 및 QR 코드 발급
- QR 스캔을 통한 명함 저장
- 명함첩 그룹/태그 관리
