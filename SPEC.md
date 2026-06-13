# Little Chords — Project Specification

A practice and exploration tool for the sixth/diminished ("Barry Harris") approach to jazz
harmony, bridged to chord-scale theory. Personal-use web app. React + TypeScript + Vite + Vitest.

Sources informing the theory layer (owner's personal study copies, concepts paraphrased):
- "Little Chords" / mini-cadence chapter (slash-chord mappings, home/away wheel, bridge formula)
- David Berkman, *The Jazz Harmony Book*, ch. 9 (pair concept, conversions, borrowing, bebop scales)
- Barry Harris pedagogy (6th-diminished scales, brothers, important minors)

## Core model: the Pair

The central domain object is the **Pair**: two interleaved 4-note pitch-class sets.

```
Pair = {
  chordSet:  4 pitch classes   // e.g. C6  = {C E G A}   — degrees 1 3 5 6
  dimSet:    4 pitch classes   // e.g. B°7 = {B D F A♭}  — degrees 7 2 4 ♯5
  scale:     8 notes, interleaved (the "rail")
  kind:      "major6" | "minor6" | "dim-dim"
}
```

Principles:
1. The union of the two sets is an 8-note bebop scale (major bebop = major + ♯5;
   melodic-minor bebop = same with ♭3; dim-dim = a diminished scale).
2. Harmonizing the rail in diatonic 4-note chords yields ONLY the two chords of the
   pair, in alternating inversions (8 stations).
3. The dimSet functions as a dominant: it is the V7♭9 (rootless) of the chordSet's key.
   Walking the rail = alternating tonic/dominant. Motion is built into the harmony.
4. dim-set notes are "borrowed notes" with resolution tendency (down/up a step into the
   chord set). There are no 9ths/11ths/maj7ths in this system — only borrowed notes.
5. Every conventional chord symbol converts to a Pair over an (implied or sounded) bass.
   The conversion IS the chord-scale choice. See `fixtures/pairs.json`.

## Modules

### `src/theory/` — pure functions, no UI imports
- `pitch.ts` — pitch classes, intervals, MIDI helpers.
- `spelling.ts` — key-aware enharmonic spelling (letter+accidental model, not lookup tables).
- `parse.ts` — chord-symbol parser (port + extend the v1 regex grammar; output typed ChordSymbol).
- `pairs.ts` — chord symbol → Pair conversions with chord-scale annotation
  (ground truth: `fixtures/pairs.json`).
- `identities.ts` — inversion identities (X6=vim7, Xm6=vim7♭5, Xm7=♭III6, Xm7♭5=♭IIIm6;
  ground truth: `fixtures/identities.json`).
- `families.ts` — the three diminished families: any dim7 ↔ its 4 brother dominants
  (lower one note) and 4 sibling m6s = important minors (raise one note).
  Powers tritone/minor-3rd substitution suggestions (ground truth: `fixtures/families.json`).
- `miniCadence.ts` — home/away wheel generator: given a key + duration, propose
  little-chord motion over static harmony (ground truth: `fixtures/mini_cadences.json`).

### `src/voicing/` — concrete notes with octaves
- `voicing.ts` — Voicing = { midi: number[]; inversion: 0..3; position: "close" | "drop2" }.
  Generate all inversions; drop2 = take close voicing, drop the 2nd-from-top voice one octave.
- `rail.ts` — the 8 harmonized stations of a Pair, in close or drop2, ascending from any start.
- `borrow.ts` — suspension generator: replace any subset of voices in a station with the
  step-adjacent notes of the partner set; emit { suspended, resolved } voicing pairs.
  Resolution is optional (modern sound = leave unresolved).
- `voiceLeading.ts` — between two chords in a progression, choose the station/inversion of
  the target's rail nearest (min total semitone motion, no voice crossing preferred)
  to the current voicing.

### `src/components/`
- `Keyboard` — 2–3 octaves, real voicings (not pitch classes), RH/LH coloring.
- `RailStrip` — step through the 8 stations of a pair; close/drop2 toggle; borrow overlay.
- `SuggestionCard` — per progression chord: pair options grouped by chord-scale color,
  with implied symbol, slash notation (e.g. E♭6/C), and family substitutions for dominants.
- `CadenceWheel` — interactive home/away diagram; tapping arcs emits little-chord sequences.
- `ProgressionBar`, key selector, presets/builder/free input (port from v1).

### `src/audio/` (phase 5)
- Tone.js playback of voicings, rails, and generated cadences.

## Phases (each = green fixtures + demo)
1. **Theory core**: pitch/spelling/parse/pairs/identities/families. All fixture tests pass.
2. **Voicing**: inversions, drop2, rails, borrowing. Snapshot tests against fixtures/rails.json.
3. **Voice-leading + suggestions**: progression → voiced realization; family substitution panel.
4. **UI**: keyboard, rail strip, suggestion cards, cadence wheel.
5. **Audio + melody harmonizer** (stretch): mechanical rule — classify melody note into
   chordSet/dimSet, harmonize with that set's chord; borrowing adds variants.

## Non-goals (v2)
- Notation rendering (consider later; keyboard view first).
- Mobile-native; responsive web is enough.
- Audio input / transcription.

## Design language
Keep the v1 aesthetic: warm dark palette, DM Sans/Fraunces, amber RH / blue LH accents.
