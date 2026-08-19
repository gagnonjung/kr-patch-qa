# kr-patch-qa

Common Korean localization QA and runtime-safety standards for retro-game patch projects.

## Purpose

This repository provides a shared QA baseline for Korean game localization projects, with emphasis on both language quality and runtime safety.

The standard covers areas such as:

- Korean punctuation, spacing, particles, register, terminology, and line breaks
- actual renderer-width and glyph-coverage verification
- fixed-span, pointer, compression, archive, and physical-layout safety
- graphics palette, CLUT, texture-structure, and protected-region preservation
- freeze/crash isolation and last-known-good regression tracking
- deterministic build/readback verification
- RC, runtime smoke, canonical promotion, patch packaging, and release stages

## Standard

The normative document is:

- [`LOCALIZATION_QA_STANDARD.md`](LOCALIZATION_QA_STANDARD.md)

Project-specific rules may be stricter. When they are, the stricter rule takes precedence.

## Relationship to create-kr-patch

`kr-patch-qa` is the shared QA/runtime-safety standard. It is intentionally separate from the broader `create-kr-patch` methodology, which covers the full retro-game Korean patch workflow from reverse engineering through build and verification.

Rules that are already owned by `create-kr-patch` should not be duplicated here unless this repository defines a stricter Korean-localization-specific requirement.

## Status

The standard is intended to evolve from issues found during real localization projects. New common rules should be promoted only when they are reusable across projects and are not already covered by the upstream methodology.
