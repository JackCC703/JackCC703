# EPF7: SSZ execution blocks and conformance work in Erigon

[← Back to profile](../README.md) · [Demo source](https://github.com/JackCC703/erigon/tree/eip7807-demo/dev/eip7807demo) · [Project proposal](https://github.com/eth-protocol-fellows/cohort-seven/blob/main/projects/erigon-ssz-execution-blocks-eip-7807.md)

**Author:** Chen Can (Jack CC) · **Program:** Ethereum Protocol Fellowship, Cohort 7  
**Mentors:** RazorClient and Etan Kissling · **Languages:** Go and Python

I worked on the Erigon-side foundations for EIP-7807: mapping execution-block data into SSZ, computing progressive Merkle roots, and generating proofs for individual payload fields. I then continued with SSZ conformance testing and fixes in Caplin's SSZ code.

## Outcomes

| Work | Outcome | Evidence |
| --- | --- | --- |
| Progressive Merkleization | Merged into Erigon | [#22446](https://github.com/erigontech/erigon/pull/22446) |
| Progressive list root | Merged into Erigon | [#22689](https://github.com/erigontech/erigon/pull/22689) |
| Progressive byte list root | Merged into Erigon | [#22783](https://github.com/erigontech/erigon/pull/22783) |
| SSZ depth calculation for non-power-of-two limits | Merged into Erigon | [#25265](https://github.com/erigontech/erigon/pull/25265) |
| EIP-7807 payload adapter and root | Closed without merging; experimental implementation on my fork | [#23854](https://github.com/erigontech/erigon/pull/23854) |
| Generalized indices, field proofs, and independent cross-check | Published on my fork's `eip7807-demo` branch | [Source and commands](https://github.com/JackCC703/erigon/tree/eip7807-demo/dev/eip7807demo) |
| List-limit checks and empty flat-leaf hashing | Open PR; not merged | [#25361](https://github.com/erigontech/erigon/pull/25361) |

The three progressive primitives were merged during the EIP-7807 project; the depth fix followed during conformance work. The adapter and demo are separate from Erigon's upstream execution path.

## 1. Mapping existing block data to an SSZ payload

The adapter takes `BlockWithReceipts` and explicit configuration, constructs the proposed 18-field `ExecutionPayload`, and computes its SSZ root.

```text
Erigon block + receipts + requests + access list + explicit configuration
    → ExecutionPayloadFromBlock
    → 18-field EIP-7807 payload
    → field roots → progressive container root
    → generalized indices and field proofs
```

The mapping required understanding each field's source and encoding:

- **Receipts:** the existing header's Merkle-Patricia trie root cannot be copied into the SSZ receipts-root field. The adapter computes the proposed root from receipt envelopes.
- **Blob base fee:** derived from excess blob gas and the supplied configuration.
- **Regular excess gas:** has no matching legacy-header field, so the caller supplies it explicitly.
- **Execution requests:** must follow the request-container shape linked by the pinned EIP revision. Independent verification exposed a mismatch with a Gloas-shaped container, which I corrected.

The prototype computes a parallel SSZ commitment without changing Erigon's canonical block hash or import process. The adapter PR was closed without merging after EIP-7807 was declined for inclusion; it remains available for research on the fork.

## 2. Field proofs and independent verification

The demo derives generalized indices for all 18 top-level fields and can generate a Merkle proof for any of them. By default it reports proofs for `parent_hash`, `transactions`, `receipts_root`, and `withdrawals`.

For example, `parent_hash` has generalized index **4** and a branch containing **2 sibling hashes**. Folding the field root with those hashes reconstructs the payload root. A verifier still needs a trusted payload root.

Progressive SSZ preserves existing field positions when fields are appended. The payload root and branch hashes can change, so a proof must correspond to the particular root being verified.

| Check | Method |
| --- | --- |
| Payload root | Python rebuilds the deterministic fixture from raw values using `remerkleable` and compares its root with Go's |
| All 18 field positions | Compare the reported generalized indices with `remerkleable` |
| Reported proof branches | Fold Go's sibling hashes using Python-computed field roots and indices, then compare with Python's payload root |
| Index stability and invalid proofs | Go tests cover appended fields, tampered branches, and truncated branches |

This is an independent cross-check of one deterministic fixture. It is not a direct comparison of every Go field root, a historical-block corpus, or a complete SSZ conformance suite.

### Reproduce the demo

Requires Git, Go compatible with the branch's `go.mod`, and Python 3.11+ with venv/pip support. Initial setup needs network access to download Go/Python dependencies and the pinned `remerkleable` source. No running node or chain database is needed.

```sh
git clone --branch eip7807-demo https://github.com/JackCC703/erigon.git erigon-demo
cd erigon-demo
./dev/eip7807demo/verify.sh
```

The script runs the Go demo tests, generates a JSON report, and runs the Python cross-check. Successful verification ends with `OK all cross-checks passed`.

- [Go schema and proof code](https://github.com/JackCC703/erigon/blob/eip7807-demo/dev/eip7807demo/internal/eip7807/schema.go)
- [Python verifier](https://github.com/JackCC703/erigon/blob/eip7807-demo/dev/eip7807demo/verify/verify.py)
- [Verification script](https://github.com/JackCC703/erigon/blob/eip7807-demo/dev/eip7807demo/verify.sh)
- [Payload adapter](https://github.com/JackCC703/erigon/blob/eip7807-demo/execution/types/sszblocks/payload.go)

The fixture targets EIP-7807 blob `3f2dc991ac65153842fd9b894c04ba590d58d3e6`, its linked consensus-specs revision `4f1df00ff567ea1855fb7f37ec23bd324a96703b`, and `remerkleable@2f0baeef0082d4278acaef7d822deb7009d7db7e`. It does not track future spec changes automatically.

## 3. Continuing with SSZ conformance

Running the `ethereum/ssz-specs` vectors against Caplin's SSZ implementation exposed incorrect roots for non-power-of-two chunk limits. For a `ByteList[96]`, the limit is three chunks, which requires a four-leaf tree after padding. The old depth calculation rounded down and omitted the third chunk from the result.

In [#25265](https://github.com/erigontech/erigon/pull/25265), I changed the depth calculation to round up, updated its callers, and added regression tests for roots and element proofs. The PR records 26 failures among 380 vectors in the `ssz` format before the fix and all 380 passing afterward. The external vector runner was a local validation tool and was not included in the merged PR.

Further reports and work:

| Report | Finding | Status checked 2026-10-29 |
| --- | --- | --- |
| [#25277](https://github.com/erigontech/erigon/issues/25277) | Over-full lists produce a root instead of an error | Open; partly addressed by open PR #25361 |
| [#25278](https://github.com/erigontech/erigon/issues/25278) | Reused `RawUint64List` hashes stale buffer bytes after shrinking | Open; my proposed fix [#25346](https://github.com/erigontech/erigon/pull/25346) was closed without merging |
| [#25279](https://github.com/erigontech/erigon/issues/25279) | Inconsistent `IndexedPayloadAttestation` list limits | Open |
| [#25280](https://github.com/erigontech/erigon/issues/25280) | Empty flat-leaf input causes a panic | Open; addressed by open PR #25361 |

Next steps are to work with maintainers on the remaining cases and broaden test coverage. Integration of the external vector runner into upstream CI remains future work.

## What this work demonstrates

Reading protocol specifications, tracing data through a large Go codebase, choosing explicit adapter boundaries, finding correctness bugs through external vectors, writing regression tests, and responding to upstream review.

For collaboration or development opportunities: [chencan0703@gmail.com](mailto:chencan0703@gmail.com).

<sub>PR and issue statuses verified on 2026-10-29. Linked discussions are the source for subsequent changes.</sub>
