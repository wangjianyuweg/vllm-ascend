# vLLM Ascend CPU audit publication evidence

This branch keeps reproduction evidence separate from upstream source PRs.

The 2026-11-04 audit used vllm-ascend main 0f85ed44bbbbdc0f0447f83ce62a82a8ad55afb3 and its verified vLLM pin 24420b7a341af37a9bc3186e6fc6062512e0aa58. Supported-release compatibility was also checked where affected.

The final publication scope is in [final-adjudication.json](final-adjudication.json): ten eligible findings, ten existing or substantially overlapping reports/limitations, and two deferred findings requiring stronger supported-contract evidence. The final adjudication supersedes any earlier candidate status in the historical research records.

Search results are bounded by queries and indexing. They establish that no matching public report was found for the submitted mechanism in those searches; they do not prove absolute novelty.

All execution here is on CPU with mocks or synthetic metadata. No Ascend NPU hardware is available. Screenshots render actual captured unittest output and are accompanied by raw logs, scripts, source patches, and hashes.
