# 메리츠증권 정보보안 지원 포트폴리오

개인정보 처리 기준을 API와 검증 로직으로 구현하고, 오픈소스 구성요소와 실행 조건을 분석했습니다.

| 경험 | 개인 담당 | 확인 결과 |
|---|---|---|
| 동의ON | 4인 팀장, 위험도 계산과 동의항목 API, 회귀 테스트 | 공용 모델 28개 테스트 및 동의항목 API 3개 테스트 통과 기록 |
| LogosFuzz | C/C++ API 추출, 빌드 정보와 KB 연결, 대상 프로필 확장 | libsndfile 92개 파일의 컴파일 정보 추출, 최소 하네스 200회 실행 |
| nodown | 4인 팀장, 거래 탐지와 기본요건 규칙 엔진 | 합성 거래 12건에서 선별 후보 7건, 알림 대상 4건 확인 |
| SBOM/SCA | 6인 팀의 SBOM 생성 자동화와 AWS 연계 | 저장소 입력부터 구성요소 분석과 취약점 조회까지 시나리오 시연 |
| AIBOMBOMBass | 3인 팀의 Model BOM 구현과 모듈 통합 | 통합 API 3종 응답 확인 |

## 동의ON - 동의항목과 위험도 계산을 검증하는 API

**기간:** 2025.03.11-2026.08.31

개인정보처리방침을 구조화해 위험도를 보여주고, 사용자의 동의 변경과 정책 변경 알림을 지원하는 프로젝트입니다. 4인 팀장으로 요구사항과 일정을 조율했으며, FAIR를 참고한 위험도 계산과 백엔드 흐름을 담당했습니다.

기업별 필수 및 선택 동의항목을 조회하는 API를 구현해 사용자의 실제 선택 여부와 민감도, 외부 제공, 보유기간, 처리 목적, AI 활용 관련 값을 함께 반환했습니다. 공용 모델에는 산출식, 등급 경계, LLM 응답 파싱과 저장 변환의 회귀 테스트를 추가했습니다. DB 저장 시 숫자 형식과 등급 매핑의 차이도 확인하고 통합 가이드에 기록했습니다.

- [구현 브랜치: develop](https://github.com/dayeon053/dynamic-consent-system/tree/develop)
- [회귀 테스트와 타입 변환 PR #20](https://github.com/dayeon053/dynamic-consent-system/pull/20): 공용 모델 테스트 28개 통과 기록
- [동의항목 조회 API PR #21](https://github.com/dayeon053/dynamic-consent-system/pull/21): 필수항목, 사용자 선택 여부, 변수 매핑 테스트 3개 통과 기록

두 PR은 develop에 병합됐습니다. 테스트 결과는 각 PR의 검증 기록 기준이며, 백엔드에는 별도의 기존 `contextLoads()` 실패가 남아 있었습니다.

## LogosFuzz - 빌드 조건을 반영한 C/C++ 분석

**기간:** 2026.04.01-현재 (진행 중)

LLM이 퍼징 하네스를 생성할 때 API 선언과 빌드 조건을 함께 사용할 수 있도록 정보를 추출하는 4인 프로젝트입니다. Clang AST 기반 API 분석, 빌드 단위별 지식베이스(KB) 구성과 생성기 연결을 맡았습니다.

Bazel XML에서 대상과 의존성, 컴파일 옵션을 추출했습니다. 이후 CMake의 `compile_commands.json`에서 파일별 플래그를 읽는 기능과 대상 프로필을 추가하고, 비-Bazel 빌드에서도 KB를 하네스 계획에 연결하도록 확장했습니다. 평가용 정답셋 생성 경로에서 컴파일 플래그가 누락되는 문제도 수정했습니다.

- [프로젝트 개발 브랜치: dev](https://github.com/sesezz/LogosFuzz/tree/dev)
- [구현 브랜치: feature/generic-embedded](https://github.com/sesezz/LogosFuzz/tree/feature/generic-embedded)
- [대상 프로필과 추출기 PR #71](https://github.com/sesezz/LogosFuzz/pull/71)
- [KB 연결과 libsndfile 평가 PR #76](https://github.com/sesezz/LogosFuzz/pull/76)
- [실행 및 평가 문서](https://github.com/sesezz/LogosFuzz/blob/feature/generic-embedded/docs/GENERIC-A-kb.md)

WSL의 libsndfile 1.2.2 환경에서 92개 파일의 컴파일 정보를 추출하고 최소 하네스 200회 실행을 확인했습니다. PR #76 기록은 1142개 통과, 5개 건너뛰기, 6개 실패이며 기존 실패 6개를 포함합니다. 선정 API 8개와 해당 환경에서 기능을 검증한 결과입니다. 전체 라이브러리의 정확도와 기존 퍼저 대비 성능 우위는 검증하지 않았습니다.

## nodown - 장기할부 피해 후보와 확인 근거 제공

**기간:** 2026.08.14-2026.09.06

장기할부 서비스 중단 시 피해 가능 거래를 선별하고 할부항변 준비를 돕는 금융 AI Challenge 프로젝트입니다. 4인 팀장으로 거래 탐지, 가맹점 상태 모의조회, 사례 API와 기본요건 규칙 엔진을 맡았습니다.

Python과 Flask로 기능을 연결하고 기본요건의 충족 여부는 결정론적 규칙으로 평가했습니다. 필요한 자료가 빠진 경우 추가자료 확인 상태를 반환하고, 제외되는 거래에는 사유를 표시했습니다. 문장 생성과 요건 평가를 구분해 결과의 근거를 확인할 수 있도록 구성했습니다.

- [원본 코드: main](https://github.com/10502R/nodown/tree/main)
- [기본요건 규칙 엔진](https://github.com/10502R/nodown/blob/main/services/rule_service.py)
- [규칙 테스트](https://github.com/10502R/nodown/blob/main/tests/test_rule_service.py)
- [담당 범위](https://github.com/10502R/nodown/blob/main/docs/A_%EB%8B%B4%EB%8B%B9_%EA%B5%AC%ED%98%84%EC%A7%80%EC%B9%A8.md)

합성 거래 12건 중 선별 조건을 통과한 후보 7건과 모의 폐업 상태에 따른 알림 대상 4건을 확인했습니다. 합성데이터와 모의조회를 사용한 프로토타입이며, 실제 금융기관 연계나 최종적인 권리 인정 판단은 검증 범위에 포함되지 않습니다.

## SBOM/SCA - 구성요소 분석 과정 자동화

**프로젝트 기간:** 2024.05.04-2024.07.14

화이트햇스쿨 6인 팀 프로젝트에서 SBOM 생성 자동화와 CodeCommit 저장소 주소의 S3 전달을 주로 맡았습니다. Lambda, Systems Manager와 EC2를 연결했으며, Amazon Inspector 결과와 Maven SBOM 명령, Package URL 조사에는 보조 담당으로 참여했습니다.

팀은 저장소 입력, SBOM 생성, 서버 패키지 진단과 취약점 조회로 이어지는 자동 분석 시나리오를 시연했습니다. 코드의 구성요소와 서버에서 수집한 패키지 정보를 함께 확인하는 흐름을 구현했으며, 검증 범위는 교육 프로젝트의 자동 분석 시나리오입니다.

## AIBOMBOMBass - AI 모델 구성정보 수집과 통합

**기간:** 2026.03.26-2026.06.04

3인 팀에서 Model BOM 구현과 팀원들이 개발한 모듈의 통합을 맡았습니다. Hugging Face 모델 메타데이터를 수집하고 `requirements.txt`, `config.json`, 모델 카드의 의존성과 설정 정보를 읽어 SPDX 3.0.1 AI Profile을 참고한 모델 구성명세를 생성했습니다.

외부 API 연결이 실패하면 규칙 기반 분석으로 전환하는 경로를 구현하고, 다른 모듈의 대체 경로도 통합했습니다. 통합 후 API 3종의 응답을 확인했습니다. Dataset BOM과 화면, Environment BOM과 취약점 조회는 각 팀원의 담당 범위입니다. 표준 적합성과 법규 준수 여부는 검증하지 않았습니다.
