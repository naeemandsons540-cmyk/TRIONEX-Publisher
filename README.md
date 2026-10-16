# TRIONEX Publisher â€” v0.1.0 (Phase 1 foundation)

C++20 / Qt 6 / CMake desktop publishing application. This is the **Phase 1 foundation** of the PRD,
not the full product.

## Status (honest)

This source tree has **not been compiled or run by its author** (it was written in a Linux sandbox without
Qt or MSVC). The first CI run may reveal compile errors; fix them before trusting any result.
No `TRIONEXPublisher.exe` exists until the GitHub Actions workflow (or your local build) produces one,
and it is not verified until you run the checklist below on a real Windows machine.

### Implemented
- New document dialog (size presets, units mm/cm/in/pt, orientation, pages, margins, bleed, columns, facing flag) with validation
- Multi-page documents: add / delete page, page list navigation
- Rectangle, ellipse, line tools; select (click, rubber band, shift/ctrl multi-select); move; resize with handles; keyboard nudge; delete
- Properties panel (position, size, fill, stroke, stroke width) in the document's units
- Bring to front / send to back; lock / unlock (enforced by the engine, not only visually)
- Undo / redo for every document mutation (command pattern, macros for multi-object edits)
- Zoom (Ctrl+wheel, menu), pan (scroll, middle-drag), fit page; margin / column / bleed guides
- `.txpub` save / save as / open: ZIP + JSON, SHA-256 integrity check, zip-bomb and path-traversal protection, atomic replace (`QSaveFile`)
- Autosave to a separate recovery file + recovery prompt after an unclean exit (never overwrites your saved file)
- Basic **vector PDF export** (all pages, real page size) through `QPdfWriter`
- Automated tests (CTest): model, validation, commands, package round-trip/corruption/traversal, PDF smoke test

### Not implemented yet (do not rely on these)
Text frames and typography (incl. Urdu/Arabic), image import, layers panel UI (the model supports layers),
styles, threaded text, parent pages, guides/snapping/rulers, align/distribute, group/ungroup,
preflight, bleed/crop marks, CMYK/ICC, PDF/X.

## Build (Windows)
Requirements: Visual Studio 2022 (MSVC, "Desktop development with C++"), Qt 6.5+ (MSVC 2019/2022 64-bit kit), CMake 3.21+, internet access for the first configure (fetches miniz 3.0.2).

```
cmake -S . -B build -G "Visual Studio 17 2022" -A x64 -DCMAKE_PREFIX_PATH="C:/Qt/6.7.3/msvc2019_64"
cmake --build build --config Release
ctest --test-dir build -C Release --output-on-failure
C:\Qt\6.7.3\msvc2019_64\bin\windeployqt.exe --release build\bin\Release\TRIONEXPublisher.exe
```
Run `build\bin\Release\TRIONEXPublisher.exe`.

## Build via GitHub Actions (no local toolchain)
1. Create a GitHub repository and push this folder to `main`.
2. Open the **Actions** tab â†’ *build-windows* â†’ the latest run.
3. Download the artifact **TRIONEXPublisher-windows**: `TRIONEXPublisher-portable.zip` (unzip and run `TRIONEXPublisher.exe`), `TRIONEXPublisher-Setup.exe` (installer, built best-effort), and `checksums.txt`.
If the build fails, send me the log and I'll fix it.

## Manual acceptance checklist (run on Windows before calling anything "verified")
1. Launch the app; File â†’ New (3 pages, A4).
2. Draw a rectangle, ellipse and line; move and resize each with the mouse; change fill in the Properties panel.
3. Undo/redo several times; Ctrl+L locks an object (it can no longer be moved or deleted).
4. Save as `test.txpub`; close the app; reopen the file; confirm pages and objects are identical.
5. Export PDF; open it in an independent reader (Acrobat, Edge, SumatraPDF); confirm 3 pages, A4 size, shapes present.
6. Kill the app from Task Manager while a document has unsaved changes (wait >60 s first); relaunch and confirm the recovery prompt.
7. Corrupt a `.txpub` (e.g. truncate it) and confirm opening shows an error, not a crash.

## Licensing
Qt is LGPLv3/GPL/commercial: review before distribution (the CI build links Qt dynamically via `windeployqt`, which is the usual LGPL-friendly arrangement; confirm with your own legal review). miniz is MIT.
