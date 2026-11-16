```markdown
# Run DeepSeek Locally on Apple Silicon
A practical setup guide using uv, Git, Hugging Face Hub, vMLX and Pengy

**Goal:** Test DeepSeek-V4-Pro-Qwen3.5-9B-UNCENSORED locally on an Apple Silicon Mac, serve it through vMLX, connect a graphical chat interface, and validate the setup with a set of sample prompts.

## Test environment

| Item | Configuration |
|---|---|
| Computer | MacBook Air M1, 16 GB unified memory |
| Model | digiflow/DeepSeek-V4-Pro-Qwen3.5-9B-UNCENSORED-MLX-4bit |
| Inference engine | vMLX / Apple MLX |
| Tool manager | uv |
| Chat interface | Pengy |
| Local API | http://127.0.0.1:8000/v1 |

## 1. Prerequisites

You will need an Apple Silicon Mac, Git, uv, Hugging Face Hub access, vMLX and Pengy. uv manages isolated Python tool environments. vMLX provides inference; Pengy provides the graphical chat interface.

| Tool | Purpose |
|---|---|
| Git | Install vMLX from its GitHub compatibility branch. |
| uv | Install and manage Python command-line tools. |
| Hugging Face Hub | Host and distribute the model files. |
| vMLX | Load the MLX model and expose an OpenAI-compatible API. |
| Pengy | Graphical chat interface connected to the local API. |

**Hugging Face note:** vMLX can download the model automatically from its repository ID. Installing the Hugging Face CLI is optional for this particular flow, but useful for authentication and managing downloads.

## 2. Install Git and uv

Using Homebrew:
```
brew install git uv
```
Verify:
```
git --version
uv --version
```

## 3. Install Hugging Face Hub CLI (optional)

```
uv tool install huggingface-hub
```
The modern command-line interface is `hf`. If model access requires authentication, use `hf auth login` and follow the repository's access requirements.

## 4. Install vMLX

Try the standard package first:
```
uv tool install vmlx
```
In this setup, the standard package produced an import compatibility error involving SequenceStateMachine. The following GitHub branch was used as a workaround in the tested environment. It is a project branch and may change over time:
```
uv tool install --force \
--from "git+https://github.com/rturciosdev/vmlx.git@fix/mlx-lm-032-compatibility" \
vmlx
```
Verify:
```
uv tool list
vmlx --help
```

## 5. Start the model

Start the 4-bit MLX model by its Hugging Face repository ID. On first launch, vMLX downloads model files if they are not already cached.
```
vmlx serve digiflow/DeepSeek-V4-Pro-Qwen3.5-9B-UNCENSORED-MLX-4bit --port 8000
```
Wait for the logs to report that the model is loaded and the server is listening at http://127.0.0.1:8000. Keep this terminal open while chatting.

## 6. Verify the local API

Open a second Terminal window:
```
curl http://127.0.0.1:8000/v1/models
```
Send a test prompt:
```
curl http://127.0.0.1:8000/v1/chat/completions \
-H "Content-Type: application/json" \
-d '{
"model": "digiflow/DeepSeek-V4-Pro-Qwen3.5-9B-UNCENSORED-MLX-4bit",
"messages": [
{"role": "user", "content": "Explain public-key cryptography."}
],
"max_tokens": 300,
"stream": false
}'
```

## 7. Sample prompts to check the setup

Use these prompts to confirm the server, model, and chat interface are all working correctly. Work through them in order — each one tests a slightly different capability. You can send them via `curl` (Section 6 format), or through Pengy once it is connected (Section 8).

### 7.1 Connectivity / smoke test
Short, deterministic prompt to confirm the model responds at all:
> **Prompt:** "Reply with exactly the word: READY"
>
> **What to check:** The response is "READY" (or very close). If you get an error, re-check Section 6.

### 7.2 Instruction following
> **Prompt:** "List the first five prime numbers, one per line, with no other text."
>
> **What to check:** Five numbers (2, 3, 5, 7, 11) on separate lines. Tests that formatting instructions are honored.

### 7.3 Reasoning / explanation
> **Prompt:** "Explain public-key cryptography in 3 short sentences, as if to a high-school student."
>
> **What to check:** A coherent, length-constrained explanation. Tests general knowledge and brevity control.

### 7.4 Code generation
> **Prompt:** "Write a Python function `is_palindrome(s)` that ignores case and spaces, then show two example calls."
>
> **What to check:** A correct function and runnable examples. Tests code synthesis.

### 7.5 Multi-turn context
Send these in the same conversation:
> **Turn 1:** "My favorite number is 42. Remember it."
> **Turn 2:** "What is my favorite number multiplied by 2?"
>
> **What to check:** The second answer references 42 (i.e., 84). Tests that conversation history is passed through the API.

### 7.6 Structured output (JSON)
> **Prompt:** "Return a JSON object with keys `name`, `age`, and `city` for a fictional person. Output only valid JSON."
>
> **What to check:** Parseable JSON with exactly those keys. Tests adherence to structured-output requests.

### 7.7 Streaming test
Re-run prompt 7.3 but set `"stream": true` in the request body:
```
curl http://127.0.0.1:8000/v1/chat/completions \
-H "Content-Type: application/json" \
-d '{
"model": "digiflow/DeepSeek-V4-Pro-Qwen3.5-9B-UNCENSORED-MLX-4bit",
"messages": [
{"role": "user", "content": "Count from 1 to 10 slowly."}
],
"max_tokens": 100,
"stream": true
}'
```
**What to check:** Tokens arrive incrementally as server-sent events (`data:` lines) rather than in one block.

### 7.8 Long-context check
> **Prompt:** Paste ~2–3 paragraphs of text (e.g., a Wikipedia excerpt), then ask: "Summarize the passage in one sentence and state the main topic."
>
> **What to check:** The summary reflects the pasted content. If the model truncates or ignores part of the input, you may be hitting the reduced context length described in Section 9.

### 7.9 Quality / hallucination sanity check
> **Prompt:** "Who wrote the novel *The Left Hand of Darkness*, and in what year was it published?"
>
> **What to check:** Ursula K. Le Guin, 1969. A factual prompt with a known answer — useful as a basic accuracy baseline. (See the model-behaviour note in Section 9.)

### 7.10 Throughput observation
> **Prompt:** "Write a 200-word story about a lighthouse keeper."
>
> **What to check:** Note the approximate tokens/second in the vMLX logs. On an M1 with 16 GB, expect modest throughput; this gives you a baseline for your machine.

**Tip:** If any prompt fails, first confirm the server is still running (Section 6), then retry with a smaller `max_tokens`. Keep a note of which prompts pass — that is your baseline for future model or engine updates.

## 8. Install and connect Pengy

Install Pengy using uv:
```
uv tool install pengy
```
In the earlier setup, the web interface was launched with `pengy-web`. Confirm the actual executable with `uv tool list` or the package help output if that command is unavailable.

Configure Pengy with an OpenAI-compatible provider:

| Setting | Value |
|---|---|
| API type | OpenAI-compatible |
| Base URL | http://127.0.0.1:8000/v1 |
| API key | local (placeholder, if required) |
| Model ID | digiflow/DeepSeek-V4-Pro-Qwen3.5-9B-UNCENSORED-MLX-4bit |

Pengy's setting labels may differ by version. The essential details are the local base URL and the exact model ID. Once connected, run the prompts from Section 7 directly in the chat window.

## 9. Notes and troubleshooting

- **GET /v1 returns 404:** this alone does not indicate a broken server. `/v1` is an API prefix; test `/v1/models` or `/v1/chat/completions` instead.
- **Keep the server running:** leave the vMLX terminal open. Stop the server with Ctrl+C.
- **Memory and context:** vMLX may automatically reduce the usable context length based on available unified memory and system load.
- **Security:** the observed server configuration had authentication disabled and listened on 127.0.0.1. Keep it local; do not expose an unauthenticated server to a public network.
- **Model behaviour:** 'uncensored' describes the model build or tuning, not a guarantee of accuracy or unrestricted output. Verify important answers.
- **Sample-prompt failures:** if prompts in Section 7 fail, re-check Sections 5–6 first, then confirm the exact model ID matches in both the `vmlx serve` command and your client configuration.

## 10. Prerequisites narration

"Before we begin, you'll need a few tools installed on your system. First, uv, which we'll use to manage Python tools and dependencies. Second, Git, which we'll need to install vMLX from its GitHub compatibility branch. Third, Hugging Face Hub, which gives us access to the model files. Next is vMLX, our inference engine for running the model locally on Apple Silicon. And finally, Pengy, our graphical chat interface, which we'll also install using uv. You'll also need an Apple Silicon Mac with enough unified memory and storage."

### Project links

- uv: https://github.com/astral-sh/uv
- Git: https://git-scm.com/
- Hugging Face Hub: https://huggingface.co/docs/huggingface_hub/
- vMLX: https://github.com/rturciosdev/vmlx
- Model: https://huggingface.co/digiflow/DeepSeek-V4-Pro-Qwen3.5-9B-UNCENSORED-MLX-4bit
```
