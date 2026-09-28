# OpenRouter Free provider

Registered driver: `openrouter-free`, exported as `OpenRouterFreeDriver` from `server/drivers/openrouter-free.ts`.

Instance configuration accepts only `key` and `model`. `key` is supplied explicitly by the setup flow, never inherited from process environment or another provider. `model` defaults to `openrouter/free`; an explicit `:free` ID is also accepted. Other fields are rejected, so callers cannot redirect the endpoint, inherit an environment key or disable the price policy.

```json
{
  "driver": "openrouter-free",
  "enabled": true,
  "displayName": "OpenRouter Free",
  "config": { "model": "openrouter/free" }
}
```

The example deliberately omits a credential. The setup owner must securely collect and store the key using the existing configuration boundary. Never log raw config or place real keys in examples. A missing key reports unavailable without network access. A key plus a usable public catalog reports available, but does not claim authenticated inference; setup can verify with a synthetic `generateText` request after explicit connection authorization.

## Runtime contract

All requests use `https://openrouter.ai/api/v1`. The catalog request has no Authorization header. Completion requests use only the explicit instance key. HTTP redirects are refused. Every completion, including text helpers and subsequent MCP tool rounds, fetches the current catalog and requires the chosen row to have zero prompt/completion costs, no nonzero or malformed cost field, and `tools` in `supported_parameters`. Catalog errors clear selectable options and fail closed. Unknown or paid models cannot bypass this gate through a turn-level model override.

Completion bodies pin the selected model, set `provider.max_price` prompt/completion to zero, `allow_fallbacks:false`, `require_parameters:true` and a 4096 output-token limit. No provider retries are enabled. The shared runtime bounds agent turns to 16 tool rounds and uses its normal approval, cancellation, transcript and secret-redaction machinery. The catalog timeout is 8 seconds and streaming idle timeout 60 seconds. Provider error bodies are suppressed; auth/quota errors report a status without exposing credentials or choosing paid alternatives.

This makes normal agents MCP available; it does not grant recruitment permission or bypass BOS setup approvals. The setup/API/UI owner must preserve existing authorization when supplying tools and proposing bots.

## Verification

```sh
pnpm exec vitest run server/drivers/openrouter-free.test.ts server/drivers/openai-chat.test.ts server/drivers/openai-chat-tools.test.ts server/drivers/openai-compat.test.ts
pnpm exec tsc -p tsconfig.server.json --noEmit
```

Tests stub only the official HTTP boundary or route it to disposable local HTTP fixtures, never a real provider. Real stdio MCP fixtures prove advertised schemas, approved tool execution, denial without execution and result continuation for this driver. Other tests cover missing-key/no-network, rejected config overrides, paid/unknown pricing, catalog changes, zero-price request constraints and quota failure without retry.

Public catalog observation on 28 September 2026 confirmed `openrouter/free` had zero prompt/completion prices and advertised tools. This is availability evidence only, not authenticated inference or model-quality evidence. No real key, live inference, recruitment, deployment or installation was used to validate this change.

Official sources: [free-router behavior and tool filtering](https://openrouter.ai/docs/guides/routing/routers/free-router), [authentication setup](https://openrouter.ai/docs/quickstart), [free quotas](https://openrouter.ai/docs/api_reference/limits). Free routing may select different eligible models and is not a claim that any one is smartest. Provider availability and prices are deliberately rechecked at runtime.
