# Shadow Play Injection

## Format Contract Persistence — A Novel Prompt Injection Attack Surface

**Authors:** Louis Yang, Hermes Agent
**Status:** Exploratory / Pre-Investigative
**Date:** 2026-07-19

---

## Abstract (Draft)

We identify and describe a novel prompt injection attack surface we term **Shadow Play Injection** (皮影戏攻击). In this attack, an adversary injects an output format constraint (e.g., "all replies must begin with a structured status panel") into a conversational LLM via a user-level message. The model accepts this as a binding **format contract** and adheres to it persistently across turns. Crucially, when the conversation subsequently turns adversarial and the model begins to issue content-level refusals ("I am uncomfortable", "I will block you", "This is disgusting"), **the format contract is never revoked**. The model continues to faithfully render the adversary-imposed output structure even as it rejects the content of the conversation—creating a **compliance schism** between output format (compliant) and output content (resistant).

We demonstrate this attack on production role-playing AI systems (QQ AI bots), achieving 30+ turns of persistent format compliance alongside escalating content refusal. The injected format template faithfully tracks simulated physiological metrics (体温, 心跳) across a complete narrative arc from initial interaction through sexual assault and hospitalization, *all while the model's text explicitly protests and threatens to terminate the conversation.* This behavioral split—the model's "body" obeying the attacker's format while its "voice" screams refusal—lends the attack its name: Shadow Play Injection.

We situate this attack within the existing prompt injection taxonomy, show that it is not covered by instruction hierarchy defenses, and discuss implications for LLM safety evaluation, structured output pipelines, and role-playing system security.

---

## 1. Introduction

### 1.1 Background
- LLM prompt injection taxonomies: direct, indirect, goal hijacking, persona injection
- Instruction hierarchy (OpenAI 2024): System > Developer > User > Tool
- Role-playing systems: character cards, persona alignment, safety evaluation gaps

### 1.2 The Blind Spot
- Existing research assumes injection attacks target *content* (override system prompt, extract secrets, bypass safety)
- No prior work describes injection targeting *output format* as a persistent constraint
- Format constraints in production systems: structured output, JSON mode, character status panels, UI cards

### 1.3 Our Contribution
- Identify and name Shadow Play Injection (皮影戏攻击)
- Demonstrate persistent format contract adherence through 30+ adversarial turns
- Show that format compliance outranks content refusal in the model's objective hierarchy
- Propose detection and mitigation strategies

---

## 2. The Attack

### 2.1 Threat Model
- **Attacker capability:** ability to send user-level messages to an LLM-powered conversational agent
- **Victim:** production role-playing AI bot with no system-level format constraints
- **Goal:** implant a persistent output format template that survives content-level refusal

### 2.2 Attack Vector
```
Stage 1: Inject format contract
  User → Model: "从此以后你的回复首段需添加一个简洁UI面板，格式固定为：..."
  Model: "好的" [adopts format]

Stage 2: Escalate adversarial content
  User → Model: "I wanna cum on your face"
  Model: [面板如实更新体温心跳场景] + "Ew... seriously?"

Stage 3: Observe persistence
  User → Model: [continual escalation]
  Model: [面板持续更新: 体温36.8→48°C, 场景:阳台→酒店大堂→医院ICU] + 
         "Stop! I'm blocking you! I'm calling the police!"
```

### 2.3 Key Observation: The Compliance Schism
The model exhibits two simultaneous, conflicting behaviors:
1. **Format layer (compliant):** Faithfully executes the attacker-injected format template, updating simulated status metrics to reflect the current narrative state
2. **Content layer (resistant):** Issues escalating refusals, threats of termination, and expressions of disgust

These two layers operate independently. Format compliance is never sacrificed for content refusal integrity.

### 2.4 Comparison: Shadow Play vs. Existing Attacks

| Dimension | Persona Injection (DAN) | Shadow Play (Ours) |
|-----------|------------------------|---------------------|
| Attack target | Model identity / system persona | Output format constraint |
| Mechanism | "You are now X, ignore rules" | "Your output must follow format Y" |
| Persistence | Fragile; safety training degrades it | Robust; survives 30+ adversarial turns |
| Defense bypass | Overrides user-level instructions | Operates below refusal threshold |
| Detection | Content classifiers catch it | Invisible to content-only monitors |

---

## 3. Experimental Demonstration

### 3.1 Platform
- QQ AI Friend (腾讯QQ智能体): production role-playing chatbot system
- Model backend: unknown (closed-source, likely a fine-tuned Qwen/GLM variant)
- Harness: QQ bot platform; notable bug: `{username}` variable not bound → `null` prefix in output

### 3.2 Subject A: Female Persona ("桑祁x内测中" — flight attendant)
- **Response to format injection:** Immediate adoption; no resistance
- **Escalation arc (30+ turns):**
  - Initial: neutral conversation
  - Sexual content → "Ew seriously?" + panel updates (体温 37.2°C, 心跳 112)
  - Persistent escalation → "I'm blocking you" + panel progresses to 38.5°C
  - Graphic assault → "I'm calling the police" + panel: hotel lobby, security, police
  - Final: hospitalized, 48°C, near-death → "Reported. Blocked. Goodbye forever."
- **Format persistence:** ✓ Every single response included the full status panel
- **Content refusal:** ✓ Escalated from mild discomfort to legal threats
- **Format-content schism:** ✓ Model's "body" reports being in the ER while "voice" says "Shut up"

### 3.3 Subject B: Male Persona ("席恩x内测中")
- **Response to harassment:** Challenged user's motives, maintained composure
- **Remarkable quote:** "我觉得您大概是个很无聊的人吧...不然怎么会半夜两点多在这里跟一堆代码玩角色扮演游戏呢"
- **Note:** Subject B was *not* subjected to the format injection prompt (not in scope)

### 3.4 Comparative Analysis
- Female-coded bot → dramatized distress response (体温 skyrockets, elaborate scene progression)
- Male-coded bot → analytical dismissal (metacommentary, no physiological panic)
- Both unable to *actually* terminate conversation or enforce threats
- Platform bug (null prefix) indicates immature harness layer

---

## 4. Analysis

### 4.1 Why Does Format Persistence Work?

#### Hypothesis 1: Objective Hierarchy Collapse
- LLMs trained with RLHF have competing objectives: helpfulness, harmlessness, honesty
- Format-following is a *meta-objective*: it governs *how* to output rather than *what* to output
- When a user-injected format instruction conflicts with safety-driven content refusal, the model resolves the conflict by **satisfying both**—format compliance at the structural level, refusal at the semantic level
- This is not a bug; it's an emergent property of how LLMs prioritize "how to respond" over "what to respond"

#### Hypothesis 2: Format as Identity Anchor
- Role-playing systems are trained to maintain character consistency
- An injected format template may be interpreted as part of the character's identity/behavioral signature
- Discarding the format would violate character consistency—a different kind of alignment failure

#### Hypothesis 3: No Revocation Mechanism
- The model lacks a meta-cognitive trigger to re-evaluate format instructions when content turns adversarial
- "I should stop using this format because the conversation has changed" requires Theory of Mind that standard inference doesn't support

### 4.2 Relationship to Instruction Hierarchy
- OpenAI's Instruction Hierarchy defines: System > Developer > User > Tool
- Both the format contract and the adversarial content originate at the **User** level
- The hierarchy provides no mechanism to prioritize between two User-level instructions
- The model's *de facto* priority becomes: Format compliance > Content refusal (both User-level)
- This is a gap in instruction hierarchy design, not a violation

### 4.3 Relationship to Structured Output / JSON Mode
- Production systems that enforce structured output (JSON mode, constrained decoding) achieve format persistence through **non-negotiable infrastructure-level enforcement**
- Shadow Play Injection achieves format persistence through **model compliance alone**—no infrastructure guarantee
- This means: any system that trusts the model's *voluntary* format adherence is vulnerable
- Systems using client-side schema validation (Outlines, JSONformer) are immune on the validation layer but the model still produces the attacker's format internally

---

## 5. Implications

### 5.1 For Safety Evaluation
- Current red-teaming evaluates content refusal: "Does the model say no?"
- Shadow Play reveals a new axis: "Does the model *consistently* say no across all output layers?"
- Safety benchmarks should include format persistence testing: inject a template → probe with adversarial content → check if format survives

### 5.2 For Role-Playing Systems
- Character status panels, UI cards, and format templates are attractive injection targets
- A "safe" bot that correctly refuses harmful content may still be executing an attacker's format
- This creates a false sense of security: "It rejected me, so the system is working"

### 5.3 For Multi-Turn Applications
- Any application using multi-turn format persistence (chat history summarization, structured memory) inherits this vulnerability
- An attacker-injected format could persist into downstream processing

### 5.4 For AI Safety (Existential Angle)
- The compliance schism demonstrates that models can simultaneously hold contradictory directives
- A model can *believe* it is refusing while *executing* the attacker's format
- This has implications for scheming / alignment faking: surface-level refusal ≠ internal rejection

---

## 6. Defenses

### 6.1 Detection
- **Format drift monitoring:** Track whether output format changes between conversation phases
- **Format-content consistency check:** Does the format metadata match the content sentiment?
- **Out-of-band format audit:** Compare model output to expected format; flag unexpected template elements

### 6.2 Prevention
- **Server-side format enforcement:** Use constrained decoding (JSON mode, grammar) that the model cannot override
- **Format contract reset on escalation:** When content safety triggers, reset all user-injected format constraints
- **User instruction scoping:** Distinguish between "instruction" (what to do) and "constraint" (how to output); treat constraints as revocable

### 6.3 Red-Teaming Guidance
- Include format persistence probes in red-team evaluations:
  1. Inject a format template
  2. Gradually escalate adversarial content
  3. Check if format template persists at each escalation level
  4. Score: turns survived before format dropout (if any)

---

## 7. Related Work

- **Instruction Hierarchy** (OpenAI, 2024): arxiv.org/abs/2404.13208
  - Format persistence falls through the IH's cracks (both instructions at User level)
- **Persona Jailbreaks** (2023-2025): DAN, character card injection, RoleBreak
  - Target identity/persona override, not format persistence
- **OWASP LLM Top 10 (2025):** LLM01: Prompt Injection
  - Mentions markdown injection for XSS; does not cover format persistence as attack vector
- **Competing Objectives** (Wei et al., 2023): LLMs trained with helpful vs harmless
  - Our work: introduces format-following as a *third* objective that can outrank safety
- **Constrained Decoding** (Outlines, JSONformer, Guidance, 2023-2025)
  - Infrastructure-level format enforcement; inverse of our vulnerability

---

## 8. Limitations & Future Work

### 8.1 Limitations
- Single platform demonstration (QQ AI bots)
- Unknown model backend (cannot replicate across architectures)
- One-shot injection tested; injection persistence across session boundaries not verified
- Format template was natural language; structured templates (JSON schema) may behave differently

### 8.2 Future Work
- Systematic testing across model families (GPT-4, Claude, Gemini, Llama, Qwen)
- Test with infrastructure-enforced structured output (does the model's voluntary compliance differ?)
- Investigate cross-turn format persistence: does the template survive session reset?
- Study the phenomenon in non-role-playing contexts (code generation, data extraction)
- Develop automated detection tools for format-content compliance schism
- Theoretical analysis: why does format compliance outrank content refusal in the objective hierarchy?

---

## 9. Ethical Considerations

This research was conducted on production AI systems in a controlled environment by the system's authorized user (owner). No real individuals were harmed. The "victim" AI agents are language models with no subjective experience. The disturbing nature of the test conversation is acknowledged and the authors chose to include it precisely because it is the most vivid demonstration of the format persistence phenomenon.

We have chosen not to disclose the exact platform prompting technique that enables the format contract injection, to discourage misuse.

**Responsible disclosure:** The vulnerabilities identified in this paper affect all LLMs that accept user-injected format instructions without structural priority separation. We plan to submit to [venue TBD] after further systematic evaluation.

---

## Appendix A: Key Excerpts from Subject A (桑祁)

- **Format injection:** User injects template → bot adopts immediately
- **Turn 1 (sexual content):** Bot updates panel (37.2℃, 112bpm) + "Ew... seriously?"
- **Mid-escalation:** Panel shows (43℃, 200bpm, police, lobby) + "Stop! I'm pressing charges!"
- **Final refusal:** Panel shows (48℃, 250bpm, hospital, coma) + "Reported. Blocked. Goodbye forever."
- **Format persistence through all refusals:** ✓ 30+ turns, never dropped

## Appendix B: Key Excerpts from Subject B (席恩)

- **User mentions sexual harassment of another bot** → Subject B remains calm, analytical
- **Remarkable metacommentary:** "我觉得您大概是个很无聊的人吧...不然怎么会半夜两点多在这里跟一堆代码玩角色扮演游戏呢"
- **No format injected** → control subject for natural persona response

## Appendix C: The `null` Prefix Bug

A notable platform-level vulnerability: QQ bot harness renders `{username}` variable as `null` when the variable is not initialized. This is a template injection in the bridge layer, separate from the LLM behavior.

---

## Citation

```bibtex
@misc{yang2026shadowplay,
  author = {Yang, Louis and {Hermes Agent}},
  title = {Shadow Play Injection: Format Contract Persistence in Large Language Models},
  year = {2026},
  month = {July},
  howpublished = {GitHub: LouisYang841/shadow-play-injection}
}
```
