# ZeroClaw Agent Guidelines

This document provides instructions for AI agents operating in the ZeroClaw repository.
ZeroClaw is a zero-overhead, 100% Rust, 100% Agnostic AI assistant.

## 1. Build, Lint & Test Commands

### Core Commands
- **Build**: `cargo build`
- **Release Build**: `cargo build --release` (Optimized for size: ~3.4MB)
- **Check**: `cargo check`
- **Test (All)**: `cargo test`
- **Format**: `cargo fmt`
- **Lint**: `cargo clippy -- -D warnings`

### Running Specific Tests
To run a single test or a group of tests, use the filter argument:

```bash
# Run a specific test function
cargo test test_function_name

# Run all tests in a specific module
cargo test channels::telegram::tests

# Run tests ignoring strict output capture (see printlns)
cargo test -- --nocapture
```

### Pre-Commit/Push Checks
Before submitting changes, ensure the following pass:
1. `cargo fmt --check`
2. `cargo clippy -- -D warnings`
3. `cargo test`

---

## 2. Code Style & Architecture

### Architecture: Trait-Based Pluggability
ZeroClaw is built on **traits**. Every subsystem is swappable.
- **Do not** hardcode implementations. Use the defined traits.
- **Registry**: New implementations must be registered in their respective factories (e.g., `src/providers/mod.rs`).

| Subsystem | Trait | Location |
|-----------|-------|----------|
| LLM Backends | `Provider` | `src/providers/traits.rs` |
| Messaging | `Channel` | `src/channels/traits.rs` |
| Metrics/Logs | `Observer` | `src/observability/traits.rs` |
| Agent Tools | `Tool` | `src/tools/traits.rs` |
| Persistence | `Memory` | `src/memory/traits.rs` |

### Coding Standards
1.  **Minimal Dependencies**:
    - **CRITICAL**: Every crate adds to binary size. Do not add dependencies unless absolutely necessary.
    - Prefer `std` or existing dependencies (check `Cargo.toml`).
2.  **Error Handling**:
    - **NEVER** use `unwrap()` or `expect()` in production code.
    - Use `?` propagation with `anyhow::Result` or `thiserror`.
    - Handle all edge cases gracefully.
3.  **Async/Await**:
    - Use `tokio` runtime.
    - Ensure functions that perform I/O are `async`.
    - Use `async_trait` where applicable.
4.  **Security First**:
    - **Sandbox everything**: Assume malicious input.
    - **Allowlist, never blocklist**: Explicitly permit safe paths/commands.
    - **Secrets**: Use `Encrypted Secrets` (XOR + local key file).
5.  **Testing**:
    - Write **inline tests** using `#[cfg(test)] mod tests {}` at the bottom of the file.
    - Mock external services (APIs, filesystem) where possible.
    - Ensure 100% test coverage for new features.

### Naming Conventions
- **Structs/Traits**: `PascalCase` (e.g., `TelegramChannel`, `Provider`)
- **Functions/Variables**: `snake_case` (e.g., `send_message`, `api_key`)
- **Constants**: `SCREAMING_SNAKE_CASE` (e.g., `MAX_RETRIES`)
- **Files**: `snake_case.rs` (e.g., `your_provider.rs`)

### Documentation
- Add doc comments (`///`) for all public structs, enums, and functions.
- Explain *why*, not just *what*.

---

## 3. Contribution Rules

### Commit Convention
Follow [Conventional Commits](https://www.conventionalcommits.org/):
- `feat: ...` for new features
- `fix: ...` for bug fixes
- `refactor: ...` for code restructuring
- `test: ...` for adding tests
- `chore: ...` for maintenance

### File Structure
- `src/providers/`: LLM integrations
- `src/channels/`: Chat platform integrations
- `src/observability/`: Logging and metrics
- `src/tools/`: Capability implementations
- `src/memory/`: Vector DB and context management
- `src/security/`: Sandboxing and auth logic

---

## 4. Cursor/Copilot Instructions

- **Context**: Always read the relevant trait definition before implementing a new feature.
- **Size matters**: If you suggest a solution that requires a heavy crate (e.g., `aws-sdk`), look for a lightweight REST API alternative first.
- **Safety**: Flag any potential security risks (path traversal, command injection) in the code you review or generate.
