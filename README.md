# 과제 2 – 신청 승인 서비스

웹서비스설계 한국어 분반 개인 과제입니다. 행사 참가 신청, 대기 취소, 직접 승인, 승인 취소와 FIFO 대기열 자동 승급을 구현했습니다. 상태 변경과 변경 이력을 하나의 트랜잭션으로 처리하여 데이터 일관성을 유지합니다.

## 기술 및 실행 환경

- Node.js v24.22.1
- npm 11.19.1
- Express, SQLite(Node.js 내장 모듈), JWT
- Windows, Visual Studio Code

## 설치 및 실행

저장소의 `package.json`이 있는 폴더에서 실행합니다.

```sh
node --version
npm --version
npm ci
npm test
npm start
```

브라우저에서 http://127.0.0.1:3000 에 접속합니다. 서버는 Ctrl+C로 종료합니다. 최초 실행 시 DB와 서명 키가 자동 생성됩니다. Node.js 20은 지원하지 않습니다.

### DB 초기화

서버 종료 후 다음 명령을 실행하고 확인 메시지에 `RESET`을 입력합니다. 기존 신청과 변경 이력이 삭제됩니다.

```sh
npm run db:reset
```

## 연습 계정

공통 연습 비밀번호: `Classroom123!` (배포 코드에 포함된 가상 계정)

| 아이디 | 이름 | 역할 |
|---|---|---|
| alice | 민지 | 학생 |
| bob | 준호 | 학생 |
| charlie | 서연 | 학생 |
| manager1 | 운영자 A | 행사 1 담당, 정원 2명 |
| manager2 | 운영자 B | 행사 2 담당, 정원 1명 |

각 브라우저 탭에서 별도의 계정으로 로그인할 수 있습니다. 페이지 자체를 새로고침하면 재로그인이 필요할 수 있으며 DB 데이터는 유지됩니다.

## 구현 및 변경 파일

주요 변경 파일: `src/application-service.js`.

| 함수 | 구현 내용 | 구현 여부 |
|---|---|---|
| `apply` | 행사 존재, 학생 권한, 중복 신청 검사 및 PENDING/APPLY 저장 | 완료 |
| `cancel` | 본인 소유 및 PENDING 검사, CANCELLED/CANCEL 저장 | 완료 |
| `approve` | 담당자, 상태 및 정원 검사, APPROVED/APPROVE 저장 | 완료 |
| `revoke` | 담당자 및 APPROVED 검사, REVOKED/REVOKE 저장, 대기자 최대 1명 승급, AUTO_APPROVE 원인 이력 연결 | 완료 |
| `transaction` | BEGIN IMMEDIATE, COMMIT, 오류 발생 시 ROLLBACK | 완료 |

자동 승급 대상은 같은 행사에 속한 PENDING 신청으로 제한하고 `ORDER BY created_at ASC, id ASC LIMIT 1`로 선택합니다. 승인 취소, 자동 승급, 두 변경 이력을 단일 트랜잭션에 포함합니다. 기존 로그인·JWT 인증, API 라우팅, 입력 검사, DB 스키마, 저장 도구, 화면 및 테스트는 제공 코드 기반입니다.

## 테스트 및 검증

```sh
npm test
npm run test:assignment
npm run test:advanced
```

- 기본 테스트: 4건 (최종 결과는 증거에서 확인)
- 필수 테스트: T01~T32, 32건 통과
- 선택 심화 테스트: 별도 실행 결과가 있는 경우에만 보고서에 기록
- T28~T30의 `TEST_INJECTED_FAILURE`는 이력 저장 오류를 의도적으로 발생시켜 롤백을 확인하는 테스트 로그입니다.

화면 검증은 다음 흐름을 수행하고 캡처했습니다.

| ID | 검증 흐름 |
|---|---|
| U01 | 학생 신청 및 담당 운영자 승인 |
| U02 | 대기 취소 이후 오래된 화면에서 승인 거부 및 상태·이력 유지 |
| U03 | 서버 재시작 이후 신청 상태와 변경 이력 확인 |
| U04 | 승인 취소, 다음 대기자 자동 승급 및 이력 원인 연결 확인 |
| U05 | 대기자 없는 승인 취소, 빈자리 및 REVOKED 확인 |

- [검증 증거](docs/evidence/)
- [보고서 PDF](docs/report.pdf) (최종 제출 시 첨부)
- [API 계약](docs/API.md)

## 파일 구조

- `src/application-service.js`: 업무 규칙, 트랜잭션 구현
- `src/repository.js`: SQL 조회·저장 도구
- `src/db.js`: SQLite 스키마 및 초기 데이터
- `src/auth.js`: JWT 인증
- `src/app.js`: API 라우팅 및 입력 검증
- `src/server.js`: 로컬 서버
- `public/`: 제공 화면
- `tests/`: 기본·필수·선택 심화 테스트
- `docs/`: API 명세, 보고서 및 검증 증거

## 구현 범위와 한계

필수 업무 규칙 R1~R8을 구현했으며 필수 자동 테스트를 통과했습니다. 실제 서비스 배포, 반려, 재신청, 알림 및 서비스 내부 신청 마감은 구현 범위에 포함하지 않았습니다. 선택 동시성 테스트 결과와 추가로 발견한 문제는 실제 검증 결과에 따라 보고서에 기록합니다.

## AI 활용

ChatGPT를 활용해 `src/application-service.js`의 `transaction`, `apply`, `cancel`, `approve`, `revoke` 함수 구현 방법과 오류 원인 분석, 검증 절차, 제출 자료 정리에 도움을 받았습니다. 직접 코드를 적용하고 `setStatus`, `forbidden`, `managerId`, `actorId`, 자동 승급 이력의 신청 ID 등을 점검·수정했으며 필수 테스트 재실행 및 실제 웹 화면 검증을 수행했습니다.

생성 DB, JWT 서명 키, 실제 `.env`, 토큰, 개인정보 및 `node_modules`는 제출에서 제외합니다.
