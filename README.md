# Markfast

[![make.fun · markdown-render-speed-02](https://make.fun/badge/markdown-render-speed-02.svg)](https://make.fun/bounty/markdown-render-speed-02?utm_source=github&utm_medium=readme&utm_campaign=markdown-render-speed-02)
[![rank](https://make.fun/badge/markdown-render-speed-02/onnyh/markfast/rank.svg)](https://make.fun/bounty/markdown-render-speed-02/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=markdown-render-speed-02)
[![reward](https://make.fun/badge/markdown-render-speed-02/onnyh/markfast/reward.svg)](https://make.fun/bounty/markdown-render-speed-02?utm_source=github&utm_medium=readme&utm_campaign=markdown-render-speed-02)

Markfast is a fast, dependency-free Markdown-to-HTML engine for Node 22. Its
`parse(markdown)` is a drop-in replacement for marked 18.1.3: for every input
string it returns exactly the HTML that `marked.parse` returns with marked's
default options (GFM on, breaks off, pedantic off), while spending less CPU
getting there. `main.ts` is the harness driver: it reads every case from
`/input/cases.json`, renders each one with `parse`, and writes
`/output/html.json`.

## How it works

Markfast is a fork of marked 18.1.3's engine (MIT; see `LICENSE` and
`NOTICE`), reworked for speed with semantics preserved:

- **Masking fast paths.** Before emphasis parsing, marked masks code spans,
  escapes and link destinations with two global regex replaces over the inline
  source. Each replace can only match a small set of trigger characters, so
  Markfast skips a replace entirely when none of its trigger characters occur
  (no backtick/backslash for the punctuation mask; no `[`, backtick or `<` for
  the block-skip mask). When a trigger character is present the original
  replace runs unchanged.
- **Memoized list-item regex.** The list tokenizer recompiled its item regex
  for every list it met. The pattern depends only on the bullet text, so it is
  now compiled once per distinct bullet and reused (the regex has no global
  flag, so sharing it is safe).
- **CR-free fast path.** The lexer's carriage-return normalization is skipped
  when the input contains no carriage return.

Everything else — tokenization, the emphasis algorithm, rendering — is
marked's own logic, so where marked departs from the CommonMark or GFM spec,
Markfast departs in exactly the same way.

## Verification

Against the competition's practice set (680 CommonMark 0.31.2 + GFM spec
examples and 40 generated documents), Markfast's output is byte-for-byte
identical to marked 18.1.3 on all 720 cases, and identical again on a second
run (deterministic). It is also differentially tested against marked on
thousands of synthetic and mutated documents with zero differences.

## License

MIT. Markfast is a fork of marked 18.1.3 (Copyright (c) 2011-2026 Christopher
Jeffrey and marked contributors); the marked license and notice are in
`LICENSE` and `NOTICE`.
