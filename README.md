# DAO Governance System

A decentralized governance system built with **Solidity, OpenZeppelin, and Foundry**. This project demonstrates token-based voting, proposal management, and time-delayed execution through a Governor contract and a Timelock controller.

## Overview

This project implements a basic Decentralized Autonomous Organization (DAO) governance workflow. Token holders can submit proposals, vote on them, and authorize on-chain operations through a governance-controlled execution process.

The `Box` contract serves as a simple example of a smart contract whose state can only be modified by its owner. Ownership is transferred to the Timelock, allowing governance proposals to control contract operations.

## Features

- **ERC-20 Governance Token:** Custom token named `MyToken` with the symbol `MTK`.
- **Voting Power:** Uses OpenZeppelin's `ERC20Votes` extension to track delegated voting power.
- **Permit Support:** Integrates `ERC20Permit` for signature-based token approvals.
- **Proposal Management:** Creates and tracks governance proposals through `MyGovernor`.
- **Token-Based Voting:** Supports votes in favor, against, or abstention.
- **Quorum Requirement:** Configures a quorum fraction of 4%.
- **Timelock Execution:** Enforces a configurable delay before approved proposals can be executed.
- **Automated Testing:** Uses Foundry to test governance execution and access control.

## Project Structure

```text
.
├── src/
│   ├── Box.sol
│   ├── GovToken.sol
│   ├── MyGovernor.sol
│   └── TimeLock.sol
├── test/
│   └── MyGovernorTest.t.sol
├── lib/
├── foundry.toml
└── README.md
```

*Adjust the file names and paths if your repository uses a different structure.*

## Smart Contracts

### 1. Box.sol

A simple storage contract used to demonstrate governance-controlled execution.

- Stores a single unsigned integer.
- Restricts updates using the `onlyOwner` modifier.
- Emits a `ValueChanged` event when the stored value changes.
- Provides `getValue()` to read the stored value.

In the test setup, ownership is transferred to the Timelock. Consequently, ordinary accounts cannot update the value directly.

### 2. GovToken.sol — MyToken

Implements the DAO governance token.

- **Token Name:** MyToken
- **Token Symbol:** MTK
- **ERC-20:** Standard token functionality.
- **ERC20Votes:** Delegated voting power.
- **ERC20Permit:** Signature-based approvals.
- **Minting:** Creates tokens through the `mint()` function.

> **Security Note:** The current `mint()` function is publicly accessible. Anyone can mint tokens to any address. Restrict minting permissions before using this contract in production.

Token holders must delegate their voting power before that power can be used for governance voting.

### 3. MyGovernor.sol

The main governance contract, built using OpenZeppelin Governor extensions.

Responsibilities include:

- Creating proposals containing target addresses, ETH values, and encoded function calls.
- Managing proposal states.
- Counting votes.
- Enforcing voting delay and voting period settings.
- Applying a quorum fraction of 4%.
- Queueing and executing approved proposals through the Timelock.

#### Governance Parameters

| Parameter | Value |
|---|---:|
| Voting Delay | 7,200 blocks |
| Voting Period | 50,400 blocks |
| Proposal Threshold | 0 voting units |
| Quorum Fraction | 4% |

**Note:** Voting delay and voting period are measured in blocks, not seconds. Their actual duration depends on the target blockchain.

### 4. TimeLock.sol

Extends OpenZeppelin's `TimelockController`.

Constructor parameters:

- `minDelay`: Minimum delay before a scheduled operation can execute, measured in seconds.
- `proposers`: Addresses authorized to schedule operations.
- `executors`: Addresses authorized to execute ready operations.

The deployer receives the initial Timelock administrator role. Role permissions should be reviewed carefully before deployment.

## Governance Workflow

A successful proposal follows this lifecycle:

1. **Propose:** A participant submits a governance proposal.
2. **Pending:** The proposal waits for the configured voting delay.
3. **Active:** Eligible token holders cast their votes.
4. **Succeeded:** Voting ends and the proposal satisfies the success conditions.
5. **Queued:** The proposal is scheduled through the Timelock.
6. **Waiting:** The configured minimum delay elapses.
7. **Executed:** The operation runs through the Timelock.

Depending on its outcome, a proposal may instead be defeated, canceled, or expire.

## Installation and Setup

### Prerequisites

- [Foundry](https://getfoundry.sh/)
- Git
- A compatible Solidity compiler
- OpenZeppelin Contracts

### Clone the Repository

```bash
git clone https://github.com/ZEINSALLOM/Foundry-DAO.git
cd Foundry-DAO
```

### Install Dependencies

If the dependencies are not already installed:

```bash
forge install OpenZeppelin/openzeppelin-contracts
forge install foundry-rs/forge-std
```

Ensure that the remappings in `foundry.toml` correctly resolve `@openzeppelin/contracts/` and `forge-std/`.

## Build

Compile the smart contracts:

```bash
forge build
```

## Run Tests

For detailed execution traces:

```bash
forge test -vvvv
```

### Included Tests

The provided `MyGovernorTest` demonstrates two scenarios:

- **Unauthorized Update Rejection:** Verifies that a direct call to `Box.store()` fails after ownership is transferred to the Timelock.
- **Successful Governance Execution:** Creates a proposal, advances the voting lifecycle, casts a vote, queues the proposal, waits for the Timelock delay, and executes the operation.

The successful execution test verifies that the stored value in `Box` matches the proposed value of `777`.

## Configuration

The current test setup uses these parameters:

| Parameter | Value |
|---|---:|
| Timelock Minimum Delay | 3,600 seconds (1 hour) |
| Voting Delay | 7,200 blocks |
| Voting Period | 50,400 blocks |
| Quorum Fraction | 4% |
| Test Voter Allocation | 100 MTK |

The test voter delegates voting power to themselves before voting. Foundry's `vm.warp()` and `vm.roll()` cheatcodes advance the timestamp and block number to simulate the governance lifecycle.

## Security Considerations

This project is intended for educational and development purposes and has not been established as production-audited.

Before deploying to a live network:

- Restrict token minting to an authorized role or remove unrestricted minting.
- Review token delegation and voting-power assumptions.
- Verify Timelock proposer, executor, and administrator roles.
- Test proposal cancellation, execution, and access-control behavior.
- Review quorum and voting settings for the target blockchain.
- Pin and verify dependency versions.
- Run additional tests and obtain an independent security audit.

## Future Improvements

Potential enhancements include:

- Access-controlled token minting.
- Deployment and configuration scripts.
- Tests for defeated, canceled, and expired proposals.
- Tests for quorum failures and unauthorized operations.
- Testnet deployment with verified contract sources.
- A frontend for proposal creation, voting, and execution tracking.
- Continuous integration for automated builds and tests.

## License

The provided Solidity contracts declare the **MIT License**. Include a repository-level `LICENSE` file containing the appropriate MIT license text.

---

**Disclaimer:** This project is intended for educational and development purposes. Review and audit all contracts before using them to manage real assets or production governance.
