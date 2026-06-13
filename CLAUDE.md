# CLAUDE.md — Little Chords

Practice tool for sixth/diminished (Barry Harris) harmony bridged to chord-scale theory.
Read SPEC.md before starting any phase.

## Commands
- `npm run dev` — Vite dev server
- `npm test` — Vitest (watch off in CI: `npm test -- --run`)
- `npm run typecheck` — `tsc --noEmit`

## Hard rules
1. **Fixtures are ground truth.** Files in `fixtures/` encode verified music theory from the
   owner's study sources. NEVER edit a fixture to make a test pass. If an implementation
   disagrees with a fixture, the implementation is wrong; if you believe a fixture itself is
   wrong, stop and ask.
2. `src/theory/` and `src/voicing/` are pure: no React, no DOM, no side effects.
   Every exported function gets unit tests.
3. Spelling is computed (letter + accidental arithmetic), not hardcoded note-name tables,
   except the 12 pitch-class anchors. A D♯ in B major must never print as E♭.
4. Pitch classes are integers 0–11 (C=0). Concrete voicings are MIDI numbers.
   Never store voicings as pitch classes.
5. TypeScript strict mode. No `any` in `src/theory/` or `src/voicing/`.

## Domain glossary (use these terms in code and comments)
- **little chord** — a simple closed-position 6th, m6, °7 (or 7♯5/7♭5) shape in the RH,
  placed over a bass note to imply a richer chord (e.g. E♭6 over C = Cm7).
- **pair** — a 6th chord plus its neighboring diminished 7th (or dim7 + neighbor dim7);
  two interleaved 4-note sets forming an 8-note bebop scale. The central domain object.
- **rail** — the harmonized 6th-diminished scale: 8 stations alternating the pair's two
  chords through their inversions. Voice-leading moves along rails.
- **chordSet / dimSet** — the two 4-note sides of a pair. dimSet notes are "borrowed
  notes": they want to resolve stepwise into the chordSet (the dimSet is a rootless V7♭9).
- **borrowing** — replacing voices of one side with step-adjacent notes of the other,
  creating suspensions that may resolve or be left unresolved.
- **brothers** — the four dominant 7ths obtained by lowering one note of a dim7
  (a minor-3rd cycle sharing that dim7 as their 3-5-♭7-♭9). Basis of tritone/m3 subs.
- **important minor** — the m6 on the 5th of a dominant (Dm6 ↔ G7 = G9 rootless).
  Raising one note of a dim7 yields the four important minors of its four brothers.
- **inversion identities** — X6 = vim7, Xm6 = vim7♭5, Xm7 = ♭III6, Xm7♭5 = ♭IIIm6
  (same four notes, different bass).
- **mini cadence** — tonic/dominant alternation generated over static harmony using
  little chords: home (I6 or V6/I) ↔ away (IV6/V = V7sus), pushed by passing
  diminisheds (♯i°7, i°7 and inversions, vii°7).
- **drop 2** — from a close voicing, drop the second voice from the top one octave.
- **chord scale annotation** — every pair maps to exactly one chord scale
  (e.g. ♭iim6 pair over a dominant root = altered scale, since ♭ii melodic minor = altered).
  Keep these annotations attached to pair conversions; they are the bridge for users who
  think in chord-scale terms.

## Theory invariants (good property-test targets)
- There are exactly 3 diminished pitch-class sets; families partition all 12 dominants
  and all 12 m6 chords.
- Harmonizing any pair's rail in diatonic 4-note chords yields only the pair's two chords.
- Every m6's dimSet equals the rootless V7♭9 of its tonic.
- drop2(invert(v)) and invert(drop2(v)) agree up to octave normalization.
- Conversions in pairs.json round-trip: realizing the pair over the given bass reproduces
  the chord symbol's tones/tensions.

## Workflow
- One phase per session (see SPEC.md phases). TDD: load the phase's fixtures into Vitest
  first, implement until green, then refactor.
- Conventional commits (`feat(theory): ...`). Commit per module, PR per phase.
