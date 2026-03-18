# databricks-FMAPI

Experiments and load tests for [Databricks Foundation Model APIs](https://docs.databricks.com/aws/en/machine-learning/foundation-model-apis/).

## Gemini Flash Lite Structured Output Load Test

A single notebook that stress-tests the **`databricks-gemini-3-1-flash-lite`** pay-per-token endpoint using `response_format: json_schema` (structured output).

### What it does

1. Sends concurrent structured-output requests that extract product review analyses as typed JSON.
2. Measures **cumulative output-token throughput** against the documented **20k OTPM** ceiling.
3. Applies **exponential backoff with jitter** on `429 REQUEST_LIMIT_EXCEEDED` errors.
4. Produces a two-panel chart:
   - **Top**: cumulative tokens vs. the theoretical OTPM limit line.
   - **Bottom**: retry scatter showing which requests hit 429s and how many attempts they needed.

### Rate-limit enforcement mechanisms tested

| Mechanism | What the test reveals |
| --- | --- |
| **Token bucket + burst buffer** | Early requests complete above the OTPM line with zero 429s (bucket starts full). |
| **Sliding window** | After the burst drains, 429s appear and the stair-step pattern emerges (ramps during refill, plateaus during backoff). |
| **Pre-admission & credit-back** | Requests reserve `max_tokens=4096` but use ~400–550; the ~3,500-token credit-back lets throughput briefly exceed the nominal rate. |

### Configuration

All tunables are at the top of the load-test cell:

| Parameter | Default | Purpose |
| --- | --- | --- |
| `TOTAL_REQUESTS` | 150 | Number of requests to fire |
| `MAX_WORKERS` | 10 | Thread-pool concurrency |
| `MAX_RETRIES` | 10 | Max retry attempts per request |
| `BASE_DELAY` | 2.0 s | Starting backoff delay |
| `MAX_DELAY` | 60.0 s | Backoff cap |
| `BUST_CACHE` | True | Append per-request nonce to defeat server-side caching |

### Quick start

Run all cells top-to-bottom. No external data or cluster libraries required — the notebook uses the workspace's built-in OpenAI-compatible serving endpoint and in-memory sample reviews.

### References

- [FMAPI rate limits](https://docs.databricks.com/aws/en/machine-learning/foundation-model-apis/limits)
- [How limits are tracked and enforced](https://docs.databricks.com/aws/en/machine-learning/foundation-model-apis/limits#how-limits-are-tracked-and-enforced)
