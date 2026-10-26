# stepwise — promise vs. implementation gap tracker

The `stepwise` landing page (`index.html`) makes claims about what the
four-tool suite does for humans, for agents, and between agents. This
document audits those claims against what&rsquo;s actually shipped in the
`ccr`, `agx`, `sift`, and `holdout` repos — and enumerates what would have to land
before the page could drop the hedge-words.

**Suite scope.** Stepwise is four tools:

- **ccr** — find the session (cross-CLI session picker).
- **agx** — walk through the session (timeline viewer).
- **sift** — snapshot what the agent did in each turn (per-turn write ledger).
- **holdout** — verify the change is correct (held-out grading oracle).

`rgx` is a *related tool* by the same author, not a suite member. The
landing page links to it in a "Related" footer; this gap tracker does
not audit rgx&rsquo;s surfaces. Any rgx &harr; stepwise integration that
actually ships (e.g. sift &rarr; rgx for policy-pattern debugging)
becomes news the day its PR merges, and can be added to the page then
— never pre-announced.

**Maintained by:** whoever most recently edited `index.html`. When a
gap closes, move its row from *Open* to *Closed* and link the
commit / PR. When the page adds a new claim, the claim must come with
its gap row here — the landing page is not allowed to be ahead of
this document.

**Last audited:** 2026-10-26 against crates.io (ccr 0.2.6, agx-tui /
agx-mcp 0.2.1, sift-tui 0.1.1, holdout 0.1.1): holdout added as the
fourth tool; sift `state` / `export` / `doctor` / `fsck` and the `t`
hand-off marked shipped; agx `--jump-to` / `doctor` / `agx-mcp` marked
shipped; ccr `export --format json` marked shipped; license and Windows claims
corrected). Previous: 2026-04-20 (ccr v0.1.0 added; rgx demoted to
related tool).

---

## The promise (what the landing page asserts)

1. **Dual-addressed tools** — every tool is reachable as a TUI *and* as
   a scriptable CLI with JSON output, so an agent can query it on the
   user&rsquo;s behalf.
2. **Agent as a first-class principal** — sift in particular is
   designed around the agent running commands via natural-language
   requests; the human&rsquo;s direct touchpoint is `git commit`.
3. **Versioned export schemas** — every artifact meant to be consumed
   by a sibling tool or downstream agent carries a schema version and
   a SemVer commitment.
4. **Agent-to-agent hand-off** — one agent publishes via stepwise
   exports, another consumes, zero instrumentation in between.
5. **Named cross-tool integrations** — *Session hand-off*
   (ccr &rarr; agx, shipped v0.1.0), *Timeline jump* (sift &rarr; agx,
   shipped at session level; step-level planned), *Sift overlay in agx*
   (planned).
6. **Communication substrate** — the tools collectively serve as a
   shared vocabulary for human &harr; agent &harr; agent coordination
   over agentic work.

---

## Gap 1 — Agent discoverability

**Claim:** &ldquo;When you ask the assistant, it runs the matching
stepwise command and answers.&rdquo;

**Requires:** the assistant knows the tool exists, knows the command
surface, and reaches for it reflexively.

| Artifact | ccr | agx | sift | holdout |
|---|---|---|---|---|
| `docs/agent-guide.md` | &mdash; open | shipped | shipped | &mdash; open |
| `--ai-help` flag | open | open | open | open |
| Project CLAUDE.md injection via `init` | open | open | open | open |
| Dogfood log of agent reflex-use | open | open | open | open (one controlled trial) |

**Closes when:** (a) `ccr/docs/agent-guide.md` and
`holdout/docs/agent-guide.md` exist alongside agx&rsquo;s and sift&rsquo;s, (b) each tool has an `--ai-help` stdout
dump, (c) `ccr init` (new) / `agx init` / `sift init` optionally drop
a stanza into the project&rsquo;s CLAUDE.md, and (d) a 20-session
dogfood in a fresh Claude Code project shows assistants reaching for
the commands unprompted at least ~50% of applicable occasions.

---

## Gap 2 — Stable, versioned, documented schemas

**Claim:** &ldquo;agent-to-agent coordination relies on versioned
schemas.&rdquo;

| Surface | Schema doc | Version field | SemVer commitment | Ships with |
|---|---|---|---|---|
| `agx --export json` | [agx/docs/stability.md](https://github.com/brevity1swos/agx/blob/main/docs/stability.md) | &mdash; see gap 2a | documented | v0.1.x |
| `agx --export trajectory-openai` | described in `eval-integration.md` | &mdash; | documented | v0.1.x |
| `agx-core` public API | stability.md | Cargo SemVer | pre-1.0 | v0.1.x |
| `sift export --format json` | [sift/docs/export-schema.md](https://github.com/brevity1swos/sift/blob/main/docs/export-schema.md) | `sift_export_version: 1` | versioned; untested by a consumer (Gap 3) | **shipped** (v0.1.x) |
| `sift state --at-turn N --json` | agent-guide.md | &mdash; | &mdash; | **shipped** (v0.1.x) |
| `ccr export <id> --format json` | &mdash; open | &mdash; open | &mdash; open | **shipped**, unversioned |
| `ccr list --json` (proposed) | &mdash; open | &mdash; | &mdash; | not shipped &mdash; `ccr list` today emits plain text only |
| `ccr` session-record schema | &mdash; open | &mdash; | &mdash; | not shipped &mdash; would formalize the `Session` shape (tool, id, cwd, title, last_activity, preview) for downstream consumers |
| `holdout grade --json` | &mdash; open | &mdash; open | &mdash; open | **shipped**, unversioned |

### Sub-gap 2a — `tool_use_id` on agx export

sift Phase 1.2 hit a blocker: `agx --export json` does not serialize
`tool_use_id` on `StepKind::ToolUse`. Without it, sift&rsquo;s ledger
cannot be joined with agx&rsquo;s timeline for step-level integration.

**Closes when:** agx adds `tool_use_id` as a
`#[serde(skip_serializing_if = "Option::is_none")]` field on the
exported step (additive, so MINOR bump under the pre-1.0 policy).

---

## Gap 3 — A working downstream consumer

**Claim:** &ldquo;One agent&rsquo;s record becomes another
agent&rsquo;s readable input.&rdquo;

**Reality:** Zero consumers exist. Until one ships, the stability
commitments are untested — they can be *held* without being
*exercised*, which means bugs don&rsquo;t surface.

Candidate consumers that would close this gap:

- A code-review agent that reads `agx --export trajectory-openai` and
  flags suspicious turns.
- An eval harness that reads `sift export --format json` to build
  labeled datasets from accept / revert decisions.
- A small CLI (`stepwise-report`) that reads both exports and prints
  a per-turn summary of &ldquo;what the agent did, what was kept,
  and what it cost.&rdquo;

**Kill criterion from sift ROADMAP 1.7.2:** if `sift export --format
json` has no consumer within 6 months of shipping, sift drops its
schema-stability commitment.

---

## Gap 4 — Step-level integration

**Claim:** *Timeline jump* — press `t` on a sift entry to open agx at
the turn that produced the write.

**Reality:**

- sift&rsquo;s `t` keybind ships (sift-tui v0.1.0) and opens agx at
  *session* level.
- agx ships `--jump-to <STEP>` (agx-tui v0.2.x), so the launch side of
  the contract exists.
- sift can&rsquo;t compute `<STEP>` for a ledger entry until sub-gap 2a
  (agx serializing `tool_use_id` on exported steps) closes.

**Closes when:** (a) agx adds `tool_use_id` to JSON export, (b) sift
maps the entry&rsquo;s tool call to a step index and passes
`--jump-to <STEP>` through on `t`.

---

## Gap 5 — Validation evidence

**Claim:** &ldquo;Agents reach for these tools reflexively.&rdquo;

**Reality:** No measurement exists. The reframe of sift as agent-first
is from 2026-04-19; the intended dogfood (sift ROADMAP Phase 1.4) has
not run. ccr&rsquo;s `v`-to-agx hand-off is shipped as of 2026-04-20
but has no usage telemetry attached.

**Closes when:** a notes file under e.g. `sift/dogfood/phase-1.4.md`
records 20+ real agent sessions with and without the guide installed,
and reports which commands were reached for, at what rates, and
whether the behavior changed review decisions.

---

## Gap 6 — Machine-consumable output parity

**Claim:** &ldquo;Dual-addressed: TUI *and* scriptable CLI.&rdquo;

**Reality:**

| Tool | Shipped machine output | Gaps |
|---|---|---|
| ccr | `ccr list` / `path` / `show` / `stats` (plain text), `ccr export --format json` | No JSON on `list` — agents can&rsquo;t query the session index programmatically; no structured errors |
| agx | `--summary`, `--export md|html|json|trajectory-openai`, `corpus --json|--jsonl` | `--summary` is human-formatted (no JSON twin); no structured errors |
| sift | `--json` on `list`, `log`, `status`, `history`, `state`, `doctor`, `fsck`; `export --format json` | Not every subcommand has `--json`; structured error shapes (e.g. `{"error": "no_entries_for_turn", "turn": 99}`) not yet a convention |
| holdout | exit codes + `--json` on `grade` | JSON parity across `verify` / `properties` / `watch` unaudited; no structured errors; no MCP server |

**Closes when:** every subcommand an agent might reasonably call has
a `--json` (or `--format json`) twin, and error paths emit structured
envelopes instead of human prose.

---

## Gap 7 — Feature detection and graceful degradation

**Claim:** &ldquo;Missing sibling &rarr; status-bar install hint, not
a hard failure.&rdquo;

**Reality:**

- ccr&rsquo;s `v`-to-agx hand-off is feature-detected (shipped).
- sift `doctor` ships (agx / rgx sibling detection, `--json`).
- agx `doctor` ships (version, features, sift / rgx / agx-mcp
  availability).
- ccr and holdout have no `doctor` equivalent.
- None of the tools version-parse their siblings for compat checks.

**Closes when:** each tool ships a `doctor` (or equivalent) that
probes for siblings on `PATH`, parses their `--version`, and reports
whether each sibling&rsquo;s CLI surface meets the minimum-supported
contract from `suite-conventions.md` §5 (kept in
[sift/docs/](https://github.com/brevity1swos/sift/blob/main/docs/suite-conventions.md)
and agx/docs/).

---

## Gap 8 — An explicit principal model

**Claim:** &ldquo;Human / agent / agent-to-agent all addressable.&rdquo;

**Reality:** There is no document that spells out, per tool, which
commands are safe for the agent to run without user confirmation,
which mutate state, and which should always surface a preview first.

**Closes when:** `stepwise/docs/principals.md` (new) enumerates, for
each subcommand of each tool, its safety class (`read-only`,
`mutating-scoped`, `mutating-global`) and is referenced by every
`docs/agent-guide.md`.

---

## Gap 9 — Cross-tool protocol definition

**Claim:** Named integrations light up automatically when siblings are
installed.

**Reality:**

| Integration | Direction | Status | Blocker |
|---|---|---|---|
| *Session hand-off* (`v`) | ccr &rarr; agx | **shipped** (ccr v0.1.0, 2026-04-20) | &mdash; |
| *Timeline jump* (`t`) | sift &rarr; agx | **shipped, session level** (sift-tui v0.1.0); step-level planned | step-level: Gap 2a + Gap 4 |
| *Sift overlay* | agx &rarr; sift (consumer) | planned | agx-side consumer (sift export has shipped) |

**Closes when:** each integration has a PR link under it, a sibling-
independent smoke test, and a line in the README cross-tool compat
table. The *Session hand-off* row is the template: the page can add
an integration only after it clears this bar.

---

## Scoring the landing page today

| Page section | Honest today? |
|---|---|
| Intro / thesis | Yes (recast as hypothesis, not shipped claim). |
| ccr tool card | Yes. `v`-to-agx integration shipped; Unix-only live-session guard called out. |
| agx tool card | Yes. agx-py/agx-wasm status caveated; `tool_use_id` gap noted; `--jump-to` / `doctor` / `agx-mcp` shipped. |
| sift tool card | Yes. `state` / `export` / `doctor` / `fsck` shipped; agent-first thesis tagged unvalidated. |
| holdout tool card | Yes. One confirmed false-green, not a measured rate; threat model stated. |
| Composition workflow | Yes. Step 4 tags agent reflex-use as unvalidated. |
| Communication surfaces | Yes. Each direction tags its gap. |
| Named integrations | Yes. *Session hand-off* shipped; *Timeline jump* shipped at session level; overlay tagged planned. |
| Shared principles | Yes. Dual-addressed and Versioned schemas caveated; sift&rsquo;s hooks named as the instrumentation exception. |
| Install | Yes. |
| Status | Yes; explicitly links here. |
| Related | Yes. rgx linked without bond-dropping — no integrations pre-announced. |

---

## Next moves (smallest-first)

1. **agx: `tool_use_id` in JSON export** (Gap 2a). Unblocks step-level
   sift integration. Should be a small change in
   `agx-core/src/timeline.rs`. MINOR bump, not breaking.
2. **ccr: `--json` / `--format json` on `list`** (Gap 2, Gap 6). The
   session index is the most obvious machine surface the suite is
   missing. Agents today can&rsquo;t ask ccr "which session has the
   thread about X?" programmatically.
3. **ccr: `docs/agent-guide.md`** (Gap 1). Brings ccr to parity with
   agx / sift for agent discoverability.
4. **All three: `--ai-help` flag** (Gap 1). Prints the agent guide
   to stdout for discovery without a browser.
5. **One downstream consumer** (Gap 3). Pick the smallest of the
   three candidates; ship it; its bugs become the feedback loop that
   tests the stability commitments.
6. ~~**sift Phase 1.7 (`sift state`, `sift export`, `sift accept
   --by-commit`)**~~ — shipped.
7. ~~**agx `--jump-to`**~~ — shipped as `--jump-to <STEP>`; sift
   still needs sub-gap 2a to use it at step level (Gap 4).
8. **Per-tool `doctor`** (Gap 7). agx and sift ship; ccr and holdout
   open.
9. **sift Phase 1.4 dogfood + notes file** (Gap 5). Falsifiable
   signal on whether the agent-as-primary-user thesis holds.
10. **`docs/principals.md`** (Gap 8). Cheap once the command
    surfaces stabilize.
11. **Named integrations, one at a time** (Gap 9). Do not claim any
    integration on the landing page until its PR is merged and a
    smoke test passes. *Session hand-off* (ccr &rarr; agx) is the
    current template.

---

## Rule for future updates to `index.html`

> The landing page is not allowed to describe a capability as shipped
> unless a row in this document places its gap as *closed*. When a
> gap closes, the landing page can be updated in the same commit —
> not before.
>
> Related tools (currently: rgx) are linked in the "Related" footer
> without bond-dropping. No integration between a related tool and a
> suite member is announced on the page before its PR merges.

This rule is the discipline that keeps the page from drifting back
into marketing.
