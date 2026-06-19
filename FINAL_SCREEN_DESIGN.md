# 셀프클레임 v3.3 — 최종 화면 설계 문서

**Project**: 셀프클레임 (Self-Claim) v3.3  
**Based On**: PRD_selfclaim_v3.3.md + DESIGN_SYSTEM_SPECIFICATION.md  
**Design Approach**: Design System 100% 준수, React Component First  
**Date**: 2026년 6월 18일

---

## 📑 목차
1. [Sitemap](#1-sitemap)
2. [Screen Structure](#2-screen-structure)
3. [Component Hierarchy](#3-component-hierarchy)
4. [Design Token Mapping](#4-design-token-mapping)
5. [UI Layout & Wireframe](#5-ui-layout--wireframe)
6. [React Component Tree](#6-react-component-tree)

---

## 1. Sitemap

### 1.1 Information Architecture (IA)

```
┌─────────────────────────────────────────────────────────────┐
│ 셀프클레임 v3.3 — Linear Chat Session (Single Session)    │
└─────────────────────────────────────────────────────────────┘

Layer 0: Common Shell
├── HeaderBar
│   ├── Logo (left)
│   ├── VehicleBadge + EditTrigger (center, conditional)
│   └── ExitButton (right)
│
└── ChatViewport (main scrollable container)

Layer 1: Session Flow (Linear + Conditional Branches)

S1. Intro Messages (Auto)
    └─> [Next] ──────────────────────────────────> S2

S2. Plate Input Card (User Action)
    ├─> [조회 Success] ─────────────────────────> S3
    └─> [조회 Fail: 99 prefix] ──> Sheet A ──> S3 (bypass)

S3. Vehicle Info Confirm Message (Auto)
    └─> [Next Auto] ────────────────────────────> S4

S4. Main Branch Choice (User Choice)
    ├─> [렌터카 신청] ────────────────────────> S6
    └─> [교통비 조회] ────────────────────────> S5

    ┌─────── Branch: ESTIMATE (S5 서브플로우) ──────────┐
    │                                                   │
    S5-1. Repair Days Selector (Sheet B)               │
         └─> [Complete] ──────────────────> S5-2       │
                                                        │
    S5-2. Fault Rate Selector (Sheet B re-use)         │
         └─> [Complete] ──────────────────> S5-3       │
                                                        │
    S5-3. Transit Cost Result (Auto message)           │
         └─> [Auto] ──────────────────────> S5-4       │
                                                        │
    S5-4. Post-Estimate Branch                         │
         ├─> [렌터카로 변경] ───────────> S6             │
         └─> [나가기] ───────────────> End             │
    │                                                   │
    └───────────────────────────────────────────────────┘

    ┌─────── Branch: RENT (S6~S12 메인 플로우) ─────────┐
    │                                                   │
    S6. Region Selection (Sheet C)                      │
        └─> [Chip select] ──────────────> S7           │
                                                        │
    S7. Phone Verification (Inline)                    │
        ├─> [인증번호 받기] ──────────────────┐         │
        │    └─> [인증번호 입력 + 확인] ──┐  │         │
        │         └─> [확인 완료] ──────┼──┴─> S8       │
        │                              │              │
        └──────────────────────────────┘               │
                                                        │
    S8. Additional Request (Optional)                  │
        ├─> [없어요] ──────────────> S9                │
        └─> [있어요] ──> Text Input ──> S9             │
                                                        │
    S9. Garage Decision                                │
        ├─> [정했어요] ─> S9-A (Shop Name Input)        │
        │    └─> [Input Complete] ──> S11             │
        └─> [못 정했어요] ──> S9-B (Shop List)         │
             ├─> [Shop Card Select] ──> S10            │
             ├─> [더 보기] ──> Full List               │
             ├─> [Shop 최종 선택] ──> S10              │
             └─> [지역 변경] ─> Sheet C ──> (갱신)     │
                                                        │
    S10. Reserve Info Card (Summary)                   │
         └─> [Auto] ──────────────────> S11            │
                                                        │
    S11. Terms Agreement (Sheet D)                     │
         └─> [All Terms Agree + Complete] ──> S12     │
                                                        │
    S12. Completion Card (Final)                       │
         └─> [Session End]                             │
    │                                                   │
    └───────────────────────────────────────────────────┘

Modal Layer (Overlay)
├── Sheet A: Vehicle Edit (VehicleEditSheet)
│   └─> [Complete] ──> Dismiss + Resume Chat
├── Sheet B: Days/Fault Selector (DaysSelectSheet / FaultRateSheet)
│   └─> [Complete] ──> Dismiss + Auto Progress
├── Sheet C: Region Select (RegionSelectSheet)
│   └─> [Complete / Region Changed] ──> Dismiss + Refresh Shop List
└── Sheet D: Terms Agreement (TermsSheet)
    └─> [Agree All] ──> Dismiss + Final Submit

Dialog Layer (Confirm)
└── Exit Confirmation Dialog (ExitConfirmDialog)
    ├─> [계속 진행] ──> Dismiss
    └─> [종료] ──> location.reload()

Error/Alert Layer (Toast / Inline Message)
├── Network Error (with Retry)
├── Validation Error (inline + button disable)
└── Success Message (transient)
```

### 1.2 User Journey Map

#### Path A: 렌터카 신청 (정보 수리점)
```
S1 → S2(차량조회) → S3(확인) → S4(렌트) → S6(지역) → S7(인증) 
→ S8(요청) → S9(정함) → S9-A(입력) → S11(약관) → S12(완료)
```

#### Path B: 렌터카 신청 (미정 수리점 → 추천)
```
S1 → S2(차량조회) → S3(확인) → S4(렌트) → S6(지역) → S7(인증) 
→ S8(요청) → S9(미정) → S9-B(리스트) → S10(예약) → S11(약관) → S12(완료)
```

#### Path C: 교통비 조회 후 종료
```
S1 → S2(차량조회) → S3(확인) → S4(교통비) → S5-1(기간) → S5-2(과실) 
→ S5-3(결과) → S5-4(나가기) → End
```

#### Path D: 교통비 조회 후 렌트로 전환
```
S1 → S2 → S3 → S4(교통비) → S5-1 → S5-2 → S5-3 
→ S5-4(렌트로 변경) → S6 → ... → S12
```

#### Path E: 중도 종료
```
(Any Stage) → [헤더 종료] → ExitConfirmDialog → [종료 확정] → Reload
```

---

## 2. Screen Structure

### 2.1 S1 — 인트로 메시지

**목적**: 사용자 정서 안정화 + 첫 입력 안내

**구조**:
```
┌─────────────────────────────────────┐
│  HeaderBar (로고, 종료)              │
├─────────────────────────────────────┤
│                                     │
│  [봇 메시지 1]                      │
│  "안녕하세요, 셀프클레임입니다.     │
│   사고 직후 필요한 렌터카 신청,     │
│   교통비 조회를 도와드립니다."      │
│                                     │
│  [봇 메시지 2]                      │
│  "먼저 사고 차량의 정보를            │
│   확인하겠습니다."                   │
│                                     │
│  [자동 다음 단계]                   │
│  └─> S2로 이동                      │
│                                     │
└─────────────────────────────────────┘
```

**컴포넌트 구성**:
- `HeaderBar`
- `ChatViewport`
  - `BotMessageBubble` (×2)
  - `TypingIndicator` (선택)

**Design Tokens**:
- Message: `--t-body1` (15px, 400)
- Spacing: `--lg` (16px) between messages
- Container: `--container-mobile`

**Data Structure**:
```typescript
type IntroState = {
  status: 'intro' | 'intro_complete';
  message1: string;
  message2: string;
};
```

---

### 2.2 S2 — 차량번호 입력 카드

**목적**: 차량번호 수집 및 자동 조회

**구조**:
```
┌─────────────────────────────────────┐
│  HeaderBar (로고, 종료)              │
├─────────────────────────────────────┤
│                                     │
│  [봇 메시지]                        │
│  "차량번호를 입력해 주세요."        │
│                                     │
│  ┌──────────────────────────────┐   │
│  │ PlateInputCard               │   │
│  │ ┌────────────────────────┐   │   │
│  │ │ 입력: "12나3456"       │   │   │
│  │ └────────────────────────┘   │   │
│  │ [차량 조회하기] (활성/비활성) │   │
│  │                             │   │
│  │ ℹ️  정규식 가이드 (선택)     │   │
│  │ "숫자(2~3자) + 한글(1자)    │   │
│  │  + 숫자(4자)" 예: 12가3456  │   │
│  └──────────────────────────────┘   │
│                                     │
└─────────────────────────────────────┘
```

**컴포넌트 구성**:
- `HeaderBar`
- `ChatViewport`
  - `BotMessageBubble`
  - `PlateInputCard`
    - `TextInput` (plate field)
    - `Button` (primary, md, 비활성 조건 포함)
    - `InlineHint` (가이드 문구)

**Design Tokens**:
- Input: `--radius-lg` (12px), height 44-48px
- Button: `--primary` (#008449), `--t-headline3` (16px, 700)
- Padding: `--lg` (16px) card 내 padding
- Spacing: `--md` (12px) field 간격

**Validation**:
```typescript
const plateRegex = /^\d{2,3}[가-힣]\d{4}$/;
const isValid = plateRegex.test(input.trim());
const isLookupFailed = plate.startsWith('99'); // Mock rule

// Button state
disabled = !isValid || isLoading
```

**Data Structure**:
```typescript
type PlateState = {
  plate: string;
  isValid: boolean;
  isLoading: boolean;
  error: string | null;
};

type VehicleInfo = {
  plate: string;
  name: string;      // e.g. "셀토스"
  cls: 'SUV' | 'sedan' | 'van' | 'truck';
  cc: '~1599' | '1600~1999' | '2000~2499' | '2500~2999' | '3000+';
  year: '~2017' | '2018+';
  source: 'auto' | 'manual';
};
```

---

### 2.3 S3 — 차량 정보 확인 메시지

**목적**: 조회된 차량 정보 확인 및 헤더 배지 노출

**구조**:
```
┌─────────────────────────────────────┐
│  HeaderBar                          │
│  ├─ Logo                            │
│  ├─ VehicleBadge: "12나3456 ✎"      │ ◄─ 새로 노출
│  └─ ExitButton                      │
├─────────────────────────────────────┤
│                                     │
│  [봇 메시지]                        │
│  "조회 완료했습니다."               │
│                                     │
│  ┌──────────────────────────────┐   │
│  │ VehicleInfoCard              │   │
│  │                              │   │
│  │ 🚗 셀토스                     │   │
│  │ 차종: SUV                     │   │
│  │ 엔진: 1600~1999cc            │   │
│  │ 연식: 2020년                 │   │
│  │                              │   │
│  │ [✎ 정보 수정]                │   │ ◄─ Sheet A 호출
│  └──────────────────────────────┘   │
│                                     │
│  [자동 다음]                        │
│  └─> S4로 이동                      │
│                                     │
└─────────────────────────────────────┘
```

**컴포넌트 구성**:
- `HeaderBar` (업데이트)
  - `VehicleBadge` (새 추가)
    - Plate 텍스트
    - EditTrigger (✎ 버튼)
- `ChatViewport`
  - `BotMessageBubble`
  - `VehicleInfoCard`
    - Icon (car)
    - Info fields (name, cls, cc, year)
    - Button (secondary/tertiary, md, edit trigger)

**Design Tokens**:
- VehicleBadge: `--radius-full` (20px), `--shadow-sm`
- VehicleInfoCard: `--radius-lg` (14px), `--primary-light` (2% bg)
- Button: `--secondary`, `--t-headline3`
- Icons: 20×20px, `--primary`

**Data Structure**:
```typescript
type VehicleConfirmState = {
  vehicle: VehicleInfo;
  isEditing: boolean;
  sheet: 'closed' | 'vehicle_edit';
};
```

---

### 2.4 S4 — 메인 분기 선택

**목적**: 렌터카 vs 교통비 조회 선택

**구조**:
```
┌─────────────────────────────────────┐
│  HeaderBar (plate 배지 표시)        │
├─────────────────────────────────────┤
│                                     │
│  [봇 메시지]                        │
│  "어떤 도움이 필요하신가요?"        │
│                                     │
│  ┌──────────────────────────────┐   │
│  │ QuickChoiceGroup (2 buttons) │   │
│  │                              │   │
│  │  [렌터카 신청]  [교통비 조회] │   │
│  │  (Primary)     (Secondary)   │   │
│  │                              │   │
│  └──────────────────────────────┘   │
│                                     │
│  (선택 후 자동 사라지고 결과 에코)   │
│                                     │
└─────────────────────────────────────┘
```

**커포넌트 구성**:
- `HeaderBar`
- `ChatViewport`
  - `BotMessageBubble`
  - `QuickChoiceGroup`
    - `Button` (Primary, md)
    - `Button` (Secondary, md)

**Design Tokens**:
- Buttons: `--radius-lg` (12px), height 44-48px
- Spacing: `--md` (12px) button간
- Primary: `--primary` (#008449)
- Secondary: outline with border `--primary`

**Logic**:
```typescript
type BranchChoice = 'rent' | 'estimate';

// Choice 후 화면 전환
if (choice === 'rent') {
  // S6로 이동 (Region Selection)
} else if (choice === 'estimate') {
  // S5-1로 이동 (Days Selection Sheet)
}

// 선택 버튼은 메시지로 에코되고 사라짐
```

---

### 2.5 S5-1 ~ S5-4 교통비 시뮬레이션 서브플로우

#### S5-1: 수리 기간 선택 (Sheet B)

**구조**:
```
Modal Overlay:
┌─────────────────────────────────────┐
│ BottomSheet                         │
│ ┌───────────────────────────────┐   │
│ │ Header                        │   │
│ │ [X]  수리 기간 선택           │   │ ◄─ Drag handle
│ ├───────────────────────────────┤   │
│ │                               │   │
│ │ ℹ️ 예상 수리 기간을            │   │
│ │    입력해 주세요.              │   │
│ │ (1~25일 권장)                  │   │
│ │                               │   │
│ │ ┌──────────────────────────┐   │   │
│ │ │ ChipGroup (1-25 칩)      │   │   │
│ │ │ [1] [2] [3] ... [25]     │   │   │
│ │ │ (선택 칩: background 강조) │   │   │
│ │ └──────────────────────────┘   │   │
│ │                               │   │
│ ├───────────────────────────────┤   │
│ │ [완료]  (선택 후 활성)         │   │
│ └───────────────────────────────┘   │
└─────────────────────────────────────┘
```

**컴포넌트 구성**:
- `BottomSheet`
  - `Header` (drag handle, close button)
  - `HintMessage`
  - `ChipGroup` (1-25)
    - Multiple `Chip` (Filled/Outlined variant)
  - `Button` (Primary, md, disabled until selected)

**Design Tokens**:
- Sheet: `--radius-2xl` (20px top), `--shadow-lg`
- Chips: `--radius-full` (20px), `--radius-xl` (12px) smaller option
- Button: `--primary`
- Spacing: `--lg`, `--md`

**Data**:
```typescript
type DaysSelectState = {
  selectedDays: number | null;
  days: number[]; // 1~25
};
```

#### S5-2: 과실 비율 선택 (Sheet B 재사용)

**구조**: S5-1과 동일, 단 칩 내용 변경

```
칩 옵션: [무과실] [10%] [20%] ... [100%]
```

**Data**:
```typescript
type FaultRateSelectState = {
  selectedRate: number | null; // 0 | 10 | 20 | ... | 100
  rates: number[];
};
```

#### S5-3: 교통비 결과 메시지

**구조**:
```
┌─────────────────────────────────────┐
│  HeaderBar                          │
├─────────────────────────────────────┤
│                                     │
│  [봇 메시지]                        │
│  "계산 완료했습니다."               │
│                                     │
│  ┌──────────────────────────────┐   │
│  │ TransitCostResultCard        │   │
│  │                              │   │
│  │ 📊 예상 일일 교통비           │   │
│  │                              │   │
│  │ 기준 단가: 32,000원/일        │   │
│  │ 과실 비율: 20%               │   │
│  │ ━━━━━━━━━━━━━━━━━━          │   │
│  │ 예상 일일 교통비:              │   │
│  │ 25,600원                      │   │
│  │                              │   │
│  │ ⓘ 정확한 교통비는             │   │
│  │   담당자 확인 후 안내         │   │
│  │                              │   │
│  └──────────────────────────────┘   │
│                                     │
└─────────────────────────────────────┘
```

**컴포넌트 구성**:
- `ChatViewport`
  - `BotMessageBubble`
  - `TransitCostResultCard`
    - Icon (chart)
    - Info fields (base rate, fault rate, result)
    - Disclaimer text

**Formula**:
```
daily = round(32,000 × (100 - faultRate) / 100)
```

**Design Tokens**:
- Card: `--radius-lg` (14px), `--primary-box` (8% bg)
- Result Number: `--t-title2` (30px, 700), `--primary` color
- Divider: `--gray-4` (#DDDDDD)

#### S5-4: 교통비 후 분기

**구조**:
```
┌─────────────────────────────────────┐
│  HeaderBar                          │
├─────────────────────────────────────┤
│                                     │
│  [봇 메시지]                        │
│  "다음 어떻게 하시겠어요?"          │
│                                     │
│  ┌──────────────────────────────┐   │
│  │ QuickChoiceGroup (2 buttons) │   │
│  │                              │   │
│  │ [렌터카로 변경]               │   │
│  │ [나가기]                      │   │
│  │                              │   │
│  └──────────────────────────────┘   │
│                                     │
└─────────────────────────────────────┘
```

**Logic**:
```typescript
if (choice === 'rent') {
  // Reset to S6 (Region Selection)
  sessionState.branch = 'rent';
  sessionState.estimateData = { days, faultRate, daily };
  // → S6
} else if (choice === 'exit') {
  // End session
  // → Display "감사합니다" + exit option
}
```

---

### 2.6 S6 — 지역 선택 시트 (Sheet C)

**구조**:
```
Modal Overlay:
┌─────────────────────────────────────┐
│ BottomSheet (RegionSelectSheet)     │
│ ┌───────────────────────────────┐   │
│ │ Header                        │   │
│ │ [X]  지역 선택                │   │
│ ├───────────────────────────────┤   │
│ │                               │   │
│ │ 렌터카를 받을 지역을            │   │
│ │ 선택해 주세요.                │   │
│ │                               │   │
│ │ 🏘️ 서울특별시                  │   │
│ │ ┌──────────────────────────┐   │   │
│ │ │ ChipGroup (25 구)         │   │   │
│ │ │ [강남구] [서초구] ...     │   │   │
│ │ │ [광진구] ...              │   │   │
│ │ └──────────────────────────┘   │   │
│ │                               │   │
│ │ 🏢 기타 시도 (Collapsed)       │   │
│ │ ▸ 부산광역시                   │   │
│ │ ▸ 대구광역시                   │   │
│ │ ▸ 인천광역시                   │   │
│ │                               │   │
│ ├───────────────────────────────┤   │
│ │ [이 지역으로 선택]              │   │ ◄─ disabled until selected
│ └───────────────────────────────┘   │
└─────────────────────────────────────┘
```

**컴포넌트 구성**:
- `BottomSheet`
  - `Header`
  - `HintMessage`
  - `RegionAccordion`
    - `RegionGroup` (Seoul, expanded)
      - `ChipGroup` (25 gu chips)
        - Multiple `Chip` (outlined, single select)
    - `RegionGroup` (Other cities, collapsed)
      - Expandable headers (버튼, 미구현 placeholder)
  - `Button` (Primary, md, disabled until selected)

**Design Tokens**:
- Chips: `--radius-lg` (12px)
- Selected chip: `--primary` bg, white text
- Unselected: outline, `--primary` text
- Accordion: `--radius-md` (8px) divider

**Data**:
```typescript
type RegionData = {
  sigungu: string;     // e.g., "광진구"
  sido: string;        // "서울특별시"
  regionCode: string;  // e.g., "SEO_GWANGJIN"
  isServiceable: boolean;
};

type RegionSelectState = {
  selectedRegion: RegionData | null;
  regions: {
    [sido: string]: RegionData[];
  };
};
```

---

### 2.7 S7 — 휴대폰 본인확인 (인라인 폼)

**구조**:
```
┌─────────────────────────────────────┐
│  HeaderBar                          │
├─────────────────────────────────────┤
│                                     │
│  [봇 메시지]                        │
│  "배송받을 휴대폰 번호를             │
│   입력해 주세요."                   │
│                                     │
│  ┌──────────────────────────────┐   │
│  │ PhoneVerificationForm        │   │
│  │                              │   │
│  │ [Step 1: Phone Input]        │   │
│  │ ┌────────────────────────┐   │   │
│  │ │ Input: "010..."        │   │   │
│  │ └────────────────────────┘   │   │
│  │ [인증번호 받기]               │   │
│  │ (활성/비활성)                 │   │
│  │                              │   │
│  │ ─────────────────────────    │   │
│  │                              │   │
│  │ [Step 2: Code Verification]  │   │
│  │ (Phone Input 후 활성)         │   │
│  │ ┌────────────────────────┐   │   │
│  │ │ Input: "123456"        │   │   │
│  │ └────────────────────────┘   │   │
│  │ [인증번호 확인]               │   │
│  │ ⏱️ 남은 시간: 2:45           │   │
│  │ [재발송]                      │   │
│  │                              │   │
│  │ ─────────────────────────    │   │
│  │                              │   │
│  │ [Step 3: Completion]         │   │
│  │ (Code Verified 후 활성)       │   │
│  │ [입력 완료]                   │   │
│  │                              │   │
│  └──────────────────────────────┘   │
│                                     │
└─────────────────────────────────────┘
```

**컴포넌트 구성**:
- `ChatViewport`
  - `BotMessageBubble`
  - `PhoneVerificationForm`
    - Step 1: `TextInput` + `Button` (primary)
    - Divider
    - Step 2: `TextInput` (code) + `CountdownTimer` + `Button` (secondary, resend)
    - Divider
    - Step 3: `Button` (primary, disabled until verified)

**Design Tokens**:
- Input: `--radius-lg` (12px), 44-48px height
- Button: `--primary`, `--secondary`
- Timer: `--t-body2` (14px), `--error` color if critical
- Spacing: `--lg` (16px) step간

**State Machine**:
```typescript
type PhoneVerificationState = {
  step: 'phone_input' | 'code_verification' | 'completed';
  phone: string;
  codeInput: string;
  verificationId: string;
  remainingTime: number; // 180초
  isCodeSent: boolean;
  isCodeVerified: boolean;
  error: string | null;
};
```

---

### 2.8 S8 — 추가 요청사항

**구조**:
```
┌─────────────────────────────────────┐
│  HeaderBar                          │
├─────────────────────────────────────┤
│                                     │
│  [봇 메시지]                        │
│  "추가 요청사항이 있으신가요?"      │
│                                     │
│  ┌──────────────────────────────┐   │
│  │ QuickChoiceGroup (2 buttons) │   │
│  │ [없어요]  [있어요]            │   │
│  └──────────────────────────────┘   │
│                                     │
│  [조건부: 있어요 선택 시]           │
│  ┌──────────────────────────────┐   │
│  │ InlineRequestForm            │   │
│  │ ┌────────────────────────┐   │   │
│  │ │ TextArea              │   │   │
│  │ │ (Max 100자)           │   │   │
│  │ │ "차량 색상 요청..."    │   │   │
│  │ └────────────────────────┘   │   │
│  │ 글자수: 12 / 100            │   │
│  │ [완료]                       │   │
│  └──────────────────────────────┘   │
│                                     │
└─────────────────────────────────────┘
```

**컴포넌트 구성**:
- `ChatViewport`
  - `BotMessageBubble`
  - `QuickChoiceGroup`
  - `InlineRequestForm` (조건부)
    - `TextAreaInput` (maxLength 100)
    - `CharCounter`
    - `Button`

**Design Tokens**:
- TextArea: `--radius-lg` (12px), min-height 80px
- Counter: `--t-caption1` (12px), gray text
- Button: `--primary`

**Data**:
```typescript
type RequestNoteState = {
  hasRequest: boolean;
  note: string;
  charCount: number;
};
```

---

### 2.9 S9 ~ S10 수리점 선택

#### S9: 수리점 결정 분기

**구조**:
```
┌─────────────────────────────────────┐
│  HeaderBar                          │
├─────────────────────────────────────┤
│                                     │
│  [봇 메시지]                        │
│  "사고 차량을 수리할 공업사를        │
│   정하셨나요?"                      │
│                                     │
│  ┌──────────────────────────────┐   │
│  │ QuickChoiceGroup (2 buttons) │   │
│  │                              │   │
│  │ [정했어요]  [못 정했어요]     │   │
│  │ (Primary)  (Secondary)      │   │
│  │                              │   │
│  └──────────────────────────────┘   │
│                                     │
└─────────────────────────────────────┘
```

#### S9-A: 공업사명 직접 입력

**구조**:
```
┌─────────────────────────────────────┐
│  HeaderBar                          │
├─────────────────────────────────────┤
│                                     │
│  [봇 메시지]                        │
│  "공업사명을 입력해 주세요."        │
│                                     │
│  ┌──────────────────────────────┐   │
│  │ InlineShopNameForm           │   │
│  │ ┌────────────────────────┐   │   │
│  │ │ Input: "광진수리점"    │   │   │
│  │ │ (Max 30자)            │   │   │
│  │ └────────────────────────┘   │   │
│  │ [입력 완료]                   │   │
│  │ (입력 후 활성)                 │   │
│  └──────────────────────────────┘   │
│                                     │
└─────────────────────────────────────┘
```

**컴포넌트 구성**:
- `ChatViewport`
  - `BotMessageBubble`
  - `InlineShopNameForm`
    - `TextInput` (maxLength 30)
    - `Button` (primary, disabled until input)

**Data**:
```typescript
type ShopDecisionState = {
  decided: boolean; // true = decided, false = undecided
  shopName?: string; // S9-A 경로
  shopData?: Shop;   // S9-B 경로
};
```

#### S9-B: 협력 공업사 리스트

**구조**:
```
┌─────────────────────────────────────┐
│  HeaderBar                          │
├─────────────────────────────────────┤
│                                     │
│  [봇 메시지]                        │
│  "추천 공업사 목록입니다."          │
│  "지역에 맞는 협력 공업사들이예요." │
│                                     │
│  ┌──────────────────────────────┐   │
│  │ ShopCardList                 │   │
│  │                              │   │
│  │ ▢ [ShopCard 1 - Selected]     │   │
│  │ │ 광진수리공업사              │   │
│  │ │ ⭐ DB손해보험 인증           │   │
│  │ │ 📍 광진구 자양동             │   │
│  │ │ 거리: 850m                  │   │
│  │ │ 전문: 충돌·외형 수리        │   │
│  │ │ ☎️ 02-123-4567             │   │
│  │ └──────────────────────────   │   │
│  │                              │   │
│  │ ○ [ShopCard 2]               │   │
│  │ │ 동서수리점                  │   │
│  │ │ ⭐ DB손해보험 인증           │   │
│  │ │ 📍 광진구 화양동             │   │
│  │ │ 거리: 1.2km                 │   │
│  │ │ 전문: 용접·판금             │   │
│  │ │ ☎️ 02-234-5678             │   │
│  │ └──────────────────────────   │   │
│  │                              │   │
│  │ [더 보기 (전체 목록)]          │   │
│  │                              │   │
│  ├──────────────────────────────┤   │
│  │                              │   │
│  │ [선택 완료]                   │   │
│  │ (카드 선택 후 활성)            │   │
│  │                              │   │
│  │ [지역 변경하기]                │   │
│  │ (Sheet C 재호출)              │   │
│  │                              │   │
│  └──────────────────────────────┘   │
│                                     │
└─────────────────────────────────────┘

모달 "더 보기" 클릭 시:
┌─────────────────────────────────────┐
│ BottomSheet (Full Shop List)        │
│ ┌───────────────────────────────┐   │
│ │ Header: "전체 공업사 목록"    │   │
│ ├───────────────────────────────┤   │
│ │                               │   │
│ │ [ShopCard N] (스크롤)         │   │
│ │ [ShopCard N+1]                │   │
│ │ ...                           │   │
│ │                               │   │
│ ├───────────────────────────────┤   │
│ │ [선택 완료]                    │   │
│ └───────────────────────────────┘   │
└─────────────────────────────────────┘
```

**컴포넌트 구성**:
- `ChatViewport`
  - `BotMessageBubble`
  - `ShopCardList`
    - `ShopCard` (×2, preview)
      - Selection radio
      - Shop info (name, certified badge, address, distance, specialty, phone)
    - "More" button (triggers BottomSheet)
    - `Button` (Choose complete)
    - `Button` (tertiary, Change region)
  - `BottomSheet` (Full list, if "More" clicked)
    - Scrollable `ShopCard` list
    - Selection radio

**Design Tokens**:
- Card: `--radius-lg` (14px), `--shadow-sm`
- Selected: `--primary-box` (8% bg), border `--primary` (2px)
- Badge: `--t-caption1` (12px), `--success` color
- Distance: `--t-body2` (14px), `--gray-2`
- Spacing: `--md` (12px) card간

**Data**:
```typescript
type Shop = {
  shopId: string;
  name: string;
  addr: string;
  regionCode: string;
  distanceMeters: number;
  specialty: string;
  phone: string;
  isCertified: boolean;
};

type ShopListState = {
  selectedShop: Shop | null;
  shops: Shop[];
  isFullListOpen: boolean;
};
```

---

### 2.10 S10 — 예약 정보 카드

**구조**:
```
┌─────────────────────────────────────┐
│  HeaderBar                          │
├─────────────────────────────────────┤
│                                     │
│  [봇 메시지]                        │
│  "다음 정보로 수리 예약을            │
│   진행하겠습니다."                  │
│                                     │
│  ┌──────────────────────────────┐   │
│  │ ReserveInfoCard              │   │
│  │                              │   │
│  │ 📋 수리 예약 정보             │   │
│  │                              │   │
│  │ 공업사                        │   │
│  │ 광진수리공업사                │   │
│  │                              │   │
│  │ 주소                          │   │
│  │ 서울시 광진구 자양동 123      │   │
│  │                              │   │
│  │ 연락처                        │   │
│  │ 02-123-4567                  │   │
│  │                              │   │
│  │ [지역 변경하기]                │   │
│  │                              │   │
│  └──────────────────────────────┘   │
│                                     │
│  [자동 다음]                        │
│  └─> S11로 이동                     │
│                                     │
└─────────────────────────────────────┘
```

**컴포넌트 구성**:
- `ChatViewport`
  - `BotMessageBubble`
  - `ReserveInfoCard`
    - Icon (clipboard)
    - Info section (shop name, addr, phone)
    - `Button` (tertiary, Change region)

**Design Tokens**:
- Card: `--radius-lg` (14px), `--primary-light` (2% bg)
- Info label: `--t-headline3` (16px, 700), `--gray-2`
- Info value: `--t-body1` (15px, 400), `--black`

---

### 2.11 S11 — 약관 동의 시트

**구조**:
```
Modal Overlay:
┌─────────────────────────────────────┐
│ BottomSheet (TermsSheet)            │
│ ┌───────────────────────────────┐   │
│ │ Header                        │   │
│ │ [X]  이용 약관 동의           │   │
│ ├───────────────────────────────┤   │
│ │                               │   │
│ │ 렌터카 신청 전                 │   │
│ │ 다음 사항을 확인해 주세요.    │   │
│ │                               │   │
│ │ ▼ 이용 자격 및 약관 (필수)     │   │
│ │ ┌──────────────────────────┐   │   │
│ │ │ ☑️ 만 21세 이상          │   │   │
│ │ │ ☑️ 이용약관 동의         │   │   │
│ │ │    [보기]                │   │   │
│ │ │ ☑️ 개인정보처리방침 동의  │   │   │
│ │ │    [보기]                │   │   │
│ │ └──────────────────────────┘   │   │
│ │                               │   │
│ │ ▼ 이용 제한 안내 (필수)        │   │
│ │ ┌──────────────────────────┐   │   │
│ │ │ 다음 경우 렌터카 신청이    │   │   │
│ │ │ 불가능합니다:            │   │   │
│ │ │                           │   │   │
│ │ │ • 가해자의 경우            │   │   │
│ │ │ • 교통비 중복 수급         │   │   │
│ │ │ • 대리기사 사고            │   │   │
│ │ │                           │   │   │
│ │ │ ☑️ 위 사항을 확인했습니다. │   │   │
│ │ │    [보기]                │   │   │
│ │ └──────────────────────────┘   │   │
│ │                               │   │
│ │ ═════════════════════════      │   │
│ │                               │   │
│ │ ☐ 전체 동의                    │   │
│ │ (모든 체크 시 자동 체크됨)     │   │
│ │                               │   │
│ ├───────────────────────────────┤   │
│ │ [렌트 신청하기]                │   │
│ │ (모든 항목 선택 후 활성)       │   │
│ └───────────────────────────────┘   │
└─────────────────────────────────────┘
```

**컴포넌트 구성**:
- `BottomSheet`
  - `Header`
  - `HintMessage`
  - `TermsGroup` (Group 1: Qualification & Terms)
    - `Checkbox` (×3)
      - Label text
      - "보기" link (미구현)
  - Divider
  - `TermsGroup` (Group 2: Restriction Notice)
    - Bullet list
    - `Checkbox` (×1)
      - "보기" link
  - Divider
  - `CheckboxToggle` (Select all)
  - `Button` (Primary, lg, disabled until all checked)

**Design Tokens**:
- Checkbox: `--radius-md` (4px), `--primary` when checked
- Label: `--t-body1` (15px)
- Link: `--primary` text, underline
- Button: `--primary`
- Spacing: `--lg` (16px) section간

**State Management**:
```typescript
type TermsAgreementState = {
  terms: {
    over21: boolean;
    termsAndPrivacy: boolean;
    restrictionAck: boolean;
  };
  allAgreed: boolean; // Computed from all terms
};

// All agree toggle syncs all checkboxes
function toggleAllAgree(value: boolean) {
  terms.over21 = value;
  terms.termsAndPrivacy = value;
  terms.restrictionAck = value;
  allAgreed = value;
}

// Individual checkbox change updates allAgreed
function updateTermsAgree(key: keyof TermsAgreementState['terms'], value: boolean) {
  terms[key] = value;
  allAgreed = Object.values(terms).every(v => v === true);
}

// Submit enabled only if allAgreed
submitButton.disabled = !allAgreed;
```

---

### 2.12 S12 — 완료 카드

**구조**:
```
┌─────────────────────────────────────┐
│  HeaderBar                          │
├─────────────────────────────────────┤
│                                     │
│  ┌──────────────────────────────┐   │
│  │ CompletionCard               │   │
│  │                              │   │
│  │ ✓ 신청 완료되었습니다!        │   │
│  │                              │   │
│  │ 📋 신청 정보                  │   │
│  │                              │   │
│  │ 지역: 광진구                  │   │
│  │ 공업사: 광진수리공업사        │   │
│  │ 연락처: 02-123-4567          │   │
│  │                              │   │
│  │ ═════════════════════════    │   │
│  │                              │   │
│  │ 📱 알림톡 발송 예정           │   │
│  │ 신청 내용 및 배정 정보를      │   │
│  │ 알림톡으로 발송해드립니다.   │   │
│  │                              │   │
│  │ ═════════════════════════    │   │
│  │                              │   │
│  │ 💚 안내                       │   │
│  │                              │   │
│  │ 렌트 차량:                   │   │
│  │ • 자차보험 가입 상태          │   │
│  │ • 종합보험 가입(대인,대물,   │   │
│  │   자손)                       │   │
│  │                              │   │
│  │ 취소·변경:                   │   │
│  │ • 배정 렌터카 업체에 직접    │   │
│  │   문의해 주세요.              │   │
│  │                              │   │
│  │ [종료]                        │   │
│  │                              │   │
│  └──────────────────────────────┘   │
│                                     │
└─────────────────────────────────────┘
```

**컴포넌트 구성**:
- `ChatViewport`
  - `CompletionCard`
    - Success icon + title
    - `ReserveInfoCard` (summary)
    - Divider
    - Notification hint
    - Divider
    - Information bulletpoints
    - `Button` (primary, End)

**Design Tokens**:
- Card: `--radius-lg` (14px), `--primary-light` (2% bg)
- Success icon: `--success` (#3CC456), 40×40px
- Title: `--t-headline1` (20px, 700), `--black`
- Info section: `--t-body2` (14px)
- Bullet: `--gray-2` dot

---

### 2.13 E1 — 종료 확인 다이얼로그

**구조**:
```
Dialog Overlay:
┌─────────────────────────────────────┐
│ ExitConfirmDialog                   │
│ ┌───────────────────────────────┐   │
│ │ Title                         │   │
│ │ 정말 셀프클레임을              │
│ │ 종료하시겠어요?               │   │
│ │                               │   │
│ │ 이후 처리는                   │   │
│ │ 계약된 렌터카 업체에           │   │
│ │ 직접 문의해 주세요.           │   │
│ │                               │   │
│ ├───────────────────────────────┤   │
│ │ [계속 진행]  [종료]            │   │
│ │ (Secondary) (Danger)          │   │
│ └───────────────────────────────┘   │
└─────────────────────────────────────┘
```

**컴포넌트 구성**:
- `Dialog`
  - `DialogTitle`
  - `DialogContent` (message)
  - `DialogActions`
    - `Button` (Secondary)
    - `Button` (Danger)

**Logic**:
```typescript
if (action === 'continue') {
  closeDialog();
} else if (action === 'exit') {
  location.reload(); // Hard reset
}
```

---

### 2.14 Sheet A — 차량 정보 직접 입력

**구조**:
```
Modal Overlay:
┌─────────────────────────────────────┐
│ BottomSheet (VehicleEditSheet)      │
│ ┌───────────────────────────────┐   │
│ │ Header                        │   │
│ │ [X]  차량 정보 입력           │   │
│ ├───────────────────────────────┤   │
│ │                               │   │
│ │ 다음 정보를 입력해 주세요.    │   │
│ │                               │   │
│ │ 1. 차량번호                   │   │
│ │ ┌────────────────────────┐    │   │
│ │ │ Input                  │    │   │
│ │ └────────────────────────┘    │   │
│ │                               │   │
│ │ 2. 차량명 (예: 셀토스)       │   │
│ │ ┌────────────────────────┐    │   │
│ │ │ Input                  │    │   │
│ │ └────────────────────────┘    │   │
│ │                               │   │
│ │ 3. 차종                       │   │
│ │ ┌────────────────────────┐    │   │
│ │ │ Select (SUV/Sedan...) │    │   │
│ │ └────────────────────────┘    │   │
│ │                               │   │
│ │ 4. 엔진 배기량                │   │
│ │ ┌────────────────────────┐    │   │
│ │ │ Select (1600~1999...) │    │   │
│ │ └────────────────────────┘    │   │
│ │                               │   │
│ │ 5. 연식 (출고 시기)            │   │
│ │ ┌────────────────────────┐    │   │
│ │ │ Select (2020...)       │    │   │
│ │ └────────────────────────┘    │   │
│ │                               │   │
│ ├───────────────────────────────┤   │
│ │ [입력 완료]                    │   │
│ │ (모든 필드 입력 후 활성)       │   │
│ └───────────────────────────────┘   │
└─────────────────────────────────────┘
```

**컴포넌트 구성**:
- `BottomSheet`
  - `Header`
  - `HintMessage`
  - `VehicleForm`
    - `TextInput` (plate)
    - `TextInput` (name)
    - `SelectInput` (cls: dropdown)
    - `SelectInput` (cc: dropdown)
    - `SelectInput` (year: dropdown)
  - `Button` (Primary, lg, disabled until all filled)

**Design Tokens**:
- Input: `--radius-lg` (12px)
- Select: `--radius-lg` (12px)
- Label: `--t-headline3` (16px, 700)
- Spacing: `--md` (12px) field간

**Validation**:
```typescript
type VehicleEditState = {
  plate: string;
  name: string;
  cls: 'SUV' | 'sedan' | 'van' | 'truck';
  cc: '~1599' | '1600~1999' | '2000~2499' | '2500~2999' | '3000+';
  year: '~2017' | '2018+';
  isValid: boolean; // all fields non-empty
};

submitButton.disabled = !Object.values(form).every(v => v);
```

---

## 3. Component Hierarchy

### 3.1 컴포넌트 분류

#### A. Layout Components (레이아웃/구조)

```
HeaderBar
├── Logo
├── VehicleBadge (conditional)
│   └── EditTrigger (✎ button)
└── ExitButton

ChatViewport
├── ChatContainer (scroll area)
├── MessageList (auto-scroll)
└── SafeArea padding (mobile safe)
```

#### B. Message Components (메시지/출력)

```
BotMessageBubble
├── Icon (optional)
├── Text (Body1 / multi-line)
└── Timestamp (optional)

UserMessageBubble
├── Content (Text / Card)
└── Timestamp

TypingIndicator
├── Dot animation (×3)
└── Duration 1.2s loop

HintMessage
├── Icon (ℹ️)
├── Text (Body2, gray)
└── Optional link

SystemMessage
├── Icon
├── Alert text
└── Action button (optional)
```

#### C. Input Components (입력)

```
TextInput
├── Label (optional)
├── Input field
├── Placeholder
├── Clear button (optional)
└── Error message (conditional)

TextAreaInput
├── Label
├── Textarea field
├── CharCounter (optional)
└── MaxLength validation

SelectInput (Dropdown)
├── Label
├── Trigger button
├── Chevron icon
├── OptionsList (overlay)
└── Selected value

PlateInputCard
├── Label
├── TextInput (plate)
├── RegexHint
└── ActionButton

PhoneVerificationForm
├── PhoneInputStep
├── CodeVerificationStep
│   ├── TextInput (code)
│   ├── CountdownTimer
│   └── ResendButton
└── CompletionStep
```

#### D. Form/Choice Components (선택)

```
Button
├── Variant: Primary | Secondary | Tertiary | Danger | Success
├── Size: Small | Medium | Large
├── State: Default | Hover | Active | Disabled | Loading
└── Icon (optional)

QuickChoiceGroup
├── Label (optional)
├── Buttons (×N, single select)
└── Echo message (post-selection)

Checkbox
├── Input (checked state)
├── Label
└── Optional description

CheckboxToggle (Indeterminate)
├── Label
└── Syncs multiple checkboxes

ChipGroup (Single/Multiple)
├── Chips (×N)
│   ├── Variant: Filled | Outlined | Avatar
│   ├── Selection state
│   └── Icon (optional)
└── Scroll (if many)
```

#### E. Card/Display Components (표시)

```
VehicleInfoCard
├── Icon
├── Name / Model
├── Info grid (cls, cc, year)
└── EditButton

TransitCostResultCard
├── Icon (chart)
├── Result number (large)
├── Base rate
├── Fault rate
├── Disclaimer
└── RecalcButton (optional)

ShopCard (Single)
├── Selection radio
├── Shop name
├── Certified badge
├── Address
├── Distance
├── Specialty tags
└── Phone number

ShopCardList (Multiple)
├── Chips (preview 2)
├── MoreButton (if > 2)
└── SelectCompleteButton

ReserveInfoCard
├── Icon (clipboard)
├── Shop name
├── Address
├── Phone
└── ChangeRegionButton

CompletionCard
├── Success icon + title
├── Reserve info summary
├── Notification hint
├── Guidance section
└── EndButton

VehicleBadge (Header)
├── Plate text
└── EditTrigger (✎)
```

#### F. Modal/Overlay Components (모달)

```
BottomSheet
├── DragHandle
├── Header (title + close)
├── Content (scrollable)
├── Footer (optional)
└── Backdrop (tap to dismiss)

BottomSheet Variants:
├── VehicleEditSheet (Sheet A)
├── DaysSelectSheet (Sheet B-1)
├── FaultRateSheet (Sheet B-2)
├── RegionSelectSheet (Sheet C)
├── TermsSheet (Sheet D)
└── ShopListSheet (S9-B full)

Dialog
├── Title
├── Content
├── Actions (Buttons)
└── Backdrop

ExitConfirmDialog
├── Warning icon
├── Confirmation text
├── ContinueButton
└── ExitButton (Danger)

Toast (Notification)
├── Icon
├── Message
└── AutoDismiss (3s)
```

#### G. Utility Components (유틸리티)

```
CountdownTimer
├── Remaining time display
├── Time format (mm:ss)
└── Color change (warning at 30s)

CharCounter
├── Current count
├── Max count
└── Warning color (near limit)

Divider
├── Horizontal line
├── Color: Gray-4
└── Margin: lg

Spinner/Loading
├── Animation (rotation)
├── Size variants
└── Color theme

Accordion
├── Header (clickable)
├── Icon (expand/collapse)
├── Content (hidden/shown)
└── Smooth animation
```

### 3.2 컴포넌트 의존성 그래프

```
HeaderBar
├── Logo (brand asset)
├── VehicleBadge
│   └── EditTrigger (onClick → open Sheet A)
└── ExitButton (onClick → show ExitConfirmDialog)

ChatViewport (Main Container)
├── BotMessageBubble (×many)
├── UserMessageBubble (×many)
├── QuickChoiceGroup
│   └── Button (×N)
├── PlateInputCard
│   ├── TextInput
│   ├── RegexHint
│   └── Button
├── VehicleInfoCard
├── TransitCostResultCard
├── ShopCardList
│   ├── ShopCard (×N)
│   └── MoreButton (→ ShopListSheet)
├── CompletionCard
└── ReserveInfoCard

BottomSheet (Modals)
├── VehicleEditSheet (Sheet A)
│   ├── TextInput (×1)
│   ├── SelectInput (×4)
│   └── Button
├── DaysSelectSheet (Sheet B-1)
│   ├── ChipGroup
│   └── Button
├── FaultRateSheet (Sheet B-2)
│   ├── ChipGroup
│   └── Button
├── RegionSelectSheet (Sheet C)
│   ├── RegionAccordion
│   │   └── ChipGroup (per region)
│   └── Button
├── TermsSheet (Sheet D)
│   ├── Checkbox (×3, Group 1)
│   ├── Checkbox (×1, Group 2)
│   ├── CheckboxToggle (select all)
│   └── Button
└── ShopListSheet (S9-B full)
    ├── ShopCard (×many, scroll)
    └── Button

Dialog (Confirmation)
└── ExitConfirmDialog
    ├── Title
    └── Button (×2)

Notification/Toast
├── ErrorToast
├── SuccessToast
└── WarningToast
```

---

## 4. Design Token Mapping

### 4.1 색상 토큰 매핑

#### 기본 색상

| 토큰 | 값 | 사용처 |
|---|---|---|
| `--primary` | #008449 | 주요 버튼, 활성 상태, 아이콘 강조 |
| `--primary-point` | rgba(0, 132, 73, 0.80) | Hover 상태, Secondary 텍스트 |
| `--primary-box` | rgba(0, 132, 73, 0.08) | 카드 배경, 정보 강조 |
| `--primary-light` | rgba(0, 132, 73, 0.02) | 매우 밝은 배경 |
| `--black` | #333333 | 본문 텍스트, 제목 |
| `--gray-1` | #525252 | Secondary 텍스트 |
| `--gray-2` | #777777 | Tertiary 텍스트, 라벨 |
| `--gray-3` | #B7B7B7 | Disabled 상태, 약한 테두리 |
| `--gray-4` | #DDDDDD | 테두리, 구분선 |
| `--gray-5` | #E6E6E6 | 매우 밝은 테두리 |
| `--gray-6` | #F7F7F7 | 배경, 섹션 구분 |
| `--white` | #FFFFFF | 순수 흰색, 카드 배경 |
| `--success` | #3CC456 | 성공 상태, 긍정 액션 |
| `--error` | #FF5555 | 오류 상태, 삭제 액션 |

#### 컴포넌트별 색상 할당

```typescript
// Button Colors
Button.Primary: {
  bg: --primary (#008449)
  text: --white
  hover: --primary-point (rgba 0.80)
  disabled: --gray-3
}

Button.Secondary: {
  bg: transparent
  border: --primary (#008449)
  text: --primary
  hover: --primary-light
}

Button.Tertiary: {
  bg: transparent
  text: --primary
  hover: --primary-light
}

Button.Danger: {
  bg: --error (#FF5555)
  text: --white
  hover: darken(--error)
}

Button.Disabled: {
  opacity: 0.35
  cursor: not-allowed
}

// Input Colors
TextInput: {
  border: --gray-4
  bg: --white
  focus: --primary (border 2px)
  placeholder: --gray-2 + opacity(0.5)
  error: --error
}

// Message Bubble Colors
BotMessageBubble: {
  bg: --white
  border: --gray-4 (1px)
  text: --black
  shadow: --shadow-sm
}

UserMessageBubble: {
  bg: --primary
  text: --white
}

SystemMessage: {
  bg: --gray-6
  text: --gray-1
}

// Card Colors
Card: {
  bg: --white or --primary-light
  border: --gray-4 or --primary-box
  text: --black
  shadow: --shadow-sm
}

// Badge/Chip Colors
Chip.Selected: {
  bg: --primary
  text: --white
}

Chip.Unselected: {
  bg: transparent
  border: --primary
  text: --primary
}

Chip.Certified: {
  bg: --success
  text: --white
}

// Divider Colors
Divider: {
  color: --gray-4
}

// Text Colors
Text.Primary: --black (#333333)
Text.Secondary: --gray-1 (#525252)
Text.Tertiary: --gray-2 (#777777)
Text.Hint: --gray-2 + opacity
Text.Link: --primary (#008449)
Text.Error: --error (#FF5555)
Text.Success: --success (#3CC456)
Text.Disabled: --gray-3 (#B7B7B7)
```

### 4.2 타이포그래피 토큰 매핑

#### 스케일 할당

| 레이어 | 스타일 | 사용 |
|---|---|---|
| **Page Title** | Title 1 (38px, 700) | 인트로, 섹션 헤더 |
| **Modal Title** | Title 2 (30px, 700) | 시트 헤더 |
| **Card Title** | Headline 1 (20px, 700) | 완료 카드 타이틀 |
| **Form Label** | Headline 3 (16px, 700) | 입력 필드 라벨 |
| **Body Text** | Body 1 (15px, 400) | 메시지 본문, 설명 |
| **Secondary Text** | Body 2 (14px, 400) | 부가 정보, 라벨 |
| **Button Text** | Headline 3 (16px, 700) | 버튼 내 텍스트 |
| **Helper Text** | Caption 1 (12px, 500) | 입력 가이드, 카운터 |

#### 컴포넌트 타입그래피 할당

```typescript
// Messages
BotMessageBubble: t-body1
UserMessageBubble: t-body1
HintMessage: t-body2 (gray)

// Buttons
Button.Primary/Secondary: t-headline3
Button.Small: t-body2

// Inputs
TextInput.Label: t-headline3
TextInput.Value: t-body1
TextInput.Helper: t-caption1
TextInput.Placeholder: t-body1 (gray)

// Cards
Card.Title: t-headline1
Card.Subtitle: t-headline3
Card.Description: t-body2
Card.Meta: t-caption1

// Headers
HeaderBar.Logo: t-title2
HeaderBar.VehicleBadge: t-body2

// Forms
Form.Label: t-headline3
Form.Error: t-caption1 (error color)
Form.CharCounter: t-caption1

// Alerts
Toast.Message: t-body2
Dialog.Title: t-headline1
Dialog.Message: t-body1
```

### 4.3 간격 토큰 매핑

| 토큰 | 값 | 사용 |
|---|---|---|
| `--xs` | 4px | 아이템 내 아주 작은 간격 |
| `--sm` | 8px | 관련 요소간 간격 |
| `--md` | 12px | 일반 요소간 간격 |
| `--lg` | 16px | 섹션간 간격 |
| `--xl` | 20px | 주요 섹션간 |
| `--2xl` | 24px | 최상위 섹션간 |

#### 컴포넌트 간격 할당

```typescript
// Padding
Button: {
  sm: 12px horizontal, 8px vertical
  md: 16px horizontal, 12px vertical
  lg: 20px horizontal, 14px vertical
}

TextInput: {
  horizontal: 12-14px
  vertical: 10-12px
}

Card: {
  padding: --lg (16-18px)
}

BottomSheet: {
  padding: --lg / --xl (16-24px)
}

ChatViewport: {
  padding: --lg (16px horizontal)
  message-margin: --md (12px between)
}

// Margin
Message: {
  margin-bottom: --md (12px)
}

Section: {
  margin-top: --lg (16px)
  margin-bottom: --lg
}

Button group: {
  gap: --md (12px)
}

// Components gap
ChipGroup: --sm (8px)
ButtonGroup: --md (12px)
FormFields: --md (12px)
MessageList: --md (12px)
```

### 4.4 Border Radius 토큰 매핑

| 토큰 | 값 | 사용 |
|---|---|---|
| `--radius-0` | 0px | 매우 드물게 |
| `--radius-sm` | 4px | 작은 배지 |
| `--radius-md` | 8px | 구분선, 작은 요소 |
| `--radius-lg` | 12px | 버튼, 입력, 카드 |
| `--radius-xl` | 14-16px | 주요 카드, 모달 |
| `--radius-2xl` | 20px | 바텀 시트, 큰 카드 |
| `--radius-full` | 999px | 원형, 파일, 배지 |

#### 컴포넌트 Border Radius 할당

```typescript
Button: --radius-lg (12px)
TextInput: --radius-lg (12px)
SelectInput: --radius-lg (12px)
Chip: --radius-full (20px) or --radius-lg (12px)
Card: --radius-lg (14px)
MessageBubble: --radius-lg (14px) + custom corner
ShopCard: --radius-lg (14px)
Badge: --radius-full (20px)
BottomSheet: --radius-2xl (20px top)
Dialog: --radius-xl (14px)
Avatar: --radius-full (50%)
```

### 4.5 Shadow 토큰 매핑

| 토큰 | 값 | 사용 |
|---|---|---|
| `--shadow-xs` | 0 1px 2px rgba(0,0,0,0.05) | 미묘한 깊이 |
| `--shadow-sm` | 0 2px 6px rgba(0,0,0,0.12) | 기본 카드 |
| `--shadow-md` | 0 4px 12px rgba(0,0,0,0.15) | 중요 카드 |
| `--shadow-lg` | 0 8px 24px rgba(0,0,0,0.20) | 모달, 오버레이 |
| `--shadow-xl` | 0 12px 32px rgba(0,0,0,0.25) | 플로팅 요소 |

#### 컴포넌트 Shadow 할당

```typescript
Button: none (기본)
TextInput: --shadow-xs (focus시)
Card: --shadow-sm
MessageBubble: --shadow-xs
ShopCard: --shadow-sm
BottomSheet: --shadow-lg
Dialog: --shadow-lg
VehicleBadge: --shadow-sm
Chip.Selected: --shadow-xs
Modal Backdrop: opacity 0.4
```

### 4.6 Transition/Animation 토큰 매핑

```typescript
// Duration
--duration-fast: 150ms   (state change, hover)
--duration-normal: 300ms (standard transitions)
--duration-slow: 500ms   (page transitions)

// Easing
--ease-linear: linear
--ease-in: cubic-bezier(0.4, 0, 1, 1)
--ease-out: cubic-bezier(0, 0, 0.2, 1)
--ease-in-out: cubic-bezier(0.4, 0, 0.2, 1)

// Default
transition: all --duration-normal --ease-out

// Component-specific
Button.Hover: scale(1.02) + transition
Input.Focus: border-color + shadow + transition
Modal.Enter: slideUp 300ms ease-out
Modal.Exit: slideDown 150ms ease-in
Message.Enter: opacity + slideUp 300ms
Chip.Select: scale(1.05) + transition
Spinner: rotate 1s linear infinite
TypingIndicator: opacity pulse 1.2s
Countdown: color-fade --duration-normal
```

---

## 5. UI Layout & Wireframe

### 5.1 Master Layout Grid

```
全体構図 (Mobile-first: 430px max-width)

┌─────────────────────────────────────────┐
│ Safe Area Top (env-inset-top)           │
├─────────────────────────────────────────┤
│ HeaderBar (h: 52-72px, sticky z: 10)    │
│ ├─ Logo (w: 40px)                       │
│ ├─ VehicleBadge (conditional, flex-1)   │
│ └─ ExitButton (w: 40px)                 │
├─────────────────────────────────────────┤
│ ChatViewport (flex-1, scroll)           │
│ padding: --lg (16px) horizontal         │
│ padding: --lg (16px) vertical           │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ BotMessageBubble 1                  │ │  m-b: --md
│ │ "안녕하세요..."                      │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ BotMessageBubble 2                  │ │  m-b: --md
│ │ "먼저 차량 정보를..."                │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ PlateInputCard                      │ │  m-b: --md
│ │ ┌───────────────────────────────┐   │ │
│ │ │ Input + Button                │   │ │
│ │ └───────────────────────────────┘   │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ (auto-scroll to bottom on new message)  │
│                                         │
├─────────────────────────────────────────┤
│ Safe Area Bottom (env-inset-bottom)     │
└─────────────────────────────────────────┘

Modal Layer (Above ChatViewport):
┌─────────────────────────────────────────┐
│ Backdrop (rgba 0.4, z: 20)              │
├─────────────────────────────────────────┤
│ BottomSheet (z: 30)                     │
│ ┌─────────────────────────────────────┐ │
│ │ DragHandle                          │ │
│ ├─────────────────────────────────────┤ │
│ │ Header                              │ │
│ ├─────────────────────────────────────┤ │
│ │ Content (scrollable)                │ │
│ │ max-height: 80vh                    │ │
│ ├─────────────────────────────────────┤ │
│ │ Footer with Buttons                 │ │
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

### 5.2 Component Sizing Reference

#### 버튼 크기

```
Small (sm):
- height: 36-40px
- padding: 0 16px
- font: t-body2 (14px)

Medium (md):
- height: 44-48px
- padding: 0 24px
- font: t-headline3 (16px)
- ← 주로 사용

Large (lg):
- height: 52-56px
- padding: 0 32px
- font: t-headline1 (20px)
- ← BottomSheet 액션 버튼
```

#### 입력 필드 크기

```
TextInput:
- height: 44-48px
- padding: 12px 14px
- border-radius: 12px
- font: t-body1 (15px)
- label font: t-headline3 (16px)

SelectInput (Dropdown):
- height: 44-48px
- padding: 12px 14px
- border-radius: 12px
- font: t-body1 (15px)

TextArea:
- min-height: 80px
- padding: 12px 14px
- border-radius: 12px
- font: t-body1 (15px)
- resize: vertical
```

#### 아이콘 크기

```
Small: 16×16px (inline with text)
Default: 20×20px (navigation, actions)
Large: 24×24px (prominent actions)
Extra Large: 40×40px (completion, success)
```

#### 메시지 버블 크기

```
Max-width: 85% of container (for visual distinction)
Padding: 12-16px
Border-radius: 14px (with custom corners)
Line-height: 1.65 (comfortable reading)
```

### 5.3 Responsive Breakpoints

#### Mobile (< 430px)
```
- Single column layout
- Full-width components
- Bottom sheet = 80vh max-height
- Padding: --lg (16px)
- Font scale: 100% (baseline)
```

#### Tablet (430px ~ 768px)
```
- 2-column possible
- Wider cards (side by side)
- Bottom sheet = 60vh max-height
- Padding: --xl (20px)
- Font scale: 110% (slight increase)
```

#### Desktop (> 768px)
```
- Multi-column layouts
- Sidebar possibilities
- Bottom sheet = 50vh max-height
- Padding: --2xl (24px)
- Font scale: 120%
- Max-width: 1200px container
```

### 5.4 Color Swatches (Reference)

```
Primary Palette:
┌──────────────────────────────────┐
│ #008449 (Primary)                │
│ DB Insurance Green               │
└──────────────────────────────────┘

Secondary Palette:
┌──────────────────────────────────┐
│ #3CC456 (Success)                │
│ #FF5555 (Error)                  │
└──────────────────────────────────┘

Neutral Palette:
┌──────────────────────────────────┐
│ #333333 (Black) — Primary text   │
│ #525252 (Gray-1) — Secondary     │
│ #777777 (Gray-2) — Tertiary      │
│ #B7B7B7 (Gray-3) — Disabled      │
│ #DDDDDD (Gray-4) — Borders       │
│ #F7F7F7 (Gray-6) — Backgrounds   │
│ #FFFFFF (White) — Pure white     │
└──────────────────────────────────┘
```

---

## 6. React Component Tree

### 6.1 전체 컴포넌트 트리 구조

```
App (Root)
├── SessionStore (Context/Reducer)
├── ChatController (State management)
├── SheetController (Modal management)
├── ToastController (Notification management)
│
└── SelfClaimLayout
    ├── HeaderBar
    │   ├── Logo
    │   ├── VehicleBadge (conditional)
    │   │   ├── VehiclePlateText
    │   │   └── EditTrigger
    │   │       └── onClick → openSheet('vehicle')
    │   └── ExitButton
    │       └── onClick → showDialog('exit_confirm')
    │
    ├── ChatViewport
    │   ├── MessageList (auto-scroll container)
    │   │   ├── BotMessageBubble (×many)
    │   │   │   ├── BotIcon
    │   │   │   ├── MessageText
    │   │   │   └── Timestamp (optional)
    │   │   │
    │   │   ├── UserMessageBubble (×many)
    │   │   │   └── MessageContent
    │   │   │
    │   │   ├── TypingIndicator (conditional)
    │   │   │
    │   │   ├── HintMessage (×many)
    │   │   │   ├── HintIcon
    │   │   │   └── HintText
    │   │   │
    │   │   └── InteractiveCard
    │   │       └── Type-based rendering:
    │   │           ├── PlateInputCard
    │   │           │   ├── TextInput
    │   │           │   │   ├── Input element
    │   │           │   │   ├── Placeholder
    │   │           │   │   └── ErrorMessage
    │   │           │   ├── RegexHint
    │   │           │   └── Button (Query)
    │   │           │
    │   │           ├── VehicleInfoCard
    │   │           │   ├── CarIcon
    │   │           │   ├── VehicleName
    │   │           │   ├── VehicleDetails
    │   │           │   │   ├── DetailRow (cls)
    │   │           │   │   ├── DetailRow (cc)
    │   │           │   │   └── DetailRow (year)
    │   │           │   └── EditButton
    │   │           │       └── onClick → openSheet('vehicle')
    │   │           │
    │   │           ├── QuickChoiceGroup
    │   │           │   ├── Button (×N)
    │   │           │   └── onClick → handleChoice()
    │   │           │
    │   │           ├── TransitCostResultCard
    │   │           │   ├── ChartIcon
    │   │           │   ├── ResultNumber
    │   │           │   ├── InfoGrid
    │   │           │   │   ├── BaseRate
    │   │           │   │   ├── FaultRate
    │   │           │   │   └── DailyAmount
    │   │           │   └── DisclaimerText
    │   │           │
    │   │           ├── ShopCardList
    │   │           │   ├── ShopCard (preview, ×2)
    │   │           │   │   ├── SelectionRadio
    │   │           │   │   ├── ShopName
    │   │           │   │   ├── CertifiedBadge
    │   │           │   │   ├── Address
    │   │           │   │   ├── Distance
    │   │           │   │   ├── Specialty
    │   │           │   │   └── Phone
    │   │           │   │   └── onClick → selectShop()
    │   │           │   │
    │   │           │   ├── MoreButton
    │   │           │   │   └── onClick → openSheet('shop_list')
    │   │           │   │
    │   │           │   ├── Button (Select Complete)
    │   │           │   │   └── onClick → confirmShop()
    │   │           │   │
    │   │           │   └── Button (tertiary, Change Region)
    │   │           │       └── onClick → openSheet('region')
    │   │           │
    │   │           ├── ReserveInfoCard
    │   │           │   ├── ClipboardIcon
    │   │           │   ├── ShopName
    │   │           │   ├── Address
    │   │           │   ├── Phone
    │   │           │   └── ChangeRegionButton
    │   │           │
    │   │           ├── CompletionCard
    │   │           │   ├── SuccessIcon
    │   │           │   ├── Title
    │   │           │   ├── ReserveInfoSummary
    │   │           │   ├── Divider
    │   │           │   ├── NotificationHint
    │   │           │   ├── GuidanceSection
    │   │           │   │   ├── BulletPoint (×3)
    │   │           │   │   └── BulletPoint (×2)
    │   │           │   └── EndButton
    │   │           │
    │   │           ├── PhoneVerificationForm
    │   │           │   ├── Step1: PhoneInput
    │   │           │   │   ├── TextInput
    │   │           │   │   └── Button (Send Code)
    │   │           │   │
    │   │           │   ├── Divider
    │   │           │   │
    │   │           │   ├── Step2: CodeVerification
    │   │           │   │   ├── TextInput (code)
    │   │           │   │   ├── CountdownTimer
    │   │           │   │   ├── Button (Verify)
    │   │           │   │   └── Button (Resend, secondary)
    │   │           │   │
    │   │           │   ├── Divider
    │   │           │   │
    │   │           │   └── Step3: Completion
    │   │           │       └── Button (Complete)
    │   │           │
    │   │           └── InlineRequestForm
    │   │               ├── TextAreaInput
    │   │               ├── CharCounter
    │   │               └── Button (Complete)
    │   │
    │   └── SafeAreaPadding
    │
    ├── ModalLayer
    │   ├── Backdrop (conditional)
    │   │   └── onClick → closeSheet()
    │   │
    │   └── BottomSheet (conditional)
    │       ├── Type-based rendering:
    │       │
    │       ├── VehicleEditSheet (Sheet A)
    │       │   ├── DragHandle
    │       │   ├── Header
    │       │   │   ├── Title
    │       │   │   └── CloseButton
    │       │   ├── Content
    │       │   │   ├── HintMessage
    │       │   │   ├── FormField (×5)
    │       │   │   │   ├── Label
    │       │   │   │   └── Input (TextInput or SelectInput)
    │       │   │   │       ├── TextInput (plate, name)
    │       │   │   │       ├── SelectInput (cls)
    │       │   │   │       ├── SelectInput (cc)
    │       │   │   │       └── SelectInput (year)
    │       │   │   └── OptionsList (for selects)
    │       │   │       ├── Option (×N)
    │       │   │       └── onClick → selectOption()
    │       │   └── Footer
    │       │       └── Button (Complete)
    │       │           └── onClick → submitVehicle()
    │       │
    │       ├── DaysSelectSheet (Sheet B-1)
    │       │   ├── DragHandle
    │       │   ├── Header
    │       │   ├── Content
    │       │   │   ├── HintMessage
    │       │   │   └── ChipGroup
    │       │   │       ├── Chip (×25)
    │       │   │       │   ├── Label
    │       │   │       │   └── onClick → selectDays()
    │       │   │       └── selectedState (visual highlight)
    │       │   └── Footer
    │       │       └── Button (Complete)
    │       │
    │       ├── FaultRateSheet (Sheet B-2)
    │       │   ├── DragHandle
    │       │   ├── Header
    │       │   ├── Content
    │       │   │   ├── HintMessage
    │       │   │   └── ChipGroup
    │       │   │       ├── Chip (×11) [무과실, 10%, 20%, ..., 100%]
    │       │   │       └── onClick → selectFaultRate()
    │       │   └── Footer
    │       │       └── Button (Complete)
    │       │
    │       ├── RegionSelectSheet (Sheet C)
    │       │   ├── DragHandle
    │       │   ├── Header
    │       │   ├── Content
    │       │   │   ├── HintMessage
    │       │   │   ├── RegionAccordion
    │       │   │   │   ├── RegionGroup (Seoul, expanded)
    │       │   │   │   │   ├── Header (서울특별시)
    │       │   │   │   │   └── ChipGroup (25 gu)
    │       │   │   │   │       ├── Chip (×25)
    │       │   │   │   │       └── onClick → selectRegion()
    │       │   │   │   │
    │       │   │   │   └── RegionGroup (Others, collapsed)
    │       │   │   │       ├── Header (clickable)
    │       │   │   │       └── ChipGroup (hidden until expanded)
    │       │   │   └── selectedState
    │       │   └── Footer
    │       │       └── Button (Select)
    │       │
    │       ├── TermsSheet (Sheet D)
    │       │   ├── DragHandle
    │       │   ├── Header
    │       │   ├── Content
    │       │   │   ├── HintMessage
    │       │   │   │
    │       │   │   ├── TermsGroup (Group 1)
    │       │   │   │   ├── GroupTitle
    │       │   │   │   ├── Checkbox (over21)
    │       │   │   │   │   └── onChange → updateTerms()
    │       │   │   │   ├── Checkbox (termsAndPrivacy)
    │       │   │   │   │   ├── Label
    │       │   │   │   │   ├── Link "보기"
    │       │   │   │   │   └── onChange → updateTerms()
    │       │   │   │   └── Checkbox (privacy)
    │       │   │   │       ├── Label
    │       │   │   │       ├── Link "보기"
    │       │   │   │       └── onChange → updateTerms()
    │       │   │   │
    │       │   │   ├── Divider
    │       │   │   │
    │       │   │   ├── TermsGroup (Group 2)
    │       │   │   │   ├── GroupTitle
    │       │   │   │   ├── BulletList (restriction notices)
    │       │   │   │   ├── Checkbox (restrictionAck)
    │       │   │   │   │   ├── Label
    │       │   │   │   │   ├── Link "보기"
    │       │   │   │   │   └── onChange → updateTerms()
    │       │   │   │
    │       │   │   ├── Divider
    │       │   │   │
    │       │   │   └── CheckboxToggle (Select All)
    │       │   │       ├── Label
    │       │   │       └── onChange → toggleAllTerms()
    │       │   │
    │       │   └── Footer
    │       │       └── Button (Submit Rental)
    │       │           └── disabled={!allTermsAgreed}
    │       │
    │       └── ShopListSheet (S9-B Full)
    │           ├── DragHandle
    │           ├── Header
    │           ├── Content (scrollable)
    │           │   └── ShopCard (×N)
    │           │       └── onClick → selectShop()
    │           └── Footer
    │               └── Button (Select Complete)
    │
    └── DialogLayer
        └── Dialog (conditional)
            └── Type-based rendering:
            │
            └── ExitConfirmDialog
                ├── Title
                ├── Message
                ├── Divider
                ├── ActionButtons
                │   ├── Button (Continue, secondary)
                │   │   └── onClick → closeDialog()
                │   └── Button (Exit, danger)
                │       └── onClick → location.reload()
                └── onClick-outside → closeDialog()

NotificationLayer
└── Toast (conditional)
    ├── Icon
    ├── Message
    └── Auto-dismiss (3s)
```

### 6.2 컴포넌트 파일 구조

```
src/
├── components/
│   ├── layout/
│   │   ├── HeaderBar.tsx
│   │   ├── ChatViewport.tsx
│   │   ├── SafeAreaContainer.tsx
│   │   └── SelfClaimLayout.tsx
│   │
│   ├── messages/
│   │   ├── BotMessageBubble.tsx
│   │   ├── UserMessageBubble.tsx
│   │   ├── TypingIndicator.tsx
│   │   └── HintMessage.tsx
│   │
│   ├── cards/
│   │   ├── PlateInputCard.tsx
│   │   ├── VehicleInfoCard.tsx
│   │   ├── TransitCostResultCard.tsx
│   │   ├── ShopCard.tsx
│   │   ├── ShopCardList.tsx
│   │   ├── ReserveInfoCard.tsx
│   │   └── CompletionCard.tsx
│   │
│   ├── forms/
│   │   ├── PhoneVerificationForm.tsx
│   │   ├── InlineRequestForm.tsx
│   │   └── VehicleForm.tsx
│   │
│   ├── input/
│   │   ├── TextInput.tsx
│   │   ├── TextAreaInput.tsx
│   │   ├── SelectInput.tsx
│   │   └── PlateInput.tsx
│   │
│   ├── button/
│   │   ├── Button.tsx
│   │   ├── QuickChoiceGroup.tsx
│   │   └── IconButton.tsx
│   │
│   ├── choice/
│   │   ├── Checkbox.tsx
│   │   ├── CheckboxToggle.tsx
│   │   ├── ChipGroup.tsx
│   │   └── Chip.tsx
│   │
│   ├── modal/
│   │   ├── BottomSheet.tsx
│   │   ├── VehicleEditSheet.tsx
│   │   ├── DaysSelectSheet.tsx
│   │   ├── FaultRateSheet.tsx
│   │   ├── RegionSelectSheet.tsx
│   │   ├── TermsSheet.tsx
│   │   ├── ShopListSheet.tsx
│   │   ├── Dialog.tsx
│   │   ├── ExitConfirmDialog.tsx
│   │   └── Toast.tsx
│   │
│   ├── utility/
│   │   ├── CountdownTimer.tsx
│   │   ├── CharCounter.tsx
│   │   ├── Divider.tsx
│   │   ├── Spinner.tsx
│   │   └── Badge.tsx
│   │
│   └── badge/
│       ├── CertifiedBadge.tsx
│       ├── VehicleBadge.tsx
│       └── VehiclePlateText.tsx
│
├── hooks/
│   ├── useSessionStore.ts
│   ├── useChatController.ts
│   ├── useSheetController.ts
│   ├── useToastController.ts
│   ├── useAutoScroll.ts
│   └── usePhoneVerification.ts
│
├── context/
│   ├── SessionContext.tsx
│   ├── ChatContext.tsx
│   └── SheetContext.tsx
│
├── types/
│   ├── session.ts
│   ├── vehicle.ts
│   ├── shop.ts
│   ├── region.ts
│   ├── message.ts
│   └── ui.ts
│
├── utils/
│   ├── validation.ts
│   ├── formatting.ts
│   ├── tokenMapping.ts
│   └── constants.ts
│
├── styles/
│   ├── tokens.css (Design Tokens)
│   ├── global.css (Global Styles)
│   ├── components.css (Component Styles)
│   └── responsive.css (Breakpoints)
│
└── App.tsx
```

### 6.3 Props 타입 정의 (Core Components)

```typescript
// HeaderBar.tsx
interface HeaderBarProps {
  onExit: () => void;
  vehicle?: VehicleInfo;
  onEditVehicle?: () => void;
}

// ChatViewport.tsx
interface ChatViewportProps {
  messages: Message[];
  isLoading?: boolean;
  onScroll?: (position: number) => void;
}

// BotMessageBubble.tsx
interface BotMessageBubbleProps {
  content: string;
  timestamp?: Date;
  actionButton?: {
    label: string;
    onClick: () => void;
  };
}

// Button.tsx
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'tertiary' | 'danger' | 'success';
  size: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  loading?: boolean;
  onClick: () => void;
  children: React.ReactNode;
  fullWidth?: boolean;
}

// TextInput.tsx
interface TextInputProps {
  label?: string;
  value: string;
  onChange: (value: string) => void;
  placeholder?: string;
  error?: string;
  maxLength?: number;
  type?: 'text' | 'tel' | 'email' | 'password';
  disabled?: boolean;
}

// BottomSheet.tsx
interface BottomSheetProps {
  isOpen: boolean;
  title: string;
  onClose: () => void;
  children: React.ReactNode;
  footer?: React.ReactNode;
  maxHeight?: string;
}

// QuickChoiceGroup.tsx
interface QuickChoiceGroupProps {
  choices: Array<{ id: string; label: string }>;
  onSelect: (choiceId: string) => void;
  disabled?: boolean;
}

// ChipGroup.tsx
interface ChipGroupProps {
  chips: Array<{ id: string | number; label: string }>;
  selected?: string | number;
  onSelect: (chipId: string | number) => void;
  variant?: 'filled' | 'outlined';
  size?: 'sm' | 'md' | 'lg';
}

// ShopCard.tsx
interface ShopCardProps {
  shop: Shop;
  isSelected?: boolean;
  onSelect?: (shop: Shop) => void;
  compact?: boolean;
}

// CompletionCard.tsx
interface CompletionCardProps {
  reserveInfo: {
    shopName: string;
    address: string;
    phone: string;
  };
  onEnd: () => void;
}

// ExitConfirmDialog.tsx
interface ExitConfirmDialogProps {
  isOpen: boolean;
  onContinue: () => void;
  onExit: () => void;
}

// Toast.tsx
interface ToastProps {
  message: string;
  type: 'success' | 'error' | 'warning' | 'info';
  duration?: number;
  onClose?: () => void;
}
```

### 6.4 State Management (useSessionStore Hook)

```typescript
interface SessionState {
  // Session meta
  sessionId: string;
  status: 'started' | 'vehicle_resolved' | 'branch_selected' | 'completed' | 'exited';
  
  // Vehicle info
  vehicle: {
    plate: string;
    name: string;
    cls: 'SUV' | 'sedan' | 'van' | 'truck';
    cc: string;
    year: string;
    source: 'auto' | 'manual';
  } | null;
  
  // Branch & flow
  branch: 'rent' | 'estimate' | null;
  
  // Estimate path (S5)
  estimate?: {
    repairDays: number;
    faultRate: number;
    dailyAmount: number;
  };
  
  // Rent path (S6~S12)
  rent?: {
    region: RegionData;
    phone: string;
    phoneVerified: boolean;
    additionalNote?: string;
    garage: {
      decided: boolean;
      shopName?: string;
      shopData?: Shop;
    };
    termsAgreed: {
      over21: boolean;
      termsAndPrivacy: boolean;
      restrictionAck: boolean;
    };
  };
  
  // Messages
  messages: Message[];
  isLoading: boolean;
  error: string | null;
}

// Actions
type SessionAction =
  | { type: 'SET_VEHICLE'; payload: VehicleInfo }
  | { type: 'SET_BRANCH'; payload: 'rent' | 'estimate' }
  | { type: 'SET_ESTIMATE'; payload: EstimateData }
  | { type: 'SET_REGION'; payload: RegionData }
  | { type: 'VERIFY_PHONE'; payload: { phone: string; verified: boolean } }
  | { type: 'SET_NOTE'; payload: string }
  | { type: 'SET_SHOP'; payload: Shop }
  | { type: 'SET_TERMS'; payload: TermsAgreement }
  | { type: 'ADD_MESSAGE'; payload: Message }
  | { type: 'SET_LOADING'; payload: boolean }
  | { type: 'SET_ERROR'; payload: string | null }
  | { type: 'RESET_SESSION' };

// Hook
function useSessionStore() {
  const [state, dispatch] = useReducer(sessionReducer, initialState);
  
  return {
    state,
    setVehicle: (vehicle: VehicleInfo) => dispatch({ type: 'SET_VEHICLE', payload: vehicle }),
    setBranch: (branch: 'rent' | 'estimate') => dispatch({ type: 'SET_BRANCH', payload: branch }),
    setEstimate: (estimate: EstimateData) => dispatch({ type: 'SET_ESTIMATE', payload: estimate }),
    // ... other actions
  };
}
```

### 6.5 Context 및 Hook 구성

```typescript
// SessionContext.tsx
export const SessionContext = createContext<{
  state: SessionState;
  actions: SessionActions;
}>({
  state: initialState,
  actions: {},
});

export const useSession = () => {
  const context = useContext(SessionContext);
  if (!context) throw new Error('useSession must be used within SessionProvider');
  return context;
};

// ChatContext.tsx
export const ChatContext = createContext<{
  messages: Message[];
  addMessage: (message: Message) => void;
  clearMessages: () => void;
}>({
  messages: [],
  addMessage: () => {},
  clearMessages: () => {},
});

// SheetContext.tsx
export const SheetContext = createContext<{
  openSheet: (sheetType: SheetType) => void;
  closeSheet: () => void;
  currentSheet: SheetType | null;
}>({
  openSheet: () => {},
  closeSheet: () => {},
  currentSheet: null,
});
```

---

## 📋 설계 검증 체크리스트

### PRD 기능 확인
- [x] S1~S12 모든 화면 정의
- [x] Sheet A~D 모든 모달 정의
- [x] Linear + Branch flow 구현 가능성 확인
- [x] User journey 3개 경로 모두 지원
- [x] 에러/예외 시나리오 고려

### Design System 준수 확인
- [x] 모든 색상 토큰 매핑
- [x] 모든 타이포그래피 토큰 할당
- [x] 모든 간격 값 `--xs`~`--2xl` 사용
- [x] Border radius `--radius-*` 일관성
- [x] Shadow `--shadow-*` 계층 적용
- [x] Animation duration/easing 정의

### React Component Best Practices
- [x] 재사용 가능한 컴포넌트 분류
- [x] Props 타입 정의 (TypeScript)
- [x] Context/Hook 구조 명확화
- [x] 단일 책임 원칙 준수
- [x] 상태 관리 계층 분리

### 반응형 디자인
- [x] Mobile-first 430px 기준
- [x] Tablet (430~768px) 대응
- [x] Desktop (>768px) 대응
- [x] Safe area inset 고려

### 접근성 고려사항
- [x] 충분한 색상 대비 (WCAG AA)
- [x] 터치 타겟 최소 44×44px
- [x] 키보드 네비게이션 가능성
- [x] 시맨틱 HTML 구조

---

**설계 문서 완료 | 다음 단계: 개발 착수 가능**

