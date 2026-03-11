  # Embeddings 接口支持 - 交接问题清单

  ## 已知事实（检索记忆）

  - OpenAI 兼容路由在 internal/api/server.go:332，目前包含 /v1/chat/completions 与 /v1/completions；端点列表在 internal/
    api/server.go:355，暂无 /v1/embeddings。
  - OpenAI handler 的入口函数位于 sdk/api/handlers/openai/openai_handlers.go:98（ChatCompletions）与 sdk/api/handlers/
    openai/openai_handlers.go:153（Completions）。
  - OpenAI-compat executor 的 endpoint 选择与 payload/think 处理在 internal/runtime/executor/
    openai_compat_executor.go:86、internal/runtime/executor/openai_compat_executor.go:99、internal/runtime/executor/
    openai_compat_executor.go:106，stream 入口在 internal/runtime/executor/openai_compat_executor.go:179。
  - 非流式执行入口在 sdk/api/handlers/handlers.go:470。
  - provider 列表可从注册表获取（internal/registry/model_registry.go:1034），auto 解析在 internal/util/provider.go:71，
    thinking 后缀解析在 internal/thinking/suffix.go:23。
  - OpenAI-compat provider key 可能是 openai-compatibility 或具体 compat 名（如 openrouter），见 sdk/cliproxy/
    service.go:881 与 sdk/cliproxy/service.go:923。

  ## 待确认问题（请先定结论）

  1. Embeddings 是否仅允许 OpenAI-compat 上游？若模型 provider 属于 gemini/claude/codex/qwen/iflow/kimi/antigravity，是
     否直接返回 400 invalid_request_error？
    ExecuteWithAuthManager(..., alt="embeddings")。
  - 在 internal/runtime/executor/openai_compat_executor.go:86 扩展 alt 逻辑：当 opts.Alt=="embeddings" 时 endpoint 设
    为 /embeddings 且强制非流；并跳过 applyPayloadConfigWithRoot 与 thinking.ApplyThinking（internal/runtime/executor/
    openai_compat_executor.go:99、internal/runtime/executor/openai_compat_executor.go:106）。
  - （可选）在 internal/runtime/executor/openai_compat_executor.go:179 的 ExecuteStream 中对 opts.Alt=="embeddings" 返回
    400/NotImplemented，防止误用。

  ## 测试/验收建议

  - POST /v1/embeddings + openai-compat 模型 → 200，返回 embeddings。
  - POST /v1/embeddings + stream=true → 400 invalid_request_error。
  - POST /v1/embeddings + 非 openai-compat 模型 → 400 invalid_request_error。