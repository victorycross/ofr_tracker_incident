# OFR Issue Tracker — Appkit4 Colour Map

**Version:** 1.0
**Date:** February 19, 2026
**Theme Variant:** Blue, Orange, Red, Black

---

## Primary UI Colours

| Token | Hex | RGBA | Usage |
|-------|-----|------|-------|
| **Primary Blue** | `#415385` | `RGBA(65,83,133,1)` | Header bar, navigation, "New" status badge, "Low" priority badge, staleness "Current" |
| **Primary Orange** | `#D04A02` | `RGBA(208,74,2,1)` | CTA buttons, "Active" status badge, accent |
| **Primary Red** | `#E0301E` | `RGBA(224,48,30,1)` | "High" priority badge, "Escalated" status badge, staleness "Stale" |
| **Neutral Black** | `#2D2D2D` | `RGBA(45,45,45,1)` | Primary body text, KPI count labels, panel headings |

---

## Extended Ramp (from Appkit4 spec)

### Primary Blue Ramp

| Step | Hex | RGBA | App Usage |
|------|-----|------|-----------|
| Blue +3 (lightest) | `#D2D7E2` | `RGBA(210,215,226,1)` | Selected row background |
| Blue +2 | `#9AA4BE` | `RGBA(154,164,190,1)` | — |
| Blue +1 | `#62719A` | `RGBA(98,113,154,1)` | Header hover state |
| **Blue (base)** | **`#415385`** | **`RGBA(65,83,133,1)`** | **Header bar, nav, badges** |
| Blue -1 | `#203570` | `RGBA(32,53,112,1)` | — |
| Blue -2 | `#1A2A5A` | `RGBA(26,42,90,1)` | — |
| Blue -3 | `#132043` | `RGBA(19,32,67,1)` | — |
| Blue -4 | `#0D152D` | `RGBA(13,21,45,1)` | — |
| Blue -5 (darkest) | `#060B16` | `RGBA(6,11,22,1)` | — |

### Primary Orange Ramp

| Step | Hex | RGBA | App Usage |
|------|-----|------|-----------|
| Orange +3 (lightest) | `#FEDACC` | `RGBA(254,218,204,1)` | — |
| Orange +2 | `#FDAB8D` | `RGBA(253,171,141,1)` | — |
| Orange +1 | `#FB7C4D` | `RGBA(251,124,77,1)` | — |
| Orange +0.5 | `#E45C2B` | `RGBA(228,92,43,1)` | "Medium" priority badge, "Monitoring" status, staleness "Aging" |
| **Orange (base)** | **`#D04A02`** | **`RGBA(208,74,2,1)`** | **CTA buttons, "Active" status** |
| Orange -1 | `#C34C2F` | `RGBA(195,76,47,1)` | Button hover state |
| Orange -2 | `#A7452C` | `RGBA(167,69,44,1)` | Button pressed state |
| Orange -3 | `#773829` | `RGBA(119,56,41,1)` | — |
| Orange -4 (darkest) | `#472B24` | `RGBA(71,43,36,1)` | — |

### Primary Red Ramp

| Step | Hex | RGBA | App Usage |
|------|-----|------|-----------|
| Red +3 (lightest) | `#F9D6D2` | `RGBA(249,214,210,1)` | — |
| Red +2 | `#F1A29A` | `RGBA(241,162,154,1)` | — |
| Red +1 | `#E96E61` | `RGBA(233,110,97,1)` | — |
| Red +0.5 | `#E44F3F` | `RGBA(228,79,63,1)` | — |
| **Red (base)** | **`#E0301E`** | **`RGBA(224,48,30,1)`** | **"High" priority, "Escalated" status, staleness "Stale"** |
| Red -1 | `#C22D1D` | `RGBA(194,45,29,1)` | — |
| Red -2 | `#A62B1E` | `RGBA(166,43,30,1)` | — |
| Red -3 | `#772820` | `RGBA(119,40,32,1)` | — |
| Red -4 (darkest) | `#472420` | `RGBA(71,36,32,1)` | — |

---

## Neutral Palette

| Token | Hex | RGBA | App Usage |
|-------|-----|------|-----------|
| Neutral Black | `#2D2D2D` | `RGBA(45,45,45,1)` | Primary body text, panel headings |
| Secondary text | `#505050` | `RGBA(80,80,80,1)` | Secondary labels |
| Muted labels | `#646464` | `RGBA(100,100,100,1)` | Tertiary/muted text |
| Placeholder text | `#8C8C8C` | `RGBA(140,140,140,1)` | Input placeholders |
| Disabled / reject btn | `#B4B4B4` | `RGBA(180,180,180,1)` | Reject button, disabled states |
| Disabled fill | `#C8C8C8` | `RGBA(200,200,200,1)` | Disabled button fill |
| Borders | `#E1E1E1` | `RGBA(225,225,225,1)` | Card/table borders |
| Dividers | `#EBEBEB` | `RGBA(235,235,235,1)` | Horizontal dividers |
| Hover background | `#F0F0F0` | `RGBA(240,240,240,1)` | Filter chip hover, row hover |
| Card background | `#F5F5F5` | `RGBA(245,245,245,1)` | KPI card fills |
| White | `#FFFFFF` | `RGBA(255,255,255,1)` | Page background, button text |
| Overlay | — | `RGBA(0,0,0,0.5)` | Modal backdrop |

---

## Semantic Colour Assignments

### Priority Badges

| Priority | Colour Token | RGBA |
|----------|-------------|------|
| High | Primary Red | `RGBA(224,48,30,1)` |
| Medium | Orange +0.5 | `RGBA(228,92,43,1)` |
| Low | Primary Blue | `RGBA(65,83,133,1)` |
| *(no priority)* | Neutral Grey | `RGBA(180,180,180,1)` |

### Status Badges

| Status | Colour Token | RGBA |
|--------|-------------|------|
| New | Primary Blue | `RGBA(65,83,133,1)` |
| Active | Primary Orange | `RGBA(208,74,2,1)` |
| Monitoring | Orange +0.5 | `RGBA(228,92,43,1)` |
| Escalated | Primary Red | `RGBA(224,48,30,1)` |
| Closed | Neutral Grey | `RGBA(180,180,180,1)` |

### Staleness Indicators

| Flag | Colour Token | RGBA |
|------|-------------|------|
| Current (0-7 days) | Primary Blue | `RGBA(65,83,133,1)` |
| Aging (8-14 days) | Orange +0.5 | `RGBA(228,92,43,1)` |
| Stale (15+ days) | Primary Red | `RGBA(224,48,30,1)` |

---

## Power Apps Formula Reference

### Priority Switch
```
Switch(ThisItem.Priority.Value,
    "High",   RGBA(224,48,30,1),
    "Medium", RGBA(228,92,43,1),
    "Low",    RGBA(65,83,133,1),
    RGBA(180,180,180,1)
)
```

### Status Switch
```
Switch(ThisItem.Status.Value,
    "New",        RGBA(65,83,133,1),
    "Active",     RGBA(208,74,2,1),
    "Monitoring", RGBA(228,92,43,1),
    "Escalated",  RGBA(224,48,30,1),
    RGBA(180,180,180,1)
)
```

### Staleness Conditional
```
If(ThisItem.DaysSinceUpdate <= 7,
    RGBA(65,83,133,1),
    If(ThisItem.DaysSinceUpdate <= 14,
        RGBA(228,92,43,1),
        RGBA(224,48,30,1)
    )
)
```

---

*Source: Appkit4 Colour Specification (Colour.docx) — Theme: Blue, Orange, Red, Black*
