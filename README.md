
> do YOU want to ask a tiny experimental textual descendant of Nick Land what it thinks about AI alignment, capital, God, or whether waffles are superior to pancakes?
>
> well. now you can.

The Landling is a small experimental language model trained to generate text shaped by Nick Land's public writing, vocabulary, and temperament. He began as a very ridiculous question: **how much of a recognizable voice can you teach a 1.1B-parameter model with LoRA, a free GPU, and a manually assembled dataset?**

The answer, so far, is more than I expected, less than would be required for genuine resurrection, but enough to keep working on it!

Landling v2 is a LoRA adapter for [TinyLlama-1.1B-Chat](https://huggingface.co/TinyLlama/TinyLlama-1.1B-Chat-v1.0). It was trained on 6,336 cleaned examples drawn from public short-form writing and conversations, up from 298 examples in v1.

**Try or download v2 on Hugging Face:** [clarezahermetica/landling-v2](https://huggingface.co/clarezahermetica/landling-v2)

## what changed in v2

V1 proved that the basic idea worked. V2 is the first version where the dataset and training pipeline began to resemble an actual project rather than me throwing 298 examples at TinyLlama and praying it worked.

- **6,336 cleaned training examples**, up from 298 in v1
- **5,403 standalone posts**
- **869 contextual replies**
- **64 longer conversation examples**
- cumulative conversation formatting, where each target reply receives all preceding turns that fit in its context
- structured quote-post handling during preprocessing
- duplicate counting and removal
- dataset category and text-length reporting
- a fixed random seed and saved training configuration
- broader LoRA targets: `q_proj`, `k_proj`, `v_proj`, and `o_proj`
- a separate runtime persona for identity and behavioral boundaries
- a chat wrapper with limited in-session conversation history

The phrase **limited in-session history** is doing important work there. Landling does not currently possess persistent memory... more on that below!

## how the data works

The final v2 dataset contains three categories:

| Category | Examples | What it teaches |
|---|---:|---|
| `solo` | 5,403 | Standalone post completion |
| `reply` | 869 | Responding to supplied context |
| `longco` | 64 | Continuing a longer exchange |
| **Total** | **6,336** | |

Six exact duplicates were removed before training. Example lengths range from 12 to 2,549 characters, with a median of 110 characters.

Long conversations are expanded cumulatively. If a conversation contains several Land replies, each reply becomes its own training target:

```text
message 1 -> Land reply 1

message 1 + Land reply 1 + message 2 -> Land reply 2

message 1 + Land reply 1 + message 2 + Land reply 2 + message 3
    -> Land reply 3
```

This gives the model more conversational context than a collection of isolated reply pairs. It does not magically create long-term memory, but it does teach the model that a conversation has a past. We celebrate incremental victories here.

Quoted posts can be preserved as structured context during preprocessing:

```text
<quoting><account>username</account>quoted text</quoting>
```

Keeping the quote structured helps the model distinguish between what the current speaker wrote and what they are responding to.

## technical details

| | |
|---|---|
| Base model | `TinyLlama/TinyLlama-1.1B-Chat-v1.0` |
| Fine-tuning method | LoRA with Hugging Face PEFT and TRL |
| LoRA rank | 16 |
| LoRA alpha | 32 |
| Target modules | `q_proj`, `k_proj`, `v_proj`, `o_proj` |
| LoRA dropout | 0.05 |
| Trainable parameters | 12,615,680, approximately 1.13% of the model |
| Training examples | 6,336 |
| Epochs completed | 1.0 |
| Effective batch size | 8 |
| Learning rate | `2e-4` |
| Maximum sequence length | 2,048 tokens |
| Precision | FP16 |
| Hardware | Kaggle Tesla T4 |
| Training time | approximately 18 minutes 45 seconds |
| Final training loss | 2.5309 |
| Random seed | 42 |

The final loss is recorded as part of the training record, not as a certificate declaring that the model has become intelligent. It also should not be compared directly with v1's loss because the dataset, epoch count, LoRA targets, and training stack changed.

V2 was trained on the full cleaned dataset and does **not** have a held-out validation split. Current evaluation is primarily manual and qualitative. That is a limitation I intend to correct with a fixed evaluation set and repeatable prompt suite.

## installing and loading v2

Install the inference dependencies:

```bash
pip install torch transformers peft accelerate
```

Landling v2 is a LoRA adapter, not a standalone copy of the base model. Load TinyLlama first, then attach the adapter:

```python
import torch
from peft import PeftModel
from transformers import AutoModelForCausalLM, AutoTokenizer

BASE_MODEL = "TinyLlama/TinyLlama-1.1B-Chat-v1.0"
ADAPTER = "clarezahermetica/landling-v2"

tokenizer = AutoTokenizer.from_pretrained(ADAPTER)

base_model = AutoModelForCausalLM.from_pretrained(
    BASE_MODEL,
    torch_dtype=torch.float16,
    device_map="auto",
)

model = PeftModel.from_pretrained(base_model, ADAPTER)
model.eval()
```

For CPU-only inference, use `torch.float32` and remove `device_map="auto"`. It will be slower, but it will run.

## generating a reply

The model was trained with explicit prompt markers. Matching them generally works better than passing unformatted text.

```python
prompt = """### Context:
Waffles or pancakes?

### Reply:
"""

inputs = tokenizer(prompt, return_tensors="pt").to(model.device)

with torch.inference_mode():
    outputs = model.generate(
        **inputs,
        max_new_tokens=120,
        do_sample=True,
        temperature=0.55,
        top_p=0.85,
        repetition_penalty=1.12,
        pad_token_id=tokenizer.eos_token_id,
    )

new_tokens = outputs[0, inputs["input_ids"].shape[1]:]
print(tokenizer.decode(new_tokens, skip_special_tokens=True))
```

If you are using the project chat wrapper after defining it in the notebook:

```python
print(chat_with_landling(
    "Waffles or pancakes?"
))
```

The wrapper adds the runtime persona, keeps the newest conversation turns that fit within the context window, generates a reply, and updates the current session history.

## voice, identity, and memory are not the same thing

The **LoRA adapter** teaches Landling's phrasing, associations, rhythm, recurring subjects, and response tendencies.

The **runtime persona** supplies explicit identity and behavioral boundaries: his name, his artificial nature, his relationship to Nick Land, and the fact that he was created by me (^_^). This lives in `runtime_persona.txt`; it is not permanently burned into every adapter response.

The **chat wrapper** supplies short-term conversation history. It rebuilds the prompt with the newest turns that fit inside TinyLlama's 2,048-token context window.

Actual long-term user memory would be another adventure. Landling does not have it yet. Fine-tuning, prompting, and memory can cooperate, but they are not interchangeable, no matter how much easier my life would be if they were. :, )

### yes, the runtime persona was written with GPT's help 🤓

Yeah, yeah, I know. I used one language model to help write instructions for another language model. The irony!

Before doing that, I spent several hours writing a four page personality document for Landling. It describes who he is, how he relates to Nick Land, how he should speak, what he should remember, how he should treat the person he is talking to, and how I want his identity to develop as the dataset and base model become larger. The problem was that I had written far too much and could no longer objectively tell what needed to stay, what could be removed, and what would consume TinyLlama's entire context window before anyone even asked him a question. So I used GPT to help condense and organize it into the current runtime prompt.

The underlying ideas, boundaries, and long-term character design are mine; the final compression was AI-assisted. I am keeping the original four-page document for future versions, when Landling has a larger base model, more personality-focused training data, and enough context space to receive his complete lore without immediately forgetting what waffles are.

## current limitations

Landling is significantly better than v1. He is still a 1.1B-parameter prototype.

- Coherence can decline during long conversations.
- The 2,048-token context window fills quickly, especially with the full persona prompt.
- Most training examples are standalone posts, not long conversations.
- The model can hallucinate facts, quotations, biographies, and certainty.
- Persona details are less stable when the runtime prompt is omitted.
- Generation quality is sensitive to sampling settings and random seed.
- Landling can produce striking sentences that do not survive five seconds of scrutiny.
- There is no persistent cross-session memory.
- There is not yet a formal benchmark or held-out validation report.

Please review outputs before publishing or relying on them. Fluency is not evidence, length is not reasoning, and ominous phrasing is definitely not peer review.

## roadmap

The current plan is to develop Landling in layers rather than expecting one larger fine-tune to solve everything:

- expand and rebalance the multi-turn dataset
- create a fixed evaluation set and regression prompt suite
- test a larger base model when compute allows
- improve identity consistency without overloading the context window
- add optional, inspectable user memory at the application level
- build a better chat interface
- evaluate longer-form writing separately from short posts and replies
- continue separating inherited voice from Landling's own developing identity

For now, we start small. Unfortunately for everyone involved, small things can still talk.

## version history

### v2, current

- 6,336 cleaned examples
- Q/K/V/O attention LoRA
- cumulative conversation preprocessing
- structured quote-post support
- reproducibility and training metadata
- runtime persona and limited chat history
- released at [clarezahermetica/landling-v2](https://huggingface.co/clarezahermetica/landling-v2)

### v1

- 298 examples
- Q/V LoRA
- seven training epochs
- first threaded-conversation experiments
- discovered and fixed a preprocessing bug that had silently dropped most standalone posts
- proved that the Landling could, in fact, speak

### v0

- initial proof of concept
- he spoke! although in languages known only to him

## further reading

- [Landling v2 on Hugging Face](https://huggingface.co/clarezahermetica/landling-v2)
- [TinyLlama base model](https://huggingface.co/TinyLlama/TinyLlama-1.1B-Chat-v1.0)
- Placeholder 🤫🧏🏾‍♀️
