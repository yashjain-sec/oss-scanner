# Threat model

## What this project does
- postman-sandbox runs pre-request and test scripts from Postman collections. It is used by postman-runtime, Newman, and the Postman app.
- It uses uvm to run each script in a separate context. Scripts may only reach the host through the pm API and the modules the sandbox allows.

## Trust model
- The sandbox is a security boundary. In the Postman app, scripts can come from public workspaces, forks, or other workspace members, so the person who wrote a script is often not the person running it. Treat script code as untrusted.
- Treat HTTP responses (status, headers, body, cookies, redirects) as untrusted too.
- Collection data that is not script code (URLs, variables, environments) is trusted. Reports that only need hostile collection JSON, without a script or a server, are out of scope.

## Components that matter most
- The boundary between the script and the host: lib/sandbox/*, execute.js, execute-context.js, pmapi.js, pm-require.js, timers.js, and the bootcode bundle.
- Any path where a script gets a reference to a host object, constructor, or function, or loads a module that is not on the allowed list.
- vault.js and cookie-jar.js: a script must not read vault secrets or cookies beyond what the user granted.
- How response data reaches scripts: pm.response, pm.cookies, xml2Json.js, and the response parsing helpers.

## How to exercise it
- test/unit drives the sandbox with sample scripts. Run "npm run cache" first, then "npm run test-unit".

## How we rate severity
- Critical: a script or a server response runs code outside the sandbox, for example by reaching require, process, fs, or child_process on the host.
- High: a script gets a host object or function without full code execution, reads vault secrets or cookies it was not granted, or reads local files. Also high: sending credentials to a host the collection did not target, or bypassing TLS checks the user turned on.
- Medium: a script changes the state of other scripts or later runs in a way the pm API does not allow, such as tampering with shared globals across sandbox contexts.
- Low: denial of service from a hostile server (ReDoS, deep recursion, huge responses).
- Every report must include a working reproducer against the current default branch. Without one, cap the severity at low.

## Anything to leave alone
- A script that hangs, loops forever, or uses a lot of memory. Callers handle this with the timeout option, so this is not a vulnerability.
- A script using intended features: pm.sendRequest to reach the network, pm.require for allowed packages, or reading data the pm API gives it.
- Code under test/, benchmark/, and npm/ build scripts.
- Known CVEs in dependencies, unless our code calls the vulnerable path in a reachable way.
- Missing hardening with no attack path, such as generic best practices.
