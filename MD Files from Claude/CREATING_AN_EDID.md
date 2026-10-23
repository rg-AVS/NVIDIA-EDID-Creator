# Creating a Custom EDID

How to build an EDID by hand for a custom resolution and refresh rate, using 5120x2160 @ 50 Hz as the worked example. It covers the byte layout, the timing maths, the DisplayID extension needed for widths over 4095, checksums, and the text format used for Hippotizer-style EDID files.

> Status: the example EDID has been checksum- and timing-verified in Python but not yet tested on hardware. Validate with `edid-decode` before relying on it.

---

## 1. What an EDID is

An EDID (Extended Display Identification Data) is a small binary blob a display (or EDID emulator) sends to a source over DDC. It tells the source the native timings, supported modes, colour capabilities and identity.

- Base block: always 128 bytes.
- Extension blocks: optional, 128 bytes each (CTA-861, DisplayID, etc.).
- Every 128-byte block ends with a checksum byte so the block's bytes sum to `0 mod 256`.

---

## 2. Base block (128 bytes)

| Bytes | Content | Notes |
|---|---|---|
| 0-7 | Header | `00 FF FF FF FF FF FF 00` |
| 8-9 | Manufacturer ID | 3 letters, 5 bits each (A=1), big-endian. `AVS` = `06 D3` |
| 10-11 | Product code | Little-endian |
| 12-15 | Serial number | Little-endian, may be 0 or 1 |
| 16 | Week of manufacture | |
| 17 | Year of manufacture | Year minus 1990 |
| 18-19 | EDID version | `01 04` = 1.4 |
| 20 | Video input | `A5` = digital, 8 bpc, DisplayPort. Bits 6-4: depth (`010` = 8 bpc). Bits 3-0: interface (`5` = DP, `2` = HDMI-a) |
| 21-22 | Screen size | cm (width, height) |
| 23 | Gamma | `(gamma x 100) - 100`; 2.2 = `120` (`0x78`) |
| 24 | Feature support | `0A`: bit 1 = preferred timing is DTD 1; bits 4-3 = `01` -> RGB 4:4:4 + YCbCr 4:4:4 |
| 25-34 | Chromaticity | sRGB: `EE 91 A3 54 4C 99 26 0F 50 54` |
| 35-37 | Established timings | Usually 0 |
| 38-53 | Standard timings | 8 x 2 bytes; unused = `01 01` |
| 54-71 | Descriptor 1 | Detailed Timing Descriptor (DTD), the preferred mode |
| 72-89 | Descriptor 2 | Usually range limits (`FD`) |
| 90-107 | Descriptor 3 | Usually monitor name (`FC`) |
| 108-125 | Descriptor 4 | Dummy (`10`) or second DTD |
| 126 | Extension count | Number of 128-byte blocks that follow |
| 127 | Checksum | `(256 - sum(bytes 0..126)) mod 256` |

### Non-timing descriptors (18 bytes)

`00 00 00 <tag> 00 <13 bytes data>`

| Tag | Meaning |
|---|---|
| `FD` | Range limits: min V, max V, min H kHz, max H kHz, max pixel clock in 10 MHz units |
| `FC` | Monitor name, ASCII, up to 13 chars, ends with `0A` and space padding |
| `FF` | Monitor serial string |
| `10` | Dummy |

---

## 3. Working out the timing

You need these for the mode:

- Active: `Ha x Va`
- Horizontal blanking `Hb`, front porch `Hfp`, sync width `Hsw` (back porch = `Hb - Hfp - Hsw`)
- Vertical blanking `Vb`, front porch `Vfp`, sync width `Vsw`
- Refresh rate `F`

```
Htotal = Ha + Hb
Vtotal = Va + Vb
Pixel clock = Htotal x Vtotal x F
H frequency = Pixel clock / Htotal
```

The EDID stores the pixel clock in 10 kHz units, so for an exact refresh rate `Htotal x Vtotal x F` must divide by 10,000. For F = 50 that means `Htotal x Vtotal` must be a multiple of 200. Adjust the blanking by a few lines or pixels until it is.

### Worked example: 5120x2160 @ 50 Hz (CVT-RB style)

| Field | Value |
|---|---|
| Ha / Hb | 5120 / 160 (Hfp 48, Hsw 32, back porch 80) |
| Va / Vb | 2160 / 55 (Vfp 3, Vsw 10, back porch 42) |
| Htotal x Vtotal | 5280 x 2215 = 11,695,200 |
| Pixel clock | 11,695,200 x 50 = 584.76 MHz -> stored as `58476` |
| H frequency | 110.75 kHz |
| Refresh | 50.0000 Hz |

Bandwidth check: 584.76 MHz is under HDMI 2.0's 600 MHz TMDS limit at 8-bit RGB, and well within DisplayPort. At 10-bit or higher HDMI will not carry it.

Fallback in the base block: 3840x2160 @ 50 (Htotal 4000, Vtotal 2205, 441.00 MHz).

---

## 4. Detailed Timing Descriptor (18 bytes)

The base-block DTD stores active width in 12 bits, so **the maximum is 4095 pixels**. 5120 does not fit; use the DisplayID extension (section 5) for the wide mode.

| Byte | Content |
|---|---|
| 0-1 | Pixel clock, 10 kHz units, little-endian |
| 2 | Ha low 8 bits |
| 3 | Hb low 8 bits |
| 4 | Ha high 4 bits (upper nibble), Hb high 4 bits (lower nibble) |
| 5 | Va low 8 bits |
| 6 | Vb low 8 bits |
| 7 | Va high 4 bits (upper), Vb high 4 bits (lower) |
| 8 | Hfp low 8 bits |
| 9 | Hsw low 8 bits |
| 10 | Vfp low 4 bits (upper nibble), Vsw low 4 bits (lower nibble) |
| 11 | Hfp bits 9-8 (bits 7-6), Hsw bits 9-8 (bits 5-4), Vfp bits 5-4 (bits 3-2), Vsw bits 5-4 (bits 1-0) |
| 12 | Image width mm, low 8 bits |
| 13 | Image height mm, low 8 bits |
| 14 | Width mm high 4 bits (upper), height mm high 4 bits (lower) |
| 15 | H border |
| 16 | V border |
| 17 | Flags: `1A` = digital separate sync, +H, +V, non-interlaced |

---

## 5. DisplayID 1.3 extension (for widths over 4095)

Extension block layout:

| Byte | Content |
|---|---|
| 0 | `70` (DisplayID extension tag) |
| 1 | DisplayID version/revision, `13` = 1.3 |
| 2 | Section payload length (bytes of data blocks) |
| 3 | Product type (`00` = extension) |
| 4 | Extension count (`00`) |
| 5... | Data blocks |
| after data | DisplayID checksum: sum of bytes 1..N including itself = 0 mod 256 |
| 127 | EDID block checksum |

### Type I Detailed Timing data block (tag `03`)

Header: `03 00 <payload length>`; each timing is 20 bytes. All "-1" values are stored as value minus 1.

| Bytes | Content |
|---|---|
| 0-2 | Pixel clock (10 kHz units) minus 1, 24-bit little-endian |
| 3 | Flags: bit 7 = preferred; bits 2-0 = aspect (`6` = 64:27, `4` = 16:9, `1` = 5:4, `2` = 4:3, `5` = 16:10, `7` = 256:135) |
| 4-5 | Ha - 1 |
| 6-7 | Hb - 1 |
| 8-9 | Hfp - 1 (bits 14-0); bit 15 = H sync polarity (1 = positive) |
| 10-11 | Hsw - 1 |
| 12-13 | Va - 1 |
| 14-15 | Vb - 1 |
| 16-17 | Vfp - 1 (bits 14-0); bit 15 = V sync polarity (1 = positive) |
| 18-19 | Vsw - 1 |

Remember to set byte 126 of the base block to `01` so readers look for the extension.

---

## 6. Checksums

Base and extension blocks each:

```
byte[127] = (256 - (sum(byte[0:127]) % 256)) % 256
```

DisplayID has its own checksum covering bytes 1 to the last data byte of the DisplayID structure (not byte 0, and not the EDID block checksum).

Recalculate both whenever any byte changes.

---

## 7. Text format

Hippotizer-style EDID text files are 16 hex bytes per line, uppercase, space-separated, trailing space, 16 lines for 256 bytes (8 lines per 128-byte block):

```
00 FF FF FF FF FF FF 00 06 D3 20 51 01 00 00 00 
2A 24 01 04 A5 50 21 78 0A EE 91 A3 54 4C 99 26 
...
```

---

## 8. Generator script

Python 3, no dependencies. Run: `python3 mk_edid.py EDID_5120x2160_50Hz.bin`, then convert to text format as in section 7.

```python
import struct, sys

OUT = sys.argv[1]

# 5120x2160 @ 50.000 Hz, CVT-RB style. Total 5280 x 2215, 584.76 MHz
T5 = dict(ha=5120, hb=160, hfp=48, hsw=32, va=2160, vb=55, vfp=3, vsw=10, pclk=58476)
# 3840x2160 @ 50.000 Hz fallback for base block. Total 4000 x 2205, 441.00 MHz
T4 = dict(ha=3840, hb=160, hfp=48, hsw=32, va=2160, vb=45, vfp=3, vsw=5, pclk=44100)

WMM, HMM = 800, 337  # approx 34" 21:9 panel

def cs(block):
    block[-1] = (256 - sum(block[:-1])) % 256

def dtd(t):
    d = bytearray(18)
    d[0:2] = struct.pack('<H', t['pclk'])
    ha, hb, va, vb = t['ha'], t['hb'], t['va'], t['vb']
    d[2] = ha & 255; d[3] = hb & 255; d[4] = ((ha >> 8) << 4) | (hb >> 8)
    d[5] = va & 255; d[6] = vb & 255; d[7] = ((va >> 8) << 4) | (vb >> 8)
    hfp, hsw, vfp, vsw = t['hfp'], t['hsw'], t['vfp'], t['vsw']
    d[8] = hfp & 255; d[9] = hsw & 255
    d[10] = ((vfp & 15) << 4) | (vsw & 15)
    d[11] = ((hfp >> 8) << 6) | ((hsw >> 8) << 4) | ((vfp >> 4) << 2) | (vsw >> 4)
    d[12] = WMM & 255; d[13] = HMM & 255; d[14] = ((WMM >> 8) << 4) | (HMM >> 8)
    d[17] = 0x1A  # digital separate sync, +H +V
    return d

# ---- Base block ----
base = bytearray(128)
base[0:8] = bytes([0, 255, 255, 255, 255, 255, 255, 0])
mfg = (ord('A') - 64) << 10 | (ord('V') - 64) << 5 | (ord('S') - 64)  # "AVS"
base[8:10] = struct.pack('>H', mfg)
base[10:12] = struct.pack('<H', 0x5120)
base[12:16] = struct.pack('<I', 1)
base[16] = 42; base[17] = 2026 - 1990
base[18] = 1; base[19] = 4           # EDID 1.4
base[20] = 0xA5                      # digital, 8 bpc, DisplayPort
base[21] = WMM // 10; base[22] = HMM // 10
base[23] = 120                       # gamma 2.2
base[24] = 0x0A                      # RGB + YCbCr 4:4:4, DTD1 = preferred
base[25:35] = bytes([0xEE, 0x91, 0xA3, 0x54, 0x4C, 0x99, 0x26, 0x0F, 0x50, 0x54])
base[38:54] = bytes([1, 1] * 8)      # no standard timings
base[54:72] = dtd(T4)
r = bytearray(18); r[3] = 0xFD
r[5:10] = bytes([24, 60, 30, 120, 60])  # V 24-60 Hz, H 30-120 kHz, max 600 MHz
r[10] = 0; r[11:18] = bytes([10] + [32] * 6)
base[72:90] = r
n = bytearray(18); n[3] = 0xFC
n[5:18] = b'AVS 5120x2160\n'[:13]
base[90:108] = n
base[108:126] = bytes([0, 0, 0, 0x10, 0] + [0] * 13)  # dummy
base[126] = 1  # one extension block
cs(base)

# ---- DisplayID 1.3 extension, Type I detailed timing ----
t = T5
tm = bytearray(20)
tm[0:3] = struct.pack('<I', t['pclk'] - 1)[:3]
tm[3] = 0x80 | 0x06          # preferred, aspect 64:27
tm[4:6] = struct.pack('<H', t['ha'] - 1)
tm[6:8] = struct.pack('<H', t['hb'] - 1)
tm[8:10] = struct.pack('<H', (t['hfp'] - 1) | 0x8000)   # +H sync
tm[10:12] = struct.pack('<H', t['hsw'] - 1)
tm[12:14] = struct.pack('<H', t['va'] - 1)
tm[14:16] = struct.pack('<H', t['vb'] - 1)
tm[16:18] = struct.pack('<H', (t['vfp'] - 1) | 0x8000)  # +V sync
tm[18:20] = struct.pack('<H', t['vsw'] - 1)

ext = bytearray(128)
ext[0] = 0x70                # DisplayID extension tag
ext[1] = 0x13                # DisplayID 1.3
blk = bytes([0x03, 0x00, 20]) + bytes(tm)   # Type I timing, rev 0, 20-byte payload
ext[2] = len(blk)
ext[3] = 0x00                # product type: extension
ext[4] = 0x00                # extension count
ext[5:5 + len(blk)] = blk
end = 5 + len(blk)
ext[end] = (256 - sum(ext[1:end]) % 256) % 256   # DisplayID checksum
cs(ext)

edid = bytes(base) + bytes(ext)
open(OUT, 'wb').write(edid)

# ---- Verify ----
assert sum(edid[:128]) % 256 == 0 and sum(edid[128:]) % 256 == 0
for name, tt in (('base 3840x2160', T4), ('DisplayID 5120x2160', T5)):
    ht = tt['ha'] + tt['hb']; vt = tt['va'] + tt['vb']
    print(f"{name}: {ht}x{vt}  pclk {tt['pclk']/100:.2f} MHz  "
          f"hfreq {tt['pclk']*1e4/ht/1e3:.2f} kHz  vrefresh {tt['pclk']*1e4/(ht*vt):.4f} Hz")
```

To change the mode: edit `T5` (and `T4` if you want a different fallback), keep `Htotal x Vtotal x refresh` divisible by 10,000, and update the monitor name, product code and physical size.

---

## 9. Verifying

- `edid-decode file.bin` (Linux, `apt install edid-decode`) decodes and flags checksum or structure errors.
- Online decoders (e.g. Parse-EDID) accept the hex text.
- For `.txt` -> `.bin`: `xxd -r -p file.txt file.bin`.
- Always confirm on the real device that the source reports the mode, not just that the file parses.

## 10. Known limitations / TODO

- Not yet tested on a Hippotizer or NVIDIA hardware.
- Some EDID loaders only read CTA-861 extensions and ignore DisplayID, in which case they will fall back to the 3840x2160 base-block mode. A CTA-861 version using a VIC 125 style timing is the next thing to try.
- No audio, HDR or colorimetry data blocks.
- Manufacturer ID `AVS` is not registered.
