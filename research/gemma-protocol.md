# Can small Gemma choose how much work a graph claim needs?

Research for [issue #3](https://github.com/rubleseth/halo-graph-verifier/issues/3), checked November 25, 2026. Start the protocol check with **`google/gemma-4-E2B-it`**. E4B-it is the comparison candidate if the track rules and compute budget allow it. Both use per-layer embeddings. The E2B and E4B names count effective parameters, so use the checkpoint and precision when estimating memory. [Google model overview](https://gemma.ai/models/gemma-4/)

## What the competition establishes

White Circle lists Google DeepMind’s **Best Gemma post-train** track for applying Halo to a task of the entrant’s choice. The public listing identifies the model family. Sections 2.4 and 4.2 of the terms assign the base model, Halo version, hardware, metrics and submission requirements to the track rules. Obtain the applicable track rules or organizer confirmation for the exact E2B-it or E4B-it checkpoint, permitted adapter and required Halo version before committing compute. Confirm how judges can reproduce a result if some source packets have redistribution limits. [Competition](https://whitecircle.com/halo/competition?utm_source=luma) · [Terms §§2.4, 4.2, 6.1–6.2](https://whitecircle.com/halo/competition/terms)

## What the templates do

I downloaded both pinned `chat_template.jinja` files and rendered one text-only system/user exchange and three final assistant targets with Jinja 3.1.6 on CPU. Their SHA-256 was identical, `0a2c8073c878ab1da004bee933a998606537bbb62016310352c7285c3f01c5b5`. The check covers rendered text only. [E2B template at 4adf157](https://huggingface.co/google/gemma-4-E2B-it/blob/4adf15748de9e1a0097b6274cee9c15836c41c37/chat_template.jinja) · [E4B template at 4ecc2ba](https://huggingface.co/google/gemma-4-E4B-it/blob/4ecc2ba63bcac23ba4a767fc87025b1450f84c9c/chat_template.jinja)

| Render case | Observed result |
| --- | --- |
| Serve with the default `enable_thinking=false` | The prompt ends at `<\|turn>model\n`. |
| Serve with `enable_thinking=true` | Same assistant prefix. The first system turn gains `<\|think\|>` and a newline. The caller selects the setting before decoding. |
| Train with the default setting and a final assistant `content` | The answer follows the model-turn prefix directly. |
| Add a nonempty final assistant `reasoning_content` | The template inserts a native thought channel, closes it, then writes the answer. |
| Paste a native thought block into assistant `content` alone | The template strips that block and retains the answer. |

Google documents a conversation-level thinking switch and suggests system instructions can influence thought length. Neither the render nor the documentation measures whether E2B selects useful deliberation on D08. A request that toggles the flag after classifying the packet has already made the routing decision outside Gemma. [Google prompt formatting and adaptive efficiency guidance](https://gemma.ai/get-started/prompt-formatting/)

## One-generation training and serving contract

Use the same neutral instruction, stock template and default flag for every packet. The first generated tokens can begin the verdict object or open the native thought channel. For training, put the reviewed verdict and evidence locators in the final assistant `content`. Add a concise, evidence-grounded check to `reasoning_content` on examples where paired offline review favors deliberation. Omit that field on direct targets. Keep the route label, generation flags and later reviewer evidence out of the packet. This is a learned behavior to test.

At Halo revision `3a21b4d`, text SFT renders conversations through the tokenizer. The row renderer supplies tools, reasoning effort or `clear_thinking` where configured. A `reasoning_effort` field does not set Gemma’s `enable_thinking`. Use the checkpoint template and its actual model-turn prefix for completion-only masking. The published Gemma parallelism guide covers the MoE variant. A small dense checkpoint still needs its own processor, loss-mask and load/training check on the competition-approved version. [SFT](https://github.com/whitecircle/halo/blob/3a21b4dbdb56cb840d42a59e98e40f967a1d2807/agent-docs/training-methods/sft.md) · [Text input path](https://github.com/whitecircle/halo/blob/3a21b4dbdb56cb840d42a59e98e40f967a1d2807/halo/data/pipeline/text_rows.py) · [Template arguments](https://github.com/whitecircle/halo/blob/3a21b4dbdb56cb840d42a59e98e40f967a1d2807/halo/data/conversation.py) · [Gemma guide](https://github.com/whitecircle/halo/blob/3a21b4dbdb56cb840d42a59e98e40f967a1d2807/agent-docs/models/gemma4.md)

## Inexpensive next check

1. On CPU with the actual Halo dataset loader and checkpoint processor, pass two fixture rows through preprocessing. Assert identical training/serving prompt token IDs up to the assistant boundary for the same packet, and compare decoded non-masked labels to the expected direct and deliberative targets. Check both delimiters, the final verdict and the stop token. Record the revisions, dtype and template checksum.
2. For a future inference smoke test, prepare six independently keyed, rights-cleared text packets at roughly matched lengths. Include direct support, a control-versus-treated footnote conflict and a missing source. Use the same instruction, flag, token cap and decoding settings across all six. Send each packet once, request one completion, and record the underlying generation count per packet. A retry, router or repair generation counts as an additional call.
3. Examine generated token IDs before the serving layer removes special tokens. Distinguish a direct answer, an empty thought channel, a nonempty closed thought channel and a malformed/truncated response. Record input, thought, final and total generated tokens, latency, finish reason, verdict correctness and citation validity. Score a missing source as insufficient when the reference calls for it. Observing both valid paths across packets demonstrates the protocol can express both behaviors. If only one appears, record a collapsed route on the sample. Either outcome leaves accuracy and savings to a larger held-out comparison with always-direct and always-thinking.

Only the template render was executed. Checkpoint eligibility, Halo preprocessing parity, runtime mode selection, quality, latency and cost remain open.
