# Thesis Verification Report: Linking Parameters

## Summary

I verified your LaTeX table "Feste Suchparameter für Spezialfälle" against the actual implementation. Here are the findings:

---

## 1. Fahrtrichtungsdetektion (GKS-Suche)

| Parameter | Thesis | Code | Location | Status |
|-----------|--------|------|----------|--------|
| d_max | 250 px | 250 px | `linking.py:753` | ✅ MATCH |
| dy_min | 30 px | 30 px | `linking.py:754` | ✅ MATCH |
| dy_max | 200 px | 200 px | `linking.py:755` | ✅ MATCH |
| dx_tol | ±120 px | 120 px | `linking.py:756-757` | ✅ MATCH |

**Function:** `detect_fahrtrichtung()` (lines 750-968)

---

## 2. Haltetafel-GKS-Fallback

| Parameter | Thesis | Code | Location | Status |
|-----------|--------|------|----------|--------|
| d_max | 250 px | 250 px | `linking.py:686` | ✅ MATCH |
| dy_tol | 100 px | 100 px | `linking.py:687` | ✅ MATCH |
| dx_tol | 300 px | 300 px | `linking.py:688` | ✅ MATCH |

**Function:** `link_haltetafel_to_gks()` (lines 685-748)

---

## 3. Fallback-Mechanismen

| Parameter | Thesis | Code | Location | Status |
|-----------|--------|------|----------|--------|
| Isolierstoß r_max | 300 px | 300 px | `linking.py:2201` | ✅ MATCH |
| Gleismittellinie d_ray | 1500 px | 1500 px | `linking.py:52-53` | ✅ MATCH |

---

## 4. Richtungsbasierte Filterung

### k_dx Values (called `dx_multiplier` in code)

| Class | Thesis | Code | Location | Status |
|-------|--------|------|----------|--------|
| Signal | 1.0 | 1.0 (default) | `config.py:268` | ✅ MATCH |
| GKS | 1.0 | 1.0 (default) | `config.py:270-271` | ✅ MATCH |
| GM-Block | 1.0 | 1.0 (default) | `config.py:269` | ✅ MATCH |
| Haltetafel | "erhöht" | 2.0 | `config.py:279` | ✅ MATCH |
| Prellbock | "erhöht" | 2.0 | `config.py:278` | ✅ MATCH |

### Formulas

| Formula | Thesis | Code | Location | Status |
|---------|--------|------|----------|--------|
| dx_max,tight | k_dx × 0.45 × max(w) | `dx_multiplier * 0.45 * max(anchor["w"], c["w"])` | `linking.py:531` | ✅ MATCH |
| dx_max,left | 1.3 × dx_max | `dx_max *= 1.3` | `linking.py:540` | ✅ MATCH |

---

## 5. Discrepancies Found

### ⚠️ CORRECTION NEEDED: Angular threshold is 12° not 15°

**Your thesis says 15°, but the code uses 12°.**

In `utils/helpers.py:107`:
```python
ANGLE_TOL = 12.0  # Thesis says 15°
```

**Action required:** Update your thesis from 15° to 12° for the angular/cardinal classification threshold.

### ℹ️ INFO: Haltepunkt-Signal Group uses different dx_tolerance

The function `detect_haltepunkt_signal_group()` uses:
- `dx_tolerance = 100 px` (line 2041)

This is **different** from the Fahrtrichtung's ±120 px. However, this is a **separate function** for a different purpose (detecting Haltepunkt-Signal-Coordinate groups), so this is intentional and not a thesis error.

### ✅ CONFIRMED: search_left only for weichengruppenende

The `search_left=True` parameter (enabling the 1.3× left bonus) is correctly only configured for `weichengruppenende`.

---

## 6. Physical Interpretation Verification

Your thesis states at 500 DPI:
- 250 px ≈ 12.7 mm ✅ (250/500 inch × 25.4 mm = 12.7 mm)
- 120 px ≈ 6 mm ✅ (120/500 inch × 25.4 mm = 6.1 mm)
- 300 px ≈ 15 mm ✅ (300/500 inch × 25.4 mm = 15.24 mm)

**All physical interpretations are correct.**

---

## Conclusion

**Overall: ✅ THESIS IS ACCURATE** (with 1 correction needed)

All parameters in your LaTeX table match the implementation exactly:
- ✅ All GKS-Suche parameters (250px, 30px, 200px, ±120px)
- ✅ All Haltetafel-GKS-Fallback parameters (250px, 100px, 300px)
- ✅ All Fallback parameters (300px, 1500px)
- ✅ All physical interpretations (mm calculations)
- ✅ k_dx values and formulas (0.45 tight, 1.3 left)

**One correction needed:**
- Change angular threshold from **15° → 12°** in your thesis

---

## Files Verified

- `core/linking.py` - Main linking functions
- `config.py` - LINK_RULES configuration
- `utils/helpers.py` - Angular detection helpers

---

## LaTeX Code for Additional Parameters (TIER 3 + TIER 4)

Add this to your existing table:

```latex
\multicolumn{4}{|c|}{\textit{Fahrtrichtung-Fallback (TIER 3: Relaxierte Spaltensuche)}} \\
\hline
GKS-Relaxed & $dx_{\text{tol}}$ & 200 px & Erweiterte horizontale Toleranz \\
& $dy_{\text{min}}$ & 30 px & Min. vertikale Separation \\
& $dy_{\text{max}}$ & 600 px & Max. vertikale Separation \\
\hline
\multicolumn{4}{|c|}{\textit{Fahrtrichtung-Fallback (TIER 4: Euklidische Suche)}} \\
\hline
GKS-Nearest & $d_{\text{max}}$ & 800 px & Euklidische Distanz \\
& $dy_{\text{min}}$ & 30 px & Min. vertikale Separation \\
\hline
```

**Physical interpretation to add to your itemize:**

```latex
\item 600 px $\approx$ 30,5 mm (erweiterter vertikaler Suchbereich TIER 3)
\item 800 px $\approx$ 40,6 mm (maximaler Fallback-Suchradius TIER 4)
```

**Code locations:**
- TIER 3: `linking.py:1147-1269` → `detect_fahrtrichtung_gks_relaxed()`
- TIER 4: `linking.py:1276-1395` → `detect_fahrtrichtung_gks_nearest()`
