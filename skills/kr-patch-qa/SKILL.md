---
name: kr-patch-qa
description: Use for Korean retro-game localization QA, RC/release validation, runtime-safety regression checks, and requests such as "한글화 QA 스킬 적용해줘", "한글화 QA", "한글 패치 QA", "한글 패치 검수", "배포 전 QA", "RC 검증", or "프리징 회귀 검수". Requests such as "한글화 QA 스킬 설치해줘" refer to installing this plugin.
---

# Korean patch QA

## Purpose

Apply the shared Korean localization QA and runtime-safety standard without duplicating the broader patch-construction methodology owned by `create-kr-patch`.

The normative standard for this plugin is the repository-root document:

- `../../LOCALIZATION_QA_STANDARD.md`

Read the relevant sections before judging a build, QA report, regression, runtime smoke result, canonical promotion, or release package.

## Relationship to create-kr-patch

When `create-kr-patch` is also installed, use both skills together:

- `create-kr-patch` owns investigation, reverse engineering, fonts/encoding, extraction, reinsertion, hooks, graphics/compression strategy, build design, and platform-specific methodology.
- `kr-patch-qa` owns the shared Korean presentation defaults, QA stage vocabulary, known-good regression safeguards, final-image readback expectations, freeze/crash reproduction metadata, canonical promotion, and release gating defined in the common standard.

Do not copy generic methodology into this skill when `create-kr-patch` already owns it. If a project-specific rule is stricter than the common standard, use the stricter project rule.

## Default workflow

1. Identify the exact source revision and last-known-good baseline.
2. Read the project-specific instructions and the relevant sections of `../../LOCALIZATION_QA_STANDARD.md`.
3. Keep source-language QA, static binary QA, RC build, final-image readback, runtime smoke, canonical promotion, patch packaging, and release as separate states.
4. Verify Korean punctuation, particles, spacing, voice/register, terminology, renderer-width line breaks, and glyph coverage against the actual game consumer.
5. Verify changed binary structures at their real consumer boundary: spans, records, pointers, sizes, alignment, compression descriptors, allocation spans, archives, graphics structures, and physical layout as applicable.
6. Re-read the final ROM/disc/archive output and verify changed and protected regions before treating it as an RC.
7. If a freeze/crash appears, preserve last-known-good and first-known-bad identities and isolate text, glyph/font, graphics, archive/compression, and physical integration rather than changing several layers at once.
8. Promote only the exact RC that passed the required runtime smoke path. Build distributable patches only from the canonical result.

## Runtime verification with emucap

When `emucap` is available, use it to obtain runtime evidence for the standard rather than treating static QA as runtime proof.

Typical composition:

1. Establish the exact ROM/disc identity.
2. Start an emucap run for the target build.
3. Reproduce the required gameplay/menu/event path.
4. Record screenshots, state/memory observations, interventions, and gates as needed.
5. Feed the observed result back into the `RUNTIME_SMOKE` and regression records required by the common standard.

`emucap` supplies emulator control and experiment evidence; it does not replace the QA policy in this skill.

## Completion rule

Never report the entire localization as PASS from one narrower check. State the scope of every result explicitly. `STATIC_BINARY_QA PASS` does not imply `RUNTIME_SMOKE PASS`, and an RC does not become canonical until the project's required runtime confirmation has passed on that exact candidate.
