# Research Note: Pipecat Framework

## Summary

Pipecat v0.0.106 (Mar 2026) is production-ready, maintained by Daily.co, BSD-2 licensed. Has native Egyptian Arabic support (AR_EG language enum), Twilio integration via TwilioFrameSerializer, and Silero VAD + SmartTurn. Multi-agent must be built manually using Pipecat Flows or context switching primitives.

## Key Findings

### Pipeline Architecture

- Core abstractions: Frame → FrameProcessor → Pipeline → PipelineTask → PipelineRunner
- Canonical pattern: transport.input() → STT → user_aggregator → LLM → TTS → transport.output() → assistant_aggregator
- ParallelPipeline enables branching; ProducerProcessor/ConsumerProcessor for cross-branch comms

### Twilio Integration

- FastAPIWebsocketTransport + TwilioFrameSerializer
- Audio: 8kHz mu-law (PCMU), must set audio_in/out_sample_rate=8000
- TwiML: `<Connect><Stream url="wss://server/ws" /></Connect>`
- Auto hang-up supported via call_sid/account_sid/auth_token params

### Multi-Agent (No Built-in Orchestration)

- **Pattern A:** LLMMessagesUpdateFrame to swap system prompt + context mid-conversation
- **Pattern B:** Function calling for routing (register route_to_agent function)
- **Pattern C:** ParallelPipeline with filters per branch
- **Pipecat Flows:** Node-based state machine with per-node prompts, tools, context strategies
- **LLMSwitcher:** Runtime LLM provider switching — BUT Anthropic NOT supported (only OpenAI, Google)

### VAD + SmartTurn

- Silero VAD: <1ms on CPU, configurable threshold/start_secs/stop_secs/min_volume
- SmartTurn: ML-based turn detection, default since v0.0.102, analyzes up to 8s audio, 23 languages
- VAD now on LLMUserAggregatorParams (not transport params — deprecated)

### Interruption Handling

- Automatic via UserTurnStartStrategy (VADUserTurnStartStrategy or MinWordsUserTurnStartStrategy)
- MinWordsUserTurnStartStrategy(min_words=2) recommended for Arabic to avoid false barge-in on fillers

### Tool Calling

- FunctionSchema + register_function pattern
- FunctionCallParams provides arguments, context, llm, result_callback
- cancel_on_interruption, timeout_secs configurable per function
- Tool calls auto-stored in conversation context

### Latency (500-800ms E2E typical)

- VAD: <1ms, SmartTurn: <100ms, STT: 200-550ms, LLM TTFT: 100-300ms, TTS TTFA: 100-300ms
- WebSocket transport adds ~50-100ms vs WebRTC
- Use ttfs_p99_latency override for benchmarked values

### Limitations

- Pre-1.0 API, breaking changes between releases
- Each call = separate pipeline instance (horizontal scaling only)
- Anthropic NOT in LLMSwitcher
- ElevenLabs WebSocket context limit (5) can be exceeded during rapid tool calls

## Impact on Architecture

- Pipecat Flows is the right abstraction for multi-agent (node = agent, transition = handoff)
- Must use OpenAI or Google LLM services for LLMSwitcher, not Anthropic directly
- Can use Anthropic via OpenAI-compatible proxy or as a non-switchable service
