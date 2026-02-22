# Claude Integration Ideas

## Approach: Custom Command Endpoint

Willow sends transcribed speech to a configurable "command endpoint." Build a lightweight HTTP server that:

1. Receives transcribed text from Willow/WIS
2. Sends it to Claude API with system prompt for voice assistant behavior
3. Returns Claude's response as text for TTS playback

> The exact request/response format Willow expects should be verified against the [Willow docs](https://heywillow.io/) and source code before building this.

### Tech Stack

- **FastAPI** — lightweight Python HTTP server
- **[Anthropic Python SDK](https://docs.anthropic.com/en/api/client-sdks)** — Claude API calls
- **Docker** — containerize alongside WAS/WIS

### Basic Flow (sketch)

```python
@app.post("/command")
async def handle_command(request: CommandRequest):
    response = client.messages.create(
        model="claude-sonnet-4-6",  # check docs.anthropic.com for latest models
        system="You are a helpful voice assistant. Keep responses brief and conversational.",
        messages=[{"role": "user", "content": request.text}]
    )
    return {"speech": response.content[0].text}
```

> See [Anthropic API docs](https://docs.anthropic.com/en/docs/about-claude/models) for current model names and pricing. Choose based on latency vs. capability tradeoff at the time of implementation.

## Considerations

### Latency
- Voice assistants need fast responses — minimize round-trip time
- Smaller/faster Claude models trade capability for speed
- Could cache common queries or use streaming responses

### Conversation Context
- Willow sends each utterance independently (no session state by default)
- Endpoint would need to manage conversation history per device
- Consider a sliding window of recent exchanges

### Smart Home Integration
- Could add [tool use](https://docs.anthropic.com/en/docs/build-with-claude/tool-use) to let Claude control smart home devices
- Home Assistant API integration via Claude tool calls
- "Turn off the living room lights" → Claude tool call → HA API

### Offline Fallback
- If Claude API is unreachable, fall back to basic command matching
- Or use WIS's built-in LLM capability (smaller local model)

## Future Ideas

- Multi-room awareness (different system prompts per device/room)
- Voice identification (different users get personalized responses)
- Proactive notifications (timer alerts, reminders via TTS)
