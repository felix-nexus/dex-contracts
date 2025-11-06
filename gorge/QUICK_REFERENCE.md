# Gorge Quick Reference - DEX Admin Operations

빠른 참조용 명령어 모음입니다. 상세 설명은 [DEX_ADMIN_GUIDE.md](./DEX_ADMIN_GUIDE.md)를 참조하세요.

---

## 기본 템플릿

### Write → Sign → Send

```bash
# 1. Write
gorge write --rpc-url $RPC --sender $SENDER --out FILENAME CONTRACT "function(args)" arg1 arg2

# 2. Sign (오프라인)
gorge sign FILENAME.json

# 3. Send
gorge send --rpc-url $RPC --wait signed-FILENAME.json
```

---

## 환경 변수 (필수)

```bash
# Mainnet
export MAINNET_RPC="https://mainnet.crosstoken.io:22001"
export MAINNET_DEX="0x89e23B854e432e5c759D49e643d3e612EadB7a6B"
export MAINNET_OWNER="0x22C1522276855B028c31a731BA10D125811Af37c"
export MAINNET_TICK_SETTER="0x04F01a3042e536dae19068E6E916CC81D563d0D7"
export GAME_MARKET="0xa0f50f79615247530fABcC3efd79B8e5b961b966"
export GAME_MARKET_OWNER="0xafcc9E7d739b03CC53e2152d368f365430CF3CCa"
export USDTX_MARKET="0xB7811907b2839d6b5CCF908D6B58dE944D8AfbA7"

# Testnet
export TESTNET_RPC="https://testnet.crosstoken.io:22001"
export TESTNET_DEX="0xa2d6eAc8Ad4C750c6d0a5Ead48896f9d5f96e873"
export TESTNET_OWNER="0x535c7b8C25eADACCAf857652cA34cF4c010957E2"
export TESTNET_TICK_SETTER="0x85756F8C044AdcC9B37Dc37447C5d1399E8998D0"
```

---

## CrossDex 관리

### TickSizeSetter 변경

```bash
gorge write --rpc-url $MAINNET_RPC --sender $MAINNET_OWNER --out set-tick-setter \
  $MAINNET_DEX "setTickSizeSetter(address)" 0xNEW_ADDRESS
```

### PairImpl 업데이트

```bash
gorge write --rpc-url $MAINNET_RPC --sender $MAINNET_OWNER --out update-pair-impl \
  $MAINNET_DEX "setPairImpl(address)" 0xNEW_IMPL
```

### MarketImpl 업데이트

```bash
gorge write --rpc-url $MAINNET_RPC --sender $MAINNET_OWNER --out update-market-impl \
  $MAINNET_DEX "setMarketImpl(address)" 0xNEW_IMPL
```

### Market 생성

```bash
FEE_DATA=$(cast abi-encode "f(uint32,uint32,uint32,uint32)" 10 10 10 10)
gorge write --rpc-url $MAINNET_RPC --sender $MAINNET_OWNER --out create-market \
  $MAINNET_DEX "createMarket(address,address,address,bytes,string)" \
  $OWNER $QUOTE_TOKEN $FEE_COLLECTOR $FEE_DATA "Market Name"
```

---

## Market 관리

### FeeCollector 변경

```bash
gorge write --rpc-url $MAINNET_RPC --sender $GAME_MARKET_OWNER --out set-fee-collector \
  $GAME_MARKET "setFeeCollector(address)" 0xNEW_COLLECTOR
```

### Market 수수료 변경

```bash
# 예: 모두 0.2% (20 BPS)
gorge write --rpc-url $MAINNET_RPC --sender $GAME_MARKET_OWNER --out set-market-fees \
  $GAME_MARKET "setMarketFees(uint32,uint32,uint32,uint32)" 20 20 20 20
```

### Pair 생성

```bash
FEE_DATA=$(cast abi-encode "f(uint32,uint32,uint32,uint32)" 0xFFFFFFFF 0xFFFFFFFF 0xFFFFFFFF 0xFFFFFFFF)
gorge write --rpc-url $MAINNET_RPC --sender $GAME_MARKET_OWNER --out create-pair \
  $GAME_MARKET "createPair(address,uint256,uint256,bytes)" \
  0xBASE_TOKEN 1000000000000000 1000000000000000000 $FEE_DATA
```

---

## Pair 관리

### TickSize 변경 (TickSizeSetter만)

```bash
gorge write --rpc-url $MAINNET_RPC --sender $MAINNET_TICK_SETTER --out set-tick-size \
  $PAIR_ADDRESS "setTickSize(uint256,uint256)" LOT_SIZE TICK_SIZE
```

### Pair 수수료 변경

```bash
gorge write --rpc-url $MAINNET_RPC --sender $GAME_MARKET_OWNER --out set-pair-fees \
  $PAIR_ADDRESS "setPairFees(uint32,uint32,uint32,uint32)" 5 5 5 5
```

### Pair 일시정지/재개

```bash
# 일시정지
gorge write --rpc-url $MAINNET_RPC --sender $GAME_MARKET_OWNER --out pause-pair \
  $PAIR_ADDRESS "setPause(bool)" true

# 재개
gorge write --rpc-url $MAINNET_RPC --sender $GAME_MARKET_OWNER --out unpause-pair \
  $PAIR_ADDRESS "setPause(bool)" false
```

### 초과 토큰 회수

```bash
gorge write --rpc-url $MAINNET_RPC --sender $GAME_MARKET_OWNER --out skim \
  $PAIR_ADDRESS "skim(address,address,uint256)" $TOKEN $RECIPIENT $AMOUNT
```

---

## 배치 작업 (--append)

```bash
# 첫 번째 트랜잭션
gorge write --rpc-url $RPC --sender $SENDER --out batch \
  $CONTRACT1 "function1()" args1

# 두 번째 트랜잭션 (append)
gorge write --rpc-url $RPC --sender $SENDER --out batch --append \
  $CONTRACT2 "function2()" args2

# 세 번째 트랜잭션 (append)
gorge write --rpc-url $RPC --sender $SENDER --out batch --append \
  $CONTRACT3 "function3()" args3

# 한 번에 서명 및 전송
gorge sign batch.json
gorge send --rpc-url $RPC --wait signed-batch.json
```

---

## 유용한 Cast 명령어

```bash
# Nonce 확인
cast nonce $ADDRESS --rpc-url $RPC

# 잔액 확인  
cast balance $ADDRESS --rpc-url $RPC

# Calldata 미리보기
cast calldata "function(type)" arg

# ABI 인코딩
cast abi-encode "constructor(uint32,uint32,uint32,uint32)" 10 10 10 10

# 트랜잭션 확인
cast tx $TX_HASH --rpc-url $RPC
cast receipt $TX_HASH --rpc-url $RPC

# 컨트랙트 호출 (read-only)
cast call $CONTRACT "function()(returnType)" --rpc-url $RPC

# Owner 확인
cast call $CONTRACT "owner()(address)" --rpc-url $RPC
```

---

## 자주 사용하는 값

### Fee BPS 값

| 수수료 | BPS 값 |
|--------|--------|
| 0.01% | 1 |
| 0.05% | 5 |
| 0.1% | 10 |
| 0.2% | 20 |
| 0.3% | 30 |
| 1% | 100 |
| Market 상속 | 0xFFFFFFFF (NO_FEE_BPS) |

### 일반적인 Decimals

| 토큰 타입 | Decimals | 1 Token |
|-----------|----------|---------|
| CROSS, ZENY, MGT | 18 | 1000000000000000000 |
| USDTx, USDCx | 6 | 1000000 |

### TickSize/LotSize 예시

```bash
# 0.001 CROSS tick, 1 token lot (18 decimals)
TICK_SIZE="1000000000000000"
LOT_SIZE="1000000000000000000"

# 0.01 USDTx tick, 0.1 token lot (6 decimals)
TICK_SIZE="10000"
LOT_SIZE="100000"
```

---

## 체크리스트

### Write 전
- [ ] RPC URL 확인 (Mainnet/Testnet)
- [ ] Sender 권한 확인
- [ ] 컨트랙트 주소 정확성
- [ ] 함수 시그니처 정확성
- [ ] 파라미터 타입/개수 일치

### Sign 전
- [ ] JSON 파일 내용 검토
- [ ] Nonce 확인
- [ ] Gas limit 충분성
- [ ] 오프라인 환경으로 안전하게 이동

### Send 전
- [ ] signed- 파일 확인
- [ ] RPC URL 확인
- [ ] 잔액 충분성
- [ ] --wait 옵션 (결과 확인용)

---

## 자주 발생하는 에러

| 에러 | 원인 | 해결 |
|------|------|------|
| Gas estimation failed | 함수 실행 실패 예상 | --gas-limit 명시 |
| Nonce too low | 이미 사용된 nonce | 새 트랜잭션 생성 |
| Insufficient funds | 잔액 부족 | 잔액 확인 및 충전 |
| Execution reverted | 권한 부족 또는 로직 오류 | Owner 확인, 파라미터 검증 |
| Invalid signature | 서명 실패 | Keystore 확인 |

---

## 실전 예시

### 수수료 긴급 조정 (0.05%로 인하)

```bash
gorge write --rpc-url $MAINNET_RPC --sender $GAME_MARKET_OWNER --out emergency-fee \
  $GAME_MARKET "setMarketFees(uint32,uint32,uint32,uint32)" 5 5 5 5

gorge write --rpc-url $MAINNET_RPC --sender $GAME_MARKET_OWNER --out emergency-fee --append \
  $USDTX_MARKET "setMarketFees(uint32,uint32,uint32,uint32)" 5 5 5 5

gorge sign emergency-fee.json
gorge send --rpc-url $MAINNET_RPC --wait signed-emergency-fee.json
```

### 새 토큰 동시 상장

```bash
TOKEN="0xTOKEN_ADDRESS"
FEE_DATA=$(cast abi-encode "f(uint32,uint32,uint32,uint32)" 0xFFFFFFFF 0xFFFFFFFF 0xFFFFFFFF 0xFFFFFFFF)

gorge write --rpc-url $MAINNET_RPC --sender $GAME_MARKET_OWNER --out list-token \
  $GAME_MARKET "createPair(address,uint256,uint256,bytes)" \
  $TOKEN 1000000000000000 1000000000000000000 $FEE_DATA

gorge write --rpc-url $MAINNET_RPC --sender $GAME_MARKET_OWNER --out list-token --append \
  $USDTX_MARKET "createPair(address,uint256,uint256,bytes)" \
  $TOKEN 1000000000000000 1000000000000000000 $FEE_DATA

gorge sign list-token.json
gorge send --rpc-url $MAINNET_RPC --wait signed-list-token.json
```

---

## 참고 문서

- [DEX_ADMIN_GUIDE.md](./DEX_ADMIN_GUIDE.md) - 상세 가이드
- [CROSS_Mainnet_Contracts.md](./CROSS_Mainnet_Contracts.md) - Mainnet 주소
- [CROSS_Testnet_Contracts.md](./CROSS_Testnet_Contracts.md) - Testnet 주소
- [commands.md](../docs/commands.md) - Gorge 명령어 상세

