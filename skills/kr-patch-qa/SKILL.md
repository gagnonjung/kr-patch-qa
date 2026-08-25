---
name: kr-patch-qa
description: Static-first Korean retro-game localization QA that works without launching an emulator or playing the game. Use for source QA, binary/layout/glyph/graphics/compression/archive checks, final-image readback, RC/release validation, runtime-safety regression checks, and requests such as "한글화 QA 스킬 적용해줘", "한글화 QA", "한글 패치 QA", "한글 패치 검수", "배포 전 QA", "RC 검증", or "프리징 회귀 검수". emucap/runtime verification is optional and extends static QA when runtime evidence is needed. Requests such as "한글화 QA 스킬 설치해줘" refer to installing this plugin.
---

# Korean patch QA

## Purpose

Apply the shared Korean localization QA and runtime-safety standard without duplicating the broader patch-construction methodology owned by `create-kr-patch`.

The normative standard for this plugin is the repository-root document:

- `../../LOCALIZATION_QA_STANDARD.md`

Read the relevant sections before judging a build, QA report, regression, runtime smoke result, canonical promotion, or release package.

## Static-first operation

This skill does not require an emulator, gameplay automation, or `emucap` to be useful. Without launching the game, it can perform and report `SOURCE_QA`, `STATIC_BINARY_QA`, `RC_BUILD`, and `RC_READBACK_QA` using source files, build reports, binary structures, hashes, renderer constraints, glyph plans, archives, ROM/disc layout, and final-image readback.

If no runtime execution is performed, record `RUNTIME_SMOKE` as `NOT RUN` or `PENDING`. Do not downgrade the completed static checks, but do not reinterpret them as runtime proof or promote a candidate through a runtime-required canonical/release gate.

`emucap` is an optional extension for collecting runtime evidence and reproducing regressions after the static QA path.

## Relationship to create-kr-patch

When `create-kr-patch` is also installed, use both skills together:

- `create-kr-patch` owns investigation, reverse engineering, fonts/encoding, extraction, reinsertion, hooks, graphics/compression strategy, build design, and platform-specific methodology.
- `kr-patch-qa` owns the shared Korean presentation defaults, QA stage vocabulary, known-good regression safeguards, final-image readback expectations, freeze/crash reproduction metadata, canonical promotion, and release gating defined in the common standard.

Do not copy generic methodology into this skill when `create-kr-patch` already owns it. If a project-specific rule is stricter than the common standard, use the stricter project rule.

## Default workflow

1. Identify the exact source revision and last-known-good baseline.
2. Read the project-specific instructions and the relevant sections of `../../LOCALIZATION_QA_STANDARD.md`.
3. Keep source-language QA, static binary QA, RC build, final-image readback, runtime smoke, canonical promotion, patch packaging, and release as separate states.
4. Verify Korean punctuation, particles, spacing, dependent-noun/auxiliary-verb usage, Japanese-translationese naturalness, voice/register, terminology, protected multiword proper nouns, renderer-width line breaks, and glyph coverage against the actual game consumer. Treat naturalness regexes as candidate generators, not blind replacements.
5. Distinguish coverage scope, canonical unique text, physical occurrences, unscanned/excluded categories, translation approval state, and runtime-generated/hardcoded text. Never report a workset/extractor as project-wide 100% without evidence that all user-facing consumer families are in scope.
6. Track speaker/listener evidence provenance. Do not treat metadata injected by a prior normalization rule as independent source evidence, and keep ambiguous repeated lines unconfirmed until direct context resolves them.
7. Verify changed binary structures at their real consumer boundary: spans, records, physical windows, pointers, sizes, decoded/raw size, raw/packed descriptors, allocator-visible sizes, alignment, compression descriptors, allocation spans, archives, graphics structures, and physical layout as applicable.
8. Verify glyph mapping identity as well as missing-glyph count. Prefer free-slot extension over reassigning an active code, and inspect baseline-to-candidate active code→glyph mapping diffs.
9. Re-read the final ROM/disc/archive output and verify changed and protected regions before treating it as an RC. Derived overlays must identify their parent/base hash; if a lower layer changes, reject or rebuild stale overlays and then reapply them in dependency order.
10. For residual Japanese scans, reverse-decode findings with the actual runtime codec/codebook before classifying them. Do not suppress Korean carrier false positives with broad ignore rules.
11. When parallel translation is used, keep canonical integration single-writer: workers produce isolated drafts, and one integration session applies only reviewed/new groups then reruns global QA.
12. If a freeze/crash appears, preserve last-known-good and first-known-bad identities and isolate text, glyph/font, graphics, archive/compression, and physical integration rather than changing several layers at once.
13. For runtime gates, record whether the path came from cold boot, a normal game save, or a save state. Do not use a post-initialization save state as the sole proof for assets that load only during boot/scene/battle initialization.
14. Promote only the exact RC that passed the required runtime smoke path. Build distributable patches only from the canonical result, preferably as a direct supported-retail-to-canonical patch with encode/decode verification.

## Runtime verification with emucap

When `emucap` is available, use it to obtain runtime evidence for the standard rather than treating static QA as runtime proof.

Typical composition:

1. Establish the exact ROM/disc identity.
2. Start an emucap run for the target build.
3. Reproduce the required gameplay/menu/event path, using cold boot when the changed asset is loaded during initialization.
4. Record whether normal saves or save states were used, including the state’s source build and creation point when applicable.
5. Record screenshots, state/memory observations, interventions, and gates as needed.
6. Feed the observed result back into the `RUNTIME_SMOKE` and regression records required by the common standard.

`emucap` supplies emulator control and experiment evidence; it does not replace the QA policy in this skill.

## Completion rule

Never report the entire localization as PASS from one narrower check. State the coverage scope, excluded/unscanned populations, and evidence level of every result explicitly. `10,283/10,283 dialogue groups`, `STATIC_BINARY_QA PASS`, or `raw SJIS residual 0` each prove only their named scope; none implies project-wide or runtime completion. An RC does not become canonical until the project's required runtime confirmation has passed on that exact candidate.
