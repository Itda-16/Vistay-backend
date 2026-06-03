# Vistay — D-2 비자 서류 자동 분석 백엔드

D-2(유학) 비자 연장에 필요한 서류를 AI로 자동 검증하는 서비스의 백엔드 워크플로우.

## 구성

n8n 기반 3개 워크플로우:

### 1. `vistay-document-parse-final.json` (메인)
사용자가 업로드한 비자 서류를 분석하고 검증 결과를 반환.

- **트리거**: Webhook POST `/webhook/upload`
- **흐름**: 파일 다운로드 → Upstage Document Parse + Information Extract → 룰 기반 검증 → Solar LLM 판단 (외화 환산, 경계선 케이스) → 교차 검증 → 한·영 안내문 생성 → 결과 응답
- **외부 서비스**: Upstage Solar API, ConvertAPI, exchangerate-api.com, Supabase

### 2. `vistay-policy-change-final.json`
비자 정책 변경 시 영향받는 사용자에게 자동 알림.

- **트리거**: 수동 실행
- **흐름**: 정책 데이터 정의 → DB 업데이트 → 영향 사용자 조회 → 중복 제거 → 이메일 발송

### 3. `send-result-email.json`
분석 결과 이메일 발송.

- **트리거**: Webhook POST `/webhook/send-result-email`
- **흐름**: 세션 조회 → 이메일 본문 생성 → SMTP 발송 → 발송 기록 저장

## 기술 스택

- **워크플로우 엔진**: n8n.cloud
- **OCR & AI**: Upstage Document Parse, Information Extract, Solar Pro
- **DB**: Supabase (PostgreSQL)
- **변환**: ConvertAPI (PDF→PNG)
- **환율**: exchangerate-api.com

## 셋업 가이드

### 사전 준비

다음 서비스의 API 키가 필요합니다:

- Upstage API
- ConvertAPI
- Supabase (URL + Service Role Key)
- SMTP (Gmail)

### n8n 가져오기

1. n8n 계정 생성 (cloud 또는 self-hosted)
2. 좌측 메뉴 → Workflows → **Import from File**
3. `workflows/` 폴더의 JSON 파일 업로드

### Credential 설정

각 워크플로우는 다음 credential을 사용합니다. n8n에서 직접 생성하여 연결해 주세요:

- `Supabase Header Auth` (HTTP Header Auth)
  - Name: `apikey`
  - Value: `[YOUR_SUPABASE_SERVICE_ROLE_KEY]`
- `Upstage API` (Bearer Token)
- `ConvertAPI` (Header Auth)
- `SMTP account` (Gmail SMTP)

### Supabase 스키마

`docs/schema.sql` 참조.

## 시연 가이드

심사용 데모 파일은 `demo-files/`에 있습니다.

## 팀

- 백엔드: 이세연, 김민서
- 프론트엔드: 곽성은
- 기획: 강규민

