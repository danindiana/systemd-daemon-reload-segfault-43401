# systemd #43401 — investigation diagrams

Five Graphviz diagrams explaining the investigation behind
[systemd/systemd#43401](https://github.com/systemd/systemd/issues/43401):
a PID 1 SIGSEGV in `cescape()`/`serialize_item_escaped()` during
`daemon-reload` on Ubuntu 22.04 (systemd 249), and the follow-up work that
walked back an early hypothesis about the cause.

## 1. Investigation timeline
Full chronology from the corrupted `Environment="PATH=..."` line
appearing on Aug 12, through four harmless reloads, the crash on Aug 15,
and the Aug 16 fix + reproducer work.

![timeline](diagrams/01_investigation_timeline.png)

## 2. daemon-reload internals: where the crash actually happened
`daemon-reload` serializes currently-running state *before* re-parsing
unit files from disk. The crash happened in the serialize step — the
parse-rejection log line that appears on every other reload is absent on
the crashing one, meaning it never reached the re-parse stage.

![reload internals](diagrams/02_reload_internals_crash_point.png)

## 3. The corrupted `PATH=` line, before/after
What the malformed `Environment=` directive actually contained (an
embedded `npm` error message, raw newlines, an unescaped quote), its
likely origin, and the fix applied.

![before/after](diagrams/03_path_corruption_before_after.png)

## 4. Reproducer methodology
An isolated systemd-255 sandbox (Docker container, own PID 1, host never
touched) used to test the same malformed input on a current release.
Result: no crash, identical parser behavior to the real v249 incident.

![reproducer](diagrams/04_reproducer_methodology.png)

## 5. Case status / evidence map
What's confirmed, what's been ruled out or weakened, and what's still an
open question.

![evidence map](diagrams/05_evidence_map_conclusion.png)

---

Source `.dot` files and `.svg` renders are in [`diagrams/`](diagrams/).
Rendered with Graphviz `dot -Tpng` / `dot -Tsvg`.
