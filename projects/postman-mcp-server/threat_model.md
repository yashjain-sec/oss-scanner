# Threat model

## What this project does and where untrusted input enters
- The Postman MCP server lets AI agents call the Postman API. It runs locally over stdio, with the user's POSTMAN_API_KEY from the environment.
- Untrusted input: tool arguments from the MCP client (which an AI model controls), and Postman API responses (which can hold content written by other workspace members).

## Components that matter most
- src/tools/*: how tool arguments become Postman API calls. Path traversal or injection in URL paths, query strings, or request bodies matters.
- src/clients/*: the API key must only go to the configured Postman API host and never to a host taken from tool input or API responses.
- Any tool that reads or writes local files, or runs commands.

## How to exercise it
- After the build, run "node dist/src/index.js" with POSTMAN_API_KEY set. src/tests/unit has drivers. Run with: pnpm exec vitest run src/tests/unit

## How we rate severity
- Critical: code execution on the host from tool input or API responses.
- High: sending the API key to a host other than the Postman API, or reading or writing local files the user did not ask for.
- Medium: tool input that reaches a different Postman API path or resource than the tool intends, where the API key allows it. The Postman API still checks access, so this is not a cross-tenant issue on its own.
- Low: denial of service and crashes.
- Prompt injection where API content changes the agent's behaviour is out of scope, unless the server itself does something unsafe as a result.
- Every report must include a working reproducer against the current default branch. Without one, cap the severity at low.

## Anything to leave alone
- Code under src/tests/ and scripts/.
- Known CVEs in dependencies, unless our code calls the vulnerable path in a reachable way.
