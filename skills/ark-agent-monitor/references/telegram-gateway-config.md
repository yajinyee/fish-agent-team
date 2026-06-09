# Telegram Gateway Configuration Patterns & Pitfalls

## Thread/Topic Filtering (Platform-Level vs LLM-Level)

### ❌ Wrong: Using channel_prompt to silence bot
Telling LLM "don't respond in this thread" via channel_prompt does NOT work reliably:
- LLM still gets called (wastes tokens)
- LLM may emit text like "（靜默）" or other artifacts
- `on_processing_start` still fires → ⏳ reaction appears on ignored messages
- `on_processing_complete` may add ✅ even when no real work was done

### ✅ Correct: Use `allowed_topics` for whitelist filtering
```yaml
telegram:
  allowed_topics:
    - "8432"
    - "8436"
    - "8441"
    - "5635"
```
Messages in threads NOT in this list are dropped at platform level in `_should_process_message()` — LLM never called, no reactions, no processing.

### ✅ Correct: Use `ignored_threads` for blacklist blocking
```yaml
telegram:
  ignored_threads:
    - 1  # 新人類聊天室 (General topic)
```
Checked in `_should_process_message()` before any processing begins.

### When to use which:
- **allowed_topics**: When bot should ONLY respond in specific threads (strict whitelist)
- **ignored_threads**: When bot responds everywhere EXCEPT specific threads (blacklist)
- Both can coexist: allowed_topics is checked first, then ignored_threads

## Reaction Emoji Behavior

### ProcessingOutcome flow:
1. Message enters `_should_process_message()` → if False, NO reaction at all
2. Message passes → `on_processing_start` → ⏳ added
3. LLM processes → `on_processing_complete`:
   - `SUCCESS` → ✅
   - `FAILURE` → 👎
   - `NOOP` (no response, no delivery) → clear reactions (no emoji left)
   - `CANCELLED` → clear reactions

### Key insight:
If you want a message truly silent (no ⏳, no ✅, nothing), it must be blocked at `_should_process_message()` level, NOT via channel_prompt.

## Telegram Output Formatting

### ❌ Causes garbled text in groups:
```
# Header
## Subheader
```

### ✅ Correct for Telegram:
```
**Header**
- bullet point

**Subheader**
- bullet point
```

Telegram supports: **bold**, *italic*, ~~strikethrough~~, `code`, ```code blocks```, [links](url).
Telegram does NOT support: # headers, tables (|---|), HTML tags in most contexts.

## Cronjob Model/Provider Specification

### ❌ Wrong:
```python
model={"model": "claude-opus-4.6", "provider": "custom:kiro-gateway"}
```
Error: `Unknown provider 'custom:kiro-gateway'`

### ✅ Correct:
```python
model={"model": "claude-opus-4.6", "provider": "custom"}
```
When there's only one custom_provider configured, use `"custom"` without the name suffix.

### ✅ Also correct (omit provider to use default):
```python
model={"model": "claude-opus-4.6"}
```

## Bot-to-Bot Communication

Telegram bots cannot see other bots' messages by default, even with privacy mode disabled.

### To enable:
1. Open BotFather on Telegram mobile
2. Select the bot → Bot Settings
3. Look for "Allow messages from other bots in groups" (newer BotFather feature)
4. Enable it

### Verification:
After enabling, messages from other bots should appear in session_search results. Search for the other bot's username to confirm.

## Model Availability (kiro-gateway)

Available models on localhost:8000/v1 (as of 2026-06-05):
- claude-haiku-4.5 ✅
- claude-sonnet-4 / 4.5 / 4.6 ✅
- claude-opus-4.5 / 4.6 ✅
- claude-opus-4.7 ❌ (listed but returns "insufficient subscription level")
- deepseek-3.2, glm-5, minimax-m2.1/m2.5, qwen3-coder-next ✅

Default model: claude-opus-4.6 (小葉's preference)
