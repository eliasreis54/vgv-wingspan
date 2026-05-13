# Validate and Fix

Run static analysis — detect and use the project's linter/analyzer.

Run tests using the project's test runner. If MCP tools are available for running tests, prefer them over shell commands. When using the `run_tests` MCP tool, always pass:
- `timeout: "30s"` — caps each individual test at 30 seconds of real wall time
- `fail-fast: true` — stops the run at the first failure or hang so one stuck test does not block the entire suite

If a test run produces no output and does not complete within a few minutes, it is hung. The `run_tests` MCP tool has a known issue where a `TimeoutException` thrown by the test isolate is not propagated back as a tool result — the test runner exits but the MCP call never returns, leaving Wingspan waiting indefinitely.

When this happens, fall back to Bash with a hard process timeout:

```bash
timeout 120 flutter test --timeout=30s [path/to/test]
```

The `timeout 120` at the shell level kills the entire process after 2 minutes regardless of what Flutter does internally, ensuring the command always returns.

Once the test run completes (or is killed), identify the offending test and look for the root cause:
- **Widget tests**: a bare `tester.pumpAndSettle()` with no timeout argument — its default is 10 minutes. Replace with `tester.pump(Duration(milliseconds: 500))` or `tester.pumpAndSettle(timeout: Duration(seconds: 5))`.
- **Unit/integration tests**: an async operation that never resolves — an unresolved `Future`, a `Stream` that never emits or closes, a real network call with no timeout, or a loop with no exit condition.

If failures occur:
- Fix the issue and re-run
- Up to 3 attempts per failure
- After 3 failed attempts, use **AskUserQuestion** to ask the user for guidance with context on what failed and what you tried

Fix all lint warnings before proceeding.
