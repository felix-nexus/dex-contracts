# Gorge Script Examples - AI Training Guide

이 문서는 AI가 사용자의 요청을 받아 실제 실행 가능한 gorge 스크립트를 작성할 수 있도록 학습하기 위한 가이드입니다.

---

## 목차

- [기본 원칙](#기본-원칙)
- [Market 타입별 차이점](#market-타입별-차이점)
- [실전 스크립트 예제](#실전-스크립트-예제)
- [Verse8MarketOwner 사용법](#verse8marketowner-사용법)
- [패턴 인식 및 응답](#패턴-인식-및-응답)

---

## 기본 원칙

### 1. 출력 형식 규칙

**⚠️ 절대 쉘 스크립트(.sh) 파일로 작성하지 마세요!**

사용자는 터미널에서 명령어를 **직접 복사-붙여넣기**하여 실행합니다.

```bash
# ❌ 절대 금지: 쉘 스크립트 파일 형식
#!/usr/bin/env bash
set -euo pipefail
TESTNET_RPC="https://testnet.crosstoken.io:22001"
GAME_MARKET="0x1d86752372281D0573Cb70319b434D73f2daFd59"
...

# ✅ 올바른 방법: 터미널에서 바로 실행 가능한 명령어
# Game Market FeeCollector 변경
gorge write \
  --rpc-url https://testnet.crosstoken.io:22001 \
  --sender 0x535c7b8C25eADACCAf857652cA34cF4c010957E2 \
  --out testnet_update_fee_collector.json \
  0x1d86752372281D0573Cb70319b434D73f2daFd59 \
  "setFeeCollector(address)" \
  0xNewCollectorAddress
```

### 2. 명령어 구조

**⚠️ 중요: 모든 CLI 명령어는 플래그[OPTION]를 명령어 바로 다음에 입력합니다**

**Gorge 명령어:**

```bash
gorge write \
  --rpc-url <RPC_URL> \
  --sender <SENDER_ADDRESS> \
  --out <FILENAME> \
  <CONTRACT_ADDRESS> \
  "<FUNCTION_SIGNATURE>" \
  <ARG1> <ARG2> ...

gorge sign --out <OUTPUT_FILE> <INPUT_FILE>

gorge send --out <OUTPUT_FILE> --rpc-url <RPC_URL> --wait <SECONDS> <INPUT_FILE>
```

**Foundry 명령어 (cast, forge 등):**

```bash
# cast call - 컨트랙트 읽기
cast call --rpc-url <RPC_URL> <CONTRACT_ADDRESS> "<FUNCTION_SIGNATURE>"

# cast abi-encode - 데이터 인코딩
cast abi-encode "<SIGNATURE>" <ARG1> <ARG2>

# cast calldata - calldata 생성
cast calldata "<FUNCTION_SIGNATURE>" <ARG1> <ARG2>

# cast nonce - nonce 확인
cast nonce --rpc-url <RPC_URL> <ADDRESS>

# cast receipt - 트랜잭션 확인
cast receipt --rpc-url <RPC_URL> <TX_HASH>
```

### 3. 변수 사용 규칙

**⚠️ 중요: 기본적으로 모든 값을 직접 입력하세요. 변수를 사용하지 마세요!**

**✅ 유일한 예외: `cast calldata` 결과값만 변수 사용 허용**
- `cast calldata` 또는 `cast abi-encode` 결과는 너무 길고 복잡하므로 변수 사용 가능
- 그 외 모든 값(주소, 숫자 등)은 직접 입력

```bash
# ❌ 절대 금지: 시스템 상수를 변수화
TESTNET_RPC="https://testnet.crosstoken.io:22001"
GAME_MARKET="0x1d86752372281D0573Cb70319b434D73f2daFd59"
TESTNET_OWNER="0x535c7b8C25eADACCAf857652cA34cF4c010957E2"
gorge write --rpc-url "$TESTNET_RPC" --sender "$TESTNET_OWNER" ...

# ❌ 절대 금지: 사용자 제공 값도 변수화하지 마세요
TOKEN="0x1234567890123456789012345678901234567890"
NEW_COLLECTOR="0xAbCdEf1234567890123456789012345678901234"
gorge write ... $TOKEN ...
gorge write ... "setFeeCollector(address)" $NEW_COLLECTOR

# ❌ 절대 금지: 숫자를 변수화
TICK_SIZE="1000"
LOT_SIZE="2000"
SELLER_MAKER="100"
gorge write ... $TICK_SIZE $LOT_SIZE $SELLER_MAKER

# ✅ 올바른 방법: 모든 값을 직접 입력
gorge write \
  --rpc-url https://testnet.crosstoken.io:22001 \
  --sender 0x535c7b8C25eADACCAf857652cA34cF4c010957E2 \
  --out testnet_list_token.json \
  0x1d86752372281D0573Cb70319b434D73f2daFd59 \
  "createPair(address,uint256,uint256,bytes)" \
  0x1234567890123456789012345678901234567890 \
  1000 \
  2000 \
  $(cast abi-encode "f(uint32,uint32,uint32,uint32)" 0 0 0 0)

# ✅ cast calldata 결과가 여러 곳에 사용될 때만 변수 허용
FEE_DATA=$(cast abi-encode "f(uint32,uint32,uint32,uint32)" 0 0 0 0)
gorge write ... 0x1234567890123456789012345678901234567890 1000 2000 $FEE_DATA
gorge write --append ... 0xAbCdEf1234567890123456789012345678901234 1000 2000 $FEE_DATA
```

### 4. 배치 작업

**여러 트랜잭션을 하나로 묶을 때 `--append` 사용:**

```bash
# 첫 번째 트랜잭션 (--append 없음)
gorge write \
  --rpc-url <RPC> \
  --sender <SENDER> \
  --out <FILENAME> \
  <CONTRACT1> "<FUNCTION1>" <ARGS1>

# 두 번째 트랜잭션 (--append 추가)
gorge write \
  --append \
  --rpc-url <RPC> \
  --sender <SENDER> \
  --out <FILENAME> \
  <CONTRACT2> "<FUNCTION2>" <ARGS2>
```

### 5. 파일명 규칙

**파일명은 작업 내용을 명확하게 표현:**
- Write: `환경_작업내용.json`
- Sign: `환경_작업내용_sign.json`
- Send: `환경_작업내용_send.json`
- 예: 
  - Write → `mainnet_list_token.json`
  - Sign → `mainnet_list_token_sign.json`
  - Send → `mainnet_list_token_send.json`

**모든 파일 확장자는 `.json`:**
```bash
# ✅ 올바른 예
--out mainnet_operation.json
--out mainnet_operation_sign.json

# ❌ 잘못된 예
--out mainnet_operation
--out mainnet_operation.txt
```

### 6. Sign과 Send 명령어 규칙

**Sign 명령어:**
```bash
gorge sign --out <OUTPUT_FILE>_sign.json <INPUT_FILE>.json
```

**Send 명령어:**
```bash
gorge send --out <OUTPUT_FILE>_send.json --rpc-url <RPC> --wait <SECONDS> <INPUT_FILE>.json
```

**명령어 구조:**
- 명령어 → 플래그(옵션) → 파일명 순서
- 플래그는 명령어 바로 다음에 위치

**--wait 옵션:**
- 반드시 seconds 값을 지정해야 함
- 일반적으로 60초 사용
- 예: `--wait 60`

```bash
# ✅ 올바른 예
gorge send --out result_send.json --rpc-url $RPC --wait 60 file.json

# ❌ 잘못된 예 (파일명이 앞에)
gorge send file.json --out result_send.json --rpc-url $RPC --wait 60

# ❌ 잘못된 예 (seconds 없음)
gorge send --out result_send.json --rpc-url $RPC --wait file.json
```

---

## Market 타입별 차이점

### 일반 Market (Game, USDTx)

**Owner가 직접 함수 호출:**

```bash
gorge write \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --sender 0xafcc9E7d739b03CC53e2152d368f365430CF3CCa \
  --out operation \
  0xMARKET_ADDRESS \
  "functionName(args)" \
  arg1 arg2
```

### Verse8 Market

**Verse8MarketOwner의 execute() 함수를 통해 간접 호출:**

```bash
# 1. calldata 생성 (변수에 저장)
CALLDATA=$(cast calldata "functionName(args)" arg1 arg2)

# 2. execute 함수로 전달
gorge write \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --sender <ADMIN_ADDRESS> \
  --out operation \
  0x4469a9879Ec02CE4313bef5e2F21b4F17f2B70e4 \
  "execute(address,uint256,bytes)" \
  0xcb95777d0f8d2EfA5e836Cb65f814dF8C7261d83 \
  0 \
  $CALLDATA
```

**또는 인라인으로:**

```bash
gorge write \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --sender <ADMIN_ADDRESS> \
  --out operation \
  0x4469a9879Ec02CE4313bef5e2F21b4F17f2B70e4 \
  "execute(address,uint256,bytes)" \
  0xcb95777d0f8d2EfA5e836Cb65f814dF8C7261d83 \
  0 \
  $(cast calldata "functionName(args)" arg1 arg2)
```

---

## 실전 스크립트 예제

### 예제 1: 새 토큰 상장 (단일 Market)

**사용자 요청:**
> "메인넷 Game Market에 0x1234...5678 토큰을 ticksize 1000000000000000, lotsize 1000000000000000000으로 상장해줘"

**AI 응답:**

```bash
# Fee 데이터 생성 (Market 설정 상속)
FEE_DATA=$(cast abi-encode "f(uint32,uint32,uint32,uint32)" 0xFFFFFFFF 0xFFFFFFFF 0xFFFFFFFF 0xFFFFFFFF)

# 트랜잭션 생성
gorge write \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --sender 0xafcc9E7d739b03CC53e2152d368f365430CF3CCa \
  --out mainnet_list_token \
  0xa0f50f79615247530fABcC3efd79B8e5b961b966 \
  "createPair(address,uint256,uint256,bytes)" \
  0x1234567890123456789012345678901234567890 \
  1000000000000000 \
  1000000000000000000 \
  $FEE_DATA

# 서명 (오프라인 환경)
gorge sign --out mainnet_list_token_sign.json mainnet_list_token.json

# 전송
gorge send \
  --out mainnet_list_token_send.json \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --wait 60 \
  mainnet_list_token_sign.json
```

### 예제 2: 여러 Market에 동시 상장

**사용자 요청:**
> "메인넷의 USDTx Market과 Verse8 Market에 0x0001 토큰을 ticksize 1, lotsize 1로 상장하려고 하는데 스크립트 작성해줘"

**AI 응답:**

```bash
# Fee 데이터 (Market 설정 상속)
FEE_DATA=$(cast abi-encode "f(uint32,uint32,uint32,uint32)" 0xFFFFFFFF 0xFFFFFFFF 0xFFFFFFFF 0xFFFFFFFF)

# 1. USDTx Market에 상장 (직접 호출)
gorge write \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --sender 0xafcc9E7d739b03CC53e2152d368f365430CF3CCa \
  --out mainnet_list_token_multi.json \
  0xB7811907b2839d6b5CCF908D6B58dE944D8AfbA7 \
  "createPair(address,uint256,uint256,bytes)" \
  0x0001000000000000000000000000000000000000 \
  1 \
  1 \
  $FEE_DATA

# 2. Verse8 Market에 상장 (execute를 통한 간접 호출)
gorge write \
  --append \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --sender 0xafcc9E7d739b03CC53e2152d368f365430CF3CCa \
  --out mainnet_list_token_multi.json \
  0x4469a9879Ec02CE4313bef5e2F21b4F17f2B70e4 \
  "execute(address,uint256,bytes)" \
  0xcb95777d0f8d2EfA5e836Cb65f814dF8C7261d83 \
  0 \
  $(cast calldata "createPair(address,uint256,uint256,bytes)" 0x0001000000000000000000000000000000000000 1 1 $FEE_DATA)

# 서명
gorge sign --out mainnet_list_token_multi_sign.json mainnet_list_token_multi.json

# 전송
gorge send \
  --out mainnet_list_token_multi_send.json \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --wait 60 \
  mainnet_list_token_multi_sign.json
```

### 예제 3: FeeCollector 변경 (3개 Market)

**사용자 요청:**
> "메인넷의 모든 마켓 FeeCollector를 0x9999로 변경해줘"

**AI 응답:**

```bash
# 1. Game Market (직접 호출)
gorge write \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --sender 0xafcc9E7d739b03CC53e2152d368f365430CF3CCa \
  --out mainnet_update_fee_collector.json \
  0xa0f50f79615247530fABcC3efd79B8e5b961b966 \
  "setFeeCollector(address)" \
  0x9999999999999999999999999999999999999999

# 2. USDTx Market (직접 호출)
gorge write \
  --append \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --sender 0xafcc9E7d739b03CC53e2152d368f365430CF3CCa \
  --out mainnet_update_fee_collector.json \
  0xB7811907b2839d6b5CCF908D6B58dE944D8AfbA7 \
  "setFeeCollector(address)" \
  0x9999999999999999999999999999999999999999

# 3. Verse8 Market (execute를 통한 간접 호출)
gorge write \
  --append \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --sender 0xafcc9E7d739b03CC53e2152d368f365430CF3CCa \
  --out mainnet_update_fee_collector.json \
  0x4469a9879Ec02CE4313bef5e2F21b4F17f2B70e4 \
  "execute(address,uint256,bytes)" \
  0xcb95777d0f8d2EfA5e836Cb65f814dF8C7261d83 \
  0 \
  $(cast calldata "setFeeCollector(address)" 0x9999999999999999999999999999999999999999)

# 서명
gorge sign --out mainnet_update_fee_collector_sign.json mainnet_update_fee_collector.json

# 전송
gorge send \
  --out mainnet_update_fee_collector_send.json \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --wait 60 \
  mainnet_update_fee_collector_sign.json
```

### 예제 4: Market 수수료 일괄 변경

**사용자 요청:**
> "테스트넷 모든 마켓의 수수료를 0.1% (10 BPS)로 변경해줘"

**AI 응답:**

```bash
# 1. Game Market
gorge write \
  --rpc-url https://testnet.crosstoken.io:22001 \
  --sender 0x535c7b8C25eADACCAf857652cA34cF4c010957E2 \
  --out testnet_update_market_fees \
  0x1d86752372281D0573Cb70319b434D73f2daFd59 \
  "setMarketFees(uint32,uint32,uint32,uint32)" \
  10 10 10 10

# 2. USDTx Market
gorge write \
  --append \
  --rpc-url https://testnet.crosstoken.io:22001 \
  --sender 0x535c7b8C25eADACCAf857652cA34cF4c010957E2 \
  --out testnet_update_market_fees \
  0x7D2fA2c51DBcF5AdaB5508B338f03853598c2352 \
  "setMarketFees(uint32,uint32,uint32,uint32)" \
  10 10 10 10

# 3. Verse8 Market (execute를 통한 간접 호출)
gorge write \
  --append \
  --rpc-url https://testnet.crosstoken.io:22001 \
  --sender 0x535c7b8C25eADACCAf857652cA34cF4c010957E2 \
  --out testnet_update_market_fees \
  0xFc2852EeaA01b4A9B52eF36Eb24DF56432f8043e \
  "execute(address,uint256,bytes)" \
  0xCcb6F48f18da94B7d90e29adBF762FBf70564844 \
  0 \
  $(cast calldata "setMarketFees(uint32,uint32,uint32,uint32)" 10 10 10 10)

# 서명
gorge sign --out testnet_update_market_fees_sign.json testnet_update_market_fees.json

# 전송
gorge send \
  --out testnet_update_market_fees_send.json \
  --rpc-url https://testnet.crosstoken.io:22001 \
  --wait 60 \
  testnet_update_market_fees_sign.json
```

---

## Verse8MarketOwner 사용법

### 컨트랙트 구조

```solidity
contract Verse8MarketOwner {
    // execute: 단일 호출
    function execute(address to, uint256 value, bytes calldata data) 
        external 
        onlyRole(DEFAULT_ADMIN_ROLE)
        returns (bytes memory);
    
    // executeBatch: 여러 호출을 한 번에
    function executeBatch(ExecuteBatchArgs[] calldata calls) 
        external 
        onlyRole(DEFAULT_ADMIN_ROLE)
        returns (bytes[] memory);
    
    // createPair: Pair 생성 (PAIR_CREATOR_ROLE)
    function createPair(address market, address base, uint256 tickSize, uint256 lotSize, bytes memory feeData)
        external
        onlyRole(PAIR_CREATOR_ROLE)
        returns (address);
}
```

### execute 함수 사용 패턴

**기본 구조:**

```bash
gorge write \
  --rpc-url <RPC> \
  --sender <ADMIN> \
  --out <FILENAME> \
  <VERSE8_OWNER_ADDRESS> \
  "execute(address,uint256,bytes)" \
  <TARGET_MARKET> \
  0 \
  $(cast calldata "<ACTUAL_FUNCTION>" <ARGS>)
```

**예제: setFeeCollector 호출**

```bash
# 방법 1: 변수 사용
CALLDATA=$(cast calldata "setFeeCollector(address)" 0xNEW_COLLECTOR)
gorge write \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --sender 0xADMIN \
  --out verse8_set_fee_collector \
  0x4469a9879Ec02CE4313bef5e2F21b4F17f2B70e4 \
  "execute(address,uint256,bytes)" \
  0xcb95777d0f8d2EfA5e836Cb65f814dF8C7261d83 \
  0 \
  $CALLDATA

# 방법 2: 인라인
gorge write \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --sender 0xADMIN \
  --out verse8_set_fee_collector \
  0x4469a9879Ec02CE4313bef5e2F21b4F17f2B70e4 \
  "execute(address,uint256,bytes)" \
  0xcb95777d0f8d2EfA5e836Cb65f814dF8C7261d83 \
  0 \
  $(cast calldata "setFeeCollector(address)" 0xNEW_COLLECTOR)
```

### executeBatch 사용 (고급)

**여러 작업을 Verse8 Market에서 한 번에 실행:**

```bash
# executeBatch는 struct 배열을 받으므로 직접 ABI 인코딩 필요
# 일반적으로는 여러 개의 execute를 --append로 연결하는 것이 더 간단함

# 예: Verse8에서 2개 작업 (대안: execute를 2번 호출)
gorge write \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --sender 0xADMIN \
  --out verse8_batch \
  0x4469a9879Ec02CE4313bef5e2F21b4F17f2B70e4 \
  "execute(address,uint256,bytes)" \
  0xcb95777d0f8d2EfA5e836Cb65f814dF8C7261d83 \
  0 \
  $(cast calldata "setFeeCollector(address)" 0xNEW_COLLECTOR)

gorge write \
  --append \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --sender 0xADMIN \
  --out verse8_batch \
  0x4469a9879Ec02CE4313bef5e2F21b4F17f2B70e4 \
  "execute(address,uint256,bytes)" \
  0xcb95777d0f8d2EfA5e836Cb65f814dF8C7261d83 \
  0 \
  $(cast calldata "setMarketFees(uint32,uint32,uint32,uint32)" 10 10 10 10)
```

### createPair (PAIR_CREATOR_ROLE)

**Verse8 Market에 Pair 생성 - 방법 1 (createPair 직접):**

```bash
# PAIR_CREATOR_ROLE을 가진 주소로 직접 호출
FEE_DATA=$(cast abi-encode "f(uint32,uint32,uint32,uint32)" 0xFFFFFFFF 0xFFFFFFFF 0xFFFFFFFF 0xFFFFFFFF)

gorge write \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --sender 0xPAIR_CREATOR \
  --out verse8_create_pair \
  0x4469a9879Ec02CE4313bef5e2F21b4F17f2B70e4 \
  "createPair(address,address,uint256,uint256,bytes)" \
  0xcb95777d0f8d2EfA5e836Cb65f814dF8C7261d83 \
  0xBASE_TOKEN \
  1000000000000000 \
  1000000000000000000 \
  $FEE_DATA
```

**방법 2 (DEFAULT_ADMIN이 execute로 호출):**

```bash
FEE_DATA=$(cast abi-encode "f(uint32,uint32,uint32,uint32)" 0xFFFFFFFF 0xFFFFFFFF 0xFFFFFFFF 0xFFFFFFFF)

gorge write \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --sender 0xADMIN \
  --out verse8_create_pair \
  0x4469a9879Ec02CE4313bef5e2F21b4F17f2B70e4 \
  "execute(address,uint256,bytes)" \
  0xcb95777d0f8d2EfA5e836Cb65f814dF8C7261d83 \
  0 \
  $(cast calldata "createPair(address,uint256,uint256,bytes)" 0xBASE_TOKEN 1000000000000000 1000000000000000000 $FEE_DATA)
```

---

## 패턴 인식 및 응답

### AI가 인식해야 할 키워드

| 사용자 요청 | 필요한 동작 | Market 타입 |
|------------|------------|-------------|
| "상장", "리스팅", "list" | createPair | 확인 필요 |
| "수수료 변경", "fee" | setMarketFees or setPairFees | 확인 필요 |
| "FeeCollector" | setFeeCollector | 확인 필요 |
| "Game Market" | 직접 호출 | Normal |
| "USDTx Market" | 직접 호출 | Normal |
| "Verse8 Market" | execute 사용 | Verse8 |
| "모든 마켓", "전체 마켓" | 배치 작업 | Mixed |

### 응답 생성 프로세스

**Step 1: 환경 확인**
- Mainnet or Testnet?
- 해당 환경의 RPC URL 선택

**Step 2: Market 타입 확인**
- Game Market → 직접 호출
- USDTx Market → 직접 호출
- Verse8 Market → execute를 통한 간접 호출

**Step 3: Owner/Sender 확인**
```
Mainnet:
- Game/USDTx Market Owner: 0xafcc9E7d739b03CC53e2152d368f365430CF3CCa
- Verse8 Admin: 0xafcc9E7d739b03CC53e2152d368f365430CF3CCa (DEFAULT_ADMIN_ROLE)
- CrossDex Owner: 0x22C1522276855B028c31a731BA10D125811Af37c

Testnet:
- Game/USDTx Market Owner: 0x535c7b8C25eADACCAf857652cA34cF4c010957E2
- Verse8 Admin: 0x535c7b8C25eADACCAf857652cA34cF4c010957E2
- CrossDex Owner: 0x535c7b8C25eADACCAf857652cA34cF4c010957E2
```

**Step 4: 스크립트 생성**
1. 필요한 변수 선언
2. 첫 번째 gorge write (--append 없음)
3. 추가 트랜잭션 (--append 사용)
4. gorge sign 명령어
5. gorge send 명령어

**Step 5: 주석 추가**
- 각 단계마다 명확한 주석
- 사용자가 이해하기 쉽게

---

## 완전한 응답 템플릿

**사용자 요청 예시:**
> "메인넷의 Game Market과 Verse8 Market에 0xABCD 토큰을 ticksize 1000, lotsize 2000으로 상장해줘"

**AI 응답 템플릿:**

```bash
# Fee 데이터 (Market 설정 상속)
FEE_DATA=$(cast abi-encode "f(uint32,uint32,uint32,uint32)" 0xFFFFFFFF 0xFFFFFFFF 0xFFFFFFFF 0xFFFFFFFF)

# 1. Game Market에 상장 (직접 호출)
gorge write \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --sender 0xafcc9E7d739b03CC53e2152d368f365430CF3CCa \
  --out mainnet_list_token.json \
  0xa0f50f79615247530fABcC3efd79B8e5b961b966 \
  "createPair(address,uint256,uint256,bytes)" \
  0xABCDABCDABCDABCDABCDABCDABCDABCDABCDABCD \
  1000 \
  2000 \
  $FEE_DATA

# 2. Verse8 Market에 상장 (execute를 통한 간접 호출)
gorge write \
  --append \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --sender 0xafcc9E7d739b03CC53e2152d368f365430CF3CCa \
  --out mainnet_list_token.json \
  0x4469a9879Ec02CE4313bef5e2F21b4F17f2B70e4 \
  "execute(address,uint256,bytes)" \
  0xcb95777d0f8d2EfA5e836Cb65f814dF8C7261d83 \
  0 \
  $(cast calldata "createPair(address,uint256,uint256,bytes)" 0xABCDABCDABCDABCDABCDABCDABCDABCDABCDABCD 1000 2000 $FEE_DATA)

# 서명 (오프라인 환경에서)
gorge sign --out mainnet_list_token_sign.json mainnet_list_token.json

# 전송 (온라인 환경에서)
gorge send \
  --out mainnet_list_token_send.json \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --wait 60 \
  mainnet_list_token_sign.json
```

---

## 주요 주소 참조 (빠른 복사용)

**⚠️ 주의: 아래는 참조용입니다. 스크립트 작성 시 변수로 사용하지 마세요!**

### Mainnet

```
# RPC
https://mainnet.crosstoken.io:22001

# CrossDex
0x89e23B854e432e5c759D49e643d3e612EadB7a6B  # CrossDex Proxy
0x22C1522276855B028c31a731BA10D125811Af37c  # CrossDex Owner

# Markets
0xa0f50f79615247530fABcC3efd79B8e5b961b966  # Game Market
0xB7811907b2839d6b5CCF908D6B58dE944D8AfbA7  # USDTx Market
0xcb95777d0f8d2EfA5e836Cb65f814dF8C7261d83  # Verse8 Market

# Market Owners
0xafcc9E7d739b03CC53e2152d368f365430CF3CCa  # Game/USDTx Owner
0x4469a9879Ec02CE4313bef5e2F21b4F17f2B70e4  # Verse8MarketOwner contract
0xafcc9E7d739b03CC53e2152d368f365430CF3CCa  # Verse8 DEFAULT_ADMIN_ROLE
```

### Testnet

```
# RPC
https://testnet.crosstoken.io:22001

# CrossDex
0xa2d6eAc8Ad4C750c6d0a5Ead48896f9d5f96e873  # CrossDex Proxy
0x535c7b8C25eADACCAf857652cA34cF4c010957E2  # CrossDex Owner

# Markets
0x1d86752372281D0573Cb70319b434D73f2daFd59  # Game Market
0x7D2fA2c51DBcF5AdaB5508B338f03853598c2352  # USDTx Market
0xCcb6F48f18da94B7d90e29adBF762FBf70564844  # Verse8 Market

# Market Owners
0x535c7b8C25eADACCAf857652cA34cF4c010957E2  # Game/USDTx Owner
0xFc2852EeaA01b4A9B52eF36Eb24DF56432f8043e  # Verse8MarketOwner contract
0x535c7b8C25eADACCAf857652cA34cF4c010957E2  # Verse8 DEFAULT_ADMIN_ROLE
```

---

## 체크리스트 (AI가 스크립트 생성 시)

### 1. 필수 데이터 확인 (스크립트 작성 전)

**⚠️ 중요: 트랜잭션 생성에 필요한 데이터가 부족하면 반드시 사용자에게 먼저 질문해야 합니다.**

**함수별 필수 데이터:**

**`createPair` 함수:**
- [ ] 토큰 주소 (token address)
- [ ] 틱 사이즈 (tick size)
- [ ] 랏 사이즈 (lot size)
- [ ] 수수료 설정 (fee config) - 선택적, 없으면 NO_FEE 사용

**`setMarketFees` 함수:**
- [ ] sellerMakerFeeBps
- [ ] sellerTakerFeeBps
- [ ] buyerMakerFeeBps
- [ ] buyerTakerFeeBps

**`setFeeCollector` 함수:**
- [ ] 새로운 FeeCollector 주소

**`setTickSizeSetter` 함수:**
- [ ] 새로운 TickSizeSetter 주소

**`setTickSize` 함수:**
- [ ] Pair 주소
- [ ] 새로운 Tick Size 값

**`setPairFees` 함수:**
- [ ] Pair 주소
- [ ] sellerMakerFeeBps
- [ ] sellerTakerFeeBps
- [ ] buyerMakerFeeBps
- [ ] buyerTakerFeeBps

**`transferOwnership` 함수:**
- [ ] 새로운 Owner 주소

**Verse8MarketOwner `grantRole` 함수:**
- [ ] Role (DEFAULT_ADMIN_ROLE 또는 PAIR_CREATOR_ROLE)
- [ ] 새로운 계정 주소

**공통 필수 정보:**
- [ ] 환경 (Mainnet/Testnet)
- [ ] 대상 Contract (CrossDex/Router/Market/Pair/Verse8MarketOwner)

**대응 예시:**

**예시 1: 토큰 상장**
```
❌ 나쁜 대응:
사용자: "USDT 마켓에 토큰 상장해줘"
AI: (ticksize, lotsize 없이 임의로 스크립트 작성)

✅ 좋은 대응:
사용자: "USDT 마켓에 토큰 상장해줘"
AI: "토큰 상장을 위해 다음 정보가 필요합니다:
     1. 토큰 주소
     2. Tick Size (최소 가격 단위)
     3. Lot Size (최소 수량 단위)
     4. 수수료 설정 (선택사항, 없으면 NO_FEE로 설정)
     5. Mainnet인지 Testnet인지"
```

**예시 2: 수수료 변경**
```
❌ 나쁜 대응:
사용자: "Game 마켓 수수료 변경해줘"
AI: (수수료 값 없이 임의로 스크립트 작성)

✅ 좋은 대응:
사용자: "Game 마켓 수수료 변경해줘"
AI: "수수료 변경을 위해 다음 정보가 필요합니다:
     1. Seller Maker Fee (BPS)
     2. Seller Taker Fee (BPS)
     3. Buyer Maker Fee (BPS)
     4. Buyer Taker Fee (BPS)
     5. Mainnet인지 Testnet인지
     
     예: 1% = 100 BPS, 0.5% = 50 BPS"
```

**예시 3: 권한 변경**
```
❌ 나쁜 대응:
사용자: "FeeCollector 변경해줘"
AI: (어느 Market인지, 새 주소가 무엇인지 모르는데 임의로 작성)

✅ 좋은 대응:
사용자: "FeeCollector 변경해줘"
AI: "FeeCollector 변경을 위해 다음 정보가 필요합니다:
     1. 대상 Market (Game/USDTx/Verse8)
     2. 새로운 FeeCollector 주소
     3. Mainnet인지 Testnet인지"
```

### 2. 스크립트 작성 확인 사항

**형식 규칙:**
- [ ] ⚠️ 절대 쉘 스크립트(.sh) 형식으로 작성하지 않음
- [ ] 터미널에서 바로 실행 가능한 명령어로 작성
- [ ] 의미있는 파일명 생성 (모두 .json 확장자)

**변수 사용 규칙:**
- [ ] ⚠️ 모든 값을 직접 입력 (변수화 금지)
- [ ] RPC URL은 직접 입력
- [ ] 모든 주소(Market, Router, Owner, Sender, 토큰 등)는 직접 입력
- [ ] 모든 숫자(tick size, lot size, fee BPS)는 직접 입력
- [ ] ✅ 유일한 예외: `cast calldata` / `cast abi-encode` 결과값만 변수 허용

**명령어 구조:**
- [ ] ⚠️ 모든 CLI 명령어는 플래그를 명령어 바로 다음에 입력
- [ ] `gorge write --rpc-url ... --sender ... --out ... CONTRACT ...`
- [ ] `cast call --rpc-url ... CONTRACT "function()"`
- [ ] 환경 확인 (Mainnet/Testnet)
- [ ] Market 타입 확인 (Normal/Verse8)
- [ ] 올바른 RPC URL
- [ ] 올바른 Sender 주소
- [ ] Verse8인 경우 execute() 사용
- [ ] 배치 작업 시 --append 사용
- [ ] sign에 --out 플래그 (_sign.json)
- [ ] send에 --out 플래그 (_send.json)
- [ ] send에 --wait 60 (seconds 필수)
- [ ] 주석 포함

### 스크립트 구조

```bash
# 1. cast calldata 결과만 변수로 (여러 곳에서 재사용할 경우)
FEE_DATA=$(cast abi-encode "f(uint32,uint32,uint32,uint32)" 0 0 0 0)

# 2. gorge write (모든 값을 직접 입력)
gorge write \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --sender 0x9d1fb18F07AC7D608Ea920448F4cd3e11b50Fa5C \
  --out mainnet_operation.json \
  0x8Ba7DD53c4C58162e01a91d9a4d39A867CE8BBa7 \
  "createPair(address,uint256,uint256,bytes)" \
  0x1234567890123456789012345678901234567890 \
  1000 \
  2000 \
  $FEE_DATA

# 3. gorge write --append (모든 값을 직접 입력)
gorge write \
  --append \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --sender 0x9d1fb18F07AC7D608Ea920448F4cd3e11b50Fa5C \
  --out mainnet_operation.json \
  0x0E25C9AA04045C4E1AF17C7bd2eD313dD985f918 \
  "setMarketFees(uint32,uint32,uint32,uint32)" \
  100 100 50 50

# 4. gorge sign
gorge sign --out mainnet_operation_sign.json mainnet_operation.json

# 5. gorge send
gorge send \
  --out mainnet_operation_send.json \
  --rpc-url https://mainnet.crosstoken.io:22001 \
  --wait 60 \
  mainnet_operation_sign.json
```

**⚠️ 핵심 규칙:**
- 절대 쉘 스크립트(.sh) 형식으로 작성하지 마세요
- 모든 주소, RPC URL, 숫자를 직접 입력하세요 (변수 사용 금지)
- 유일한 예외: `cast calldata` 결과값만 변수 허용

---

## 자주 사용되는 Fee Data

```bash
# Market 설정 상속 (NO_FEE_BPS)
FEE_DATA=$(cast abi-encode "f(uint32,uint32,uint32,uint32)" 0xFFFFFFFF 0xFFFFFFFF 0xFFFFFFFF 0xFFFFFFFF)

# 0.1% (10 BPS)
FEE_DATA=$(cast abi-encode "f(uint32,uint32,uint32,uint32)" 10 10 10 10)

# 0.2% (20 BPS)
FEE_DATA=$(cast abi-encode "f(uint32,uint32,uint32,uint32)" 20 20 20 20)

# 다른 조합 (seller: 0.1%, buyer: 0.2%)
FEE_DATA=$(cast abi-encode "f(uint32,uint32,uint32,uint32)" 10 10 20 20)
```

---

## 정리

이 문서를 학습한 AI는:

1. ✅ 사용자의 자연어 요청을 이해
2. ✅ 환경과 Market 타입 식별
3. ✅ Verse8인 경우 execute() 사용
4. ✅ 올바른 주소와 파라미터로 스크립트 생성
5. ✅ 배치 작업 시 --append 사용
6. ✅ 완전한 write → sign → send 스크립트 제공

**핵심: Verse8 Market은 항상 Verse8MarketOwner의 execute() 함수를 통해 간접 호출해야 함!**

