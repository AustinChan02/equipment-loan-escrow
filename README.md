# Smart Contract Escrow for Equipment Loan Deposit Disputes

CSIT5760 Blockchain and Smart Contracts individual project. This prototype is designed for student clubs that lend equipment and need a predictable way to settle refundable deposits.

Contract implementation, testing, and Sepolia deployment are planned.

## Loan workflow

One `EquipmentDepositEscrow` contract will manage multiple loans. Each `loanId` identifies the equipment, lender, borrower, referee, deposit, deadlines, and agreed fallback split. The contract tracks each loan’s status and withdrawal credits separately.

1. The lender creates a loan offer. The appointed referee accepts before the funding deadline.
2. The borrower checks the terms, inspects and accepts the equipment in person, then deposits the exact amount in test ETH.
3. The borrower reports the return before the reporting deadline. The lender can confirm the return for a full refund or dispute it before the challenge deadline.
4. For a dispute, the referee can set the borrower’s share before the ruling deadline. The lender receives a credit for the remainder.
5. After settlement, each credited party can withdraw its own credit for that `loanId`.

Equipment handover, inspection, and evidence review take place off-chain. The referee submits the resulting allocation to the contract.

## Timeout rules

An account can call `finalizeTimeout(loanId)` at or after the applicable cutoff. The contract checks the current loan state before applying the outcome.

| Loan state at cutoff | Cutoff | Outcome |
| --- | --- | --- |
| `Offered` | `fundBy` | Expire the unfunded offer with zero withdrawal credits. |
| `Funded` | `reportBy` | Credit the full deposit to the lender. |
| `ReturnReported` | `challengeBy` | Credit the full deposit to the borrower. |
| `Disputed` | `ruleBy` | Allocate credits using the fallback split agreed at creation. |

## Planned technology stack

| Technology | Use |
| --- | --- |
| Solidity `0.8.35` | Implement the escrow, role checks, state transitions, and deposit accounting. [Compiler release](https://www.soliditylang.org/blog/2026/04/29/solidity-0.8.35-release-announcement/) |
| OpenZeppelin Contracts `5.7.0` | Use `ReentrancyGuard` on withdrawal. [Versioned source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.7.0/contracts/utils/ReentrancyGuard.sol) |
| Remix IDE | Compile, run Solidity unit tests, review static analysis findings, and deploy the contract. [Remix documentation](https://remix-ide.readthedocs.io/en/latest/) |
| MetaMask | Sign Sepolia transactions from separate participant test accounts. [MetaMask testnet guide](https://support.metamask.io/configure/networks/how-to-view-testnets-in-metamask/) |
| Ethereum Sepolia | Demonstrate the contract on a public test network with test ETH. Chain ID `11155111`. [Sepolia network configuration](https://github.com/eth-clients/sepolia#meta-data-sepolia) |
| Etherscan | Verify the deployed source and inspect transaction receipts and event logs. [Verification with Remix](https://docs.etherscan.io/contract-verification/verify-with-remix) |

## Testing and deployment

The Remix test suite will cover successful settlement, invalid inputs and unauthorized calls, deadline boundaries, repeated operations, and failed or reentrant withdrawals. It will also check that withdrawing from one loan preserves the same wallet’s credit on another loan.

The Sepolia demonstration will record funding, timeout settlement, and withdrawal through Remix with MetaMask. The deployment record will include the contract address, compiler settings, transaction hashes, and verified Etherscan link.
