# ToDo

## Current issue: Editable Spline points won't let go after dragging
Clicking a point and moving it works, but releasing the mouse button doesn't drop the point. It keeps following the cursor.
The drag only ends when `dragCB` in `freecad/Curves/graphics.py` sees a left-button release, so that release is probably not getting through.

### Suspected causes
1. `dragCB` checks event types with `type(event) == coin.SoMouseButtonEvent`, which may fail with pivy 0.6.10.
2. The plugin never marks the press as handled, so FreeCAD's navigation style may treat it as its own gesture and keep the release.
3. Esc can't cancel a drag: `getKey()` (a number) is compared to a text character (`graphics.py`, "abort_grab").

### Steps (one at a time; the user tests after each)
- [x] 0. Decide how edits reach FreeCAD: we now work directly in `~/.local/share/FreeCAD/Mod/Curves`.
- [ ] 1. Use `event.isOfType(...getClassTypeId())` instead of `type(event) == ...` in `dragCB`.
- [ ] 2. Call `event_callback.setHandled()` in `grabCB` when a drag starts, and in `dragCB` while dragging.
- [ ] 3. Safety net: on mouse move, end the drag if Qt says the left button is no longer held.
- [ ] 4. Fix Esc so it cancels the drag (compare against `coin.SoKeyboardEvent.ESCAPE`).
- [ ] 5. Cleanups in `graphics.py` (optional):
  - `select_all_cb` calls methods that don't exist (`ColorSelected`, `selection_changed`).
  - `getPointOnScreen` uses `SbVec3f` without the `coin.` prefix.
  - `Arrow` uses `np` without importing numpy.

### Test checklist
- Drag a point and release it: the point stays put and the curve updates.
- Press Esc during a drag: the point goes back and the drag ends.
- Check the Report view for errors.
- Optional: repeat with another navigation style (Edit → Preferences → Display → Navigation).

## Log
- 2026-10-17: Investigated the drag bug and made the plan above. Created CLAUDE.md and ToDo.md. No code changed yet.
- 2026-10-17: Moved the work to `~/.local/share/FreeCAD/Mod/Curves` (git clone, clean at `c08e21b`). Next: step 1.
- 2026-10-17: Git setup: fork pcarrick/CurvesWB is now `origin`, tomate44 is `upstream`. Created branch `pc-edits` for all our work. Next: step 1.
