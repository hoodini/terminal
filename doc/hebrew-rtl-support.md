# RTL Language Support in Windows Terminal

## Overview

Windows Terminal now includes native support for Right-to-Left (RTL) text rendering for Hebrew and other RTL languages. This enhancement enables the terminal to properly display and handle RTL text with correct text alignment and rendering.

## Implementation Details

### Text Direction Detection

The terminal uses the "first strong character" heuristic from the Unicode Bidirectional Algorithm to determine paragraph direction:

- Scans text from the beginning to find the first strong directional character
- **Strong LTR characters**: Basic Latin letters (A-Z, a-z)
- **Strong RTL characters**: Hebrew (U+0590-U+05FF), Arabic (U+0600-U+06FF)
- Returns the appropriate reading direction based on the first strong character found
- Defaults to Left-to-Right if no strong directional character is found

### Technical Changes

1. **DWriteTextAnalysis.cpp**
   - Modified `GetParagraphReadingDirection()` to use "first strong character" logic
   - Checks for strong LTR characters (Basic Latin) and strong RTL characters (Hebrew, Arabic)
   - Returns `DWRITE_READING_DIRECTION_RIGHT_TO_LEFT` when first strong character is RTL
   - Returns `DWRITE_READING_DIRECTION_LEFT_TO_RIGHT` when first strong character is LTR

2. **Bidirectional Text Analysis**
   - Added `AnalyzeBidi()` call to the text processing pipeline
   - Stores bidirectional levels in `TextAnalysisSinkResult` for proper rendering
   - Implemented `SetBidiLevel()` with overlap detection to handle different text segmentation
   - BiDi and Script analysis may segment text differently, so overlap logic ensures correct level assignment

3. **Data Structures**
   - Extended `TextAnalysisSinkResult` to include `bidiLevel` field
   - BiDi level information helps DirectWrite render text correctly

## Usage

RTL text in the terminal will automatically be rendered right-to-left based on the first strong character. No configuration is required.

### Example RTL Text

```
שלום עולם  (Hello World in Hebrew)
مرحبا بالعالم  (Hello World in Arabic)
```

When entered or displayed in the terminal, this text will appear with correct RTL alignment.

### Mixed Text Example

```
Hello שלום  (Starts with LTR, will render LTR with embedded RTL)
שלום Hello  (Starts with RTL, will render RTL with embedded LTR)
```

The paragraph direction is determined by the first strong character.

## Supported Characters

The implementation supports RTL characters in the following Unicode ranges:

- Hebrew: U+0590 to U+05FF
  - Hebrew letters (U+05D0 to U+05EA)
  - Hebrew punctuation
  - Hebrew vowel points and cantillation marks

- Arabic: U+0600 to U+06FF
  - Arabic letters
  - Arabic punctuation
  - Arabic vowel marks
- Hebrew vowel points and cantillation marks

## Technical Notes

### Mixed Text Handling

The implementation uses the "first strong character" rule to determine paragraph direction:
- If the first strong character is LTR (e.g., Latin letter), the paragraph is LTR
- If the first strong character is RTL (e.g., Hebrew/Arabic), the paragraph is RTL
- Within the paragraph, the BiDi algorithm handles proper ordering of mixed content

### Rendering Engine

The implementation uses DirectWrite's native BiDi support through:
- `IDWriteTextAnalyzer::AnalyzeBidi()` for bidirectional text analysis
- `IDWriteTextAnalysisSource::GetParagraphReadingDirection()` for base direction
- BiDi levels for proper glyph ordering

## Future Enhancements

Potential areas for future improvement:

1. User-configurable RTL mode (manual override)
2. Extended RTL support for other languages (Arabic, etc.)
3. Improved cursor movement in mixed RTL/LTR text
4. RTL-specific selection behavior

## References

- [Unicode Hebrew Block](https://www.unicode.org/charts/PDF/U0590.pdf)
- [DirectWrite BiDi Documentation](https://learn.microsoft.com/en-us/windows/win32/directwrite/bidirectional-and-vertical-text)
- [Unicode Bidirectional Algorithm](https://unicode.org/reports/tr9/)
