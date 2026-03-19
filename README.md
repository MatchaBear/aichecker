# aichecker

`aichecker` is a small Bash utility that performs a quick health check against major AI providers and prints a compact terminal summary.

Current version: `v2.0.0`

Right now it checks:

- OpenAI
- Anthropic (Claude)

For each provider, the script collects:

- Basic HTTP reachability
- A simple interpretation of whether the HTTP result looks normal
- Total response time
- A simple public API sanity check
- Official status page summary
- A plain-language impact hint
- Non-operational components reported on the status page

The result is shown as a one-line status summary per provider with color-coded health states.
The project also includes a continuous monitor mode that refreshes every 30 seconds.

## What the script does

The `aicheck` script:

1. Sends a request to a documented provider API endpoint and records the HTTP status code.
2. Measures total request time with `curl`.
3. Calls the provider models endpoint and uses `jq` to inspect the response.
4. Fetches the provider's official status page summary JSON.
5. Extracts the overall status description and any affected components.
6. Classifies service health as:
   - `UP`
   - `DEGRADED`
   - `BROKEN`
   - `DOWN`

## Health classification

The script currently uses this logic:

- `DOWN`: the API returns HTTP code `000`
- `BROKEN`: the documented API endpoint sanity check fails
- `BROKEN`: the provider status page reports a `major` or `critical` incident
- `DEGRADED`: the documented API endpoint returns an unusual HTTP status
- `DEGRADED`: total response time is greater than `1.5` seconds
- `DEGRADED`: the provider status page reports a `minor` incident
- `UP`: none of the above conditions are triggered

`DEGRADED` means the service appears reachable but not fully normal.
`BROKEN` means the service check found a stronger signal that core functionality may be failing.

The output also labels the HTTP result directly:

- `normal for API probe`: `200`, `401`, or `403`
- `abnormal`: anything outside those expected probe responses
- `no response`: `000`

The output also includes an `Impact:` field to explain the likely meaning of the detected issue in plain language.

Status-page severity is also colorized:

- Green for `none`
- Yellow for `minor`
- Red for `major` or `critical`

In the terminal output, the `Status:` label stays uncolored while the status value itself is colorized based on provider severity.

## Requirements

- `bash`
- `curl`
- `jq`
- Network access to:
  - `https://api.openai.com`
  - `https://status.openai.com`
  - `https://api.anthropic.com`
  - `https://status.anthropic.com`

## Usage

Run the script directly:

```bash
./aicheck
```

Or make it available globally in your shell so you can run:

```bash
aicheck
```

For continuous monitoring with auto-refresh every 30 seconds:

```bash
aicm
```

One simple setup is to symlink it into `~/bin`:

```bash
chmod +x ~/Desktop/Projects/aichecker/aicheck
ln -sf ~/Desktop/Projects/aichecker/aicheck ~/bin/aicheck
chmod +x ~/Desktop/Projects/aichecker/aicheck-monitor
ln -sf ~/Desktop/Projects/aichecker/aicheck-monitor ~/bin/aicm
source ~/.zshrc
aicheck
aicm
```

This assumes `~/bin` is already on your `PATH`.

## Example output

```text
=== AI HEALTH CHECK ===
Wed Mar 19 09:00:00 +08 2026

OpenAI  UP | 0.214s | HTTP:200 (normal) | API:✔ | Status:All Systems Operational | Impact:No obvious service impact detected | Issues:None
Claude  DEGRADED | 1.812s | HTTP:200 (normal) | API:✔ | Status:Minor Service Outage | Impact:Responses are slower than expected | Issues:Console, API Requests
```

Continuous monitor mode displays the same output in a refresh loop and redraws the screen every 30 seconds until you press `Ctrl+C`.

Example of a broken result:

```text
ProviderX  BROKEN | 0.051044s | HTTP:200 (normal) | API:✖ | Status:Major Service Outage | Impact:API endpoint behavior looks broken | Issues:API Requests
```

Example of a degraded result:

```text
OpenAI  UP | 0.051044s | HTTP:401 (normal for API probe) | API:✔ | Status:All Systems Operational | Impact:No obvious service impact detected | Issues:None
Claude  DEGRADED | 0.133106s | HTTP:401 (normal for API probe) | API:✔ | Status:Minor Service Outage | Impact:Partial service disruption reported | Issues:claude.ai, platform.claude.com, Claude API, Claude Code
```

## Notes

- The API sanity check uses public responses from the models endpoints and does not send an authenticated request.
- Because provider edge behavior can vary, the script is best treated as a quick operational check rather than a formal uptime monitor.
- Output uses ANSI color codes for readability in a terminal.

## Roadmap

- Add more providers
- Add JSON output mode
- Add timeout and retry options
- Add alerting or cron-friendly exit codes
- Add CLI flags for threshold tuning

## License

This project includes a `LICENSE` file in the repository.
