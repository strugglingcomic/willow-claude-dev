# Claude Integration Ideas

## Approach: Custom Command Endpoint

Willow sends transcribed speech to a configurable "command endpoint." Build a lightweight HTTP server that:

1. Receives transcribed text from Willow/WIS
2. Sends it to Claude API with system prompt for voice assistant behavior
3. Returns Claude's response as text for TTS playback

### Tech Stack

- **FastAPI** — lightweight Python HTTP server
- **Anthropic Python SDK** — Claude API calls
- **Docker** — containerize alongside WAS/WIS

### Basic Flow

```python
@app.post("/command")
async def handle_command(request: CommandRequest):
    response = client.messages.create(
        model="claude-sonnet-4-6",
        system="You are a helpful voice assistant. Keep responses brief and conversational.",
        messages=[{"role": "user", "content": request.text}]
    )
    return {"speech": response.content[0].text}
```

## Considerations

### Latency
- Voice assistants need fast responses (<2s total round-trip)
- Claude API latency is typically 1-3s for short responses
- Haiku would be faster but less capable; Sonnet is a good balance
- Could cache common queries

### Conversation Context
- Willow sends each utterance independently (no session state)
- Endpoint needs to manage conversation history per device
- Consider a sliding window of recent exchanges

### Smart Home Integration
- Could add tool use to let Claude control smart home devices
- Home Assistant API integration via Claude tool calls
- "Turn off the living room lights" → Claude tool call → HA API

### Offline Fallback
- If Claude API is unreachable, fall back to basic command matching
- Or use WIS's built-in LLM capability (smaller local model)

## Future Ideas

- Multi-room awareness (different system prompts per device/room)
- Voice identification (different users get personalized responses)
- Proactive notifications (timer alerts, reminders via TTS)
