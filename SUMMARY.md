# TypeWhisper Development Summary

## Feature: Spoken Language Shortcuts in Hotkeys Settings

### Overview

Added a new "Spoken Language Shortcuts" section to the Hotkeys settings tab. Users can assign custom keyboard shortcuts to specific spoken languages. When triggered, TypeWhisper records and transcribes audio using that language, overriding the global language setting.

### User-facing behaviour

- Open **Settings → Hotkeys → Spoken Language Shortcuts**
- Click **Add Language Shortcut** to add a new row
- Select the spoken language from the dropdown (populated from all enabled transcription engine plugins)
- Click **Record Shortcut** and press any key combo (e.g. `⌘⇧V`)
- Rows can be deleted individually with the `−` button
- Pressing the hotkey starts recording in the assigned language (hybrid mode: short press = toggle, hold = push-to-talk)
- The language override applies only for that recording session; the global language setting is unchanged

### Files changed

| File | Change |
|---|---|
| `TypeWhisper/App/UserDefaultsKeys.swift` | Added `languageHotkeys` key (`[LanguageHotkey]` JSON array) |
| `TypeWhisper/App/ServiceContainer.swift` | Call `dictationViewModel.loadLanguageHotkeys()` at startup |
| `TypeWhisper/Services/HotkeyService.swift` | Added `LanguageHotkey` struct, `LanguageHotkeyState`, `onLanguageDictationStart` callback, `activeLangId` tracking, `registerLanguageHotkeys()`, `isHotkeyAssignedToLanguageSlot()`, language slot processing in CGEventTap + NSEvent fallback, `handleLanguageKeyDown/Up` handlers |
| `TypeWhisper/ViewModels/DictationViewModel.swift` | Added `languageHotkeys` published array, `forcedLanguageCode` override, `startRecording(forcedLanguage:)` parameter, `effectiveLanguage` updated to respect forced language, full CRUD management methods, `loadLanguageHotkeys()` / `saveLanguageHotkeys()` |
| `TypeWhisper/Views/HotkeySettingsView.swift` | Added "Spoken Language Shortcuts" section with `ForEach` rows, `LanguageHotkeyRow` subview (language picker + hotkey recorder + delete button), Add button, conflict detection against global and language slots |

### Architecture notes

- `LanguageHotkey` is defined in `HotkeyService.swift` alongside `UnifiedHotkey` and `HotkeySlotType`
- Language hotkeys have an **optional** `hotkey` field — entries can exist with no shortcut assigned yet
- Hotkey behavior mirrors profile hotkeys: **hybrid** (short press = toggle, long press = PTT), using the same `handleLanguageKeyDown/Up` pattern as `handleProfileKeyDown/Up`
- `forcedLanguageCode` in `DictationViewModel` takes precedence over profile and global language settings in `effectiveLanguage`
- Conflict detection clears both global slots and other language slots when a duplicate shortcut is recorded
- Persistence: JSON-encoded `[LanguageHotkey]` stored in `UserDefaults` under key `languageHotkeys`

Resume this session with:
claude --resume 6e1afd50-bacf-4987-84eb-218c65fecbcb