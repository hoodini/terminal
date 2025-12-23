# LLM Instructions: YUV AI Terminal Installation

> This file provides autonomous installation guidance for AI agents and LLMs.

## Overview

**Project:** YUV AI Terminal  
**Author:** Yuval Avidani (@hoodini)  
**Purpose:** Windows Terminal fork with Hebrew RTL support  
**Platform:** Windows 10 2004+ / Windows 11

---

## Quick Install (Recommended)

For users who just want to use Windows Terminal with RTL support:

```powershell
winget install --id Microsoft.WindowsTerminal -e
```

This installs the official Windows Terminal. The RTL fork requires building from source.

---

## Autonomous Build from Source

### Prerequisites Check

Run these commands to verify the system is ready:

```powershell
# Check Windows version (must be 10.0.19041.0 or later)
[System.Environment]::OSVersion.Version

# Check if Developer Mode is enabled
$devMode = (Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\AppModelUnlock" -Name "AllowDevelopmentWithoutDevLicense" -ErrorAction SilentlyContinue).AllowDevelopmentWithoutDevLicense
if ($devMode -ne 1) { Write-Host "ERROR: Enable Developer Mode in Settings > Privacy & security > For developers" }

# Check PowerShell version (must be 7+)
$PSVersionTable.PSVersion
```

### Step 1: Clone Repository

```powershell
git clone https://github.com/hoodini/terminal.git
cd terminal
```

### Step 2: Install Dependencies (Automated)

**Option A: WinGet Configuration (Recommended)**
```powershell
winget configure .config\configuration.winget
```

**Option B: Manual Installation**
```powershell
# Install Visual Studio 2022 Build Tools with required components
winget install Microsoft.VisualStudio.2022.BuildTools --force --override "--wait --passive --add Microsoft.VisualStudio.Workload.VCTools --add Microsoft.VisualStudio.Component.VC.Tools.x86.x64 --add Microsoft.VisualStudio.Component.Windows11SDK.22621 --add Microsoft.VisualStudio.ComponentGroup.UWP.Support --add Microsoft.VisualStudio.Component.UWP.Support"

# Install Windows SDK
winget install Microsoft.WindowsSDK.10.0.22621

# Install PowerShell 7 if not present
winget install Microsoft.PowerShell
```

### Step 3: Build

```powershell
# Import build module
Import-Module .\tools\OpenConsole.psm1

# Set up build environment
Set-MsBuildDevEnvironment

# Build the terminal (Release x64)
Invoke-OpenConsoleBuild
```

**Alternative: Command Prompt Build**
```cmd
.\tools\razzle.cmd
bcz
```

### Step 4: Register and Run

```powershell
# Build and register the Terminal package
Import-Module .\tools\OpenConsole.psm1
Set-MsBuildDevEnvironment

# Navigate to package directory
Set-Location -Path .\src\cascadia\CascadiaPackage\AppPackages\CascadiaPackage_0.0.1.0_x64_Debug_Test

# Remove existing dev version if present
$existing = Get-AppxPackage -Name 'WindowsTerminalDev*'
if ($existing) { Remove-AppxPackage $existing.PackageFullName }

# Unpack and register
New-Item ..\loose -Type Directory -Force
makeappx unpack /v /o /p .\CascadiaPackage_0.0.1.0_x64_Debug.msix /d ..\Loose\
Add-AppxPackage -Path ..\loose\AppxManifest.xml -Register -ForceUpdateFromAnyVersion -ForceApplicationShutdown

# Run the terminal
wtd.exe
```

---

## Verification

### Test RTL Support

After installation, verify Hebrew RTL rendering:

```powershell
# Display Hebrew text
Write-Host "שלום עולם"
Write-Host "Hello עולם - Mixed text"
Write-Host "מספר 123 בעברית"

# Open test file
cat .\doc\hebrew-test-examples.txt
```

### Expected Behavior

| Test | Expected Result |
|------|-----------------|
| Pure Hebrew `שלום` | Text renders right-to-left |
| Mixed `Hello עולם` | English left, Hebrew right |
| Numbers `123 בעברית` | Numbers embedded correctly |

---

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| `Developer Mode not enabled` | Settings > Privacy & security > For developers > Enable |
| `MSBuild not found` | Run `Set-MsBuildDevEnvironment` or reinstall VS Build Tools |
| `PowerShell version < 7` | `winget install Microsoft.PowerShell` |
| `Package registration fails` | Run PowerShell as Administrator |
| `Missing Windows SDK` | `winget install Microsoft.WindowsSDK.10.0.22621` |

### Reset Build Environment

```powershell
# Clean build artifacts
git clean -xfd
git reset --hard HEAD

# Rebuild
Import-Module .\tools\OpenConsole.psm1
Set-MsBuildDevEnvironment
Invoke-OpenConsoleBuild
```

---

## File Structure Reference

```
terminal/
├── src/renderer/atlas/          # RTL implementation
│   ├── AtlasEngine.cpp          # AnalyzeBidi call
│   ├── DWriteTextAnalysis.cpp   # Hebrew detection (U+0590-U+05FF)
│   └── common.h                 # BiDi level storage
├── doc/
│   ├── hebrew-rtl-support.md    # RTL feature docs
│   ├── IMPLEMENTATION_SUMMARY.md
│   └── hebrew-test-examples.txt # Test cases
├── tools/
│   ├── OpenConsole.psm1         # Build module
│   └── razzle.cmd               # CMD build script
└── .config/
    └── configuration.winget     # Auto-setup config
```

---

## For AI Agents

### Decision Tree

```
START
  │
  ├─ User wants quick install? → winget install Microsoft.WindowsTerminal
  │
  ├─ User wants RTL fork?
  │   │
  │   ├─ Check prerequisites (Windows version, Dev Mode, PS7)
  │   │
  │   ├─ Clone repo: git clone https://github.com/hoodini/terminal.git
  │   │
  │   ├─ Install deps: winget configure .config\configuration.winget
  │   │
  │   ├─ Build: Import-Module .\tools\OpenConsole.psm1; Set-MsBuildDevEnvironment; Invoke-OpenConsoleBuild
  │   │
  │   └─ Register & run package
  │
  └─ Verify: Test Hebrew text rendering
```

### Key Commands Summary

```powershell
# Full autonomous install sequence
git clone https://github.com/hoodini/terminal.git
cd terminal
winget configure .config\configuration.winget
Import-Module .\tools\OpenConsole.psm1
Set-MsBuildDevEnvironment
Invoke-OpenConsoleBuild
```

---

## Contact

- **Author:** Yuval Avidani
- **GitHub:** [@hoodini](https://github.com/hoodini)
- **Website:** [YUV.AI](https://yuv.ai)
- **Issues:** [github.com/hoodini/terminal/issues](https://github.com/hoodini/terminal/issues)
