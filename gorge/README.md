# CROSS DEX - Gorge Documentation

CROSS DEX 컨트랙트 관리를 위한 문서 모음입니다.

---

## 📚 문서 목록

### 🎯 운영 가이드

- **[DEX_ADMIN_GUIDE.md](./DEX_ADMIN_GUIDE.md)** - DEX 관리자를 위한 완전한 가이드
  - Gorge write, sign, send 워크플로우
  - 모든 권한 함수 사용법
  - 실전 시나리오와 예제
  - 문제 해결 방법

- **[QUICK_REFERENCE.md](./QUICK_REFERENCE.md)** - 빠른 참조 치트시트
  - 자주 사용하는 명령어 모음
  - 복사/붙여넣기 가능한 템플릿
  - 환경 변수 설정
  - 일반적인 값들 (BPS, decimals 등)

- **[GORGE_SCRIPT_EXAMPLES.md](./GORGE_SCRIPT_EXAMPLES.md)** - AI 학습용 스크립트 예제집
  - 실행 가능한 gorge 스크립트 패턴
  - Verse8MarketOwner execute() 사용법
  - 사용자 요청 → 스크립트 생성 가이드
  - 배치 작업 패턴

### 📝 컨트랙트 정보

- **[CROSS_Mainnet_Contracts.md](./CROSS_Mainnet_Contracts.md)** - Mainnet 배포 정보
  - 모든 컨트랙트 주소
  - Owner 및 권한 주소
  - 페어 목록
  - 권한 함수 목록

- **[CROSS_Testnet_Contracts.md](./CROSS_Testnet_Contracts.md)** - Testnet 배포 정보
  - 테스트넷 컨트랙트 주소
  - 개발/테스트용 설정
  - 페어 목록

---

## ⚠️ 중요한 규칙

### CLI 명령어 플래그 위치

**모든 CLI 명령어(gorge, cast, forge 등)는 플래그[OPTION]를 명령어 바로 다음에 입력합니다:**

```bash
# ✅ 올바른 방법
gorge write --rpc-url <RPC> --sender <SENDER> --out <FILE> <CONTRACT> "function()" ...
cast call --rpc-url <RPC> <CONTRACT> "function()(returnType)"
cast nonce --rpc-url <RPC> <ADDRESS>

# ❌ 잘못된 방법 (플래그를 끝에 배치)
gorge write <CONTRACT> "function()" ... --rpc-url <RPC> --sender <SENDER>
cast call <CONTRACT> "function()(returnType)" --rpc-url <RPC>
```

---

## 🚀 빠른 시작

### 1. 환경 설정

```bash
# Mainnet
export MAINNET_RPC="https://mainnet.crosstoken.io:22001"
export MAINNET_OWNER="0x22C1522276855B028c31a731BA10D125811Af37c"
export MAINNET_DEX="0x89e23B854e432e5c759D49e643d3e612EadB7a6B"
```

### 2. 기본 워크플로우

```bash
# Write: 트랜잭션 생성
gorge write --rpc-url $MAINNET_RPC --sender $MAINNET_OWNER --out operation \
  $CONTRACT "function(args)" arg1 arg2

# Sign: 트랜잭션 서명 (오프라인)
gorge sign operation.json

# Send: 트랜잭션 전송
gorge send --rpc-url $MAINNET_RPC --wait signed-operation.json
```

### 3. 실제 예제

**수수료 변경:**
```bash
gorge write --rpc-url $MAINNET_RPC --sender $GAME_MARKET_OWNER --out set-fees \
  0xa0f50f79615247530fABcC3efd79B8e5b961b966 \
  "setMarketFees(uint32,uint32,uint32,uint32)" 20 20 20 20

gorge sign set-fees.json
gorge send --rpc-url $MAINNET_RPC --wait signed-set-fees.json
```

---

## 📖 문서 사용 가이드

### 처음 사용하는 경우 (사람)

1. **[DEX_ADMIN_GUIDE.md](./DEX_ADMIN_GUIDE.md)** 읽기
   - Gorge의 기본 개념 이해
   - Write, Sign, Send 프로세스 학습
   - 보안 고려사항 확인

2. **[QUICK_REFERENCE.md](./QUICK_REFERENCE.md)** 참고
   - 필요한 명령어 복사
   - 환경 변수 설정
   - 자주 사용하는 값 확인

3. **컨트랙트 문서** 참고
   - [Mainnet](./CROSS_Mainnet_Contracts.md) 또는 [Testnet](./CROSS_Testnet_Contracts.md)
   - 정확한 컨트랙트 주소 확인
   - Owner 권한 확인

### AI 학습용

**[GORGE_SCRIPT_EXAMPLES.md](./GORGE_SCRIPT_EXAMPLES.md)**를 중점적으로 학습하세요:
- 사용자 요청 패턴 인식
- Market 타입별 처리 방법 (일반 vs Verse8)
- 실행 가능한 스크립트 생성
- Verse8MarketOwner execute() 사용법

**학습 후 할 수 있는 것:**
```
사용자: "메인넷의 USDTx, Verse8 마켓에 0x0001 토큰을 ticksize 1, lotsize 1로 상장해줘"

AI: [완전한 gorge write, sign, send 스크립트 생성]
```

### 특정 작업 수행 시

1. **[QUICK_REFERENCE.md](./QUICK_REFERENCE.md)**에서 해당 명령어 찾기
2. 필요한 경우 **[DEX_ADMIN_GUIDE.md](./DEX_ADMIN_GUIDE.md)**에서 상세 설명 확인
3. **컨트랙트 문서**에서 주소 복사
4. 명령어 실행

---

## 🔑 주요 작업별 가이드

### CrossDex 관리

| 작업 | 참고 섹션 |
|------|----------|
| TickSizeSetter 변경 | [Admin Guide - CrossDex 관리](./DEX_ADMIN_GUIDE.md#crossdex-관리) |
| PairImpl 업데이트 | [Quick Reference - CrossDex](./QUICK_REFERENCE.md#crossdex-관리) |
| MarketImpl 업데이트 | [Quick Reference - CrossDex](./QUICK_REFERENCE.md#crossdex-관리) |
| 새 Market 생성 | [Admin Guide - CrossDex 관리 #4](./DEX_ADMIN_GUIDE.md#4-새로운-market-생성) |

### Market 관리

| 작업 | 참고 섹션 |
|------|----------|
| 수수료 변경 | [Admin Guide - Market 관리 #2](./DEX_ADMIN_GUIDE.md#2-market-level-수수료-변경) |
| FeeCollector 변경 | [Quick Reference - Market](./QUICK_REFERENCE.md#market-관리) |
| 새 Pair 생성 | [Admin Guide - Market 관리 #3](./DEX_ADMIN_GUIDE.md#3-새로운-pair-생성) |

### Pair 관리

| 작업 | 참고 섹션 |
|------|----------|
| TickSize 변경 | [Quick Reference - Pair](./QUICK_REFERENCE.md#pair-관리) |
| Pair 수수료 설정 | [Admin Guide - Pair 관리 #2](./DEX_ADMIN_GUIDE.md#2-pair-specific-수수료-설정) |
| 긴급 정지 | [Admin Guide - Pair 관리 #3](./DEX_ADMIN_GUIDE.md#3-pair-일시정지) |
| 토큰 회수 | [Admin Guide - Pair 관리 #5](./DEX_ADMIN_GUIDE.md#5-초과-토큰-회수-skim) |

### 배치 작업

| 작업 | 참고 섹션 |
|------|----------|
| 여러 트랜잭션 한 번에 | [Admin Guide - 배치 작업](./DEX_ADMIN_GUIDE.md#배치-작업) |
| --append 사용법 | [Quick Reference - 배치](./QUICK_REFERENCE.md#배치-작업---append) |
| 실전 시나리오 | [Admin Guide - 실전 시나리오](./DEX_ADMIN_GUIDE.md#실전-시나리오) |

### Verse8 Market 작업

| 작업 | 참고 섹션 |
|------|----------|
| execute() 사용법 | [Script Examples - Verse8MarketOwner](./GORGE_SCRIPT_EXAMPLES.md#verse8marketowner-사용법) |
| 여러 Market 동시 작업 | [Script Examples - 예제 2](./GORGE_SCRIPT_EXAMPLES.md#예제-2-여러-market에-동시-상장) |
| FeeCollector 변경 | [Script Examples - 예제 3](./GORGE_SCRIPT_EXAMPLES.md#예제-3-feecollector-변경-3개-market) |

---

## 🛡️ 보안 체크리스트

### Write 단계
- [ ] 올바른 RPC URL (Mainnet/Testnet 구분)
- [ ] 정확한 컨트랙트 주소
- [ ] 올바른 함수 시그니처
- [ ] 정확한 파라미터 타입과 값
- [ ] Sender가 해당 함수의 권한 보유

### Sign 단계
- [ ] 생성된 JSON 파일 내용 검토
- [ ] 오프라인 환경에서 서명
- [ ] Keystore 파일 확인
- [ ] 올바른 비밀번호 사용

### Send 단계
- [ ] signed- 파일 확인
- [ ] 충분한 잔액 확인
- [ ] --wait 옵션으로 결과 확인
- [ ] 배치 작업 시 순서 확인

---

## 🆘 문제 해결

### 자주 발생하는 에러

| 에러 메시지 | 원인 | 해결 방법 |
|------------|------|----------|
| Gas estimation failed | 함수 실행 실패 예상 | `--gas-limit` 명시 |
| Nonce too low | 중복 nonce | 새 트랜잭션 생성 |
| Insufficient funds | 잔액 부족 | 잔액 충전 |
| Execution reverted | 권한 부족/로직 오류 | Owner 및 파라미터 확인 |
| Invalid signature | 서명 실패 | Keystore 위치 확인 |

자세한 문제 해결은 [DEX_ADMIN_GUIDE.md - 문제 해결](./DEX_ADMIN_GUIDE.md#문제-해결) 참조

---

## 📞 추가 자료

### Cast 명령어 활용

```bash
# Nonce 확인
cast nonce --rpc-url $RPC $ADDRESS

# Owner 확인
cast call --rpc-url $RPC $CONTRACT "owner()(address)"

# FeeCollector 확인
cast call --rpc-url $RPC $MARKET "feeCollector()(address)"

# 트랜잭션 상태 확인
cast receipt --rpc-url $RPC $TX_HASH
```

### Gorge 명령어 도움말

```bash
# 각 명령어의 도움말
gorge write --help
gorge sign --help
gorge send --help
```

---

## 📝 문서 기여

문서 개선 제안이나 오류 발견 시:
1. 이슈 생성 또는
2. PR 제출

---

## 📄 라이선스

이 문서는 CROSS DEX 프로젝트의 일부입니다.

