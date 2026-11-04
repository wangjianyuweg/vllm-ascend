# PR 20985 review follow-up (2026-11-05)

Source PR: <https://github.com/vllm-project/vllm-ascend/pull/20985>

Production baseline: `0f85ed44bbbbdc0f0447f83ce62a82a8ad55afb3`.
Validated PR commit: `09ea84e6f84d3e1d481f1bf6195af615da115496`.

The initial evidence in the parent directory is retained unchanged. This follow-up
fixes the standalone test loader for the Python 3.10–3.12 mypy targets and adds
completion and failure-retry checks requested during review. Production changes
are unchanged from the original source fix.

## CPU reproduction

Use a checkout of the baseline and Python 3.10 or newer. Save this directory
outside that checkout, then copy `repro.py` into the checkout as
`tests/unit/test_repro_mooncake_shutdown.py` (create `tests/unit` if needed).
The test finds the repository source by searching its parent directories.

```bash
python3 tests/unit/test_repro_mooncake_shutdown.py -v
# Expected on the production baseline: 5 tests, 5 failures, no errors.
git apply /path/to/this/evidence/fix.patch
python3 tests/unit/test_repro_mooncake_shutdown.py -v
# Expected after the production fix: 5 tests, OK.
```

The checked-in PR test is
`tests/ut/kv_offload/mooncake_v2/test_pull_worker_shutdown_cpu.py`.
Its standalone run requires only the Python standard library. Hardware-dependent
modules are mocked; host threads and queues are real. No NPU, CANN, model-serving
E2E, or performance execution is represented.

## Recorded results

- Original production source: 5 tests, 5 failures, 0 errors (`before.log`).
- Patched production source: 5 tests, OK (`after.log`).
- mypy 1.11.1, repository flags, Python targets 3.10, 3.11, 3.12: passed.
- Full `bash format.sh ci`: passed (`format.log`).
- Local checking was serial with pre-commit concurrency and Rayon threads set to 1.

`validation.png` renders actual CPU unittest log excerpts; it is not an NPU run.
`checks.json` records the commands, test hash, and format result. The local source
paths in that metadata identify the checked working tree; reproduction only
requires the relative command above.

## Review reasoning

Retain the receiver until synchronous cleanup completes. Clearing references
first would discard the retry path on failure and could allow a later call to
return before cleanup completes. A read-and-clear sequence alone does not provide
thread synchronization. This patch covers sequential repeat shutdown, not a
promise of general concurrent shutdown safety.

The new cases hold executor cleanup behind an event and verify retained ownership,
then inject a cleanup exception and verify successful retry.
