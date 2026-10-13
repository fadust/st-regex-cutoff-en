# Regex Cutoff (English)

> **Unofficial English port.** This is a community translation of
> [st-regex-cutoff (正则截断)](https://github.com/LeenHawk/st-regex-cutoff) by **LeenHawk (碎叶哨所设定)**. All credit for the extension goes to the original author.
> Only user-facing text (UI, toasts, logs, docs) was translated; behavior is unchanged.
> The original Chinese README is kept as [README.zh-CN.md](README.zh-CN.md).
>
> The original repository has no license file. This port is published without the author's explicit permission, as a courtesy translation.
>
> **If you are the original author and want this port taken down or changed, open an issue here and it will be removed promptly.**
>
> Report translation problems here. Report bugs in the extension itself upstream.

A third-party UI extension for SillyTavern. It checks AI replies in real time against configurable regex groups. On a match it aborts generation and cuts the message.

## Features

- **Multiple regex groups.** Each group holds several regexes, one per line. Group logic is selectable:
  - **Any match (OR):** the group matches if any one of its regexes matches.
  - **All match (AND):** the group matches only if every regex in it matches.
- Groups are OR-ed together: any matching group triggers.
- **What happens on a trigger:**
  1. While streaming, text is checked live and generation is aborted on a match (can be turned off).
  2. The message is cut at the **earliest match position** (the matched text and everything after it are deleted). Then **X more characters before it** are deleted too. X is set in the settings and counted in code points; each CJK character or emoji counts as 1.
  3. For non-streaming replies, or matches missed during streaming, a fallback check runs after the message is saved.
- The cut result is written to the current swipe and the chat is saved.
- Optionally append text or script content to the end of a message after a regex cut. Leave it empty to append nothing. Appended content is not re-checked by the regexes.
- Built-in test box: paste text to preview which groups match and how it gets cut.
- Slash command `/regexcut`: manually run detection and cutoff on the last AI message.

## Segmented auto-continue

Generates long replies in segments (streaming must be on). There are two segment conditions. **Each can be toggled on its own; they run in parallel and whichever is met first cuts:**

- **By tokens:** counted live while streaming; the segment is cut when it reaches the threshold.
- **By time:** timed from the segment's first streamed chunk; the segment is cut after the set number of seconds (default 30, can be turned off).
- With neither ticked, no segmenting happens. With both ticked, whichever comes first cuts.

After a cut:

- A **continuation prompt** is injected automatically and ST's native "Continue" generates the next segment. The token count and timer reset at the start of each segment.
- **Max total tokens:** continuation stops once the whole message reaches this value.
- The continuation prompt is injected via `setExtensionPrompt`. **It goes into this request's prompt only and is never written to the chat.** Inject role is system or user; the default is user, for Claude APIs that reject requests ending with assistant.
- Before continuing, trailing whitespace is trimmed from assistant content in both the chat message and the final request. If the final request still ends with assistant, a temporary user prompt is appended to pass Claude's message validation.
- Pressing **Stop** manually does not trigger a continuation. A message cut by a regex match is not continued either.
- Built-in round safety limit: ⌈max total ÷ per segment⌉ + 3 when segmenting by tokens, 200 rounds when segmenting by time only. This prevents runaway loops.

## Regex syntax

One per line, two forms:

```
To be continued
/to be continued/i
```

- Plain patterns have no flags.
- The `/pattern/flags` form supports flags (`g` and `y` are stripped automatically).
- Empty lines are ignored. An invalid regex does not throw; it is shown in red inside its group and skipped.

## Installation

SillyTavern → Extensions → Install extension, and enter this repository's URL:

```
https://github.com/fadust/st-regex-cutoff-en
```

Or put this folder in `SillyTavern/data/<user>/extensions/` (or `public/scripts/extensions/third-party/`).

Do not install this alongside the original Chinese version; both use the same settings key and would conflict. Your existing settings carry over if you switch.
