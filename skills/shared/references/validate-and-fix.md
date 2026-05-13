# Validate and Fix

Run static analysis — detect and use the project's linter/analyzer.

Run tests using the project's test runner. Detect how tests are run from the project's configuration, scripts, or CI setup.

**Flutter projects — known MCP hang:** The `run_tests` MCP tool has a fundamental issue with hung tests. When a Flutter test hangs (e.g., from an unbounded `pumpAndSettle()` call), the Flutter process does not terminate — it keeps running indefinitely. The MCP server waits for process termination, so the tool call never returns. Per-test timeouts passed to `run_tests` do not help because the Flutter process ignores them and keeps running.

For Flutter projects, run tests via Bash with an OS-level timeout instead:

```bash
timeout 120 flutter test [path/to/test]
```

`timeout 120` sends SIGTERM to the entire process group after 2 minutes regardless of what Flutter does internally, guaranteeing the command always returns. This command is not blocked by the flutter plugin hook.

If the test run is killed by the timeout, identify the offending test and look for the root cause:
- **Widget tests**: a bare `tester.pumpAndSettle()` with no timeout argument — its default is 10 minutes. Replace with `tester.pump(Duration(milliseconds: 500))` or `tester.pumpAndSettle(timeout: Duration(seconds: 5))`.
- **Unit/integration tests**: an async operation that never resolves — an unresolved `Future`, a `Stream` that never emits or closes, a real network call with no timeout, or a loop with no exit condition.

If failures occur:
- Fix the issue and re-run
- Up to 3 attempts per failure
- After 3 failed attempts, use **AskUserQuestion** to ask the user for guidance with context on what failed and what you tried

Fix all lint warnings before proceeding.
