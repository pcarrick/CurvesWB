# CLAUDE.md

Notes for Claude about this project. Keep this short and current. Active work and history live in `ToDo.md`.

## What this is
CurvesWB ("Curves workbench", v0.6.82): a FreeCAD add-on for NURBS curves and surfaces, written in Python.
Upstream: https://github.com/tomate44/CurvesWB (author: Christophe Grellier). The user is fixing bugs locally.

## Environment
- Debian 13, FreeCAD 1.0.0 (Debian package), pivy 0.6.10 (python3-pivy).
- Work happens in `~/.local/share/FreeCAD/Mod/Curves`, the folder FreeCAD loads the add-on from, so edits are live after a FreeCAD restart.
- Git setup: work on branch `pc-edits` (based on `c08e21b`). Remotes: `origin` = the user's fork `git@github.com:pcarrick/CurvesWB.git`
  (SSH; `gh` is logged in as pcarrick), `upstream` = tomate44/CurvesWB. `main` mirrors upstream; never edit it.
  - Get upstream updates: `git fetch upstream`, then `git merge upstream/main` on `pc-edits`. Resolve conflicts with the user.
  - See all local changes: `git diff upstream/main`.
  - Don't commit or push unless the user asks. Don't update Curves via the Addon Manager (it could reset the folder).
- `.gitignore` ignores `*.md`, so `CLAUDE.md` and `ToDo.md` are not tracked by git.
- An old, unused copy is at `/home/data/9C33-6BBD/aaa_software/GitHub/CurvesWB-main` (not a git repo). Ignore it.

## Layout
- `package.xml`: add-on metadata.
- `freecad/Curves/init_gui.py`: workbench registration; imports every tool module.
- `freecad/Curves/*.py`: one module per tool, mostly `*FP.py` = FeaturePython objects (App proxy + ViewProvider + Gui command).
- `freecad/Curves/lib/`: newer shared geometry helpers.
- `freecad/Curves/resources/icons/`: icons. `TestFiles/`: sample .FCStd files.

## Editable Spline (current focus)
The user creates a B-spline with the Editable Spline tool, then drags its points in the 3D view.
- `gordon_profile_FP.py`: the FeaturePython object. `setEdit` builds the editor, `unsetEdit` writes points back to the object's properties.
- `profile_editor.py`: `InterpoCurveEditor` (markers, lines, keyboard shortcuts, rebuilds the curve while dragging) and `MarkerOnShape`
  (a point that can snap to a support shape).
- `graphics.py`: generic pivy/Coin interaction layer. `Object3D` and its subclasses are the drawn items, and `InteractionSeparator` handles events:
  - `register()` adds the hover-highlight, select, grab, delete and select-all callbacks.
  - `grabCB`: left-button press on a selected point → `unregister()` and add `dragCB` for all events.
  - `dragCB`: mouse moves drag the point. Left-button release (or Esc) ends the drag and calls `register()` again.
- Also built on `graphics.py`: `splitCurves_2.py`, `ParametricBlendCurve.py` (with `manipulators.py`). Changes to `graphics.py` affect them too.

## How we work
- One change at a time. After each one the user tests in FreeCAD, then we log the result in `ToDo.md` before moving on.
- Keep changes small. Don't refactor unrelated code without asking.
- To test: restart FreeCAD (Python modules are cached), open the Curves workbench, create or edit an Editable Spline, and watch the
  Report view (View → Panels → Report view) for errors.
