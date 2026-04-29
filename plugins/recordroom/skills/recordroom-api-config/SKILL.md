---
name: recordroom-api-config
description: Create, debug, and manage API configurations for RecordRoom evaluations. Use when the user needs to set up how RecordRoom calls their agent's API, fix failing eval tests due to HTTP errors, or modify an existing API config.
---

# RecordRoom API Config — Agent Workflow

You are working with API configurations that define how RecordRoom calls the system under test during evaluations. Each config is a multi-step HTTP request chain: authentication, setup, and the actual API call whose output gets judged.

## Prerequisites

- CLI must be authenticated: `npx recordroom auth status`
- A datasource must be connected: `npx recordroom status`
- If either fails, follow the recordroom-setup workflow first

## When You Need This

- An eval test is failing with HTTP errors (504, 401, 403) instead of a real response
- The test output is an HTML error page, not the expected AI response
- You need to set up or modify how RecordRoom calls your system's API
- `recordroom evals show` told you the API config is wrong

## Understanding API Configs

An API config has:

- **name** — Human label (e.g., "Production API", "Staging E2E")
- **base_url** — Shared base URL for all steps (can be overridden per step with absolute URLs)
- **headers** — Shared headers (auth tokens, content-type) applied to all steps
- **steps** — Ordered chain of HTTP requests:
  - **Setup steps** run once before the conversation (e.g., get auth token, create a session)
  - **Per-turn steps** (`per_turn: true`) run for each conversation turn (e.g., send chat message)
  - **Output steps** (`is_output_step: true`) — their response gets sent to the LLM grader for judging
- **extract** rules — Pull values from one step's response to use in later steps (e.g., extract `access_token` from auth response, use it as `${access_token}` in headers)
- **body_template** — Request body with `${variable}` placeholders. `${input}` is the user message for per-turn steps.
- **output_extraction_path** — Dot-path to extract the relevant output from the response (e.g., `response.text`)
- **output_transform_code** — JavaScript code to transform the raw response into readable text for display

## Commands

### See what's configured

```
recordroom api-config list
```

Shows all configs for the current project with their status (default/inactive, last test result).

### Inspect a config in detail

```
recordroom api-config show <config-id>
```

Returns the full JSON config including all steps, headers, body templates, and extract rules. This is how you understand the exact request chain.

### Test a config

```
recordroom api-config test <config-id>
```

Executes the full step chain with a sample input and shows per-step results: HTTP status, timing, extracted variables, and the final output. Use `--verbose` to see full request/response bodies.

```
recordroom api-config test <config-id> --input "Hello, how are you?" --verbose
```

Pass variables with `--var`:

```
recordroom api-config test <config-id> --var API_KEY=sk-xxx --var USER_ID=123
```

### Export, edit, and apply

Export the current config to a file:

```
recordroom api-config export <config-id> > api-config.json
```

Edit the JSON file, then apply changes:

```
recordroom api-config apply --file api-config.json
```

Apply uses upsert logic: if the file contains an `id` field, it updates that config. Otherwise it matches by `name` + `environment`. If no match, it creates a new config.

### Delete a config

```
recordroom api-config delete <config-id>
```

## Config File Structure

```json
{
  "name": "My API",
  "base_url": "https://api.example.com",
  "headers": {
    "Content-Type": "application/json",
    "Authorization": "Bearer ${access_token}"
  },
  "steps": [
    {
      "name": "Authenticate",
      "method": "POST",
      "path": "/auth/token",
      "body_template": {
        "grant_type": "refresh_token",
        "refresh_token": "${REFRESH_TOKEN}"
      },
      "extract": {
        "access_token": "access_token"
      }
    },
    {
      "name": "Create Session",
      "method": "POST",
      "path": "/sessions",
      "body_template": {
        "name": "Eval ${timestamp}",
        "owner": "${user_id}"
      },
      "extract": {
        "session_id": "id"
      }
    },
    {
      "name": "Send Message",
      "method": "POST",
      "path": "/sessions/${session_id}/chat",
      "per_turn": true,
      "is_output_step": true,
      "body_template": {
        "message": "${input}",
        "user_id": "${user_id}"
      }
    }
  ],
  "timeout_ms": 60000,
  "output_extraction_path": "response.text"
}
```

### Key fields in steps

| Field | Purpose |
|-------|---------|
| `name` | Label shown in logs and UI |
| `method` | HTTP method (GET, POST, PUT, DELETE) |
| `path` | Relative to `base_url`, or an absolute URL |
| `headers` | Per-step header overrides (merged with shared headers) |
| `body_template` | Request body. Use `${var}` for variables. `${input}` is the user message. |
| `extract` | `{"var_name": "dot.path"}` — extract values from the response JSON for use in later steps as `${var_name}` |
| `per_turn` | If `true`, this step runs for every conversation turn. If `false` (default), it runs once during setup. |
| `is_output_step` | If `true`, this step's response is what the LLM grader evaluates. Exactly one per-turn step should have this. |
| `wait_before_ms` | Delay before executing (useful for rate-limited APIs) |
| `sse_config` | For Server-Sent Events endpoints — `{"enabled": true, "completion_event_type": "done"}`. Set `enabled: true` to stream the response. `completion_event_type` optionally specifies which SSE event type signals the end of the stream. |

### Variable resolution

Variables use `${lowercase_var}` syntax. Environment variables use `${UPPERCASE_VAR}`.

The variable bag is built in this order (later values overwrite earlier ones on collision):

1. **Test `input_variables`** — When a test is created from a trace, RecordRoom automatically extracts all fields from the original trace data and flattens them into dot-notation keys. For example, a trace row like `{"user_id": "abc", "session_id": "xyz", "user_message_json": {"text": "hello", "sender_type": "USER"}}` becomes:
   - `${user_id}` → `"abc"`
   - `${session_id}` → `"xyz"`
   - `${user_message_json.text}` → `"hello"`
   - `${user_message_json.sender_type}` → `"USER"`

   These are stored on the eval test and passed to every run automatically. You can see them with `recordroom evals show <id> --json` (look at the `input_variables` field).

2. **Built-in variables** (overwrite input_variables on collision, though collisions are rare):
   - `${uuid}` — A fresh UUID (useful for idempotency keys)
   - `${timestamp}` — Current epoch seconds (useful for unique names)

3. **`${input}` and `${test_id}`** — Set last, always available:
   - `${input}` — The user message for the current turn
   - `${test_id}` — The eval test ID

4. **Extract rules from previous steps** — Values pulled from earlier step responses via `extract` are merged into the variable bag as each step completes. For example, if step 1 returns `{"access_token": "tok_xxx", "projectId": "p_123"}` and has `"extract": {"access_token": "access_token", "project_id": "projectId"}`, then `${access_token}` and `${project_id}` are available in all subsequent steps.

5. **Environment variables** — `${UPPERCASE_VAR}` (matched by pattern `[A-Z_][A-Z0-9_]*`) resolves from the server's environment. These are substituted first during template rendering, before template variables.

6. **Variables from `--var` flag** — When testing manually with `recordroom api-config test`.

### How trace fields map to API requests

This is the critical piece: the API config's `body_template` must use the same field names that exist in the original trace data.

To set this up correctly:

1. Look at a raw trace to see its field structure:
   ```
   recordroom traces show <trace-id>
   ```

2. Note the field names the API expects (e.g., `user_id`, `session_id`, `project_id`, `message`)

3. In the API config's `body_template`, reference these as `${field_name}`. For nested fields, use dot notation: `${user_message_json.text}`.

4. When a test is created from this trace, the `input_variables` are automatically extracted with the same field names, so they match the `body_template` placeholders.

**Example:** If your trace rows look like:
```json
{"user_id": "U123", "session_id": "S456", "user_message_json": {"text": "How do I upload?", "research_id": "R789"}}
```

Your API config's per-turn step should have:
```json
{
  "body_template": {
    "user_id": "${user_id}",
    "session_id": "${session_id}",
    "message": "${input}",
    "research_id": "${user_message_json.research_id}"
  }
}
```

`${input}` is automatically set to the user message text for each turn. The other fields come from `input_variables` extracted from the trace.

### Multi-turn variable flow

For multi-turn tests, variables flow like this:

1. **Setup phase:** `input_variables` from the test + built-ins populate the initial variable bag. Setup steps run once, and their `extract` rules add to the bag (e.g., auth tokens, session IDs).

2. **Each turn:** A fresh copy of the setup variables is made. The current turn's fields are merged in (overriding any matching keys). `${input}` is set to this turn's user message. `${turn_index}` is set to the current turn number. Per-turn steps run with these variables.

This means extracted values from setup steps (like `${project_id}` from a "Create Project" step) persist across all turns, while `${input}` changes per turn. Additionally, `session_id`, `conversation_id`, and `thread_id` extracted during any turn are automatically carried forward to subsequent turns.

### Output extraction

`output_extraction_path` uses dot notation to drill into the response JSON:
- `"text"` → `response["text"]`
- `"response.message"` → `response["response"]["message"]`
- `"choices.0.message.content"` → standard OpenAI format
- `"[-1].messageText"` → last element of a response array

Supports negative array indexing and bracket notation (e.g., `field[0].subfield`). Falls back to the full response if the path doesn't resolve.

`output_transform_code` is JavaScript that receives the full response as `data` and returns a string:
```javascript
// Example: extract text from a complex response
const resp = data.response || data;
return resp.state?._response_backup || resp.text || JSON.stringify(resp);
```

## Common Fixes

**401 Unauthorized** — Auth token expired. Check the auth step's `body_template` for expired refresh tokens or API keys. Update and `apply`.

**504 Gateway Timeout** — The API is too slow. Increase `timeout_ms` (default 30000ms). Some AI APIs need 60000-120000ms.

**Wrong output extracted** — The grader is judging the wrong part of the response. Check `output_extraction_path` and `is_output_step`. Use `api-config test --verbose` to see the raw response and find the correct path.

**Missing variables** — A step fails because `${var}` wasn't resolved. Two common causes:
1. A previous step's `extract` rules don't match the actual response structure. Use `api-config test --verbose` to see what the response actually returns, then fix the dot-path in `extract`.
2. The test's `input_variables` don't have the field the `body_template` expects. Check the test with `recordroom evals show <id> --json` and look at `input_variables`. If a field is missing, the trace data didn't have it — you may need to hard-code the value in the `body_template` or add it to the test's `input_variables`.

**Rate limiting** — Add `wait_before_ms` to the rate-limited step (e.g., `"wait_before_ms": 2000` for a 2-second delay).

## Setting Up a New API Config from Traces

When creating an API config from scratch to replay existing traces:

1. **Examine a raw trace** to understand the data structure:
   ```
   recordroom traces show <trace-id>
   ```

2. **Identify what the API needs** — Look at the trace data to find: the API endpoint that was called, what authentication was used, what fields were in the request body. If you have access to the codebase, read the route definitions and request schemas.

3. **Map trace fields to body_template** — Every field in the trace's raw data gets auto-extracted as `input_variables` when a test is created. Use those same field names in your `body_template` with `${field_name}` syntax.

4. **Set up the step chain:**
   - Auth step (setup, runs once): gets tokens, extracts `${access_token}`
   - Session/project creation (setup, runs once): creates the context, extracts `${session_id}` or `${project_id}`
   - Chat/message step (per-turn, output): sends `${input}` with context variables, response gets judged

5. **Write the config JSON and apply it:**
   ```
   recordroom api-config apply --file api-config.json
   ```

6. **Test with a sample input:**
   ```
   recordroom api-config test <id> --input "test message" --verbose
   ```
   Read the per-step output. If any step fails, fix the config and re-apply.

7. **Verify with an actual eval test:**
   ```
   recordroom evals run <test-id>
   ```
   This uses real test data (with `input_variables` from the trace) rather than the sample input, so it catches variable mapping issues.

8. **Iterate** — If the eval fails, check the execution log in `recordroom evals show <test-id>` for the exact error. Common issues: wrong extraction path, missing variables, auth expiry, timeout too short.

Once configured, use `recordroom evals run --all` to execute all tests. See the `recordroom-evals` skill for the full eval workflow.
