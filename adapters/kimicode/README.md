# TencentDB Agent Memory adapter for Kimi Code CLI

This directory documents the configuration-only integration between [Kimi Code CLI](https://github.com/MoonshotAI/kimi-cli) and TencentDB Agent Memory through the MemoryProxy OpenAI-compatible endpoint.

Kimi Code CLI supports custom providers using the OpenAI Chat Completions protocol (`openai_legacy`). No Kimi Code source changes or plugin are required.

## Configuration

Edit `~/.kimi/config.toml`:

```toml
[providers.tencentdb-memory]
type = "openai_legacy"
base_url = "http://127.0.0.1:8096/codebuddy/default"
api_key = "sk-tdai-replace-me"

[models.deepseek-v4-flash]
provider = "tencentdb-memory"
model = "deepseek-v4-flash"
capabilities = ["thinking"]
```

Replace the placeholder with a TencentDB user key locally, or use the Kimi Code credential mechanism supported by your installation. Do not commit a real key. The upstream model API key remains on the MemoryProxy host and is never committed to this file.

The `default` path segment is the MemoryProxy memory instance/space ID used by the local deployment. Use the instance ID from your deployment in other environments.

The model ID must match the model configured by MemoryProxy (`PROXY_UPSTREAM_MODEL`). The proxy route is OpenAI-compatible; the route name is retained for compatibility with the current MemoryProxy route table. If a dedicated `/kimicode/<spaceId>` route is added upstream, only `base_url` needs to change.

## What this provides

After the first request, MemoryProxy can perform its normal session initialization and bind the Kimi Code session to a Team, Agent, and optional Task. Subsequent turns can receive the bound memory context, and completed turns are written back to MemoryCore according to the proxy extraction configuration.

This adapter is configuration-only. It does not modify Kimi Code, TencentDB Core, or MemoryProxy.

## Limitations

- This integration requires a running TencentDB MemoryProxy and MemoryCore Gateway.
- Kimi Code must send OpenAI Chat Completions requests through the configured provider.
- The model name must be identical on the Kimi Code and MemoryProxy sides.
- Kimi Code CLI's platform-specific search/fetch services are not provided by the TencentDB upstream; use Kimi Code's local fallback or configure those services separately.
- This directory does not duplicate the shared MCP/Gateway client implementation from the existing Kimi Code MCP adapter.

## Verification

1. Start MemoryCore and MemoryProxy.
2. Create or select a TencentDB user key with access to at least one Team and Agent.
3. Export `TENCENTDB_MEMORY_API_KEY` and start Kimi Code CLI.
4. Start a new Kimi Code session and complete the Team → Agent → Task selection if enabled by the proxy.
5. Send a prompt, confirm the streamed answer, and verify the turn in MemoryCore Chat Memory.
6. Start a second session and verify that relevant memory is recalled without sharing credentials or changing Kimi Code source.

## 中文说明

本适配通过 Kimi Code CLI 官方支持的 `openai_legacy` 自定义 Provider，将请求发送到 TencentDB MemoryProxy。用户只需配置 `base_url` 和 TencentDB 用户 Key，不需要修改 Kimi Code 源码或安装插件。详细中文说明见 [README_CN.md](./README_CN.md)。
