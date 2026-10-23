# Green Hippo GUI Style Guide

A generic guide for building any tool window, web page or dashboard so it sits
alongside HippoLauncher and ZooKeeper. Dark, flat, grey-on-charcoal, with one
olive-green accent that means "this is the active thing".

> **Colour values are estimated from screenshots** (HippoLauncher and ZooKeeper
> 4.9.6). They are close, not sampled. Before locking them in, eyedrop the
> real app and update the tokens in section 2. Everything else in this guide
> refers to tokens, so only that one table needs correcting.

---

## 1. The look in one paragraph

Near-black window, mid-grey rounded buttons and panels, white text and white
line icons, and a single olive green used for selection, the active tab, the
active bank and anything "on". No gradients, no shadows worth mentioning, no
coloured text. Controls are big and chunky: icon above, short label below,
a tiny keyboard hint in the corner. Everything is usable with a mouse in a
dark venue.

---

## 2. Colour tokens

| Token | Approx. hex | Used for |
|---|---|---|
| `bg` | `#1E1E1E` | Window background, title bar, empty areas |
| `bg-inset` | `#262626` | Group-box interiors, lists, list rows |
| `panel` | `#404040` | Buttons, cards, empty slots, inactive tabs |
| `panel-hover` | `#4D4D4D` | Hover on buttons, cards and slots |
| `panel-raised` | `#666666` | Active top-level tab when it is not green |
| `border` | `#333333` | Group-box outlines, dividers (1px, subtle) |
| `text` | `#FFFFFF` | All primary text and icons |
| `text-muted` | `#A6A6A6` | Column headers, hints, disabled labels, key hints |
| `brand` | `#5C7F1E` | Selected row, active slot frame, bank grid, fader fill, active toggle |
| `brand-bright` | `#8DC63F` | Active bank tab, small "on" pills, logo glyph |
| `brand-dark` | `#455F17` | Pressed state, scrollbar thumbs on green |
| `black` | `#000000` | Media thumbnails, empty clip tiles, number-entry wells |

**Rules**

1. **Grey does the structure, green does the meaning.** If something is not
   selected, active or on, it is grey.
2. **One green at a time per area.** `brand` for large fills (grid, row,
   fader), `brand-bright` for small things (a tab pill, a dot).
3. **White text on green stays white.** Don't switch to dark text on `brand`.
   Row text on `brand` is the one place contrast is tight, so keep the font
   weight at 500 there.
4. **Number wells are white on black or dark on white**, as in the layer
   faders: a light box with dark digits under the fader. Use sparingly.

### Status colours (decision needed)
Hippo's own UI has no red/amber/green traffic lights, because green already
means "active". For tools that need status, use this and always pair with a word:

| Meaning | Colour |
|---|---|
| Healthy / connected | `brand-bright` dot + the word |
| Warning | `#E0A030` dot + the word |
| Error / live-danger | `#C0392B` dot + the word |
| Unknown / offline | `#808080` dot + the word |

Dots and pills only. Never colour body text with these.

---

## 3. Typography

| Property | Value |
|---|---|
| Family | Roboto (the apps use it); fall back to `"Segoe UI", system-ui, sans-serif` |
| Window title | 16px, weight 500, centred in the title bar |
| Top-level tab | 36–40px light/regular, with a 13px version line underneath |
| Section / group title | 14–16px, regular, white, sits on the group outline |
| Button label | 13px, weight 600 (bold-ish), centred under the icon |
| Body / list | 13px, regular |
| Column header | 13px, `text-muted`, with thin vertical dividers between columns |
| Key hint (F1…F7) | 9–10px, `text-muted`, top-right corner of the button |
| Fader value | 11px, tabular figures |

No italics, no underlines, no all-caps except the odd short acronym (NDI, TC, EULA).

---

## 4. Shape, spacing and size

| Property | Value |
|---|---|
| Button / card radius | 6px |
| Slot / tile radius | 6px |
| Group-box radius | 3px |
| Border | 1px, `border`, and only on group boxes |
| Big action button | about 160 × 90, icon 48px on top, label below |
| Small action button | about 125 × 70 (the Help row) |
| Grid slot | about 190 × 105, 8–10px gaps, inside a `brand` frame with 10px padding |
| Gaps between buttons | 10px |
| Gap between groups | 20px |
| Hit target minimum | 32px (the real buttons are far bigger) |

Spacing is generous around action buttons and tight inside dense control
panels (mix controls use 4–6px).

---

## 5. Components

### Window chrome
- Custom title bar in `bg`: title centred as `AppName on HOSTNAME`.
- Three small square buttons top-right: `-` `+` `x`, `panel` fill, 6px radius.
- Optional round logo tile at top-left in `brand-bright`.

### Top-level tabs (HippoLauncher)
Four equal-width tabs across the full width, very large text with the version
beneath. Active tab: `panel-raised`. Inactive: `panel`. A small notch marker
sits at the top edge between some tabs. No underline.

### Icon action button (the signature control)
`panel` fill, 6px radius, white line icon above a bold 13px label. Hover goes
to `panel-hover`. Pressed goes to `brand-dark` or a 2px `brand` outline. Disabled:
icon and label `text-muted`. A keyboard hint (`F1`) sits top-right.

### Group box
A `border` outline with the title sitting on the top edge in white,
`bg` behind the title so it cuts the line. Buttons go inside with 10px padding.
Use for "Current Show", "New Show", "Help" style clusters.

### Table / list
`bg-inset` background, no row borders, 13px rows about 19px tall. Column
headers `text-muted` with thin vertical dividers. Sorted column header gets a
`brand` fill and a small triangle. **Selected row is a full `brand` bar**
with white text. Truncate long text with an ellipsis, never wrap.

### Side rail (ZooKeeper)
A narrow vertical strip, 30px wide, `bg`, holding stacked square icon buttons
(`panel`) inside a rounded `brand` outline. Two clusters: main navigation on
top, a second block of small dot-buttons (recall slots) beneath. Lock and
utility buttons sit at the bottom.

### Tabs inside a panel
Flat `panel` strips. Active tab is lighter or `brand`, inactive darker. Used for
Media / Audio / Sync, Effects / Notch Params / TouchEngine, Media Select / Preset
Select. They are full-width and share the panel edge: no gaps and no rounded
tops.

### Bank grid and tiles
A `brand` rounded frame containing a regular grid of `panel` slots. Empty slot
shows a white `+`. Filled slot shows a thumbnail with a small index number
top-left and the name beneath. The active bank is a `brand-bright` pill above
the grid with a `+` button next to it. Empty media tiles are `black` with a
grey cross.

### Faders and sliders
Vertical channel strip: `panel` track, `brand` fill rising from the bottom,
a thin white line as the handle, a white rounded value box underneath with
dark digits (`0.500`), label below. Horizontal transport scrub bar: `brand`
fill with white end caps.

### Toggles and switches
Small pill switch. Off: `panel`, white knob left. On: `brand`, knob right.
Label sits beside it, never inside it.

### Dropdowns and spin boxes
`panel` fill, white text, small chevron on the right (dropdown) or up/down
arrows (spin). Spin boxes sit in a `panel` pill with the unit label outside
(`Level`, `Time`). **Every number box must accept typed digits** (see 8).

### Transport and large icon button rows
Equal-width `panel` buttons with white glyphs (play, pause, loop, skip).
The active mode gets a 2px `brand-bright` outline, not a fill change.

---

## 6. Layout patterns

1. **Landing screen**: list at the top, grouped action buttons centred below,
   help row at the bottom. Centre the clusters; don't left-align them.
2. **Navigation screen**: a single row of big icon buttons with `F1…F7`
   hints, content beneath.
3. **Control surface**: dense panel of mini-groups (media info, transport,
   geometry, colour, effects) separated by 2–4px gaps on a slightly lighter
   `bg`; the content grid fills the remaining height.
4. **Window background outside the main panel** may show a faint repeated
   `panel` tile pattern at very low contrast. Optional.

---

## 7. Writing style (labels and messages)

- Short, title-case labels: `Start Engine Only`, `Create New Show`, `Import Show`.
- Name the thing, not the variable. Dropdowns show names; numbers stay on the wire.
- Errors say what to do next. No exclamation marks, no blame.
- British English.
- Show host and version in the title bar or tab subtitle.

---

## 8. Do and don't

**Do**
- Keep icons white, single-weight line art, about 2px stroke.
- Keep hover/press states to a one-step grey change.
- Put the active state in green, always.
- Make every spin box typeable as the user types (Qt: `setKeyboardTracking(True)`;
  web: listen to `input`, not only `change`).
- Use inline status text, not modals, for routine feedback. Modals are for
  destructive confirmation only.

**Don't**
- Don't add coloured text, gradients, drop shadows or glows.
- Don't use green for anything that isn't active, selected or on.
- Don't use more than two fonts or any web-font CDN.
- Don't put text under 11px except key hints.
- Don't rely on colour alone: pair status with a word or icon.

---

## 9. Starter code

### CSS tokens
```css
:root {
  --bg: #1E1E1E;       --bg-inset: #262626;
  --panel: #404040;    --panel-hover: #4D4D4D;  --panel-raised: #666666;
  --border: #333333;
  --text: #FFFFFF;     --text-muted: #A6A6A6;
  --brand: #5C7F1E;    --brand-bright: #8DC63F; --brand-dark: #455F17;
  --radius: 6px;
  font-family: Roboto, "Segoe UI", system-ui, sans-serif;
  font-size: 13px;
}
body { background: var(--bg); color: var(--text); }
.btn { background: var(--panel); border: 0; border-radius: var(--radius);
       color: var(--text); font-weight: 600; padding: 10px 14px; min-height: 32px; }
.btn:hover { background: var(--panel-hover); }
.btn:active, .btn.on { background: var(--brand-dark); outline: 2px solid var(--brand); }
.group { border: 1px solid var(--border); border-radius: 3px; padding: 10px; }
.row-selected { background: var(--brand); color: #fff; }
```

### Qt (PyQt6 / PySide6) stylesheet
```python
HIPPO_QSS = """
* { font-family: Roboto, 'Segoe UI'; font-size: 13px; color: #FFFFFF; }
QWidget { background: #1E1E1E; }
QPushButton { background: #404040; border: none; border-radius: 6px;
              padding: 10px 14px; font-weight: 600; min-height: 32px; }
QPushButton:hover { background: #4D4D4D; }
QPushButton:pressed, QPushButton:checked { background: #455F17; border: 2px solid #5C7F1E; }
QPushButton:disabled { color: #A6A6A6; }
QGroupBox { border: 1px solid #333333; border-radius: 3px; margin-top: 10px; padding: 10px; }
QGroupBox::title { subcontrol-origin: margin; left: 12px; padding: 0 4px; }
QTabBar::tab { background: #404040; padding: 10px 24px; }
QTabBar::tab:selected { background: #666666; }
QHeaderView::section { background: #262626; color: #A6A6A6; border: 0;
                       border-right: 1px solid #333333; padding: 3px 6px; }
QTableView, QListView { background: #262626; border: 0; }
QTableView::item:selected, QListView::item:selected { background: #5C7F1E; color: #FFFFFF; }
QComboBox, QSpinBox, QLineEdit { background: #404040; border: 0; border-radius: 6px; padding: 4px 8px; }
QLineEdit:focus, QComboBox:focus, QSpinBox:focus { border: 2px solid #8DC63F; }
"""
```

---

## 10. Checklist

- [ ] Colours come from the tokens; no stray hexes
- [ ] Grey for structure, green only for active/selected/on
- [ ] Icon buttons: icon above, bold label below, key hint top-right if bound
- [ ] Selected row is a full green bar
- [ ] Group boxes carry their title on the outline
- [ ] Number inputs accept typing
- [ ] Status shown as a dot plus a word, never coloured text
- [ ] No modals for routine feedback
- [ ] Colour tokens re-checked against the real app with an eyedropper
