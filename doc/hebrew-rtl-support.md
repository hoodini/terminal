# Hebrew RTL Support in Windows Terminal

## Overview

Windows Terminal now includes native support for Right-to-Left (RTL) text rendering for Hebrew characters. This enhancement enables the terminal to properly display and handle Hebrew text with correct text alignment and rendering.

## Implementation Details

### Text Direction Detection

The terminal automatically detects Hebrew characters in the text buffer and sets the appropriate reading direction:

- **Hebrew Unicode Range**: U+0590 to U+05FF
- When Hebrew characters are detected in a line, the reading direction is set to Right-to-Left
- Lines without Hebrew characters default to Left-to-Right

### Technical Changes

1. **DWriteTextAnalysis.cpp**
   - Modified `GetParagraphReadingDirection()` to detect Hebrew characters
   - Returns `DWRITE_READING_DIRECTION_RIGHT_TO_LEFT` for text containing Hebrew
   - Returns `DWRITE_READING_DIRECTION_LEFT_TO_RIGHT` for other text

2. **Bidirectional Text Analysis**
   - Added `AnalyzeBidi()` call to the text processing pipeline
   - Stores bidirectional levels in `TextAnalysisSinkResult` for proper rendering
   - Implemented `SetBidiLevel()` to capture BiDi analysis results

3. **Data Structures**
   - Extended `TextAnalysisSinkResult` to include `bidiLevel` field
   - BiDi level information helps DirectWrite render text correctly

## Usage

Hebrew text in the terminal will automatically be rendered right-to-left. No configuration is required.

### Example Hebrew Text

```
שלום עולם  (Hello World in Hebrew)
```

When entered or displayed in the terminal, this text will appear with correct RTL alignment.

## Supported Characters

The implementation supports all Hebrew characters in the Unicode range U+0590 to U+05FF, including:

- Hebrew letters (U+05D0 to U+05EA)
- Hebrew punctuation
- Hebrew vowel points and cantillation marks

## Technical Notes

### Mixed Text Handling

When a line contains both Hebrew (RTL) and English (LTR) text, the BiDi algorithm automatically handles the proper rendering order based on the detected paragraph reading direction.

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
- [DirectWrite BiDi Documentation](https://docs.microsoft.com/en-us/windows/win32/directwrite/bidirectional-and-vertical-text)
- [Unicode Bidirectional Algorithm](https://unicode.org/reports/tr9/)
