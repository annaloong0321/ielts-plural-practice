# IELTS Plural Noun Practice — Project Reference

## Location
`/Users/long/Desktop/复数练习/`

## Repository
https://github.com/annaloong0321/ielts-plural-practice

## Live URL
https://annaloong0321.github.io/ielts-plural-practice/

## File Structure
```
复数练习/
├── ielts-plural-practice.html      ← Main application (single HTML file, ~58 KB)
├── ielts-plural-practice-v3.html   ← Backup (v3 with frontend-design)
├── ielts-plural-practice-backup.html ← Backup (v1)
├── ielts-plural-practice-v2-backup.html ← Backup (v2)
├── 使用手册.docx                    ← User manual (Chinese)
├── CLAUDE.md                       ← This file
├── audio/                          ← 48 MP3 files (C13-C18, each T1-T4 S1 & S4)
│   └── XX-Y-Z.mp3                  ← Book-Test-Section naming
├── transcripts/                    ← 48 TXT files (one per practice set)
│   └── cXXtYsZ.txt                 ← c13t1s1.txt through c18t4s4.txt
├── C5-20 听力音频/                  ← Source audio (gitignored)
├── 雅思真题1-20【无水印】/          ← Source PDFs (gitignored, copyrighted)
└── .gitignore
```

## Architecture
Single self-contained HTML file with:
- **CSS**: Custom design (warm academic theme, Nunito + Playfair Display fonts)
- **NLP**: compromise.js v14 via jsDelivr CDN (with unpkg fallback)
- **Audio**: HTML5 Audio API, fetch-to-blob for seeking support
- **Storage**: localStorage for practice sets, text cache, history

## Key JavaScript Functions

### Noun Detection (`detectNouns`)
Per-sentence NLP; filters out:
1. Words starting with capital letters (proper nouns)
2. Verbs tagged per-occurrence (not globally)
3. Quantifier set: `lot, lots, kind, kinds, sort, sorts, couple, couples`
4. Uncountable nouns (`#Uncountable`)
5. Gerunds (`#Gerund`)
6. Possessive pronouns (`#Possessive`)
7. Irregular plurals without `-s` ending
8. Attributive nouns (adjacent nouns with only spaces between)

### Audio (`loadAudioPath`)
- Fetches entire audio file as blob for seeking (HTTP Range not required)
- Applies `audioStart` from practice set metadata (skip intro)
- Restores `state.playbackRate` across set changes

### Rendering (`renderPracticeArea`)
- `<optgroup>` dropdown grouped by Cambridge book (c13-c18)
- Blank bars with fixed proportional width, absolute-positioned state labels

## Practice Set Data Structure
```javascript
{
  id: 'c18t1s1',                    // c{book}t{test}s{section}
  title: 'C18 T1 S1: Transport Survey',
  audioPath: 'audio/18-1-1.mp3',    // relative path
  audioStart: 90,                   // optional, seconds to skip intro
  textUrl: 'transcripts/c18t1s1.txt' // external text file
}
```

## CDs/External Dependencies
- compromise.js: `https://cdn.jsdelivr.net/npm/compromise@14.14.0/builds/compromise.min.js`
- Google Fonts: Playfair Display + Nunito

## Adding New Practice Sets
1. Extract audio scripts from PDF via OCR
2. Save cleaned text to `transcripts/cXXtYsZ.txt`
3. Add entry to `getDefaultSets()` in HTML (id, title, audioPath, textUrl)
4. Copy audio MP3 to `audio/`
5. Verify with `node --check` on extracted JS
6. Test via `python3 -m http.server 8080`

## Known Quirks
- C13 T1 S1 & S4 have c13t1s1.txt and c13t1s4.txt (was missing from first push — fixed)
- Python SimpleHTTP server lacks Range support → audio loaded via fetch+blob
- Safari may block `fetch()` from `file://` → use HTTP server locally
- Google Fonts require internet; system fonts fallback in place
- localStorage auto-merge: old sets preserved, new default sets added
- History limited to 50 entries

## Naming Convention
- `c{book}t{test}s{section}` — e.g., `c18t2s4` = Cambridge 18, Test 2, Section 4
- Audio files: `{book}-{test}-{section}.mp3` — e.g., `18-2-4.mp3`
- Transcript files: `c{book}t{test}s{section}.txt`

## Audio Start Times
| Book | T1S1 | T1S4 | T2S1 | T2S4 | T3S1 | T3S4 | T4S1 | T4S4 |
|------|------|------|------|------|------|------|------|------|
| C13  | —    | —    | —    | —    | —    | —    | —    | —    |
| C14  | —    | —    | —    | —    | —    | —    | —    | —    |
| C15  | —    | —    | —    | —    | —    | —    | —    | —    |
| C16  | 85   | 70   | 75   | 70   | 86   | 72   | 87   | 70   |
| C17  | 90   | 71   | 90   | 73   | 86   | 68   | 79   | 70   |
| C18  | 90   | 72   | 82   | 70   | 74   | 73   | 81   | 71   |

## Design Tokens (from v3)
```css
--ink: #1e1b18;        --ink-light: #5c554f;
--paper: #faf6f0;       --paper-warm: #f5efe5;
--surface: #fffdf9;     --border: #e0d8cc;
--amber: #b87419;       --amber-bg: #fdf3e4;
--sage: #4a7c59;        --sage-bg: #eef5ef;
--rust: #c96b5a;        --rust-bg: #fdf2ef;
--navy: #3b4f63;        --navy-bg: #edf1f5;
--gold: #c49b3f;        --gold-bg: #fef9ef;
--font-text: 'Nunito', sans-serif;
--font-display: 'Playfair Display', serif;
```

## OCR Pipeline for New Books
1. `pdftoppm -f {page} -l {page} -r 300 -png PDF /tmp/prefix`
2. `pytesseract.image_to_string(img, lang='eng')` via Python
3. Identify section boundaries by "PART 1"/"PART 4"/"SECTION" markers
4. Clean OCR errors (fix |→I, check line breaks, remove Q numbers)
5. Check for cross-page bleed (intro on previous page, conclusion on next page)
6. Save to `/transcripts/`, add to `getDefaultSets()`, copy audio
7. Verify with `node --check` before testing
