# Mentor Agent Directory

This directory contains the Mentor AI configuration files for voice note processing.

## Required Files

### persona.md
Core identity, safety rails, and tone guidelines for the Mentor AI.
- **Permissions**: Read-only for Mentor worker
- **Format**: Markdown with XML-tagged sections

### skills.md
Explicit instructions on how to evaluate the Main Agent's actions.
- **Permissions**: Read-only for Mentor worker
- **Format**: Markdown with capability descriptions

### master-voice.wav (Required for Voice Mode)
Reference audio sample for zero-shot voice cloning via Chutes.ai CSM-1B model.

**Requirements:**
- **Format**: WAV, 16-bit, 22050Hz or 44100Hz
- **Duration**: 10-15 seconds (optimal), max 30 seconds
- **Content**: Clean speech without background noise
- **File size**: Max 10MB

**Note:** The voice sample file is already placed at `/mentor/master-voice.wav` in the lippyclaw directory.

**Security (Optional):**
- SHA256 checksum verification is available but NOT required
- Only used if `MASTER_VOICE_CHECKSUM` environment variable is set
- To enable: `sha256sum master-voice.wav` and add to docker-compose.yml

## Directory Permissions

| File | Read | Write | Delete |
|------|------|-------|--------|
| persona.md | Mentor | ❌ | ❌ |
| skills.md | Mentor | ❌ | ❌ |
| master-voice.wav | Mentor, chutes_tts | ❌ | ❌ |

## Quick Start - Running with ghostclaw.sh

The lippyclaw voice system integrates with the existing ghostclaw.sh workflow.

### Prerequisites

1. **CHUTES_API_KEY** must be set in your `.env` file (already configured)
2. **master-voice.wav** must exist (already at `lippyclaw/mentor/master-voice.wav`)

### Build and Run

```bash
# From the ghostclaw root directory (/Users/mac/Documents/ironclaw)

# 1. Build all images (including mentor-mcp)
./scripts/ghostclaw.sh build

# 2. Start the stack (auto-detects voice sample)
./scripts/ghostclaw.sh up

# 3. Check health
./scripts/ghostclaw.sh health
```

### Voice Note Flow

1. Send a voice note to your Telegram bot
2. ghostclaw receives it via webhook
3. mentor-mcp transcribes via Chutes.ai Whisper
4. Mentor AI evaluates the transcription
5. Response is synthesized using your cloned voice
6. Voice note is sent back to Telegram

### Manual Voice Bootstrap (if needed)

```bash
# Manually bootstrap voice context
./scripts/ghostclaw.sh mentor:clone /Users/mac/Documents/ironclaw/lippyclaw/mentor/master-voice.wav
```

### Troubleshooting

```bash
# Check mentor-mcp logs
./scripts/ghostclaw.sh logs mentor-mcp

# Check ironclaw logs
./scripts/ghostclaw.sh logs ironclaw

# Restart with fresh voice sync
./scripts/ghostclaw.sh restart
```

## Architecture

```
Telegram Voice Note (.ogg)
         ↓
   ghostclaw.sh (orchestrator)
         ↓
   mentor-mcp (Node.js MCP server)
         ↓
   chutes_stt (WASM) → Chutes.ai Whisper → Transcription
         ↓
   Mentor AI (evaluation)
         ↓
   chutes_tts (WASM) → Chutes.ai CSM-1B → Voice Response (.wav)
         ↓
   Telegram sendVoice API → User receives voice reply
```

## Configuration

Voice mode is enabled by default when:
- `ENABLE_MENTOR_VOICE=true` in `.env`
- `MENTOR_AUTO_BOOTSTRAP_VOICE=true` in `.env`
- `CHUTES_API_KEY` is configured
- `master-voice.wav` file exists
