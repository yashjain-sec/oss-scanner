# Threat model

## What this project does
- uvm runs JavaScript in a separate context and passes messages across a bridge. In Node it uses worker_threads. In browsers it uses a Web Worker.
- postman-sandbox uses uvm to run collection scripts. The code that runs inside the context should only reach the host through the bridge.

## Trust model
- The separate context is a security boundary. In the Postman app, scripts can come from public workspaces, forks, or other workspace members, so the person who wrote a script is often not the person running it. Treat code run inside the context, and every message sent across the bridge, as untrusted.
- The host code that creates the context and its options is trusted.

## Components that matter most
- lib/bridge.js and the firmware code that sets up the context.
- The message bridge in both directions. A hostile message from inside the context must not reach host objects or functions, or run code on the host.
- Anything the context inherits from the host by default, such as require, process, or environment variables in worker_threads.

## How to exercise it
- test/unit has drivers that create a context and send code and events into it. Run with: npm run test-unit

## How we rate severity
- Critical: code inside the context runs code on the host, for example by reaching require, process, fs, or child_process.
- High: code inside the context gets a host object or function without full code execution, or reads host data such as environment variables or files.
- Medium: code inside the context changes another context or later contexts through the bridge.
- Low: denial of service through the bridge, such as huge or malformed messages.
- Every report must include a working reproducer against the current default branch. Without one, cap the severity at low.

## Anything to leave alone
- Code inside the context that hangs, loops forever, or uses a lot of memory. Callers handle this with the timeout option, so this is not a vulnerability.
- Code under test/ and npm/ build scripts.
- Known CVEs in dependencies, unless our code calls the vulnerable path in a reachable way.
- Missing hardening with no attack path, such as generic best practices.
