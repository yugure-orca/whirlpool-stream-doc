🚨 THIS FEATURE IS STILL EXPERIMENTAL

![whirlpool-stream-landing](https://github.com/user-attachments/assets/29d360b6-798c-4d81-9bdc-bc2b47579fac)

# What is Whirlpool Stream
Whirlpool Stream is a semi-real-time distribution of the followings:
- event: instructions executed by Whirlpool program along with useful data
- account delta: delta data of account data owned by Whirlpool program

Useful data that cannot be obtained from block data, such as price changes before and after swaps, or liquidity changes before and after position operations, can be obtained.

It also provides strong data consistency.

The data delivered is always for the slot with the next block height after the previously delivered block height, and is never missing.
In addition, delivery can start from any slot within a 3-day period, so if the stream is disconnected, the stream can be resumed without losing any data.

## Client Library
https://github.com/yugure-orca/whirlpool-stream-client

## Demo
### event stream (default)
https://orcanauts-a.whirlpool-stream.pleiades.dev/demo/stream/refined/sse?limit=20

### account delta stream
https://orcanauts-a.whirlpool-stream.pleiades.dev/demo/stream/refined/sse?limit=20&event=none&account=all

### event & account delta stream
https://orcanauts-a.whirlpool-stream.pleiades.dev/demo/stream/refined/sse?limit=20&event=all&account=all

# Whirlpool Stream Endpoint
## Endpoint
There are 2 endpoints (`orcanauts-a` and `orcanauts-b`) for high availability purpose.

| Protocol           | Endpoint                                                                        |
| ------------------ | ------------------------------------------------------------------------------- |
| Server-Sent Events | `https://orcanauts-a.whirlpool-stream.pleiades.dev/<APIKEY>/stream/refined/sse` |
|                    | `https://orcanauts-b.whirlpool-stream.pleiades.dev/<APIKEY>/stream/refined/sse` |
| WebSocket          | `https://orcanauts-a.whirlpool-stream.pleiades.dev/<APIKEY>/stream/refined/ws`  |
|                    | `https://orcanauts-b.whirlpool-stream.pleiades.dev/<APIKEY>/stream/refined/ws`  |

You can try the following "DEMO" endpoint.
But its concurrency is 1, so connection will be closed when another request is made.

🙏 Please use this endpoint only as a "taste" of the Whirlpool Stream.
If you like the functionality of the Whirlpool Stream, obtain your own APIKEY.

| Protocol           | Endpoint                                                   |
| ------------------ | ---------------------------------------------------------- |
| Server-Sent Events | `https://orcanauts-a.whirlpool-stream.pleiades.dev/demo/stream/refined/sse` |
|                    | `https://orcanauts-b.whirlpool-stream.pleiades.dev/demo/stream/refined/sse` |
| WebSocket          | `https://orcanauts-a.whirlpool-stream.pleiades.dev/demo/stream/refined/ws`  |
|                    | `https://orcanauts-b.whirlpool-stream.pleiades.dev/demo/stream/refined/ws`  |

## Parameters
| Parameter | Required | Type                                      | Default     | Purpose                                                                       |
| --------- | -------- | ----------------------------------------- | ----------- | ----------------------------------------------------------------------------- |
| slot      | no       | u64                                       | latest slot | Start sending from the slot right AFTER the specified slot                    |
| limit     | no       | u64                                       | 256         | The stream is closed after the specified number of blocks have been sent out. |
| event     | no       | "trade" \| "liquidity" \| "all" \| "none" | "all"       | Send out only specific Whirlpool events                                       |
| account   | no       | "trade" \| "all" \| "none"                | "none"      | Send out only specific Whirlpool account delta                                |

### Notes
- `slot` can be a slot number within the last 3 days. In other words, this endpoint has the ability to start a stream going back 3 days in the past.
- Even if a large number is specified as `limit`, the stream will be disconnected once a day. For continuous connection, implement a reconnection mechanism. `slot` will help implementing reconnection without data loss.

## APIKEY
Because this endpoint sends out large amounts of data, abuse must be prevented. Therefore, it uses APIKEY and accepts only one connection per APIKEY at a time.

You can get APIKEY freely if you have Orcanauts NFT.
APIKEY is issued for each Orcanauts, and you can view APIKEY by signing by the wallet that holds the orcanauts at the following site (OrcanautsApikey).

https://orcanauts-apikey.pleiades.dev/

### Notes
- In OrcanautsApikey site, only message signing is required and transaction signing is never requested.
- When a Orcanauts NFT is sent from the wallet, APIKEY is deactivated. A new APIKEY will be issued for the destination wallet.
- Since Orcanauts ownership information is refreshed approximately every 30 minutes, it may take up to 30 minutes for Orcanauts moves to be reflected in APIKEY.
- It should be noted that this feature is being made available to Orcanauts owners because it was convenient for APIKEY management, with no intention of providing any permanent added value to Orcanauts.

## `event` Parameter
### `none`
No events is sent out.

### `all`
All events is sent out.

### `trade`
Only the following events are sent out.

- Traded

### `liquidity`
Only the following events are sent out.

- Traded
- LiquidityDeposited
- LiquidityWithdrawn
- LiquidityPatched
- PoolInitialized
- PoolFeeRateUpdated
- PoolProtocolFeeRateUpdated
- ConfigInitialized
- ConfigUpdated
- ConfigExtensionInitialized
- ConfigExtensionUpdated
- FeeTierInitialized
- FeeTierUpdated
- TokenBadgeInitialized
- TokenBadgeDeleted
- TickArrayInitialized
- RewardInitialized
- RewardEmissionsUpdated
- RewardAuthorityUpdated

## `account` Parameter
### `none`
No account deltas is sent out.

### `all`
All account deltas is sent out.

### `trade`
Only the account deltas for the following accounts are sent out.

- Whirlpool
- TickArray

# Event

## Instruction / Event Mapping
- All instructions are mapped to specific events.
- Some instructions are mapped to the same event when the effects are very similar.
- TwoHopSwap and TwoHopSwapV2 generate two Traded events.
- The upgrade of the program generate special event `ProgramDeployed`.

| Event                      | Instruction                          |
| -------------------------- | ------------------------------------ |
| ProgramDeployed            | (no whirlpool instruction)           |
| ConfigExtensionInitialized | InitializeConfigExtension            |
| ConfigExtensionUpdated     | SetConfigExtensionAuthority          |
|                            | SetTokenBadgeAuthority               |
| ConfigInitialized          | InitializeConfig                     |
| ConfigUpdated              | SetFeeAuthority                      |
|                            | SetCollectProtocolFeesAuthority      |
|                            | SetRewardEmissionsSuperAuthority     |
|                            | SetDefaultProtocolFeeRate            |
| FeeTierInitialized         | InitializeFeeTier                    |
| FeeTierUpdated             | SetDefaultFeeRate                    |
| LiquidityDeposited         | IncreaseLiquidity                    |
|                            | IncreaseLiquidityV2                  |
| LiquidityPatched           | AdminIncreaseLiquidity               |
| LiquidityWithdrawn         | DecreaseLiquidity                    |
|                            | DecreaseLiquidityV2                  |
| PoolFeeRateUpdated         | SetFeeRate                           |
| PoolInitialized            | InitializePool                       |
|                            | InitializePoolV2                     |
| PoolProtocolFeeRateUpdated | SetProtocolFeeRate                   |
| PositionBundleDeleted      | DeletePositionBundle                 |
| PositionBundleInitialized  | InitializePositionBundle             |
|                            | InitializePositionBundleWithMetadata |
| PositionClosed             | ClosePosition                        |
|                            | ClosePositionWithTokenExtensions     |
|                            | CloseBundledPosition                 |
| PositionFeesHarvested      | CollectFees                          |
|                            | CollectFeesV2                        |
| PositionHarvestUpdated     | UpdateFeesAndRewards                 |
| PositionOpened             | OpenPosition                         |
|                            | OpenPositionWithMetadata             |
|                            | OpenPositionWithTokenExtensions      |
|                            | OpenBundledPosition                  |
| PositionRewardHarvested    | CollectReward                        |
|                            | CollectRewardV2                      |
| ProtocolFeesCollected      | CollectProtocolFees                  |
|                            | CollectProtocolFeesV2                |
| RewardAuthorityUpdated     | SetRewardAuthority                   |
|                            | SetRewardAuthorityBySuperAuthority   |
| RewardEmissionsUpdated     | SetRewardEmissions                   |
|                            | SetRewardEmissionsV2                 |
| RewardInitialized          | InitializeReward                     |
|                            | InitializeRewardV2                   |
| TickArrayInitialized       | InitializeTickArray                  |
| TokenBadgeDeleted          | DeleteTokenBadge                     |
| TokenBadgeInitialized      | InitializeTokenBadge                 |
| Traded                     | Swap                                 |
|                            | SwapV2                               |
|                            | TwoHopSwap                           |
|                            | TwoHopSwapV2                         |

## Event Definition
- To reduce data size in the stream, field names and values are abbreviated.

### ProgramDeployed (`PD`)
no fields

### ConfigExtensionInitialized (`CEI`)
| JSON Field | Name                       | Type         | Notes                     |
| ---------- | -------------------------- | ------------ | ------------------------- |
| o          | origin                     | "ice"        | InitializeConfigExtension |
| c          | config                     | PubkeyString |                           |
| ce         | config_extension           | PubkeyString |                           |
| cea        | config_extension_authority | PubkeyString |                           |
| tba        | token_badge_authority      | PubkeyString |                           |

### ConfigExtensionUpdated (`CEU`)
| JSON Field | Name                           | Type         | Notes                       |
| ---------- | ------------------------------ | ------------ | --------------------------- |
| o          | origin                         | "scea"       | SetConfigExtensionAuthority |
|            |                                | "stba"       | SetTokenBadgeAuthority      |
| c          | config                         | PubkeyString |                             |
| ce         | config_extension               | PubkeyString |                             |
| ocea       | old_config_extension_authority | PubkeyString | value before the event      |
| ncea       | new_config_extension_authority | PubkeyString | value after the event       |
| otba       | old_token_badge_authority      | PubkeyString | value before the event      |
| ntba       | new_token_badge_authority      | PubkeyString | value after the event       |

### ConfigInitialized (`CI`)
| JSON Field | Name                             | Type         | Notes            |
| ---------- | -------------------------------- | ------------ | ---------------- |
| o          | origin                           | "ic"         | InitializeConfig |
| c          | config                           | PubkeyString |                  |
| fa         | fee_authority                    | PubkeyString |                  |
| cpfa       | collect_protocol_fees_authority  | PubkeyString |                  |
| resa       | reward_emissions_super_authority | PubkeyString |                  |
| dpfr       | default_protocol_fee_rate        | u16          |                  |

### ConfigUpdated (`CU`)
| JSON Field | Name                                 | Type         | Notes                            |
| ---------- | ------------------------------------ | ------------ | -------------------------------- |
| o          | origin                               | "sfa"        | SetFeeAuthority                  |
|            |                                      | "scpfa"      | SetCollectProtocolFeesAuthority  |
|            |                                      | "sresa"      | SetRewardEmissionsSuperAuthority |
|            |                                      | "sdpfr"      | SetDefaultProtocolFeeRate        |
| c          | config                               | PubkeyString |                                  |
| ofa        | old_fee_authority                    | PubkeyString | value before the event           |
| nfa        | new_fee_authority                    | PubkeyString | value after the event            |
| ocpfa      | old_collect_protocol_fee_authority   | PubkeyString | value before the event           |
| ncpfa      | new_collect_protocol_fee_authority   | PubkeyString | value after the event            |
| oresa      | old_reward_emissions_super_authority | PubkeyString | value before the event           |
| nresa      | new_reward_emissions_super_authority | PubkeyString | value after the event            |
| odpfr      | old_default_protocol_fee_rate        | u16          | value before the event           |
| ndpfr      | new_default_protocol_fee_rate        | u16          | value after the event            |

### FeeTierInitialized (`FTI`)
| JSON Field | Name             | Type         | Notes             |
| ---------- | ---------------- | ------------ | ----------------- |
| o          | origin           | "ift"        | InitializeFeeTier |
| c          | config           | PubkeyString |                   |
| ft         | fee_tier         | PubkeyString |                   |
| ts         | tick_spacing     | u16          |                   |
| dfr        | default_fee_rate | u16          |                   |

### FeeTierUpdated (`FTU`)
| JSON Field | Name                 | Type         | Notes             |
| ---------- | -------------------- | ------------ | ----------------- |
| o          | origin               | "sdfr"       | SetDefaultFeeRate |
| c          | config               | PubkeyString |                   |
| ft         | fee_tier             | PubkeyString |                   |
| ts         | tick_spacing         | u16          |                   |
| odfr       | old_default_fee_rate | u16          |                   |
| ndfr       | new_default_fee_rate | u16          |                   |

### LiquidityDeposited (`LD`)
| JSON Field | Name                         | Type         | Notes                  |
| ---------- | ---------------------------- | ------------ | ---------------------- |
| o          | origin                       | "il"         | IncreaseLiquidity      |
|            |                              | "ilv2"       | IncreaseLiquidityV2    |
| w          | whirlpool                    | PubkeyString |                        |
| pa         | position_authority           | PubkeyString |                        |
| p          | position                     | PubkeyString |                        |
| lta        | lower_tick_array             | PubkeyString |                        |
| uta        | upper_tick_array             | PubkeyString |                        |
| ld         | liquidity_delta              | u128         |                        |
| ta         | transfer_a                   | TransferInfo |                        |
| tb         | transfer_b                   | TransferInfo |                        |
| lti        | lower_tick_index             | i32          |                        |
| uti        | upper_tick_index             | i32          |                        |
| ldp        | lower_decimal_price          | DecimalPrice |                        |
| udp        | upper_decimal_price          | DecimalPrice |                        |
| opl        | old_position_liquidity       | u128         | value before the event |
| npl        | new_position_liquidity       | u128         | value after the event  |
| owl        | old_whirlpool_liquidity      | u128         | value before the event |
| nwl        | new_whirlpool_liquidity      | u128         | value after the event  |
| wsp        | whirlpool_sqrt_price         | u128         |                        |
| wcti       | whirlpool_current_tick_index | i32          |                        |
| wdp        | whirlpool_decimal_price      | DecimalPrice |                        |

### LiquidityPatched (`LP`)
| JSON Field | Name                    | Type         | Notes                  |
| ---------- | ----------------------- | ------------ | ---------------------- |
| o          | origin                  | "ail"        | AdminIncreaseLiquidity |
| w          | whirlpool               | PubkeyString |                        |
| ld         | liquidity_delta         | u128         |                        |
| owl        | old_whirlpool_liquidity | u128         | value before the event |
| nwl        | new_whirlpool_liquidity | u128         | value after the event  |

### LiquidityWithdrawn (`LW`)
| JSON Field | Name                         | Type         | Notes                  |
| ---------- | ---------------------------- | ------------ | ---------------------- |
| o          | origin                       | "dl"         | DecreaseLiquidity      |
|            |                              | "dlv2"       | DecreaseLiquidityV2    |
| w          | whirlpool                    | PubkeyString |                        |
| pa         | position_authority           | PubkeyString |                        |
| p          | position                     | PubkeyString |                        |
| lta        | lower_tick_array             | PubkeyString |                        |
| uta        | upper_tick_array             | PubkeyString |                        |
| ld         | liquidity_delta              | u128         |                        |
| ta         | transfer_a                   | TransferInfo |                        |
| tb         | transfer_b                   | TransferInfo |                        |
| lti        | lower_tick_index             | i32          |                        |
| uti        | upper_tick_index             | i32          |                        |
| ldp        | lower_decimal_price          | DecimalPrice |                        |
| udp        | upper_decimal_price          | DecimalPrice |                        |
| opl        | old_position_liquidity       | u128         | value before the event |
| npl        | new_position_liquidity       | u128         | value after the event  |
| owl        | old_whirlpool_liquidity      | u128         | value before the event |
| nwl        | new_whirlpool_liquidity      | u128         | value after the event  |
| wsp        | whirlpool_sqrt_price         | u128         |                        |
| wcti       | whirlpool_current_tick_index | i32          |                        |
| wdp        | whirlpool_decimal_price      | DecimalPrice |                        |

### PoolFeeRateUpdated (`PFRU`)
| JSON Field | Name         | Type         | Notes                  |
| ---------- | ------------ | ------------ | ---------------------- |
| o          | origin       | "sfr"        | SetFeeRate             |
| c          | config       | PubkeyString |                        |
| w          | whirlpool    | PubkeyString |                        |
| ofr        | old_fee_rate | u16          | value before the event |
| nfr        | new_fee_rate | u16          | value after the event  |
|            |              |              |                        |

### PoolInitialized (`PI`)
| JSON Field | Name               | Type         | Notes            |
| ---------- | ------------------ | ------------ | ---------------- |
| o          | origin             | "ip"         | InitializePool   |
|            |                    | "ipv2"       | InitializePoolV2 |
| ts         | tick_spacing       | u16          |                  |
| sp         | sqrt_price         | u128         | initial value    |
| dp         | decimal_price      | DecimalPrice |                  |
| c          | config             | PubkeyString |                  |
| tma        | token_mint_a       | PubkeyString |                  |
| tmb        | token_mint_b       | PubkeyString |                  |
| f          | funder             | PubkeyString |                  |
| w          | whirlpool          | PubkeyString |                  |
| ft         | fee_tier           | PubkeyString |                  |
| tpa        | token_program_a    | TokenProgram |                  |
| tpb        | token_program_b    | TokenProgram |                  |
| tda        | token_decimals_a   | Decimals     |                  |
| tdb        | token_decimals_b   | Decimals     |                  |
| cti        | current_tick_index | i32          | initial value    |
| fr         | fee_rate           | u16          |                  |
| pfr        | protocol_fee_rate  | u16          |                  |

### PoolProtocolFeeRateUpdated (`PPFRU`)
| JSON Field | Name                  | Type         | Notes                  |
| ---------- | --------------------- | ------------ | ---------------------- |
| o          | origin                | "spfr"       | SetProtocolFeeRate     |
| c          | config                | PubkeyString |                        |
| w          | whirlpool             | PubkeyString |                        |
| opfr       | old_protocol_fee_rate | u16          | value before the event |
| npfr       | new_protocol_fee_rate | u16          | value after the event  |

### PositionBundleDeleted (`PBD`)
| JSON Field | Name                  | Type         | Notes                |
| ---------- | --------------------- | ------------ | -------------------- |
| o          | origin                | "dpb"        | DeletePositionBundle |
| pb         | position_bundle       | PubkeyString |                      |
| pbm        | position_bundle_mint  | PubkeyString |                      |
| pbo        | position_bundle_owner | PubkeyString |                      |

### PositionBundleInitialized (`PBI`)
| JSON Field | Name                  | Type         | Notes                                |
| ---------- | --------------------- | ------------ | ------------------------------------ |
| o          | origin                | "ipb"        | InitializePositionBundle             |
|            |                       | "ipbwm"      | InitializePositionBundleWithMetadata |
| pb         | position_bundle       | PubkeyString |                                      |
| pbm        | position_bundle_mint  | PubkeyString |                                      |
| pbo        | position_bundle_owner | PubkeyString |                                      |

### PositionClosed (`PC`)
| JSON Field | Name                  | Type         | Notes                            |
| ---------- | --------------------- | ------------ | -------------------------------- |
| o          | origin                | "cp"         | ClosePosition                    |
|            |                       | "cbp"        | CloseBundledPosition             |
|            |                       | "cpwte"      | ClosePositionWithTokenExtensions |
| w          | whirlpool             | PubkeyString |                                  |
| p          | position              | PubkeyString |                                  |
| lti        | lower_tick_index      | i32          |                                  |
| uti        | upper_tick_index      | i32          |                                  |
| ldp        | lower_decimal_price   | DecimalPrice |                                  |
| udp        | upper_decimal_price   | DecimalPrice |                                  |
| pa         | position_authority    | PubkeyString |                                  |
| pt         | position_type         | "p"          | Position                         |
|            |                       | "bp"         | BundledPosition                  |
| pm         | position_mint         | PubkeyString | Position only                    |
| pbm        | position_bundle_mint  | PubkeyString | BundledPosition only             |
| pb         | position_bundle       | PubkeyString | BundledPosition only             |
| pbi        | position_bundle_index | u16          | BundledPosition only             |

### PositionFeesHarvested (`PFH`)
| JSON Field | Name               | Type         | Notes         |
| ---------- | ------------------ | ------------ | ------------- |
| o          | origin             | "cf"         | CollectFees   |
|            |                    | "cfv2"       | CollectFeesV2 |
| w          | whirlpool          | PubkeyString |               |
| pa         | position_authority | PubkeyString |               |
| p          | position           | PubkeyString |               |
| ta         | transfer_a         | TransferInfo |               |
| tb         | transfer_b         | TransferInfo |               |

### PositionHarvestUpdated (`PHU`)
| JSON Field | Name               | Type         | Notes                |
| ---------- | ------------------ | ------------ | -------------------- |
| o          | origin             | "ufar"       | UpdateFeesAndRewards |
| w          | whirlpool          | PubkeyString |                      |
| p          | position           | PubkeyString |                      |

### PositionOpened (`PO`)
| JSON Field | Name                  | Type         | Notes                           |
| ---------- | --------------------- | ------------ | ------------------------------- |
| o          | origin                | "op"         | OpenPosition                    |
|            |                       | "opwm"       | OpenPositionWithMetadata        |
|            |                       | "obp"        | OpenBundledPosition             |
|            |                       | "opwte"      | OpenPositionWithTokenExtensions |
| w          | whirlpool             | PubkeyString |                                 |
| p          | position              | PubkeyString |                                 |
| lti        | lower_tick_index      | i32          |                                 |
| uti        | upper_tick_index      | i32          |                                 |
| ldp        | lower_decimal_price   | DecimalPrice |                                 |
| udp        | upper_decimal_price   | DecimalPrice |                                 |
| pa         | position_authority    | PubkeyString |                                 |
| pt         | position_type         | "p"          | Position                        |
|            |                       | "bp"         | BundledPosition                 |
| pm         | position_mint         | PubkeyString | Position only                   |
| pbm        | position_bundle_mint  | PubkeyString | BundledPosition only            |
| pb         | position_bundle       | PubkeyString | BundledPosition only            |
| pbi        | position_bundle_index | u16          | BundledPosition only            |

### PositionRewardHarvested (`PRH`)
| JSON Field | Name               | Type         | Notes           |
| ---------- | ------------------ | ------------ | --------------- |
| o          | origin             | "cr"         | CollectReward   |
|            |                    | "crv2"       | CollectRewardV2 |
| w          | whirlpool          | PubkeyString |                 |
| pa         | position_authority | PubkeyString |                 |
| p          | position           | PubkeyString |                 |
| ri         | reward_index       | u8           |                 |
| tr         | transfer_reward    | TransferInfo |                 |

### ProtocolFeesCollected (`PFC`)
| JSON Field | Name                            | Type         | Notes                 |
| ---------- | ------------------------------- | ------------ | --------------------- |
| o          | origin                          | "cpf"        | CollectProtocolFees   |
|            |                                 | "cpfv2"      | CollectProtocolFeesV2 |
| c          | config                          | PubkeyString |                       |
| w          | whirlpool                       | PubkeyString |                       |
| cpfa       | collect_protocol_fees_authority | PubkeyString |                       |
| ta         | transfer_a                      | TransferInfo |                       |
| tb         | transfer_b                      | TransferInfo |                       |

### RewardAuthorityUpdated (`RAU`)
| JSON Field | Name                 | Type         | Notes                                 |
| ---------- | -------------------- | ------------ | ------------------------------------- |
| o          | origin               | "sra"        | SetRewardAuthority                    |
|            |                      | "srabsa"     | SetRewardAuthorityBySuperAuthhhhority |
| w          | whirlpool            | PubkeyString |                                       |
| ri         | reward_index         | u8           |                                       |
| ora        | old_reward_authority | PubkeyString | value before the event                |
| nra        | new_reward_authority | PubkeyString | value after the event                 |

### RewardEmissionsUpdated (`REU`)
| JSON Field | Name                        | Type         | Notes                  |
| ---------- | --------------------------- | ------------ | ---------------------- |
| o          | origin                      | "sre"        | SetRewardEmissions     |
|            |                             | "srev2"      | SetRewardEmissionsV2   |
| w          | whirlpool                   | PubkeyString |                        |
| ri         | reward_index                | u8           |                        |
| rm         | reward_mint                 | PubkeyString |                        |
| rd         | reward_decimals             | Decimals     |                        |
| oepsx64    | old_emissions_per_secod_x64 | u128         | value before the event |
| nepsx64    | new_emissions_per_secod_x64 | u128         | value after the event  |

### RewardInitialized (`RI`)
| JSON Field | Name                 | Type         | Notes              |
| ---------- | -------------------- | ------------ | ------------------ |
| o          | origin               | "ir"         | InitializeReward   |
|            |                      | "irv2"       | InitializeRewardV2 |
| w          | whirlpool            | PubkeyString |                    |
| ri         | reward_index         | u8           |                    |
| rm         | reward_mint          | PubkeyString |                    |
| rtp        | reward_token_program | TokenProgram |                    |
| rd         | reward_decimals      | Decimals     |                    |

### TickArrayInitialized (`TAI`)
| JSON Field | Name             | Type         | Notes               |
| ---------- | ---------------- | ------------ | ------------------- |
| o          | origin           | "ita"        | InitializeTickArray |
| w          | whirlpool        | PubkeyString |                     |
| sti        | start_tick_index | i32          |                     |
| ta         | tick_array       | PubkeyString |                     |

### TokenBadgeDeleted (`TBD`)
| JSON Field | Name             | Type         | Notes            |
| ---------- | ---------------- | ------------ | ---------------- |
| o          | origin           | "dtb"        | DeleteTokenBadge |
| c          | config           | PubkeyString |                  |
| ce         | config_extension | PubkeyString |                  |
| tm         | token_mint       | PubkeyString |                  |
| tb         | token_badge      | PubkeyString |                  |

### TokenBadgeInitialized (`TBI`)
| JSON Field | Name             | Type         | Notes                |
| ---------- | ---------------- | ------------ | -------------------- |
| o          | origin           | "itb"        | InitializeTokenBadge |
| c          | config           | PubkeyString |                      |
| ce         | config_extension | PubkeyString |                      |
| tm         | token_mint       | PubkeyString |                      |
| tb         | token_badge      | PubkeyString |                      |

### Traded (`T`)
| JSON Field | Name                   | Type         | Notes                  |
| ---------- | ---------------------- | ------------ | ---------------------- |
| o          | origin                 | "s"          | Swap                   |
|            |                        | "sv2"        | SwapV2                 |
|            |                        | "thso"       | TwoHopSwap (one)       |
|            |                        | "thst"       | TwoHopSwap (two)       |
|            |                        | "thsv2o"     | TwoHopSwapV2 (one)     |
|            |                        | "thsv2t"     | TwoHopSwapV2 (two)     |
| w          | whirlpool              | PubkeyString |                        |
| ta         | token_authority        | PubkeyString |                        |
| tm         | trade_mode             | "ei"         | ExactInput             |
|            |                        | "eo"         | ExactOutput            |
| td         | trade_direction        | "ab"         | A to B                 |
|            |                        | "ba"         | B to A                 |
| ti         | transfer_in            | TransferInfo |                        |
| to         | transfer_out           | TransferInfo |                        |
| osp        | old_sqrt_price         | u128         | value before the event |
| nsp        | new_sqrt_price         | u128         | value after the event  |
| octi       | old_current_tick_index | i32          | value before the event |
| ncti       | new_current_tick_index | i32          | value after the event  |
| odp        | old_decimal_price      | DecimalPrice | value before the event |
| ndp        | new_decimal_price      | DecimalPrice | value after the event  |
| fr         | fee_rate               | u16          |                        |
| pfr        | protocol_fee_rate      | u16          |                        |

## Type Definition
### PubkeyString
PublicKey address in Base58 string

### Decimals
u8

### TokenProgram
| Type | Notes              |
| ---- | ------------------ |
| "t"  | Token program      |
| "t2" | Token-2022 program |

### DecimalPrice
decimal adjusted price with 10 significant digits in scientific notation

### TransferInfo
| JSON Field | Name             | Type         | Notes                      |
| ---------- | ---------------- | ------------ | -------------------------- |
| m          | mint             | PubkeyString |                            |
| a          | amount           | u64          |                            |
| d          | decimals         | u8           |                            |
| tfb        | transfer_fee_bps | u16          | TransferFee extension only |
| tfm        | transfer_fee_max | u64          | TransferFee extension only |

## u64 and u128
Fields with type u64 or u128 will be expressed as string to avoid number type issue in Javascript.

# Account Delta
Account change is categorized in the following 3 types:
- Initialized: a new account has been initialized in the slot
- Updated: an existing account has been updated in the slot
- Closed: an existing account has been closed in the slot

## Delta
### Initialized (`I`)
To initialize an account, you need to initialize account as `[0; length]` first, then apply each segment data to the zero initialized account.

| JSON Field | Name         | Type         | Notes           |
| ---------- | ------------ | ------------ | --------------- |
| p          | pubkey       | PubkeyString | account address |
| t          | account_type | AccountType  |                 |
| d          | delta        | "I"          | Initialized     |
| l          | length       | u16          | account length  |
| s          | segments     | Segment[]    |                 |

### Updated (`U`)
To update an account data, you need to apply each segment data to the account.

| JSON Field | Name         | Type         | Notes           |
| ---------- | ------------ | ------------ | --------------- |
| p          | pubkey       | PubkeyString | account address |
| t          | account_type | AccountType  |                 |
| d          | delta        | "U"          | Updated         |
| s          | segments     | Segment[]    |                 |

### Closed (`C`)
You need to close the account.

| JSON Field | Name         | Type         | Notes           |
| ---------- | ------------ | ------------ | --------------- |
| p          | pubkey       | PubkeyString | account address |
| t          | account_type | AccountType  |                 |
| d          | delta        | "C"          | Closed          |

## Type Definition
### AccountType
| Account           | AccountType |
| ----------------- | ----------- |
| Config            | "C"         |
| ConfigExtension   | "CE"        |
| FeeTier           | "FT"        |
| Whirlpool         | "W"         |
| TickArray         | "TA"        |
| Position          | "P"         |
| PositionBundle    | "PB"        |
| TokenBadgeDeleted | "TB"        |

### Segment
| JSON Field | Name   | Type   | Notes           |
| ---------- | ------ | ------ | --------------- |
| o          | offset | u16    |                 |
| d          | data   | String | Base64 encoding |

## Notes About Delta
The deltas are derived from state changes on a per-slot basis.
Therefore, the behavior is as follows:

- Deltas for accounts updated multiple times within a single slot are consolidated into a single record.
- If the result of updates within a slot matches the final state of the previous slot, no deltas are output.
- If an account is initialized and then closed within the same slot, no deltas are output.
- If an account is closed and then reinitialized within the same slot, the delta is recorded as Updated.

The purpose of account delta is to maintain snapshots of account states at the slot level, not to track changes for each instruction or transaction.
The responsibility of tracking instructions or transactions lies with events.

# Stream Format
## Control Message
### opened
If the stream is successfully started, this will be the first push.
```
{ "ctrl": "opened" }
```

### nodata
This is pushed every 5 seconds if there is no event to be pushed.
It works as a heartbeat to check the availability of the stream.
```
{ "ctrl": "nodata" }
```

### closed
This will be pushed when the stream is closed by the server.
```
{ "ctrl": "closed", reason: "why stream was closed" }
```

## Data Message
- Data is sent out for each slot.
- Data will be sent out even if the slot has no Whirlpool events or Whirlpool account delta.
- If both event and account are sent out, the data for event is always sent first.
- Field names are abbreviated to reduce data size.

### Event Data Layout
- The data consists of `slot(s)`, `block height(h)`, `block time(t)`, and an array of successful transactions(`x`) that generated Whirlpool events.
- A transaction consists of `signature(s)`, `payer(p)`, and Whirlpool events(`e`).

```
{
  "ctrl": "data.event",
  "data": {
    "s": slot(u64),
    "h": block_height(u64),
    "t": block_time(i64),
    "x": [
      {
        "s": signature(String),
        "p": payer(PubkeyString),
        "e": [
          {
            "n": event_name,
            "p": event_payload
          },
          ...
        ]
      },
      {
        "s": signature(String),
        "p": payer(PubkeyString),
        "e": [
          {
            "n": event_name,
            "p": event_payload
          },
          {
            "n": event_name,
            "p": event_payload
          },
          ...
        ]
      },
      ...
    ]
  }
}
```

### Event Data Example
```
{
  "ctrl": "data.event",
  "data": {
    "s": 291135497,
    "h": 270037006,
    "t": 1726923014,
    "x": [
      {
        "s": "2utjgteoUEHwTye5c9FcSFrYtSXPuqqd8rUyjfaDzC4orGnmpKGaz6YuQPC5UL8xy6EM1E9aBuo9ggspLsBQqmjZ",
        "p": "EV5yEvwrvxtCEKCYPeQfZjFbSus8am6imeuSmuiy8nwk",
        "e": [
          {
            "n": "T",
            "p": {
              "o": "s",
              "w": "u41JpbKrcPuzJwRTPU6gDysoFqKPpEZe9YmHdszQTJ1",
              "ta": "EV5yEvwrvxtCEKCYPeQfZjFbSus8am6imeuSmuiy8nwk",
              "tm": "ei",
              "td": "ba",
              "ti": {
                "m": "AT79ReYU9XtHUTF5vM6Q4oa9K8w7918Fp5SU7G1MDMQY",
                "a": "483000000000",
                "d": 9
              },
              "to": {
                "m": "So11111111111111111111111111111111111111112",
                "a": "40676329",
                "d": 9
              },
              "osp": "2007772056178763944824",
              "nsp": "2012273396536558242462",
              "octi": 93802,
              "ncti": 93847,
              "odp": "1.184648110e4",
              "ndp": "1.189965927e4",
              "fr": 100,
              "pfr": 1
            }
          }
        ]
      },
      {
        "s": "2ds3GGMsqRjLNosNYsV6UG7sagNtrwKEZfkeY8p7Q6KJCT762sTXmtgBnxHizHsJ2r5XUEJjej9LnAGGawDfaDZD",
        "p": "kitrFdc3LuHJPuEstuyP7a9XsYo6DCeEBjeW6AasXxV",
        "e": [
          {
            "n": "T",
            "p": {
              "o": "sv2",
              "w": "9Vh6fqJjDkqSTZ8bDXseVxGb2yQEMkEhhtte2anQCHSf",
              "ta": "kitrFdc3LuHJPuEstuyP7a9XsYo6DCeEBjeW6AasXxV",
              "tm": "ei",
              "td": "ab",
              "ti": {
                "m": "So11111111111111111111111111111111111111112",
                "a": "77179200",
                "d": 9
              },
              "to": {
                "m": "2b1kV6DkPAnxd5ixfnxCpjxmKwqjjaYmCZfHsFu24GXo",
                "a": "11330995",
                "d": 6,
                "tfb": 0,
                "tfm": "0"
              },
              "osp": "7071226517991932064",
              "nsp": "7067833458265632609",
              "octi": -19179,
              "ncti": -19188,
              "odp": "1.469433898e2",
              "ndp": "1.468024049e2",
              "fr": 400,
              "pfr": 1
            }
          },
          {
            "n": "T",
            "p": {
              "o": "sv2",
              "w": "JDELdNKDVFQqAodu79QcgnYbZJA8H3HXKxLQ7MzxByZb",
              "ta": "kitrFdc3LuHJPuEstuyP7a9XsYo6DCeEBjeW6AasXxV",
              "tm": "ei",
              "td": "ba",
              "ti": {
                "m": "2b1kV6DkPAnxd5ixfnxCpjxmKwqjjaYmCZfHsFu24GXo",
                "a": "11330995",
                "d": 6,
                "tfb": 0,
                "tfm": "0"
              },
              "to": {
                "m": "KMNo3nJsBXfcpJTVhZcXLW7RmTwTt4GVFE7suUBo9sS",
                "a": "146352254",
                "d": 6
              },
              "osp": "5132516088839478017",
              "nsp": "5132563004262961042",
              "octi": -25588,
              "ncti": -25587,
              "odp": "7.741430048e-2",
              "ndp": "7.741571575e-2",
              "fr": 100,
              "pfr": 1
            }
          }
        ]
      }
    ]
  }
}
```

### Account Data Layout
- The data consists of `slot(s)`, `block height(h)`, `block time(t)`, and an array of updated accounts(`a`) whose data was updated in the slot.
- An account delta data consists of `pubkey(p)`, `account type(t)`, `delta(d)` and some additional info for the delta.

```
{
  "ctrl": "data.account",
  "data": {
    "s": slot(u64),
    "h": block_height(u64),
    "t": block_time(i64),
    "a": [
      {
        "p": account_address(PubkeyString)
        "t": account_type,
        "d": "U" (updated),
        "s": [
          { "o": offset,"d": delta(Base64) },
          { "o": offset,"d": delta(Base64) },
          ...
        ]
      },
      {
        "p": account_address(PubkeyString)
        "t": account_type,
        "d": "I" (initialized),
        "l": account_data_length,
        "s":[
          { "o": offset,"d": delta(Base64) },
          { "o": offset,"d": delta(Base64) },
          ...
        ]
      },
      {
        "p": account_address(PubkeyString)
        "t": account_type,
        "d": "C" (closed)
      },
      ...
    ]
  }
}
```

### Account Data Example
```
{
  "ctrl": "data.account",
  "data": {
    "s": 310236169,
    "h": 288569517,
    "t": 1735343857,
    "a": [
      {
        "p": "5xBsx79W4mbfqf4r81wJztwDmsH3FK7enbBYaYjppEQ6",
        "t": "TA",
        "d": "U",
        "s": [
          {"o": 1482, "d": "Ifhk7fn//////////////98HmxIG"}
        ]
      },
      {
        "p": "5xMH1GGP1jyrbMoSsxRNQXGuvjnyte2eYHJy3hVGd6sz",
        "t": "TA",
        "d": "U",
        "s": [
          {"o": 2047, "d": "DZOoTBMAAAAAAAAAAAAAAEUdwcMW"},
          {"o": 6341, "d": "JUZbMAgAAAAAAAAAAAAAAMfAsP0W"}
        ]
      },
      {
        "p": "4Eq688gAJRJiRzhFR4tKpVyRhknDvC8UkqsLmfsEzFFX",
        "t": "W",
        "d": "U",
        "s": [
          {"o": 261, "d": "8Q=="}
        ]
      },
      ...
      {
        "p": "CDhRbmFymzdNXAnkLDgLHwCwZewqMVBbkLQSMSyJhpu2",
        "t": "P",
        "d": "I",
        "l": 216,
        "s": [
          {"o": 0, "d": "qryP5HpA99CyNpDX0HWNHV2LiVDOx6m018ea6P+1xroNvWKhmDeTW2OOeGAMXNoW139U0T4A1hdjw/acLuJuN4g+pscc+WXxE4dd6oYAAAAAAAAAAAAAANi8//9Yv///aJkovOtGTwAAAAAAAAAAAAAAAAAAAAAAwIBct8gpCw=="}
        ]
      },
      ...
      {
        "p": "AresKgKg6TsuiFBcTeVxYXQoJC6E5d11JN8rw4YWQ5Q9",
        "t": "P",
        "d": "C"
      },
      ...
    ]
  }
}
```
