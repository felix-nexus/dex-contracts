# DEX Admin Operations Guide - Gorge CLI

이 문서는 CROSS DEX의 권한 함수 호출을 위한 실용 가이드입니다. Gorge CLI를 사용하여 트랜잭션을 생성(write), 서명(sign), 전송(send)하는 방법을 설명합니다.

---

## 목차

- [기본 개념](#기본-개념)
- [Gorge 워크플로우](#gorge-워크플로우)
- [필수 옵션](#필수-옵션)
- [CrossDex 관리](#crossdex-관리)
- [Market 관리](#market-관리)
- [Pair 관리](#pair-관리)
- [Verse8MarketOwner 사용](#verse8marketowner-사용)
- [배치 작업](#배치-작업)
- [문제 해결](#문제-해결)

---

## ⚠️ 중요한 규칙

### CLI 명령어 플래그 위치

**모든 CLI 명령어(gorge, cast, forge 등)는 플래그[OPTION]를 명령어 바로 다음에 입력합니다:**

```bash
# ✅ 올바른 방법
gorge write --rpc-url <RPC> --sender <SENDER> --out <FILE> <CONTRACT> "function()" ...
cast call --rpc-url <RPC> <CONTRACT> "function()(returnType)"

# ❌ 잘못된 방법 (플래그를 끝에 배치하면 안 됨)
gorge write <CONTRACT> "function()" ... --rpc-url <RPC>
cast call <CONTRACT> "function()" --rpc-url <RPC>
```

---

## 기본 개념

### Gorge 3단계 프로세스

```
1. Write  → 트랜잭션 생성 (온라인 환경)
2. Sign   → 트랜잭션 서명 (오프라인 환경)
3. Send   → 트랜잭션 전송 (온라인 환경)
```

**왜 3단계로 나누나?**
- **보안**: 개인키를 오프라인 환경에서만 사용
- **검증**: 서명 전에 트랜잭션 내용 확인 가능
- **배치**: 여러 트랜잭션을 한 번에 서명/전송

---

## Gorge 워크플로우

### 1. Write - 트랜잭션 생성

```bash
gorge write \
  --rpc-url <RPC_ENDPOINT> \
  --sender <SENDER_ADDRESS> \
  --out <OUTPUT_FILE> \
  <CONTRACT_ADDRESS> \
  "<FUNCTION_SIGNATURE>" \
  <ARG1> <ARG2> ...
```

**출력**: `<OUTPUT_FILE>.json` - 서명되지 않은 트랜잭션

### 2. Sign - 트랜잭션 서명

```bash
# 오프라인 컴퓨터로 JSON 파일 이동 후
gorge sign <OUTPUT_FILE>.json
# Password 입력 프롬프트
```

**출력**: `signed-<OUTPUT_FILE>.json` - 서명된 트랜잭션

### 3. Send - 트랜잭션 전송

```bash
# 온라인 컴퓨터로 signed JSON 파일 이동 후
gorge send \
  --rpc-url <RPC_ENDPOINT> \
  --wait \
  signed-<OUTPUT_FILE>.json
```

**출력**: 트랜잭션 해시 및 Receipt

---

## 필수 옵션

### Write 필수 옵션

| 옵션 | 설명 | 예시 |
|------|------|------|
| `--rpc-url` | RPC 엔드포인트 | `https://mainnet.crosstoken.io:22001` |
| `--sender` | 트랜잭션 발신자 (owner) | `0x22C1522276855B028c31a731BA10D125811Af37c` |
| `--out` | 출력 파일명 (확장자 제외) | `set-tick-setter` |

### 선택 옵션

| 옵션 | 설명 | 사용 시기 |
|------|------|----------|
| `--append` | 기존 파일에 트랜잭션 추가 | 여러 트랜잭션을 하나의 배치로 만들 때 |
| `--gas-limit` | Gas limit 지정 | Gas estimation이 부정확할 때 |
| `--value` | ETH 전송량 | Payable 함수 호출 시 |

---

## CrossDex 관리

### 환경 변수 설정

```bash
# Mainnet
MAINNET_RPC="https://mainnet.crosstoken.io:22001"
MAINNET_DEX="0x89e23B854e432e5c759D49e643d3e612EadB7a6B"
MAINNET_OWNER="0x22C1522276855B028c31a731BA10D125811Af37c"

# Testnet
TESTNET_RPC="https://testnet.crosstoken.io:22001"
TESTNET_DEX="0xa2d6eAc8Ad4C750c6d0a5Ead48896f9d5f96e873"
TESTNET_OWNER="0x535c7b8C25eADACCAf857652cA34cF4c010957E2"
```

### 1. TickSizeSetter 변경

**시나리오**: TickSizeSetter를 새 주소로 변경

```bash
# 1. Write
gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $MAINNET_OWNER \
  --out set-tick-setter \
  $MAINNET_DEX \
  "setTickSizeSetter(address)" \
  0xNEW_TICK_SIZE_SETTER_ADDRESS

# 2. Sign (오프라인)
gorge sign set-tick-setter.json

# 3. Send (온라인)
gorge send --rpc-url $MAINNET_RPC --wait signed-set-tick-setter.json
```

### 2. PairImpl 업데이트

**시나리오**: 새로운 PairImpl 구현체로 업데이트

```bash
gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $MAINNET_OWNER \
  --out update-pair-impl \
  $MAINNET_DEX \
  "setPairImpl(address)" \
  0xNEW_PAIR_IMPL_ADDRESS

gorge sign update-pair-impl.json
gorge send --rpc-url $MAINNET_RPC --wait signed-update-pair-impl.json
```

### 3. MarketImpl 업데이트

```bash
gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $MAINNET_OWNER \
  --out update-market-impl \
  $MAINNET_DEX \
  "setMarketImpl(address)" \
  0xNEW_MARKET_IMPL_ADDRESS

gorge sign update-market-impl.json
gorge send --rpc-url $MAINNET_RPC --wait signed-update-market-impl.json
```

### 4. 새로운 Market 생성

**시나리오**: JPYx 마켓을 생성하고 싶음

```bash
# Fee 데이터 준비 (4개의 uint32: sellerMaker, sellerTaker, buyerMaker, buyerTaker)
# 예: 모두 0.1% (10 BPS)
FEE_DATA=$(cast abi-encode "f(uint32,uint32,uint32,uint32)" 10 10 10 10)

gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $MAINNET_OWNER \
  --out create-jpyx-market \
  $MAINNET_DEX \
  "createMarket(address,address,address,bytes,string)" \
  $MAINNET_OWNER \
  0xJPYX_TOKEN_ADDRESS \
  0xFEE_COLLECTOR_ADDRESS \
  $FEE_DATA \
  "JPYx Market"

gorge sign create-jpyx-market.json
gorge send --rpc-url $MAINNET_RPC --wait signed-create-jpyx-market.json
```

---

## Market 관리

### 환경 변수 설정

```bash
# Mainnet Game Market
GAME_MARKET="0xa0f50f79615247530fABcC3efd79B8e5b961b966"
GAME_MARKET_OWNER="0xafcc9E7d739b03CC53e2152d368f365430CF3CCa"

# Mainnet USDTx Market
USDTX_MARKET="0xB7811907b2839d6b5CCF908D6B58dE944D8AfbA7"
```

### 1. FeeCollector 변경

```bash
gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $GAME_MARKET_OWNER \
  --out update-fee-collector \
  $GAME_MARKET \
  "setFeeCollector(address)" \
  0xNEW_FEE_COLLECTOR_ADDRESS

gorge sign update-fee-collector.json
gorge send --rpc-url $MAINNET_RPC --wait signed-update-fee-collector.json
```

### 2. Market-level 수수료 변경

**시나리오**: 모든 수수료를 0.2% (20 BPS)로 변경

```bash
gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $GAME_MARKET_OWNER \
  --out update-market-fees \
  $GAME_MARKET \
  "setMarketFees(uint32,uint32,uint32,uint32)" \
  20 20 20 20

gorge sign update-market-fees.json
gorge send --rpc-url $MAINNET_RPC --wait signed-update-market-fees.json
```

### 3. 새로운 Pair 생성

**시나리오**: SOLx/CROSS 페어를 생성하고 싶음

```bash
# 파라미터 준비
BASE_TOKEN="0xSOLX_TOKEN_ADDRESS"
TICK_SIZE="1000000000000000"      # 0.001 CROSS (18 decimals)
LOT_SIZE="1000000000000000000"    # 1 SOLx (18 decimals)

# Fee 데이터 (NO_FEE_BPS 사용 = Market 설정 상속)
NO_FEE_BPS="0xFFFFFFFF"
FEE_DATA=$(cast abi-encode "f(uint32,uint32,uint32,uint32)" $NO_FEE_BPS $NO_FEE_BPS $NO_FEE_BPS $NO_FEE_BPS)

gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $GAME_MARKET_OWNER \
  --out create-solx-pair \
  $GAME_MARKET \
  "createPair(address,uint256,uint256,bytes)" \
  $BASE_TOKEN \
  $TICK_SIZE \
  $LOT_SIZE \
  $FEE_DATA

gorge sign create-solx-pair.json
gorge send --rpc-url $MAINNET_RPC --wait signed-create-solx-pair.json
```

---

## Pair 관리

### 환경 변수 설정

```bash
# 예: ZENY/CROSS Pair (Mainnet)
ZENY_PAIR="0xc4BFB8E247ebf889e13828521247dc66c0DB5976"
PAIR_OWNER=$GAME_MARKET_OWNER  # Pair는 Market owner를 상속
```

### 1. TickSize 변경 (TickSizeSetter 권한 필요)

```bash
TICK_SETTER="0x04F01a3042e536dae19068E6E916CC81D563d0D7"

gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $TICK_SETTER \
  --out update-tick-size \
  $ZENY_PAIR \
  "setTickSize(uint256,uint256)" \
  2000000000000000000 \
  2000000000000000

gorge sign update-tick-size.json
gorge send --rpc-url $MAINNET_RPC --wait signed-update-tick-size.json
```

### 2. Pair-specific 수수료 설정

**시나리오**: 이 Pair만 수수료 0.05% (5 BPS)로 설정

```bash
gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $PAIR_OWNER \
  --out set-pair-fees \
  $ZENY_PAIR \
  "setPairFees(uint32,uint32,uint32,uint32)" \
  5 5 5 5

gorge sign set-pair-fees.json
gorge send --rpc-url $MAINNET_RPC --wait signed-set-pair-fees.json
```

### 3. Pair 일시정지

**긴급 상황**: 페어를 긴급 중단

```bash
gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $PAIR_OWNER \
  --out pause-pair \
  $ZENY_PAIR \
  "setPause(bool)" \
  true

gorge sign pause-pair.json
gorge send --rpc-url $MAINNET_RPC --wait signed-pause-pair.json
```

### 4. Pair 재개

```bash
gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $PAIR_OWNER \
  --out unpause-pair \
  $ZENY_PAIR \
  "setPause(bool)" \
  false

gorge sign unpause-pair.json
gorge send --rpc-url $MAINNET_RPC --wait signed-unpause-pair.json
```

### 5. 초과 토큰 회수 (Skim)

**시나리오**: 실수로 전송된 토큰 회수

```bash
# BASE 토큰 회수
BASE_TOKEN="0xe9013a5231BEB721f4F801F2d07516b8ca19d953"
AMOUNT="1000000000000000000"  # 1 token

gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $PAIR_OWNER \
  --out skim-tokens \
  $ZENY_PAIR \
  "skim(address,address,uint256)" \
  $BASE_TOKEN \
  $PAIR_OWNER \
  $AMOUNT

gorge sign skim-tokens.json
gorge send --rpc-url $MAINNET_RPC --wait signed-skim-tokens.json
```

---

## Verse8MarketOwner 사용

Verse8 Market은 특별한 권한 관리 컨트랙트를 사용합니다.

### 환경 변수 설정

```bash
# Mainnet Verse8
VERSE8_MARKET="0xcb95777d0f8d2EfA5e836Cb65f814dF8C7261d83"
VERSE8_OWNER="0x4469a9879Ec02CE4313bef5e2F21b4F17f2B70e4"

# Pair Creator는 직접 createPair 호출 가능
PAIR_CREATOR_ADDRESS="0xPAIR_CREATOR"
```

### 1. Pair 생성 (PAIR_CREATOR_ROLE)

```bash
# Fee 데이터 준비
FEE_DATA=$(cast abi-encode "f(uint32,uint32,uint32,uint32)" 10 10 10 10)

gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $PAIR_CREATOR_ADDRESS \
  --out verse8-create-pair \
  $VERSE8_OWNER \
  "createPair(address,address,uint256,uint256,bytes)" \
  $VERSE8_MARKET \
  0xBASE_TOKEN \
  1000000000000000 \
  1000000000000000000 \
  $FEE_DATA

gorge sign verse8-create-pair.json
gorge send --rpc-url $MAINNET_RPC --wait signed-verse8-create-pair.json
```

### 2. Market 설정 변경 (DEFAULT_ADMIN_ROLE)

**execute()를 통해 임의의 함수 호출**

```bash
# setFeeCollector 호출을 위한 calldata 생성
CALLDATA=$(cast calldata "setFeeCollector(address)" 0xNEW_FEE_COLLECTOR)

# execute 호출
gorge write \
  --rpc-url $MAINNET_RPC \
  --sender 0xDEFAULT_ADMIN \
  --out verse8-set-fee-collector \
  $VERSE8_OWNER \
  "execute(address,uint256,bytes)" \
  $VERSE8_MARKET \
  0 \
  $CALLDATA

gorge sign verse8-set-fee-collector.json
gorge send --rpc-url $MAINNET_RPC --wait signed-verse8-set-fee-collector.json
```

---

## 배치 작업

### --append 옵션 사용

여러 트랜잭션을 하나의 파일로 묶어서 순차 실행:

```bash
# 1번 트랜잭션 - 첫 번째는 append 없이
gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $GAME_MARKET_OWNER \
  --out batch-operations \
  $GAME_MARKET \
  "setFeeCollector(address)" \
  0xNEW_FEE_COLLECTOR

# 2번 트랜잭션 - append로 추가
gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $GAME_MARKET_OWNER \
  --out batch-operations \
  --append \
  $GAME_MARKET \
  "setMarketFees(uint32,uint32,uint32,uint32)" \
  20 20 20 20

# 3번 트랜잭션 - append로 추가
gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $GAME_MARKET_OWNER \
  --out batch-operations \
  --append \
  $ZENY_PAIR \
  "setPairFees(uint32,uint32,uint32,uint32)" \
  15 15 15 15

# 한 번에 서명
gorge sign batch-operations.json

# 한 번에 전송 (순차 실행됨)
gorge send --rpc-url $MAINNET_RPC --wait signed-batch-operations.json
```

### 배치 작업 주의사항

1. **Nonce 순차성**: 모든 트랜잭션은 순차적으로 실행됨
2. **실패 시**: 중간 트랜잭션 실패 시 이후 트랜잭션 모두 실패
3. **Gas 설정**: 각 트랜잭션마다 충분한 gas 확보

---

## 실전 시나리오

### 시나리오 1: 긴급 수수료 조정

**상황**: 시장 상황으로 인해 모든 수수료를 즉시 0.05%로 낮춰야 함

```bash
# Game Market
gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $GAME_MARKET_OWNER \
  --out emergency-fee-reduction \
  $GAME_MARKET \
  "setMarketFees(uint32,uint32,uint32,uint32)" \
  5 5 5 5

# USDTx Market
gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $GAME_MARKET_OWNER \
  --out emergency-fee-reduction \
  --append \
  $USDTX_MARKET \
  "setMarketFees(uint32,uint32,uint32,uint32)" \
  5 5 5 5

# 오프라인에서 서명
gorge sign emergency-fee-reduction.json

# 즉시 전송
gorge send --rpc-url $MAINNET_RPC --wait signed-emergency-fee-reduction.json
```

### 시나리오 2: 새 토큰 상장

**상황**: SOLx 토큰을 Game Market과 USDTx Market에 모두 상장

```bash
# 파라미터 설정
SOLX="0xSOLX_ADDRESS"
TICK="1000000000000000"
LOT="1000000000000000000"
FEE_DATA=$(cast abi-encode "f(uint32,uint32,uint32,uint32)" 0xFFFFFFFF 0xFFFFFFFF 0xFFFFFFFF 0xFFFFFFFF)

# Game Market에 상장
gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $GAME_MARKET_OWNER \
  --out list-solx \
  $GAME_MARKET \
  "createPair(address,uint256,uint256,bytes)" \
  $SOLX $TICK $LOT $FEE_DATA

# USDTx Market에 상장
gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $GAME_MARKET_OWNER \
  --out list-solx \
  --append \
  $USDTX_MARKET \
  "createPair(address,uint256,uint256,bytes)" \
  $SOLX $TICK $LOT $FEE_DATA

gorge sign list-solx.json
gorge send --rpc-url $MAINNET_RPC --wait signed-list-solx.json
```

### 시나리오 3: 단계별 업그레이드

**상황**: PairImpl과 MarketImpl을 모두 업그레이드

```bash
NEW_PAIR_IMPL="0xNEW_PAIR_IMPL"
NEW_MARKET_IMPL="0xNEW_MARKET_IMPL"

# CrossDex의 PairImpl 업데이트
gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $MAINNET_OWNER \
  --out system-upgrade \
  $MAINNET_DEX \
  "setPairImpl(address)" \
  $NEW_PAIR_IMPL

# CrossDex의 MarketImpl 업데이트
gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $MAINNET_OWNER \
  --out system-upgrade \
  --append \
  $MAINNET_DEX \
  "setMarketImpl(address)" \
  $NEW_MARKET_IMPL

# Game Market의 PairImpl 업데이트
gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $GAME_MARKET_OWNER \
  --out system-upgrade \
  --append \
  $GAME_MARKET \
  "setPairImpl(address)" \
  $NEW_PAIR_IMPL

# USDTx Market의 PairImpl 업데이트
gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $GAME_MARKET_OWNER \
  --out system-upgrade \
  --append \
  $USDTX_MARKET \
  "setPairImpl(address)" \
  $NEW_PAIR_IMPL

gorge sign system-upgrade.json
gorge send --rpc-url $MAINNET_RPC --wait signed-system-upgrade.json
```

---

## 문제 해결

### Gas Estimation 실패

**증상**: Write 단계에서 gas estimation 실패

**해결**:
```bash
# --gas-limit으로 고정 gas 지정
gorge write \
  --rpc-url $MAINNET_RPC \
  --sender $SENDER \
  --out operation \
  --gas-limit 500000 \
  $CONTRACT "function()" args
```

### Nonce Too Low

**증상**: Send 단계에서 "nonce too low" 에러

**원인**: 이미 같은 nonce로 전송된 트랜잭션이 있음

**해결**:
```bash
# 1. 현재 nonce 확인
cast nonce $SENDER --rpc-url $MAINNET_RPC

# 2. JSON 파일의 nonce 확인 및 수정
# (또는 새로운 트랜잭션 생성)
```

### Transaction Reverted

**증상**: 트랜잭션이 블록에 포함되었지만 revert됨

**원인**:
- 권한 부족 (sender가 owner가 아님)
- 잘못된 파라미터
- 컨트랙트 로직 실패 (예: taker fee < maker fee)

**디버깅**:
```bash
# 트랜잭션 해시로 상세 정보 확인
cast tx $TX_HASH --rpc-url $MAINNET_RPC

# Receipt 확인
cast receipt $TX_HASH --rpc-url $MAINNET_RPC
```

### 서명 실패

**증상**: Sign 단계에서 keystore를 찾지 못함

**해결**:
```bash
# 1. Keystore 파일 위치 확인
ls ~/.foundry/keystores/

# 2. 올바른 keystore 사용
# 파일명이 sender 주소와 일치하는지 확인
```

---

## 체크리스트

### Write 전

- [ ] RPC URL이 올바른가? (Mainnet/Testnet 확인)
- [ ] Sender 주소가 해당 함수의 권한이 있는가?
- [ ] 컨트랙트 주소가 정확한가?
- [ ] 함수 시그니처가 올바른가?
- [ ] 파라미터 개수와 타입이 맞는가?

### Sign 전

- [ ] 생성된 JSON 파일을 확인했는가?
- [ ] Nonce가 순차적인가?
- [ ] Gas limit이 충분한가?
- [ ] 파일을 오프라인 환경으로 안전하게 전송했는가?

### Send 전

- [ ] 서명된 파일인가? (signed- 접두사 확인)
- [ ] RPC URL이 올바른가?
- [ ] Sender 계정에 충분한 잔액이 있는가?
- [ ] --wait 옵션을 사용하여 결과를 확인할 것인가?

---

## 참고

### 유용한 Cast 명령어

```bash
# Nonce 확인
cast nonce $ADDRESS --rpc-url $RPC

# 잔액 확인
cast balance $ADDRESS --rpc-url $RPC

# Calldata 생성 (미리 확인용)
cast calldata "transfer(address,uint256)" $TO $AMOUNT

# ABI 인코딩
cast abi-encode "constructor(uint256,uint256)" 100 200

# 트랜잭션 상태 확인
cast tx $TX_HASH --rpc-url $RPC
cast receipt $TX_HASH --rpc-url $RPC
```

### 환경 변수 파일 (.env)

```bash
# .env 파일 생성
cat > .env << 'EOF'
MAINNET_RPC=https://mainnet.crosstoken.io:22001
TESTNET_RPC=https://testnet.crosstoken.io:22001

# Mainnet
MAINNET_DEX=0x89e23B854e432e5c759D49e643d3e612EadB7a6B
MAINNET_OWNER=0x22C1522276855B028c31a731BA10D125811Af37c
GAME_MARKET=0xa0f50f79615247530fABcC3efd79B8e5b961b966
GAME_MARKET_OWNER=0xafcc9E7d739b03CC53e2152d368f365430CF3CCa

# Testnet
TESTNET_DEX=0xa2d6eAc8Ad4C750c6d0a5Ead48896f9d5f96e873
TESTNET_OWNER=0x535c7b8C25eADACCAf857652cA34cF4c010957E2
EOF

# 사용
source .env
gorge write --rpc-url $MAINNET_RPC --sender $MAINNET_OWNER ...
```

---

## 요약

### 핵심 명령어 패턴

```bash
# 단일 작업
gorge write --rpc-url $RPC --sender $SENDER --out name $CONTRACT "func()" args
gorge sign name.json
gorge send --rpc-url $RPC --wait signed-name.json

# 배치 작업
gorge write --rpc-url $RPC --sender $SENDER --out batch $CONTRACT1 "func1()" args1
gorge write --rpc-url $RPC --sender $SENDER --out batch --append $CONTRACT2 "func2()" args2
gorge sign batch.json
gorge send --rpc-url $RPC --wait signed-batch.json
```

### 주의사항

1. **항상 CLI로 직접 입력** - .sh 스크립트 사용 금지
2. **오프라인 서명** - 개인키를 인터넷에 연결된 환경에서 사용하지 않음
3. **검증 후 전송** - JSON 파일을 열어서 내용 확인
4. **배치 작업 신중** - Nonce 순차성 때문에 하나 실패하면 전부 실패
5. **Gas 여유있게** - Estimation은 정확하지 않을 수 있음

이 가이드를 따라하면 DEX 컨트랙트의 모든 권한 함수를 안전하게 호출할 수 있습니다.

