# NVIDIA EDID Creator

`edid_creator.html` is a single-file app. Open it in any browser, no install or network needed.

1. Enter Width, Height and Refresh Rate, or switch to SMPTE and pick a standard format.
2. Choose CVT, CVT RB, CVT RB v2 or SMPTE, a colour space, interface and bit depth.
3. Read the actual refresh rate the timing produces. If it is not exact, pick one of the slightly larger sizes listed that lands on the rate exactly.
4. Press Export EDID to save a text file (16 hex bytes per line, 128 bytes per block).

Timings were checked against `edid-decode --cvt`, and the generated EDIDs pass `edid-decode --check`. They have not been tested on hardware yet. See `MD Files from Claude/` for the background notes.
