> Translated from the Chinese version; if they differ, the Chinese version prevails.

# Hubo-Tack Backpack ID Card Specification (SPEC-004 Public Edition V2.2)

> **DIY backpack rules (mandatory)**
>
> 1. **DIY backpacks must use the 0x57 programmed ID card**: tie `A0`–`A2` of the `AT24C02` all to `3V3` and write the 12-byte descriptor. 0x50–0x56 are reserved for official models and must not be used by DIY backpacks.
> 2. `product_id` is taken from the **DIY range `0xF000`–`0xFFFE`**: for personal use, not guaranteed globally unique, no application needed. `0x0000` and `0xFFFF` are forbidden.
> 3. To get a **globally unique** `product_id`, apply through a GitHub Issue (see 4.2, using the [issue template](../../.github/ISSUE_TEMPLATE/product-id-request.md)).
> 4. For programming, see Chapter 5: use `idcard_image.py` from hubo-backpack-skill to generate a 16-byte image, pull the `WP` test point low, and write it with a generic I²C EEPROM programmer to offset 0x00 of address 0x57.
> 5. Host firmware treats an unrecognized `form_id` as 0 (multiplexed zone unused) without reporting an error, so the DIY range needs no firmware change.

> This is the public edition of the internal specification SPEC-004 V2.2, and it incorporates the 12-byte backpack descriptor format and the `form_id` values (3.6, 3.7). Section and clause numbers are kept unchanged; internal-process clauses are not part of the public edition, and their numbers are marked "(Not included in this public edition)". For interface and mechanical requirements see the [SPEC-003 public edition](./30pin-backpack-spec.md).

## Revision history

(Not included in this public edition)

---

## 1. Scope

This specification applies to all 30-Pin backpacks that plug into the 30-pin connector of three kinds of host: the Hubo-Tack mainboard (ESP32-P4), the S3 host (ESP32-S3) and the CM series (in development); the specification is based on Hubo-Tack. The S3 host currently cannot recognize ID cards built to this specification (its mainboard has no ID card I2C pull-up and its firmware only probes 0x50); the CM series is in development, and the release version prevails. For per-host differences see [SPEC-003](./30pin-backpack-spec.md) 1.4. The ID card sits on the mainboard's ID card I2C bus (`GPIO30`/`GPIO31`; backpack pad 14 = `SCL`, pad 15 = `SDA`, see [SPEC-003](./30pin-backpack-spec.md) 3.1).

The host identification flow in this specification (scanning 0x50–0x57, reading and validating the descriptor) is a requirement on host firmware; current firmware implements only part of it (probing 0x50), and the full flow is under development.

---

## 2. Address planning

### 2.1 Principles

1. **Official models need no programming**: official models use a blank `AT24C02`, and the host distinguishes models by 7-bit I2C address; each address in 0x50–0x56 corresponds to one official model and is reserved for official use. DIY backpacks must not use them (the host would drive the multiplexed zone according to the official form of that address).
2. **0x57 is the programmed version**: it holds a 12-byte descriptor (including `product_id`). **DIY backpacks always use 0x57**.
3. **Host identification flow**: scan 0x50–0x57. When exactly one address responds: for 0x50–0x56 the descriptor is **not read** and the backpack is handled as an official model; for 0x57 the 12-byte descriptor is read and its signature and CRC are validated. If the 0x57 descriptor fails validation, or more than one address responds at the same time, the host always treats it as `form_id = 0x00`, `flags = 0` (multiplexed zone all high-impedance) and reports "unknown backpack" or "backpack address conflict" respectively.

### 2.2 Address allocation table

| Address | A2 A1 A0 | Use | Host handling |
|---|---|---|---|
| 0x50 | 0 0 0 | Reserved for official models | Handled as an official model |
| 0x51 | 0 0 1 | Reserved for official models | Handled as an official model |
| 0x52 | 0 1 0 | Reserved for official models | Handled as an official model |
| 0x53 | 0 1 1 | Reserved for official models | Handled as an official model |
| 0x54 | 1 0 0 | Reserved for official models | Handled as an official model |
| 0x55 | 1 0 1 | Reserved for official models | Handled as an official model |
| 0x56 | 1 1 0 | Reserved for official models | Handled as an official model |
| **0x57** | 1 1 1 | **Programmed version** (descriptor + `product_id`), **used by DIY backpacks** | Per descriptor |

### 2.3 Address exclusion zone

| Clause | Requirement |
|---|---|
| 2.3.1 | 0x50–0x57 are all reserved for the ID card. No `24Cxx`-family device, and no other device occupying 0x50–0x57, may be placed on the backpack |
| 2.3.2 | The addresses of other I2C devices on the backpack that sit on the ID card bus must avoid 0x50–0x57 |

### 2.4 ID card device

| Clause | Requirement |
|---|---|
| 2.4.1 Part | `AT24C02` family (2 Kbit). **Forbidden:** 24C04/08/16, which use the address pins for page select and would occupy several addresses |
| 2.4.2 Write protect | `WP` must protect the whole array (verify against the datasheet when selecting the part); `WP` is pulled up to `3V3` through 10 kΩ and brought out to a test point; when programming, short (pull low) it to `GND`. `WP` **must not** be tied directly to ground |
| 2.4.3 Speed | 100 kHz |
| 2.4.4 Address pins | `A0`–`A2` must be tied directly to `GND` or to backpack pads 3, 4 (`3V3`) and must not float; on DIY backpacks `A0`–`A2` are all tied to `3V3` (0x57). When connected through 0 Ω solder options, the design document must state the populated state and the corresponding address, and the address is read back to confirm on the finished board |
| 2.4.5 Package | An 8-pin package with all three pins `A0`–`A2` brought out must be chosen (SOIC-8/TSSOP-8; use TSSOP-8 on the bottom side, see SPEC-003 4.1). 5-pin packages such as SOT23-5 have no address pins and a fixed address, and must not be used |
| 2.4.6 Memory areas | `0x00`–`0x0F` is the descriptor area: the programmed version (0x57) holds the 12-byte descriptor, with the other 4 bytes reserved. `0x10`–`0xFF` is the backpack user area: its content and format are defined by that backpack's design document and are not read by the host identification flow; short the `WP` test point when writing |

---

## 3. product_id and related fields

The fields in this chapter are written only to the programmed (0x57) ID card.

| Clause | Requirement |
|---|---|
| 3.1 Allocation unit | **One `product_id` per model**. Revisions of the same model (V1.0 → V1.1) keep the `product_id` and are told apart by `hw_ver`; parallel versions with the same function but a different main chip each use a separate `product_id` |
| 3.2 Numbering | 16 bits. `0x0000` (unassigned) and `0xFFFF` (value read from blank EEPROM) are forbidden. **`0xF000`–`0xFFFE` is the DIY range**: for personal use, not guaranteed globally unique, no application needed. `0x0001`–`0xEFFF` are globally unique numbers that must be applied for under Chapter 4 and are never reused once retired |
| 3.3 `hw_ver` | High 4 bits are the major version, low 4 bits the minor version (V1.0 = `0x10`, V1.1 = `0x11`, V2.0 = `0x20`); when a version number exceeds 15, an encoding is arranged separately |
| 3.4 `form_id` | Values are listed in 3.7. If a new form is needed, propose it through an Issue; it is not assigned privately under this specification |
| 3.5 Other fields | `signature`, `spec_ver`, `flags` and `crc16` are described in 3.6 |

### 3.6 Descriptor format (12 bytes, fixed offsets)

```
Offset  Field        Size  Description
─────────────────────────────────────────────────────────
0..3    signature    4 B   ASCII "HuBo"
4       spec_ver     1 B   Version of this specification, currently 0x01
5       form_id      1 B   Multiplexed-zone form, see 3.7
6..7    product_id   2 B   Model-unique ID (little-endian), used only by the
                           0x57 programmed version;
                           0x0000 and 0xFFFF are forbidden
8       hw_ver       1 B   Backpack hardware version: high 4 bits major,
                           low 4 bits minor (V1.0 = 0x10, V1.1 = 0x11)
9       flags        1 B   bit0 = ESP_INT polarity (0 active low / 1 active high)
                           bit1 = PACK_PEN (1 = pad 28 is this backpack's enable pin)
                           bit2~7 reserved, must be written as 0
10..11  crc16        2 B   CRC-16/CCITT-FALSE over offsets 0..9 (little-endian)
─────────────────────────────────────────────────────────
                     12 B  Total
```

- CRC-16/CCITT-FALSE: polynomial `0x1021`, initial value `0xFFFF`, covering offsets 0..9, with the result stored little-endian at offsets 10..11.
- EEPROM `0x00`–`0x0F` is the descriptor area (12-byte descriptor + 4 reserved bytes, the reserved bytes programmed as `0xFF`); `0x10`–`0xFF` is the user area (2.4.6).
- Multi-byte fields are little-endian.
- **When the host meets an unrecognized `form_id`, it treats it as `0x00`** (multiplexed zone all high-impedance) and reports no error.
- **If `spec_ver` is higher than the host supports, parse only the known fields**; the meaning of offsets 0..11 must not change in later versions (fields may only be appended after them).
- If reserved bits in `flags` are not 0, they are ignored and do not affect parsing.
- **`flags.bit1 = 1`: pad 28 (mainboard `U1-27`) is this backpack's enable pin (`PACK_PEN`)**: after the descriptor is validated, the host configures that pin as an output driven high, regardless of `form_id`. The enable target may be the backpack power (the backpack side must gate it through a load switch) or a function (such as an amplifier enable); the backpack-side enable pin must have a pull-down so it is off by default before the host drives it high. With `bit1 = 0`, that pin is handled according to `form_id`. For backpacks with `form_id = 0x03` (LoRa) and `form_id = 0x04` (parallel input), `bit1` must be 0 (that pin is a `BUSY` input and a logic-analysis input respectively, and host driving would cause contention).

### 3.7 `form_id` values

| Value | Form |
|---|---|
| `0x00` | **Multiplexed zone not used** (all 8 pins stay high-impedance) |
| `0x01` | SPI |
| `0x02` | I2S (audio) |
| `0x03` | LoRa (SPI + control lines) |
| `0x04` | Parallel input (8-channel logic analysis, all 8 pins are host inputs, not driven by the host) |
| `0x05` | SPI slave + UART + ready handshake (for backpacks with their own processor; pad 28 is `PACK_PEN`) |
| `0x06`–`0xFF` | Reserved; open an Issue if you need a new form; the host treats them as `0x00` |

The function of each multiplexed-zone pin depends on `form_id`; on the mainboard side they are only called multiplexed-zone GPIO (pin positions in SPEC-003 3.1). The functions each pin can carry are shown in the interface figure under SPEC-003 3.1; the full timing and direction conventions of each form are not included in this public edition.

### 3.8 How developers choose

1. **Address**: `0x50`–`0x56` are reserved for official models (no programming needed); DIY backpacks always use the programmable `0x57`.
2. **`product_id`**: DIY uses `0xF000`–`0xFFFE`, no application needed; for a globally unique number, open an Issue in this repository.
3. **`form_id`**: choose from the table above according to how the 8 multiplexed-zone lines are used; the host firmware configures those lines accordingly. If none fits, choose `0x00` and use the lines yourself on the USR nets; open an Issue if you need a new form.

---

## 4. Application process

### 4.1 Timing

An application for a globally unique `product_id` through an Issue must be completed **before the schematic is frozen** and the number written into that backpack's design document. The DIY range (`0xF000`–`0xFFFE`) needs no application.

### 4.2 Steps

| Step | Action | Responsible |
|---:|---|---|
| 1 | Submit the application in this repository with the [issue template](../../.github/ISSUE_TEMPLATE/product-id-request.md): backpack name, function and `form_id`, other I2C devices on the ID card bus and their addresses (must avoid 0x50–0x57), first `hw_ver`, project link | Applicant |
| 2 | A maintainer checks that `form_id` and the address are consistent with the host firmware's identification logic, and replies in the Issue with the assigned `product_id` | Maintainer |
| 3 | Write the `product_id` into the design document and into the note beside the ID card in the schematic | Applicant |

### 4.3 Changes and retirement

- Project cancelled: the number is retired and never reused;
- Change of form (`form_id` changes): treated as a new model, apply again;
- Revision: same `product_id`, told apart by `hw_ver`.

### 4.4 Addresses without programming and lookup values (allocation and changes)

(Not included in this public edition)

### 4.5 DIY backpacks

- Use the 0x57 programmed version with `product_id` chosen from `0xF000`–`0xFFFE`; choose it yourself, no application needed. This range is not guaranteed globally unique, so backpacks from different people may share a number.
- When a globally unique number is needed, submit an Issue per 4.2.
- Host firmware treats an unrecognized `form_id` as `0x00` and reports no error, so the DIY range needs no firmware change.

---

## 5. Programming the programmed version

The clauses in this chapter apply to all 0x57 programmed ID cards, and DIY backpacks must be programmed as described here.

| Clause | Requirement |
|---|---|
| 5.1 Content | Use `idcard_image.py` from hubo-backpack-skill to generate a 16-byte image (12-byte descriptor + 4 bytes of `0xFF`, including the CRC) from `product_id`, `form_id`, `hw_ver` and so on; do not assemble it by hand. Each backpack is programmed with its own content; do not reuse someone else's |
| 5.2 Write protect | Pull the `WP` test point low (short it to `GND`) while programming, and release it afterwards |
| 5.3 Verification | After programming, read back the 12 bytes and check `signature`, `product_id` and `crc16` |
| 5.4 Programming method | Use a generic I²C EEPROM programmer and write the image to offset `0x00` of address 0x57 |

---

## 6. Open issues

(Not included in this public edition)

---

## 7. References

(Not included in this public edition)
