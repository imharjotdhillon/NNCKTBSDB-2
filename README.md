# Punjabi Book Translator — Phase 1 (Reader)

Native Android app (Kotlin, Jetpack Compose, Material 3, Room, DataStore) for reading PDF and EPUB books,
the foundation for a manual, no-API Gemini translation workflow (Phase 2+).

## STATUS — READ THIS FIRST

**This code has NOT been compiled or run.** It was written in an environment with no Android SDK, no Gradle,
and no access to Google/Maven repositories. Only the pure-Kotlin text logic (`core/TextUtils.kt`) was compiled
and tested (22 checks, all passed, via a standalone harness; the same checks are in
`app/src/test/.../TextUtilsTest.kt` as JUnit tests, which have not been run through Gradle).
Expect a few compile errors on first build. Paste them back and they get fixed.

## Build (Android Studio, on your PC)

1. Install **Android Studio** (current stable, JDK 17+ bundled). Accept the SDK platform 35 download.
2. *File > Open* this folder. Let Gradle sync. (Android Studio downloads Gradle 8.9 from
   `gradle/wrapper/gradle-wrapper.properties`; there is no `gradlew` script yet, the IDE does not need one.
   If it offers to upgrade AGP/Gradle, you may accept.)
3. Run on an emulator or a USB-debugging phone (Android 8.0+, API 26).
4. APK: *Build > Build APK(s)*, output at `app/build/outputs/apk/debug/app-debug.apk`.
5. Unit tests: `./gradlew :app:testDebugUnitTest` (or right-click `TextUtilsTest` > Run).

## Pinned versions (see `gradle/libs.versions.toml`)

AGP 8.6.1 · Gradle 8.9 · Kotlin 2.0.20 · KSP 2.0.20-1.0.0 · Compose BOM 2024.09.03 (Material 3 1.3) ·
Room 2.6.1 · Paging 3.3.2 · DataStore 1.1.1 · Navigation 2.8.1 · Jsoup 1.18.1 · PDFBox-Android 2.0.27.0.
compileSdk/targetSdk 35, minSdk 26. These were chosen as a mutually compatible set from known release notes;
they could not be verified against live repositories from the build environment.

## What Phase 1 implements

- Library: import PDF/EPUB through the system picker, duplicate detection (SHA-256), covers, title/author,
  format, language direction (auto-detected: Gurmukhi vs Latin), read % and translation %, search, sort
  (recent / title / translation progress), grid/list, resume reading, details, delete with confirmation,
  import progress, failed-import retry/remove, interrupted imports recovered on next launch.
- Extraction into stable text blocks (`P0001`…, contiguous reading order, source page/resource/locator,
  block type). Original text is never updated after import. PDF: PDFBox-Android per page with paragraph
  heuristics; pages with no text keep a placeholder block and are counted as OCR candidates.
  EPUB: Jsoup over the spine; chapters from nav/NCX, falling back to headings.
- Reader: one text view with **Original / Translated** toggle; **light/dark toggle at top-left**; font family
  (Sans / Serif / bundled Noto Sans Gurmukhi), size, line and paragraph spacing, margins, justify, brightness;
  chapter list, go-to-page, bookmarks; reading position restored; PDF **Text / Pages** views (Pages uses
  Android PdfRenderer, inverted in dark mode). Text blocks load progressively (Paging 3 + Room).
- Translated mode only shows saved translations. There are none yet, so it says
  "This section has not been translated yet." Nothing is simulated.
- Room schema v1 with all later-phase tables already present (translations, bookmarks, progress),
  schema export on, migration scaffold, no destructive fallback.
- No INTERNET permission, `allowBackup=false`.

## Deviations and limitations (honest list)

- **Readium is not used yet.** Its EPUB/PDF navigators are fragment-based and its API changes between versions;
  writing against it blind was too risky. Phase 1 uses Jsoup + PDFBox-Android + PdfRenderer, which fit the
  "single text view" design. Revisit Readium once the project builds and its docs can be checked.
- Not implemented in Phase 1 (by design): Translation Queue and Glossary tabs (nothing real to show yet),
  Gemini prompts/import, verification, export, OCR, backup. Bookmarks and Pages-view are included early.
- No pinch-zoom in Pages view. Inline italics/bold in EPUB text are not preserved (block type is).
  EPUB images are not shown. Only the first TOC entry per spine file names a chapter.
- PDF text extraction depends on the PDF's text layer. PDFs using legacy (non-Unicode) Gurmukhi fonts will
  extract as garbage; that needs OCR or conversion. Password-protected PDFs are rejected with a message.
- Reader preferences are global, not per book. Strings are English-only. WorkManager is not used yet
  (imports run in an app-scoped coroutine and are recoverable via retry).
- Font settings sliders apply when you release the slider.

## Manual test checklist for the first build

1. Import a text PDF, an EPUB 2, an EPUB 3, a scanned PDF, a Gurmukhi PDF/EPUB. Re-import one: expect "already in your library".
2. Library shows cover, title, progress; sort/search/grid-list work; long-press > Details / Delete.
3. Open each book: scroll, close, reopen: position restored. Toggle dark/light (top-left) mid-page: position kept.
4. Original/Translated toggle: Translated shows the "not translated yet" message and position is kept.
5. PDF: Text <-> Pages keeps the page; go-to-page; bookmarks add/open/delete.
6. Change font/size/spacing/margins; Gurmukhi text renders with correct conjuncts (try the Noto Gurmukhi font chip).
7. Kill the app during a large import: next launch shows "Import failed", Retry works.
8. Delete a book: files, cover and data gone.

## Licenses

Noto Sans Gurmukhi is SIL OFL 1.1 (`app/src/main/assets/licenses/NotoSansGurmukhi-OFL.txt`).
Jsoup MIT; PDFBox-Android Apache 2.0; AndroidX/Compose Apache 2.0.
