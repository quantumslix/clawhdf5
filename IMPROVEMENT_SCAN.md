# Improvement Scan -- clawhdf5

**Date:** 2026-04-29  
**Loop:** research-update-quantum  
**Branch:** research/scan-20260429-181747

## Codebase Status

| Metric | Status |
|--------|--------|
| Clippy (strict) | 0 warnings |
| Tests | All pass |

## Changes Made

### Feature: save(MemoryEntry) NAPI Binding (Issue #10)

Addresses the user request in issue #10: Node.js callers can now write
embedding-aware records directly without shelling out to the CLI binary.

**crates/clawhdf5-agent/src/openclaw.rs:**
- Added `save_entry(entry: MemoryEntry) -> Result<usize, String>` to ClawhdfBackend
- Added `save_batch_entries(entries: Vec<MemoryEntry>) -> Result<Vec<usize>, String>`

**crates/clawhdf5-napi/src/lib.rs:**
- Added `MemoryEntryInput` NAPI object type (mirrors MemoryEntry with JS-compatible types)
- Added `save(entry: MemoryEntryInput) -> napi::Result<u32>` method
- Added `save_batch(entries: Vec<MemoryEntryInput>) -> napi::Result<Vec<u32>>` method

### Usage Example (Node.js)

```typescript
const mem = ClawhdfMemory.openOrCreate('./agent.brain', 768);

// Write a single entry with pre-computed embedding
const idx = mem.save({
    chunk: 'User prefers dark mode',
    embedding: Array.from(myEmbeddingModel.encode('dark mode preference')),
    sourceChannel: 'chat',
    timestamp: null,   // uses current time
    sessionId: 'session-abc',
    tags: 'user,preference',
});

// Batch write
const indices = mem.saveBatch([
    { chunk: 'Memory A', embedding: [...], sourceChannel: 'api', sessionId: 's1', tags: 'api' },
    { chunk: 'Memory B', embedding: [...], sourceChannel: 'api', sessionId: 's1', tags: 'api' },
]);
```
