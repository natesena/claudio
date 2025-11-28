# Claudio macOS App - Project Definition & Strategy

## Executive Summary

Transform claudio from a CLI-only hook tool into a professional macOS application with native UI for settings management, following the proven architecture of OpenSuperWhisper.

**Timeline:** 4-6 weeks
**Effort:** 40-60 hours
**Result:** Native macOS app with Homebrew distribution, DMG installer, code signing, and notarization

---

## Current State

### What Claudio Is Today

- **CLI binary** written in Go
- **Claude Code hook** that plays contextual sounds based on tool usage
- **Config-driven** via JSON at `/etc/xdg/claudio/config.json`
- **5-level fallback** sound mapping system
- **Cross-platform** audio using malgo (miniaudio wrapper)
- **XDG-compliant** configuration and logging
- **Installed via** `go install claudio.click/cmd/claudio@latest`

### Current Limitations

1. **No GUI** - All configuration requires manual JSON editing
2. **No visual feedback** - Users don't know if claudio is running or what state it's in
3. **No easy discovery** - Users must know config file location
4. **No quick toggles** - Can't easily mute/unmute or adjust volume
5. **Developer-only installation** - Requires Go toolchain
6. **No auto-update** - Users must manually update

---

## Desired State

### What Claudio Will Become

A **dual-component system**:

1. **Claudio.app** (Swift/SwiftUI)
   - Native macOS settings application
   - Beautiful, intuitive UI matching macOS design language
   - Optional menu bar icon for quick access
   - Real-time config editing with visual feedback
   - Soundpack preview and management
   - Log viewer and status display

2. **claudio binary** (Go - existing)
   - Continues as Claude Code hook handler
   - Reads config.json (no changes to core logic)
   - Audio playback engine (unchanged)
   - Hook processing system (unchanged)

### Key Improvements

1. **Professional Installation**
   - Homebrew cask: `brew install --cask claudio`
   - DMG download for non-Homebrew users
   - Code-signed and notarized
   - Automatic dependency handling

2. **Native macOS Experience**
   - SwiftUI interface following Apple HIG
   - Menu bar integration
   - System preferences-style settings
   - Dark/light mode support
   - Native controls and animations

3. **User-Friendly Configuration**
   - Visual toggles for all settings
   - Volume slider with live preview
   - Soundpack browser with audio previews
   - Per-category enable/disable (loading, success, error, interactive)
   - Config file location display

4. **Developer Quality**
   - Automated build pipeline
   - GitHub releases with DMG artifacts
   - Homebrew tap for easy distribution
   - Semantic versioning
   - Update notifications (optional)

---

## Architecture Overview

### Component Interaction

```
┌─────────────────────────────────────────────────────────────┐
│                        Claudio.app                          │
│                     (Swift/SwiftUI)                         │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │ Audio        │  │ Soundpack    │  │ Categories   │     │
│  │ Settings     │  │ Manager      │  │ Toggles      │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │ Advanced     │  │ Logs         │  │ About        │     │
│  │ Settings     │  │ Viewer       │  │ Info         │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                             │
│           Reads/Writes config.json                         │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
          ┌────────────────────────┐
          │   config.json          │
          │   /etc/xdg/claudio/    │
          └────────────┬───────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────────────────┐
│                    claudio binary                            │
│                      (Go - Hook)                             │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Hook         │  │ Sound        │  │ Audio        │      │
│  │ Parser       │  │ Mapper       │  │ Player       │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                              │
│              Claude Code stdin/stdout                        │
└──────────────────────────────────────────────────────────────┘
```

### Data Flow

1. **User adjusts settings** in Claudio.app
2. **App writes** to `config.json`
3. **Hook executes** (triggered by Claude Code)
4. **Hook reads** `config.json`
5. **Hook plays** sound based on config

### Shared Configuration Schema

```json
{
  "enabled": true,
  "volume": 0.5,
  "default_soundpack": "default",
  "soundpack_paths": ["/usr/local/share/claudio/default"],
  "log_level": "warn",
  "file_logging": {
    "enabled": true,
    "filename": "",
    "max_size_mb": 10,
    "max_backups": 5,
    "max_age_days": 30,
    "compress": true
  },
  "categories": {
    "loading": true,
    "success": true,
    "error": true,
    "interactive": true
  },
  "audio_backend": "malgo"
}
```

---

## Implementation Strategy

### Phase 1: Swift App Foundation (Week 1)

**Goal:** Create basic Claudio.app that can read/write config

#### Tasks
- [ ] Create Xcode project for Claudio
- [ ] Set up SwiftUI app structure
- [ ] Implement ConfigManager in Swift (JSON read/write)
- [ ] Create main window with tab view
- [ ] Build basic Audio Settings tab
  - [ ] Enable/disable toggle
  - [ ] Volume slider (0-100%)
  - [ ] Backend selector dropdown
- [ ] Test config reading/writing
- [ ] Add app icon (basic)

#### Deliverables
- Working Claudio.app that can modify config.json
- Single tab with basic settings
- Builds and runs locally

---

### Phase 2: Complete Settings UI (Week 2)

**Goal:** Full-featured settings interface

#### Tasks
- [ ] **Soundpack Tab**
  - [ ] List available soundpacks
  - [ ] Selection dropdown
  - [ ] Add custom soundpack path
  - [ ] Preview sounds button (plays sample)

- [ ] **Categories Tab**
  - [ ] Toggle for Loading sounds
  - [ ] Toggle for Success sounds
  - [ ] Toggle for Error sounds
  - [ ] Toggle for Interactive sounds
  - [ ] Description for each category

- [ ] **Advanced Tab**
  - [ ] File logging enable/disable
  - [ ] Log level dropdown (debug/info/warn/error)
  - [ ] Max file size slider
  - [ ] Config file location display
  - [ ] "Open Config Folder" button
  - [ ] "Open Log Folder" button

- [ ] **About Tab**
  - [ ] Version display
  - [ ] GitHub link
  - [ ] Documentation link
  - [ ] License info

#### Deliverables
- Complete settings UI with all tabs
- All settings sync to config.json
- Polished interface following macOS HIG

---

### Phase 3: Menu Bar Integration (Week 3)

**Goal:** Optional menu bar icon for quick access

#### Tasks
- [ ] Implement NSStatusBar integration
- [ ] Create menu bar icon (16x16 template)
- [ ] Build menu structure:
  - [ ] "Show Settings" → Opens main window
  - [ ] Separator
  - [ ] "Enabled" → Checkbox toggle
  - [ ] "Volume" → Submenu (25%, 50%, 75%, 100%)
  - [ ] Separator
  - [ ] "Quit Claudio"
- [ ] Implement window show/hide behavior
- [ ] Set app activation policy (.accessory when hidden)
- [ ] Handle menu bar icon clicks
- [ ] Sync menu checkmarks with config state

#### Deliverables
- Working menu bar integration
- Quick toggles without opening main window
- App stays in background when window closed

---

### Phase 4: Build System & Distribution (Week 4)

**Goal:** Professional build and release pipeline

#### Tasks

**Build Scripts**
- [ ] Create `build.sh`
  - [ ] Build for arm64 (Apple Silicon)
  - [ ] Code signing with Developer ID
  - [ ] Version stamping from git tag

- [ ] Create `notarize_app.sh`
  - [ ] Archive app for notarization
  - [ ] Submit to Apple notary service
  - [ ] Wait for notarization
  - [ ] Staple notarization ticket

- [ ] Create `make_release.sh`
  - [ ] Build app
  - [ ] Notarize app
  - [ ] Create DMG using create-dmg or swifty-dmg
  - [ ] Notarize DMG
  - [ ] Upload to GitHub releases
  - [ ] Generate Homebrew cask SHA256

**Packaging**
- [ ] Set up entitlements file
- [ ] Configure Info.plist
- [ ] Add app icon (full iconset)
- [ ] Create DMG background image (optional)
- [ ] Set DMG window size/position

**GitHub Actions** (optional automation)
- [ ] Workflow for building on tag push
- [ ] Automated notarization
- [ ] DMG creation and upload
- [ ] Homebrew tap update

#### Deliverables
- Build scripts that create signed, notarized app
- DMG installer
- GitHub release with artifacts

---

### Phase 5: Homebrew Distribution (Week 5)

**Goal:** Easy installation via Homebrew

#### Tasks
- [ ] Create Homebrew tap repository
  - [ ] `homebrew-claudio` (or add to existing tap)
  - [ ] Initialize with README

- [ ] Create Cask formula (`Casks/claudio.rb`)
```ruby
cask "claudio" do
  version "2.0.0"
  sha256 "..."

  url "https://github.com/natesena/claudio/releases/download/v#{version}/Claudio-#{version}.dmg"
  name "Claudio"
  desc "Audio feedback for Claude Code"
  homepage "https://github.com/natesena/claudio"

  app "Claudio.app"

  binary "#{appdir}/Claudio.app/Contents/MacOS/claudio"

  postflight do
    # Symlink CLI binary
    system_command "ln", args: ["-sf",
      "#{appdir}/Claudio.app/Contents/Resources/claudio",
      "/usr/local/bin/claudio"]
  end

  uninstall quit: "com.natesena.claudio"

  zap trash: [
    "~/.config/claudio",
    "~/.cache/claudio",
    "~/Library/Preferences/com.natesena.claudio.plist"
  ]
end
```

- [ ] Test installation: `brew install --cask natesena/tap/claudio`
- [ ] Document installation in README
- [ ] Add to homebrew-cask (optional, submit PR)

#### Deliverables
- Working Homebrew cask
- Installation command: `brew install --cask claudio`
- Automatic updates via `brew upgrade`

---

### Phase 6: Polish & Testing (Week 6)

**Goal:** Production-ready release

#### Tasks

**Visual Polish**
- [ ] App icon design (hire designer or use SF Symbols)
- [ ] Menu bar icon (template mode for dark/light)
- [ ] Consistent spacing/padding
- [ ] Loading states for config operations
- [ ] Error handling with user-friendly alerts
- [ ] Animations for settings changes (subtle)

**Testing**
- [ ] Test on fresh macOS install
- [ ] Test Homebrew installation flow
- [ ] Test DMG installation flow
- [ ] Test config migration from old version
- [ ] Test with different soundpacks
- [ ] Test with Claude Code integration
- [ ] Test menu bar on multi-monitor setup
- [ ] Test dark/light mode switching
- [ ] Test localization (if needed)

**Documentation**
- [ ] Update main README with app installation
- [ ] Create user guide (screenshots)
- [ ] Document keyboard shortcuts
- [ ] Update CLAUDE.md with new architecture
- [ ] Add troubleshooting guide
- [ ] Record demo video (optional)

**Release**
- [ ] Version bump to 2.0.0
- [ ] Tag release
- [ ] Run build/notarization/release pipeline
- [ ] Upload DMG to GitHub releases
- [ ] Update Homebrew cask
- [ ] Announce on relevant channels

#### Deliverables
- Claudio 2.0.0 released
- Polished UI and UX
- Complete documentation
- Public announcement

---

## Technical Specifications

### Swift/SwiftUI App Structure

```
Claudio/
├── ClaudiaApp.swift              # @main entry point, menu bar setup
├── ContentView.swift             # Main window with TabView
├── Views/
│   ├── Settings/
│   │   ├── AudioSettingsView.swift
│   │   ├── SoundpackView.swift
│   │   ├── CategoriesView.swift
│   │   ├── AdvancedView.swift
│   │   └── AboutView.swift
│   └── Components/
│       ├── VolumeSlider.swift
│       ├── ToggleRow.swift
│       └── SoundpackPicker.swift
├── Models/
│   ├── ClaudiaConfig.swift       # Codable struct matching JSON
│   └── Soundpack.swift
├── Managers/
│   ├── ConfigManager.swift       # Read/write config.json
│   ├── SoundpackManager.swift    # Discover/list soundpacks
│   └── MenuBarManager.swift      # NSStatusBar handling
├── Utils/
│   ├── XDGPaths.swift            # XDG directory resolution
│   ├── FileWatcher.swift         # Monitor config changes (optional)
│   └── AudioPreview.swift        # Play sample sounds
├── Resources/
│   ├── Assets.xcassets/
│   │   ├── AppIcon.appiconset/
│   │   └── MenuBarIcon.imageset/
│   └── Localizable.strings (optional)
├── Info.plist
└── Claudio.entitlements
```

### Key Swift Components

#### ConfigManager.swift
```swift
class ConfigManager: ObservableObject {
    @Published var config: ClaudiaConfig

    private let configURL: URL

    init() {
        // Resolve XDG config path
        let xdgConfig = ProcessInfo.processInfo.environment["XDG_CONFIG_HOME"]
            ?? FileManager.default.homeDirectoryForCurrentUser
                .appendingPathComponent(".config").path

        configURL = URL(fileURLWithPath: xdgConfig)
            .appendingPathComponent("claudio/config.json")

        config = Self.load(from: configURL)
    }

    func save() {
        // Write JSON to configURL
    }

    static func load(from url: URL) -> ClaudiaConfig {
        // Read and decode JSON
    }
}
```

#### ClaudiaConfig.swift
```swift
struct ClaudiaConfig: Codable {
    var enabled: Bool = true
    var volume: Double = 0.5
    var defaultSoundpack: String = "default"
    var soundpackPaths: [String] = []
    var logLevel: String = "warn"
    var fileLogging: FileLogging = FileLogging()
    var categories: Categories = Categories()
    var audioBackend: String = "malgo"

    struct FileLogging: Codable {
        var enabled: Bool = true
        var filename: String = ""
        var maxSizeMb: Int = 10
        var maxBackups: Int = 5
        var maxAgeDays: Int = 30
        var compress: Bool = true
    }

    struct Categories: Codable {
        var loading: Bool = true
        var success: Bool = true
        var error: Bool = true
        var interactive: Bool = true
    }
}
```

### Build Configuration

#### Info.plist Essentials
```xml
<key>CFBundleName</key>
<string>Claudio</string>
<key>CFBundleIdentifier</key>
<string>com.natesena.claudio</string>
<key>CFBundleShortVersionString</key>
<string>2.0.0</string>
<key>CFBundleVersion</key>
<string>1</string>
<key>LSMinimumSystemVersion</key>
<string>14.0</string>
<key>LSUIElement</key>
<false/>  <!-- true = hide from Dock always -->
```

#### Claudio.entitlements
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- Disable sandboxing for file system access -->
    <key>com.apple.security.app-sandbox</key>
    <false/>

    <!-- If we want to add auto-update later -->
    <key>com.apple.security.network.client</key>
    <true/>
</dict>
</plist>
```

---

## Distribution Strategy

### Primary: Homebrew Cask

**Why Homebrew Cask?**
- Standard for macOS developer tools
- Automatic updates via `brew upgrade`
- Trusted installation source
- Easy uninstall/reinstall
- Version management

**Installation Flow:**
```bash
# Add tap (first time only)
brew tap natesena/claudio

# Install
brew install --cask claudio

# Update
brew upgrade claudio

# Uninstall
brew uninstall --cask claudio
```

### Secondary: DMG Download

**For users without Homebrew:**
- Download from GitHub releases
- Drag Claudio.app to Applications folder
- First launch: Right-click → Open (to bypass Gatekeeper warning if any)

### Tertiary: Keep `go install` for CLI-only

**For minimalists:**
```bash
go install claudio.click/cmd/claudio@latest
```

---

## Code Signing & Notarization

### Requirements

1. **Apple Developer Account** ($99/year)
2. **Developer ID Application Certificate**
3. **App-specific password** for notarization

### Process

```bash
# 1. Sign the app
codesign --deep --force --verify --verbose \
  --sign "Developer ID Application: Your Name (TEAMID)" \
  --options runtime \
  Claudio.app

# 2. Create ZIP for notarization
ditto -c -k --keepParent Claudio.app Claudio.zip

# 3. Submit for notarization
xcrun notarytool submit Claudio.zip \
  --apple-id "your@email.com" \
  --team-id "TEAMID" \
  --password "app-specific-password" \
  --wait

# 4. Staple the notarization
xcrun stapler staple Claudio.app

# 5. Create DMG
create-dmg \
  --volname "Claudio Installer" \
  --window-pos 200 120 \
  --window-size 600 400 \
  --icon-size 100 \
  --app-drop-link 450 185 \
  Claudio.dmg \
  Claudio.app

# 6. Sign the DMG
codesign --sign "Developer ID Application: Your Name (TEAMID)" Claudio.dmg

# 7. Notarize the DMG
xcrun notarytool submit Claudio.dmg ...
xcrun stapler staple Claudio.dmg
```

---

## Version Strategy

### Semantic Versioning

- **Current:** v1.x.x (CLI-only era)
- **New:** v2.0.0 (macOS app debut)

### Version Increments

- **2.0.0** - Initial macOS app release
- **2.1.0** - Add soundpack preview feature
- **2.2.0** - Add log viewer in app
- **2.x.0** - Minor feature additions
- **2.x.x** - Bug fixes, patch releases
- **3.0.0** - Major architectural changes (if ever needed)

### Compatibility

- **Config format:** Maintain backward compatibility
- **CLI binary:** v2.x CLI can read v1.x configs
- **Migration:** Automatic on first launch if needed

---

## Success Metrics

### Technical
- [ ] App installs via Homebrew without errors
- [ ] App passes notarization
- [ ] Config changes reflect immediately in hook behavior
- [ ] No crashes in normal usage
- [ ] Works on macOS 14.0+ (Sonoma and later)

### User Experience
- [ ] Settings UI is intuitive (no documentation needed for basic use)
- [ ] Volume changes feel responsive
- [ ] Soundpack switching is instant
- [ ] Menu bar provides quick access to common actions

### Distribution
- [ ] DMG downloads from GitHub releases
- [ ] Homebrew cask formula works on fresh install
- [ ] Auto-update notification (if implemented)
- [ ] Less than 30 seconds from brew command to usable app

---

## Future Enhancements (Post-2.0)

### v2.1+
- [ ] In-app soundpack installer (download from repo)
- [ ] Soundpack creator/editor
- [ ] Visual waveform preview
- [ ] Custom keyboard shortcuts
- [ ] Launch at login option

### v2.2+
- [ ] Real-time log viewer with filtering
- [ ] Event history (last 100 hook events)
- [ ] Statistics dashboard (most common tools, sounds played)
- [ ] Export/import settings

### v3.0+
- [ ] Multi-profile support (work vs personal)
- [ ] Sound customization per tool (beyond categories)
- [ ] Cloud sync of settings (iCloud)
- [ ] iOS companion app (view stats on phone)

---

## Risk Mitigation

### Risk: Code Signing Issues
**Mitigation:** Test notarization early, have backup Apple ID, document process

### Risk: Homebrew Cask Rejection
**Mitigation:** Follow official guidelines, provide checksums, test on clean system

### Risk: Config Format Incompatibility
**Mitigation:** Version config schema, write migration logic, test v1→v2 upgrade

### Risk: Swift/Go Integration Complexity
**Mitigation:** Keep concerns separated, use file-based communication (config.json)

### Risk: User Confusion (Two Installation Methods)
**Mitigation:** Clear documentation, recommend Homebrew as primary, deprecate `go install` in v3.0

---

## Resources Required

### Development Tools
- Xcode 15+ (free)
- macOS 14+ (for testing)
- Apple Developer Account ($99/year)
- GitHub account (free)

### Design Assets
- App icon (hire designer ~$50-200, or use SF Symbols)
- Menu bar icon (can create with SF Symbols)
- DMG background (optional, ~$0-50)

### Infrastructure
- GitHub repository (existing)
- GitHub releases storage (free)
- Homebrew tap repository (free)
- Domain for homepage (optional, ~$15/year)

### Time Investment
- **Phase 1:** 8-12 hours
- **Phase 2:** 10-15 hours
- **Phase 3:** 6-8 hours
- **Phase 4:** 8-12 hours
- **Phase 5:** 4-6 hours
- **Phase 6:** 8-12 hours
- **Total:** 44-65 hours

---

## Conclusion

This project transforms claudio from a developer-focused CLI tool into a polished macOS application while maintaining its core strengths. By following OpenSuperWhisper's proven architecture, we ensure a native macOS experience and professional distribution pipeline.

The dual-component approach (Swift UI + Go binary) allows us to leverage the best of both worlds: native macOS UI frameworks and Go's excellent audio/cross-platform capabilities.

**Next Steps:**
1. Review and approve this document
2. Set up Apple Developer account (if not already)
3. Create Xcode project (Phase 1 start)
4. Begin implementation following the phased approach

---

**Document Version:** 1.0
**Created:** 2025-11-28
**Author:** Claude Code with @natesena
**Status:** Awaiting Approval
