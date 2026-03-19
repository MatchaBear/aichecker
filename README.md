# aichecker

`aichecker` is a small Bash utility that performs a quick health check against major AI providers and prints a compact terminal summary.

Right now it checks:

- OpenAI
- Anthropic (Claude)

For each provider, the script collects:

- Basic HTTP reachability
- Total response time
- A simple public API sanity check
- Official status page summary
- Non-operational components reported on the status page

The result is shown as a one-line status summary per provider with color-coded health states.

## What the script does

The `aicheck` script:

1. Sends a request to the provider base API URL and records the HTTP status code.
2. Measures total request time with `curl`.
3. Calls the provider models endpoint and uses `jq` to inspect the response.
4. Fetches the provider's official status page summary JSON.
5. Extracts the overall status description and any affected components.
6. Classifies service health as:
   - `UP`
   - `SLOW`
   - `BROKEN`
   - `DOWN`

## Health classification

The script currently uses this logic:

- `DOWN`: the API returns HTTP code `000`
- `BROKEN`: the models endpoint sanity check fails
- `SLOW`: total response time is greater than `1.5` seconds
- `UP`: none of the above conditions are triggered

Status-page severity is also colorized:

- Green for `none`
- Yellow for `minor`
- Red for `major` or `critical`

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

One simple setup is to symlink it into `~/bin`:

```bash
chmod +x ~/Desktop/Projects/aichecker/aicheck
ln -sf ~/Desktop/Projects/aichecker/aicheck ~/bin/aicheck
source ~/.zshrc
aicheck
```

This assumes `~/bin` is already on your `PATH`.

## Example output

```text
=== AI HEALTH CHECK ===
Wed Mar 19 09:00:00 +08 2026

OpenAI  UP | 0.214s | HTTP:200 | API:✔ | Status:All Systems Operational | Issues:None
Claude  SLOW | 1.812s | HTTP:200 | API:✔ | Status:Minor Service Outage | Issues:Console, API Requests
```

## Notes

- The API sanity check uses public responses from the models endpoints and does not send an authenticated request.
- Because provider edge behavior can vary, the script is best treated as a quick operational check rather than a formal uptime monitor.
- Output uses ANSI color codes for readability in a terminal.

## Possible future improvements

- Add more providers
- Add JSON output mode
- Add timeout and retry options
- Add alerting or cron-friendly exit codes
- Add CLI flags for threshold tuning

## License

This project includes a `LICENSE` file in the repository.
