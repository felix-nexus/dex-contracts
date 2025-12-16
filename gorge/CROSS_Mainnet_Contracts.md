# CROSS Mainnet Contract Addresses

**RPC Endpoint:**  
`https://mainnet.crosstoken.io:22001`

---

## Proxy Contracts

| Name | Address |
|------|----------|
| cross dex proxy | `0x89e23B854e432e5c759D49e643d3e612EadB7a6B` |
| cross router proxy | `0x6690844Aac584AcA982E195B7BDeBd48740fbcb1` |
| CROSSvsCROSS | `0x52D3256c7d6C7522C6D593b2aC662dBF610E6813` |

---

## Access Control

### CrossDex Proxy (`0x89e23B854e432e5c759D49e643d3e612EadB7a6B`)

**Owner:** `0x22C1522276855B028c31a731BA10D125811Af37c`

**Owner Functions:**
- `createMarket(address _owner, address quote, address feeCollector, bytes data, string message)` - Create new market
- `setTickSizeSetter(address setter)` - Set tick size setter address
- `setPairImpl(address _pairImpl)` - Update pair implementation
- `setMarketImpl(address _marketImpl)` - Update market implementation
- `upgradeTo(address newImplementation)` - Upgrade contract

**TickSizeSetter:** `0x04F01a3042e536dae19068E6E916CC81D563d0D7`
- Can call `setTickSize(uint256 _lotSize, uint256 _tickSize)` on any Pair

### CrossDex Router (`0x6690844Aac584AcA982E195B7BDeBd48740fbcb1`)

**Owner:** Same as CrossDex (inherits from CrossDex proxy)

**Owner Functions:**
- `setFindPrevPriceCount(uint256 _findPrevPriceCount)` - Update search depth
- `setMaxMatchCount(uint256 _maxMatchCount)` - Update max match limit
- `setCancelLimit(uint256 _cancelLimit)` - Update cancel limit
- `setWhitelistedCodeAccount(address[] accounts, bool whitelisted)` - Whitelist contract accounts
- `upgradeTo(address newImplementation)` - Upgrade contract

---

## Game Market

**Contract Address:** `0xa0f50f79615247530fABcC3efd79B8e5b961b966`

**Owner:** `0xafcc9E7d739b03CC53e2152d368f365430CF3CCa`

**FeeCollector:** `0xC053e2B67d9CB281e20d4C0C8e8d7b72e1629686`

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
| ZENY/CROSS | `0xc4BFB8E247ebf889e13828521247dc66c0DB5976` | `0xe9013a5231BEB721f4F801F2d07516b8ca19d953` |
| MGT/CROSS | `0xB53B4436d2BFBfF8d139129411975d4706Dc14eE` | `0x5B1579a758916560F00212B78a7AF728eAA0ffa9` |
| BNGO/CROSS | `0xb103bc89bc8b53D130da0452f8202ceeFd3d35Db` | `0x3Ead8192316d86Bc6Ea5Edaf1Ee98eA785C57047` |
| RUBYX/CROSS | `0x77F322F2e7913193B0eF6beaa356aC06B8448DA6` | `0x7bD648B4B0169C1c12d1060dFDb4005f2Ac881c0` |
| SHOUT/CROSS | `0x8fbC85A89B82B733208aa4aE4764ea4006f1Eb90` | `0x366aBA15a502e37e72c0Bbfad98c25Dd5322E20F` |
| DBS/CROSS | `0xC87640982648eb7B0A6BAb9551fEf5FCF93543f3` | `0x219eb5611C7838c1180291352E07AE21684e0d03` |

---

## USDTx Market

**Contract Address:** `0xB7811907b2839d6b5CCF908D6B58dE944D8AfbA7`

**Owner:** `0xafcc9E7d739b03CC53e2152d368f365430CF3CCa` (Same as Game Market)

**FeeCollector:** `0x2AAbB65C85E8B6Fa278BC15CcFC3E7F7C9E2Ee94`

**Owner Functions:** Same as Game Market

| Name | Pair | Base |
|------|------|------|
| ZENY/USDTx | `0xE67656cF8327d51DD510984032Bd14b118524565` | `0xe9013a5231BEB721f4F801F2d07516b8ca19d953` |
| MGT/USDTx | `0xdC4dbf1FbB916F854f6D80fc8BbA285d15a29A00` | `0x5B1579a758916560F00212B78a7AF728eAA0ffa9` |
| BNGO/USDTx | `0x425d36bDd3d7968CdD38C452701A1aE6595fC606` | `0x3Ead8192316d86Bc6Ea5Edaf1Ee98eA785C57047` |
| RUBYX/USDTx | `0x44AC60d1b6AfBb79beC09F4ebB7dE020F5267aBA` | `0x7bD648B4B0169C1c12d1060dFDb4005f2Ac881c0` |
| CROSS/USDTx | `0x71622fFD5461cb729Aaf20728F426f7DEDd9Dc73` | `0x52D3256c7d6C7522C6D593b2aC662dBF610E6813` |
| SHOUT/USDTx | `0xa8784FAA2ce3748F894dcDab052894cd55289256` | `0x366aBA15a502e37e72c0Bbfad98c25Dd5322E20F` |
| DBS/USDTx | `0x7AdD9C2eceAA89FB0e8B57342234be3397c86258` | `0x219eb5611C7838c1180291352E07AE21684e0d03` |

**USDTx Token Address:**  
`0x1c3744f44ed3e8c6d9686081CfA89e97112a85c5`

---

## Verse8 Market

**Contract Address:** `0xcb95777d0f8d2EfA5e836Cb65f814dF8C7261d83`

**Owner:** `0x4469a9879Ec02CE4313bef5e2F21b4F17f2B70e4` (Verse8MarketOwner Contract)

**FeeCollector:** `0x2AAbB65C85E8B6Fa278BC15CcFC3E7F7C9E2Ee94`

### Verse8MarketOwner Contract (`0x4469a9879Ec02CE4313bef5e2F21b4F17f2B70e4`)

This contract uses AccessControl with role-based permissions:

**DEFAULT_ADMIN_ROLE (0x00...00):**
- `execute(address to, uint256 value, bytes data)` - Execute arbitrary call
- `executeBatch(ExecuteBatchArgs[] calls)` - Execute multiple calls
- Can call any Market/Pair owner functions through `execute()`

**PAIR_CREATOR_ROLE (`0x02d639b3242e624c4062ce3346179a769447ef0a01fb09608afc904d0268f190`):**
- `createPair(address market, address base, uint256 tickSize, uint256 lotSize, bytes feeData)` - Create single pair
- `createPairs(CreatePairArgs[] args)` - Create multiple pairs in batch

| Name | Pair | Base |
|------|------|------|
| KKUL789/USDTx | `0x9a52cC333Ca7647FA8fDE626DCcBffBCc2630b66` | `0x51Ee9ebA1D95051a48316AF2B2c3462f3ceeFd00` |
| NW/USDTx | `0x08A8132639d8E493dA3752C65F8Dc695D6D03f64` | `0x869962620bF8178D43E6B18f0Ff8636b4273413c` |
| Z/USDTx | `0x289f0e66e48498292799DF65610Eb85Fb3e7F36f` | `0x7AcDABE25b949Db05850037824cB7743Bea2021d` |
| ZEDDYZZANG/USDTx | `0x83CcF5064322EC3d0d7b2719c12125E8F8A09e96` | `0xcBe30d7F1D0811D1e9bBF591F9D4168474656137` |
| BAHT/USDTx | `0xb692eBb4D9c08A67Fc8361E0700cd99896931dbF` | `0xb82116D3d9c2309738b1B95b97d73f12A460e95A` |
| JIN/USDTx | `0xC10b055D8b887DC09091f51d7F6e8c7507086775` | `0x51476F9D029F8F19b75134e4F9Ad3b8d2addE929` |
| PP/USDTx | `0x312F5C69aCE9B8fb1869D43Ac7a5a6F065A5353d` | `0x4DBE69bA9DE8EF6FB8D9B837cD3508A9568Bf2e6` |
| TRUMP/USDTx | `0x9A2E6643D7326391e474eaAd62feA530245343a8` | `0xA8069BCE353877a03c5C2f0fdB8EA3c2d4E8D739` |

**USDTx Token Address:** `0x1c3744f44ed3e8c6d9686081CfA89e97112a85c5`
