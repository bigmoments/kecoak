# Changelog

All notable changes to this skill are documented here. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [Unreleased]

## [1.0.0] - 2026-10-16

### Added
- Initial release (renamed from `tuntas` → `kecoak` before release, per user request
  for a quirkier name). Phase 0 classification (Bug Rusak vs Security Hardening, with
  misclassification traps). Track A: permanent-fix loop (reproduce first, root cause,
  sibling sweep, blast radius, minimal fix, regression test, verify, record).
  Track B: adversarial security discipline (threat model, fix at trust boundary,
  no symptom-treating, codebase-wide pattern sweep, adversarial verification,
  safe recording). **Bugraph**: graph visualization of cause chain + blast radius
  (+ attack path for Track B) via HTML widget. **Do & Don't** table.
  Merged mp-diagnosing-bugs-style diagnosis into Track A (tight feedback loop,
  ranked falsifiable hypotheses, targeted instrumentation, tagged debug logs,
  cleanup) — kecoak is now fully standalone, no longer referencing mp-diagnosing-bugs.
  Bugraph: output ganda otomatis (widget chat + file HTML di ~/workspace/bugraph/),
  terikat ke fase A2/A4/A6/B4-B5 (wajib, bukan opsional); aturan hemat token
  (render sekali, update delta, collapse cabang bersih).
  Relations with vulture, app-optimizer (optional only).
