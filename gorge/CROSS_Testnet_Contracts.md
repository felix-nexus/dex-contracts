# CROSS Testnet Contract Addresses

**RPC Endpoint:**  
`https://testnet.crosstoken.io:22001`

---

## Proxy Contracts

| Name | Address |
|------|----------|
| cross dex proxy | `0xa2d6eAc8Ad4C750c6d0a5Ead48896f9d5f96e873` |
| cross router proxy | `0xAa7B538655ce087116B1bfbb0e3C6057A405C391` |
| CROSS(wCROSS) | `0x977699d909c0142fD3c8E18163cA3528b9C779C6` |

---

## Access Control

### CrossDex Proxy (`0xa2d6eAc8Ad4C750c6d0a5Ead48896f9d5f96e873`)

**Owner:** `0x535c7b8C25eADACCAf857652cA34cF4c010957E2`

**Owner Functions:**
- `createMarket(address _owner, address quote, address feeCollector, bytes data, string message)` - Create new market
- `setTickSizeSetter(address setter)` - Set tick size setter address
- `setPairImpl(address _pairImpl)` - Update pair implementation
- `setMarketImpl(address _marketImpl)` - Update market implementation
- `upgradeTo(address newImplementation)` - Upgrade contract

**TickSizeSetter:** `0x85756F8C044AdcC9B37Dc37447C5d1399E8998D0`
- Can call `setTickSize(uint256 _lotSize, uint256 _tickSize)` on any Pair

### CrossDex Router (`0xAa7B538655ce087116B1bfbb0e3C6057A405C391`)

**Owner:** Same as CrossDex (inherits from CrossDex proxy)

**Owner Functions:**
- `setFindPrevPriceCount(uint256 _findPrevPriceCount)` - Update search depth
- `setMaxMatchCount(uint256 _maxMatchCount)` - Update max match limit
- `setCancelLimit(uint256 _cancelLimit)` - Update cancel limit
- `setWhitelistedCodeAccount(address[] accounts, bool whitelisted)` - Whitelist contract accounts
- `upgradeTo(address newImplementation)` - Upgrade contract

---

## Game Market

**Contract Address:** `0x1d86752372281D0573Cb70319b434D73f2daFd59`

**Owner:** `0x535c7b8C25eADACCAf857652cA34cF4c010957E2` (Same as CrossDex)

**FeeCollector:** `0x51389f17bc07DD2674b620909446EacC6d8A863e`

**Owner Functions:**
- `createPair(address base, uint256 tickSize, uint256 lotSize, bytes feeData)` - Create new trading pair
- `setFeeCollector(address _feeCollector)` - Update fee collector address
- `setMarketFees(uint32 sellerMakerFee, uint32 sellerTakerFee, uint32 buyerMakerFee, uint32 buyerTakerFee)` - Update market-level fees (BPS)
- `setPairImpl(address _pairImpl)` - Update pair implementation
- `upgradeTo(address newImplementation)` - Upgrade contract

**Pair Owner Functions (inherited from Market owner):**
- `setPairFees(uint32 sellerMakerFee, uint32 sellerTakerFee, uint32 buyerMakerFee, uint32 buyerTakerFee)` - Set pair-specific fees
- `skim(IERC20 erc20, address to, uint256 amount)` - Withdraw excess tokens
- `emergencyCancelOrder(uint256[] orderIds)` - Cancel orders when paused
- `setPause(bool pause)` - Pause/unpause pair
- `upgradeTo(address newImplementation)` - Upgrade pair contract

| Name | Pair | Base |
|------|------|------|
| ZENY/CROSS | `0x7e89461523BD231F541516313408336870deA365` | `0xe934057Ac314cD9bA9BC17AE2378959fd39Aa2E3` |
| GHUBx/CROSS | `0x8AC006418F792E90B912fc2Aa81a4DaeD3b89A3B` | `0xD58bfe146ffBfbf440446edDaB358D26DaD16221` |
| MGT/CROSS | `0xB92DcF3829d9716B6677472D7ee609Bfde075a74` | `0x5B1579a758916560F00212B78a7AF728eAA0ffa9` |
| RUBYX/CROSS | `0xf88101f2c0ba73d6821536E39AbD2c714a713399` | `0x0e2F9543Ad4Fe561DCA895E4B5fb63D4B47Aa435` |
| BNGO/CROSS | `0x709e7570AA4AD4be7Bc73210b951f9EA42ed7ac9` | `0x86a3481d65c4325EaC45B2734482561B5a94D15D` |
| RUBYX/CROSS (2) | `0x12a69726dE5b7cb1cA9850F0E0f9d27475f3d469` | `0x2C76121125e62cf865E093e78484a86e4c85F2c1` |
| DBS/CROSS | `0x108c2e2bf4e962b986D8097EAF771Ec7467916b0` | `0x7CD1Ff20320A31ee6a2fb2fECc5CF1116C4f5154` |
| BRLx/CROSS | `0x2Ec47363573306bDa0206236E2AC2f2b0CEe2Ff5` | `0x4813d43A77bb073a1Ddf016850F1a3c5c3a029f7` |
| IDRx/CROSS | `0xcc8A68570FEb07e83B9495B61866F3f5b872aC4f` | `0x3f958A9a92acfbfd75CD80d1141Fa1717bB8913A` |
| GDO/CROSS | `0x68B1c5078c83bE3964Fc5F45Ee74E9187F30F587` | `0x149577C6110f19C2774d3A6CEB65C1FB8F8085dC` |
| tSHOUT/CROSS | `0xC30d815C8dA7eACCE0353C12E7BA4E8E56a65b88` | `0xeaA2acfB600f72DbACA7439Ffa8035156d83cb1A` |
| PHPx/CROSS | `0x0aaCd38791C68EF66C23AB37D3c490fbfE84Ba2B` | `0x2387322d78efD075F4584A54bCbA2c3754c79CcC` |
| NGNx/CROSS | `0x7E89f998Eec0e20E022283e709cd6A5f5F6F0987` | `0xbd968AaB8A0552FD09EFD3FeAf4A2cD913085A8c` |
| INRx/CROSS | `0x5f02ea33F628ED13da5002dF20BA5577B70Abb46` | `0xb248115e46cE084cAC84f22C86831d51D40f561B` |
| BDTx/CROSS | `0xDcB477102C3c6f818f7e109293D2D6B8945597aB` | `0xA8F2d220B2578A17a75d52C002f7BAc47F217dD8` |
| TRYx/CROSS | `0xBD881C447Fe999b52Aa67B77Bf578E52D6F1F2aD` | `0x36CdB382328A415a6370D6e960Ace329ec8e7123` |
| PKRx/CROSS | `0xD3375bB3D5dB06Eb13c7C3801AA0F6B0447EF4c3` | `0x00381BdbF45d23c3658316e00970E10F3E922821` |
| ARSx/CROSS | `0x91c7079D67A605Fc2ABcB53AF28879981bb91F71` | `0xE4b95bD9a2E4f0e034D50eaF4479EE32f7720bB7` |
| COPx/CROSS | `0xc18C2Ea25f7190805DB5772bDA87b07b14d15506` | `0xD9a385ba3F45e858f59760B1105F48853450A4a0` |

---

## USDTx Market

**Contract Address:** `0x7D2fA2c51DBcF5AdaB5508B338f03853598c2352`

**Owner:** `0x535c7b8C25eADACCAf857652cA34cF4c010957E2` (Same as CrossDex)

**FeeCollector:** `0xa76662F3C93C3981bEF6EE676472215Fb9171dca`

**Owner Functions:** Same as Game Market

| Name | Pair | Base |
|------|------|------|
| ZENY/USDTx | `0x2A239357968836D69Cda8323bA38f9A62F35462a` | `0xe934057Ac314cD9bA9BC17AE2378959fd39Aa2E3` |
| MGT/USDTx | `0xAB27E2857868F6cA5650AEf442fd55704103Afa7` | `0x5B1579a758916560F00212B78a7AF728eAA0ffa9` |
| BNGO/USDTx | `0x66C24A9f913BfaE6459A23d9C800BEA7a87b36DE` | `0x86a3481d65c4325EaC45B2734482561B5a94D15D` |
| RUBYX/USDTx | `0x66Bb6deBe6B65BEa0BE37331f6d1B418ADf51B3C` | `0x2C76121125e62cf865E093e78484a86e4c85F2c1` |
| CROSS/USDTx | `0xB703a41926E601d2Af5498DC6B07493B6b0ed47b` | `0x977699d909c0142fD3c8E18163cA3528b9C779C6` |
| tSHOUT/USDTx | `0x95054dc24800a2bDD570bbbC62876d1d4a5426F6` | `0xeaA2acfB600f72DbACA7439Ffa8035156d83cb1A` |

**USDTx Token Address:**  
`0x9F85c75B7637E18f946cc8AF9C131318c6833d9`

---

## Verse8 Market (USDTx)

**Contract Address:** `0xCcb6F48f18da94B7d90e29adBF762FBf70564844`

**Owner:** `0xFc2852EeaA01b4A9B52eF36Eb24DF56432f8043e` (Verse8MarketOwner Contract)

**FeeCollector:** `0x51389f17bc07DD2674b620909446EacC6d8A863e`

### Verse8MarketOwner Contract (`0xFc2852EeaA01b4A9B52eF36Eb24DF56432f8043e`)

This contract uses AccessControl with role-based permissions:

**DEFAULT_ADMIN_ROLE (0x00...00):**
- `execute(address to, uint256 value, bytes data)` - Execute arbitrary call
- `executeBatch(ExecuteBatchArgs[] calls)` - Execute multiple calls
- Can call any Market/Pair owner functions through `execute()`

**PAIR_CREATOR_ROLE (`0x02d639b3242e624c4062ce3346179a769447ef0a01fb09608afc904d0268f190`):**
- `createPair(address market, address base, uint256 tickSize, uint256 lotSize, bytes feeData)` - Create single pair
- `createPairs(CreatePairArgs[] args)` - Create multiple pairs in batch

**USDTx Token Address:**  
`0x9F85c75B7637E18f946cc8AF9C131318c6833d9`

