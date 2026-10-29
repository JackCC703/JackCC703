<p align="center">
  <img width="760" src="./pic/logo-ascii-scan.svg" alt="JACK CC Scanline Logo" />
</p>

# Hi, I'm Jack CC

**Ethereum Protocol Fellowship · Cohort 7 | Erigon contributor | M.Eng. student**

I build and test Ethereum client software in Go. During EPF7, I contributed progressive SSZ primitives to Erigon and built an experimental EIP-7807 payload adapter with field proofs. My current focus is SSZ correctness and conformance testing.

**[EPF7 project & technical walkthrough](projects/epf7-erigon-ssz.md)** · **[Run the proof demo](https://github.com/JackCC703/erigon/tree/eip7807-demo/dev/eip7807demo)** · **[Email](mailto:chencan0703@gmail.com)**

## Selected upstream contributions

### Erigon · Go, SSZ, Merkle trees

| Contribution | Result | Status |
| --- | --- | --- |
| [#25265](https://github.com/erigontech/erigon/pull/25265) · SSZ Merkle tree depth | Fixed incorrect roots and element-proof failures for non-power-of-two limits; added regression tests | Merged |
| [#22446](https://github.com/erigontech/erigon/pull/22446) · Progressive Merkleization | Implemented the EIP-7916 tree primitive with reference-vector tests | Merged |
| [#22689](https://github.com/erigontech/erigon/pull/22689) · Progressive list root | Combined progressive Merkleization with the logical element-count mix-in | Merged |
| [#22783](https://github.com/erigontech/erigon/pull/22783) · Progressive byte list root | Added byte packing and root computation with boundary-case reference vectors | Merged |

Current work: [#25361](https://github.com/erigontech/erigon/pull/25361) adds rejection of over-full lists and handles empty flat-leaf input. **Open; not merged.**

### Ethereum execution-specs · Python, protocol tooling

- [#2738](https://github.com/ethereum/execution-specs/pull/2738) — avoided unnecessary fork-template clones in `ForkCache`, with regression tests. **Merged.**
- [#2778](https://github.com/ethereum/execution-specs/pull/2778) — centralized fork lookup and module loading through the `Hardfork` abstraction. **Merged.**
- [#3053](https://github.com/ethereum/execution-specs/pull/3053) — registered the JSON fixture fork marker to remove pytest warnings. **Merged.**

## EPF7 project: SSZ execution blocks in Erigon

Built a pipeline from Erigon block data to an 18-field EIP-7807 payload, its SSZ root, and proofs for individual fields.

- **Adapter:** maps block data, receipts, and additional inputs to the proposed payload layout. [#23854](https://github.com/erigontech/erigon/pull/23854) was **closed without merging**; the implementation remains experimental on my fork.
- **Proof demo:** reports all 18 generalized indices and generates field proofs. A Python implementation using `remerkleable` independently rebuilds the fixture and checks the payload root, positions, and reported proof branches.
- **Reproduction:** a local script runs the Go demo tests and Python cross-check. No running Ethereum node is required; initial setup downloads dependencies.

The cross-check covers a deterministic fixture. Broader historical-block coverage remains future work.

**[Read the project walkthrough →](projects/epf7-erigon-ssz.md)**

## What I'm working toward

Continuing Erigon contributions while developing deeper Go, testing, and distributed-systems skills. I am interested in internships and contributor opportunities in Ethereum clients, blockchain infrastructure, protocol testing, and Go backend development.

**Based in China (UTC+8).** Contact: [chencan0703@gmail.com](mailto:chencan0703@gmail.com).

## Other work

- [upgrading-ethereum-zh](https://github.com/JackCC703/upgrading-ethereum-zh) — independent Chinese translation of *Upgrading Ethereum*.
- [github-ai-helper](https://github.com/JackCC703/github-ai-helper) — tooling for collecting GitHub PR context and generating PR descriptions.

Primary tools: **Go · Python · Git · Linux · Docker**. Also working with TypeScript and learning Rust.

<sub>Contribution statuses checked on 2026-10-29. Follow the linked PRs for updates.</sub>
