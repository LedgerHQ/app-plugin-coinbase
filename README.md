# Badges

[![Code style check](https://github.com/LedgerHQ/app-plugin-kiln/actions/workflows/lint-workflow.yml/badge.svg)](https://github.com/LedgerHQ/app-plugin-kiln/actions/workflows/lint-workflow.yml)
[![Compilation & tests](https://github.com/LedgerHQ/app-plugin-kiln/actions/workflows/ci-workflow.yml/badge.svg)](https://github.com/LedgerHQ/app-plugin-kiln/actions/workflows/ci-workflow.yml)

# app-plugin-coinbase

This plug-in is compatible with Nano S / X devices, it enables to interact in a secure way with the Kiln v2 contracts.

## Prerequisite

Clone the plugin to a new folder.

```shell
git clone https://github.com/coinbase/ledger-plugin-coinbase-staking.git
```

Then in the same folder clone two more repositories, which is the plugin-tools and app-ethereum.

```shell
git clone https://github.com/LedgerHQ/plugin-tools.git                          # plugin-tools
git clone --recurse-submodules https://github.com/LedgerHQ/app-ethereum.git     # app-ethereum
```

## Documentation

### Plugin codebase

This plugin follows the structure of a Ledger Ethereum app plugin described [here](https://developers.ledger.com/docs/device-app/develop/code/plugin).

It receives data from the Ethereum app, processes it and displays (ie "clearsign") on the device screen.

All the plugin code is in the `src` folder.

The plugin codebase follows the 6 steps of the plugin flow:

1. **initialize** (in `src/handle_init_contract.c`): plugin receives the selector to manage, and init the correct parser initial state for the function (`context->next_param`).
2. **provide parameter** (in `src/provide_parameter/handle_provide_parameter.c`): selected parser receives parameters 32 bytes by 32 bytes, and parse them to the correct state.
3. **finalize** (in `src/handle_finalize.c`): plugin returns the number of screens to display
4. **provide_token** not used here
5. **query_contract_id** (in `src/handle_query_contract_id.c`): plugin returns the contract name as the first displayed screen on the device
6. **query_contract_ui** (in `src/query_contract_ui/handle_query_contract_ui.c`): plugin returns the screens to display on the device, each product has its own file.

Other important files:

- `src/contracts.c` holds all the constant representing contract functions, parameters or known contracts mappings.
- `src/kiln_plugin.h` is where we define all the parser states and the plugin context structure.

### Plugin context

The plugin context is defined in `src/kiln_plugin.h` and holds the state of the plugin. It is passed to all the plugin functions.

It is defined using a union of all the possible parser parameters of the plugin, and a `next_param` field to know which parser state to use next.

The plugin context is memset to 0 everytime init_contract is called (beginning of a clearsigning flow), so we make sure that the plugin context union is in a clean state before using it in the parsers.

### Provide parameter parsers

The provide parameter parsers are defined in `src/provide_parameter/` and are called by the `handle_provide_parameter` function.

Each parser is a state machine that receives 32 bytes of data at a time (abi format), and updates the plugin context state accordingly.

### Build

Go to the plugin-tools folder and run the "./start" script.

```shell
cd plugin-tools  # go to plugin folder
./start.sh       # run the script start.sh
```

The script will build a docker image and attach a console.
When the docker image is running go to the "app-plugin-kiln" folder and build the ".elf" files.

```shell
cd app-plugin-kiln/tests        # go to the tests folder in app-plugin-kiln
./build_local_test_elfs.sh      # run the script build_local_test_elfs.sh
```

### Tests

The tests are in the `tests/src` folder. They are written in javascript and use the `@ledgerhq/hw-app-eth` library to simulate the device.

The tests are run using the `yarn test` command.

The output of the tests is a series of screenshots stored in the `tests/snapshots-tmp` folder that are compared to the expected screenshots in the `tests/snapshots` folder. The latter are stored in the git repository to make sure we expect the correct outputs.

To test the plugin go to the tests folder from the "app-plugin-kiln/tests" and run the script "test"

```shell
cd app-plugin-kiln/tests        # go to the tests folder in app-plugin-kiln
yarn                            # install dependencies
yarn test                       # run the script test
```

### Continuous Integration

The flow processed in [GitHub Actions](https://github.com/features/actions) is the following:

- Code formatting with [clang-format](http://clang.llvm.org/docs/ClangFormat.html)
- Compilation of the application for Ledger Nano S in [ledger-app-builder](https://github.com/LedgerHQ/ledger-app-builder)

### Supported Smart Contracts

Smart contracts covered by this plugin are:
ks
| Network | Name | Address |
| ------- | ----------------------------- | -------------------------------------------- |
| Mainnet | Coinbase On-Chain v2 Native20 | `0x2e3956e1ee8b44ab826556770f69e3b9ca04a2a7` |
| Mainnet | Coinbase On-Chain v2 ExitQueue | `0x86358f7b33b599c484e0335b8ee4f7f7f92d8b60` |

## Clearsigning

### KILN ON-CHAIN v2 - Stake

Function: `stake()`
Clearsigned:

- tx.value

![](/tests/snapshots/nanos_stakev2/00000.png) ![](/tests/snapshots/nanos_stakev2/00001.png) ![](/tests/snapshots/nanos_stakev2/00002.png) ![](/tests/snapshots/nanos_stakev2/00003.png) ![](/tests/snapshots/nanos_stakev2/00004.png) ![](/tests/snapshots/nanos_stakev2/00005.png)

![](/tests/snapshots/nanox_stakev2/00000.png) ![](/tests/snapshots/nanox_stakev2/00001.png) ![](/tests/snapshots/nanox_stakev2/00002.png) ![](/tests/snapshots/nanox_stakev2/00003.png) ![](/tests/snapshots/nanox_stakev2/00004.png) ![](/tests/snapshots/nanox_stakev2/00005.png)

### KILN ON-CHAIN v2 - RequestExit

Function: `requestExit(uint256 shares)`
Clearsigned:

- none as shares is not 1:1 with ETH value, it would confuse users

![](/tests/snapshots/nanos_requestExitv2/00000.png) ![](/tests/snapshots/nanos_requestExitv2/00001.png) ![](/tests/snapshots/nanos_requestExitv2/00002.png) ![](/tests/snapshots/nanos_requestExitv2/00003.png) ![](/tests/snapshots/nanos_requestExitv2/00004.png) ![](/tests/snapshots/nanos_requestExitv2/00005.png)

![](/tests/snapshots/nanox_requestExitv2/00000.png) ![](/tests/snapshots/nanox_requestExitv2/00001.png) ![](/tests/snapshots/nanox_requestExitv2/00002.png) ![](/tests/snapshots/nanox_requestExitv2/00003.png) ![](/tests/snapshots/nanox_requestExitv2/00004.png) ![](/tests/snapshots/nanox_requestExitv2/00005.png)

### KILN ON-CHAIN v2 - MultiClaim

Function: `multiClaim(address[] exitQueues, uint256[][] ticketIds, uint32[][] casksIds)`
Clearsigned:

- none as shares is not 1:1 with ETH value, it would confuse users

![](/tests/snapshots/nanos_multiClaimv2/00000.png) ![](/tests/snapshots/nanos_multiClaimv2/00001.png) ![](/tests/snapshots/nanos_multiClaimv2/00002.png) ![](/tests/snapshots/nanos_multiClaimv2/00003.png) ![](/tests/snapshots/nanos_multiClaimv2/00004.png) ![](/tests/snapshots/nanos_multiClaimv2/00005.png)

![](/tests/snapshots/nanox_multiClaimv2/00000.png) ![](/tests/snapshots/nanox_multiClaimv2/00001.png) ![](/tests/snapshots/nanox_multiClaimv2/00002.png) ![](/tests/snapshots/nanox_multiClaimv2/00003.png) ![](/tests/snapshots/nanox_multiClaimv2/00004.png) ![](/tests/snapshots/nanox_multiClaimv2/00005.png)

### KILN ON-CHAIN v2 - Claim

Function: `claim(uint256[] ticketIds, uint32[] caskIds, uint16 maxClaimDepth)`
Clearsigned:

- none as shares is not 1:1 with ETH value, it would confuse users

![](/tests/snapshots/nanos_claimv2/00000.png) ![](/tests/snapshots/nanos_claimv2/00001.png) ![](/tests/snapshots/nanos_claimv2/00002.png) ![](/tests/snapshots/nanos_claimv2/00003.png) ![](/tests/snapshots/nanos_claimv2/00004.png) ![](/tests/snapshots/nanos_claimv2/00005.png)

![](/tests/snapshots/nanox_claimv2/00000.png) ![](/tests/snapshots/nanox_claimv2/00001.png) ![](/tests/snapshots/nanox_claimv2/00002.png) ![](/tests/snapshots/nanox_claimv2/00003.png) ![](/tests/snapshots/nanox_claimv2/00004.png) ![](/tests/snapshots/nanox_claimv2/00005.png)
