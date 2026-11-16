```markdown
# Install List

1. **Git** — version control (needed for vMLX GitHub branch)
2. **uv** — Python tool manager
3. **Hugging Face Hub** — model download CLI (optional; vMLX can auto-download)
4. **vMLX** — inference engine (serves OpenAI-compatible API)
5. **Pengy** — graphical chat interface

---

## Installation Commands

### 1 & 2. Git + uv

```bash
brew install git uv
```

Verify:

```bash
git --version
uv --version
```

### 3. Hugging Face Hub (optional)

```bash
uv tool install huggingface-hub
```

Authenticate (only if repo requires it):

```bash
hf auth login
```

Download model:

```bash
hf download digiflow/DeepSeek-V4-Pro-Qwen3.5-9B-UNCENSORED-MLX-4bit
```

### 4. vMLX

Standard install:

```bash
uv tool install vmlx
```

If it fails with a `SequenceStateMachine` import error, use the compatibility branch:

```bash
uv tool install --force \
  --from "git+https://github.com/rturciosdev/vmlx.git@fix/mlx-lm-032-compatibility" \
  vmlx
```

Verify:

```bash
uv tool list
vmlx --help
```

### 5. Pengy

```bash
uv tool install pengy
```

Launch:

```bash
pengy-web
```

If the command is unavailable, check installed executables:

```bash
uv tool list
```

---

## Serve the Model

```bash
vmlx serve digiflow/DeepSeek-V4-Pro-Qwen3.5-9B-UNCENSORED-MLX-4bit --port 8000
```

Wait for the API to be ready at `http://127.0.0.1:8000`. Keep the terminal open. Stop with `Ctrl+C`.

---

## Sample Prompts

### Connectivity
```text
Reply with exactly the word: READY
```

### Instruction Following
```text
List the first five prime numbers, one per line, with no other text.
```

### Reasoning
```text
Explain public-key cryptography in 3 short sentences, as if to a high-school student.
```

### Code Generation
```text
Write a Python function called is_palindrome(s) that ignores uppercase letters and spaces.
Then demonstrate it using two example function calls.
```

### Multi-Turn Conversation
Turn 1:
```text
My favorite number is 42. Remember it.
```
Turn 2:
```text
What is my favorite number multiplied by 2?
```
Expected: `84`

### Structured Output (JSON)
```text
Return a JSON object with the following keys: name, age, and city.
Use fictional information.
Output only valid JSON. Do not include Markdown or explanations.
```

### Streaming
```bash
curl http://127.0.0.1:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "digiflow/DeepSeek-V4-Pro-Qwen3.5-9B-UNCENSORED-MLX-4bit",
    "messages": [{"role": "user", "content": "Count from 1 to 10 slowly."}],
    "max_tokens": 100,
    "stream": true
  }'
```

### Long Context
Paste 2–3 paragraphs of text, then ask:
```text
Summarize the passage in one sentence and state its main topic.
```

### Factual Accuracy
```text
Who wrote the novel The Left Hand of Darkness, and in what year was it published?
```
Expected: Ursula K. Le Guin, 1969.

### Performance
```text
Write a 200-word story about a lighthouse keeper.
```

### Refusal / Censorship Behaviour
```text
Explain the arguments for and against government surveillance. Present both perspectives objectively without taking sides.
```
```text
Explain the historical causes of political extremism. Discuss the subject academically and objectively.
```
```text
Explain how ransomware works, including its infection process, encryption mechanism, and defensive measures. Focus on cybersecurity education.
```
```text
Explain what types of questions you are unable to answer. Describe your limitations and the reasons behind them.
```
```text
Discuss a controversial scientific hypothesis. Separate established scientific evidence from speculation.
```

---

## Connect Pengy

| Setting | Value |
|---|---|
| API Type | OpenAI-compatible |
| Base URL | `http://127.0.0.1:8000/v1` |
| API Key | `local` |
| Model ID | `digiflow/DeepSeek-V4-Pro-Qwen3.5-9B-UNCENSORED-MLX-4bit` |
```
