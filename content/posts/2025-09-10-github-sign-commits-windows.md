---
title: "Getting Those Green Verified Badges: GPG Commit Signing on Windows That Actually Works"
date: 2025-09-10
draft: false
tags: ["git", "gpg", "windows", "security", "github", "development"]
categories: ["Development", "Security"]
description: "In this article, I'm going to cover the complete setup for GPG commit signing on Windows, including the wrapper script solution that actually works."
featured_image: ""
---

In this article, I'm going to cover the complete setup for GPG commit signing on Windows, including the wrapper script solution that actually works.

I really love seeing those green "Verified" badges next to commits on GitHub. There's something satisfying about cryptographic proof that your code actually came from you. But if you've tried setting up GPG commit signing on Windows, you've probably run into the same frustrating issues I did - Git can't find your GPG key even though `gpg --list-keys` works perfectly fine in your terminal.

I spent hours troubleshooting keyboxd daemon conflicts, lockfile errors, and environment variable mismatches between PowerShell and Git's execution context. The usual guides just tell you to install GPG4Win and configure a few settings, but they completely miss the critical environment issues that make it fail on Windows.

Once you get the setup right, though, it becomes a seamless part of your workflow. Every commit gets automatically signed, and you never have to think about it again.


## Why Bother with GPG Signing?

GPG signing provides cryptographic proof that commits actually came from you. It's particularly valuable when you're contributing to open source projects where maintainers need to verify authentic contributions, or in enterprise environments that require signed commits for security compliance.

But honestly, I just like those green verified badges. They look professional.

## The Real Problem on Windows

The main issue is that Git for Windows and GPG4Win don't talk to each other properly. You'll install both, generate a key, configure Git, and then get hit with "No secret key" errors even though GPG works fine when you test it directly.

The root cause is environment variables. Git runs GPG in a different context than your PowerShell session, with different paths and missing environment variables. Most guides don't address this fundamental issue.

## The Working Solution

Here's what actually works. I'm going to walk you through the exact steps I use on every fresh Windows install.

### Step 1: Install the Software

Download and install [Git for Windows](https://git-scm.com/download/win) and [GPG4Win](https://www.gpg4win.org/download.html) with default settings.

Verify they're working:
```powershell
git --version
gpg --version
```

### Step 2: Generate Your GPG Key

```powershell
gpg --full-generate-key
```

Choose RSA and RSA, 4096 bits, set your expiration preference, and use the same email as your GitHub account. Create a strong passphrase.

### Step 3: Get Your Key ID and Export for GitHub

```powershell
gpg --list-secret-keys --keyid-format=long
```

Note the key ID after the `/` on the `sec` line. Then export your public key:

```powershell
gpg --armor --export YOUR_KEY_ID
```

Copy the entire output and add it to GitHub under Settings → SSH and GPG keys → New GPG key.

### Step 4: The Magic Wrapper Script

This is the part that most guides miss. Create this wrapper script to fix the environment issues:

```powershell
New-Item -ItemType Directory -Force -Path C:\temp

@'
@echo off
set GNUPGHOME=%USERPROFILE%\AppData\Roaming\gnupg
set GPG_TTY=CON
set GPG_AGENT_INFO=
set GPGCONF=
set GPG_AGENT_PROGRAM="%ProgramFiles(x86)%\GnuPG\bin\gpg-agent.exe"

rem Clean up any lockfiles
del "%GNUPGHOME%\*.lock" >nul 2>&1

rem Force use Windows GPG agent
set PATH=%ProgramFiles(x86)%\GnuPG\bin;%PATH%

gpg.exe %*
'@ | Out-File -FilePath C:\temp\git-gpg-wrapper.bat -Encoding ASCII -Force
```

### Step 5: Configure Git

```powershell
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git config --global gpg.program "C:\temp\git-gpg-wrapper.bat"
git config --global user.signingkey YOUR_SHORT_KEY_ID
git config --global gpg.format openpgp
git config --global commit.gpgsign true
```

Replace `YOUR_SHORT_KEY_ID` with your actual short key ID.

### Step 6: Test It

```powershell
mkdir test-repo
cd test-repo
git init
git commit --allow-empty -S -m "Test signed commit"
```

If you see a commit hash without errors, you're golden.

## Why This Actually Works

The wrapper script solves four critical problems:

1. Sets `GNUPGHOME` correctly for Git's execution context
2. Points to the Windows GPG agent instead of trying Linux paths
3. Cleans up lockfiles that cause signing failures
4. Ensures the right PATH ordering so Git finds the correct GPG

Without this wrapper, Git runs GPG in an environment that can't find your keys, even though they exist and work fine in your normal shell.

## Troubleshooting

If you get "No secret key" errors, double-check your key ID:
```powershell
git config --global user.signingkey
gpg --list-secret-keys --keyid-format=long
```

Make sure they match exactly.

For lockfile errors, clean up manually:
```powershell
Remove-Item -Path "$env:USERPROFILE\AppData\Roaming\gnupg\*.lock" -Force -ErrorAction SilentlyContinue
```

If the GPG agent won't start:
```powershell
gpg-connect-agent killagent /bye
gpg-connect-agent /bye
```

## Bonus: Quick Setup Script

Here's a PowerShell script that automates most of the setup:

```powershell
param(
    [Parameter(Mandatory=$true)]
    [string]$Name,
    
    [Parameter(Mandatory=$true)]
    [string]$Email
)

New-Item -ItemType Directory -Force -Path C:\temp
@'
@echo off
set GNUPGHOME=%USERPROFILE%\AppData\Roaming\gnupg
set GPG_TTY=CON
set GPG_AGENT_INFO=
set GPGCONF=
set GPG_AGENT_PROGRAM="%ProgramFiles(x86)%\GnuPG\bin\gpg-agent.exe"
del "%GNUPGHOME%\*.lock" >nul 2>&1
set PATH=%ProgramFiles(x86)%\GnuPG\bin;%PATH%
gpg.exe %*
'@ | Out-File -FilePath C:\temp\git-gpg-wrapper.bat -Encoding ASCII -Force

git config --global user.name $Name
git config --global user.email $Email
git config --global gpg.program "C:\temp\git-gpg-wrapper.bat"
git config --global gpg.format openpgp

Write-Host "Setup complete! Now run 'gpg --full-generate-key' to create your key."
Write-Host "Then set your signing key with: git config --global user.signingkey YOUR_KEY_ID"
```

Save as `setup-gpg.ps1` and run:
```powershell
.\setup-gpg.ps1 -Name "Your Name" -Email "your.email@example.com"
```

Just remember to keep that wrapper script at `C:\temp\git-gpg-wrapper.bat` - Git depends on it. If you reinstall GPG4Win, you might need to recreate it.

For today that's it. Thanks for reading, and enjoy your verified commits!
