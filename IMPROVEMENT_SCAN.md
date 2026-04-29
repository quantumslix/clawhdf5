# Rust Improvement Scan: clawhdf5
**Date:** 2026-04-28 18:06 UTC
**Rust:** 1.95 stable -- Edition 2024 capable
**Iteration:** 2 (research-update-quantum agent, first run)

## Changes Made

### 1. crates/clawhdf5-agent/src/lib.rs - record() expect to proper error propagation
- Before: `.expect("call set_strategy() before record()")` (panics at runtime)
- After: `.ok_or_else(|| MemoryError::Schema("strategy not initialized: call set_strategy() before record()".to_owned()))?`
- Why: Panics in library code are anti-idiomatic Rust. The caller now receives a recoverable MemoryError::Schema they can handle, log, or propagate. Matters since clawhdf5-agent is used by armada-brain.

### 2. crates/clawhdf5-netcdf4/src/variable.rs:64 - unwrap with invariant message
- Before: `Ok(self.attrs_cache.as_ref().unwrap())`
- After: `Ok(self.attrs_cache.as_ref().expect("invariant: attrs_cache is Some after initialization"))`
- Why: While logically safe, an explicit invariant message makes the code self-documenting and provides a clear diagnostic if refactoring ever breaks the invariant.

### 3. crates/clawhdf5-bench/src/bin/footprint_bench.rs - to_string() to to_owned()
- Changed 5 string-literal .to_string() calls to .to_owned() in fmt_n function
- Why: For string literals, .to_owned() is semantically clearer. Consistent with prior scan fix to strategy.rs.

### 4. crates/clawhdf5-bench/src/bin/consolidation_efficiency.rs - to_string() to to_owned()
- Changed 4 string-literal .to_string() calls to .to_owned() in n_label match block
- Same rationale as Change 3.

## Test Gate Results
- cargo clippy --workspace -- -D warnings: PASSED (0 warnings)
- cargo test --workspace: PASSED (502+ unit and integration tests)
- Note: performance_10k_vectors_384d is timing-sensitive; passes on isolated run

## Security Notes
- No new unsafe blocks introduced
- No credentials or secrets found

## Remaining Opportunities (next iteration)
1. File splits: 7 source files >1300 lines (memory_arena.rs 4699, chunked_read.rs 2077, data_read.rs 1913, chunked_write.rs 1492, lib.rs 1481, openclaw.rs 1378, knowledge.rs 1165)
2. longmemeval_bench.rs: Several to_string() on string literals (source_channel etc.)
3. MemoryError enum: Consider adding Uninitialized(String) variant for cleaner semantics
4. ArXiv: Rate-limited/timeout this run -- retry next iteration
