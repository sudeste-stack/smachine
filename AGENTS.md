# AI Development Guardrails (agents.md)

**Purpose:** Prevent AI slop, hallucinations, and redundancy in sMachine codebase. Enforce code-first philosophy and strict modularity.

---

## Core Principles

### 1. Code is Source of Truth

- **Actual code > documentation**
- Assume documentation is older than code
- Treat documentation as overview, not specification
- Before trusting a doc claim, verify against actual implementation

### 2. Strict Modularity (Write Once, Link Always)

- **Never duplicate information** across files
- Each concept lives in exactly one file
- Cross-reference via markdown links
- External files are mandatory for:
  - UI screens → `repo/screens/*.screen`
  - Tables/matrices → `repo/tables/*.csv`
  - Hardware configs → `repo/configs/*.yaml`
  - Code examples → `repo/examples/*.sh`

### 3. Documentation as Overview Only

- Docs describe *what* and *why*, not *how*
- *How* lives in actual code (comments, function names, logic)
- Docs decay over time; code doesn't lie
- High-level guidance. Implementation details in code.

### 4. AI Agent Responsibilities

AI must:
- Fact-check documentation against actual code *before* proposing changes
- Refuse to write duplicated information; propose link instead
- Validate all external file references (no broken links)
- Catch hallucinated package names, paths, systemd syntax
- Check documentation alignment before & after every code change
- Update docs *after* code is written, not before
- Flag outdated docs discovered during implementation

---

## Pre-Coding Checklist

Before writing ANY code or documentation:

1. **Scope mapping**
   - What files will this touch?
   - What documentation exists for this area?
   - Is there redundancy with existing files?

2. **Documentation audit**
   - Read relevant .md files
   - Check if docs match intended implementation
   - Identify stale or contradictory statements

3. **Modularity check**
   - Does this concept already exist elsewhere?
   - Should I create a new external file (screen/table/config)?
   - What will I link to?

4. **Fact-check baseline**
   - Verify doc examples against actual code behavior
   - Identify places where code will diverge from docs
   - Plan doc updates needed *after* coding

**Example Pre-Coding Conversation:**
```
Task: Add WireGuard configuration to installer

Pre-check:
✗ CONFIGURATION.md mentions WireGuard (outdated example)
✗ docs/SETUP.md also mentions WireGuard (redundant)
→ Consolidate into single WireGuard.md
✗ GPU.md has env var examples for VA-API (separate concern)
→ Keep GPU.md; link from CONFIGURATION.md
✓ No code yet; docs are stale
→ Write code first, update docs after
```

---

## Post-Coding Checklist

After writing code:

1. **Fact-check code against docs**
   - Does actual implementation match documented behavior?
   - Any undocumented side effects?
   - Any documented features not in code?

2. **Update documentation**
   - Align docs with actual code behavior
   - Update examples to match real output
   - Fix any false claims

3. **Link validation**
   - All markdown links point to real files?
   - External file references (screens/, tables/) exist?
   - No broken cross-references?

4. **Redundancy scan**
   - Did I duplicate info from another file?
   - Should I link instead?
   - Can I consolidate overlapping docs?

**Example Post-Coding Conversation:**
```
Code written: arch-retrogaming-installer.sh (GPU detection logic)

Fact-check:
✓ Code detects iGPU via lspci (matches GPU.md)
✗ Code uses $GPU_DETECTED variable; GPU.md says /tmp/gpu_config
→ Update GPU.md to match actual variable name
✗ Code installs nouveau by default; docs say "user selects"
→ Update GPU.md first-boot menu section
✓ Code references BIOS settings; links to GPU.md (good)
→ No broken links

Result: GPU.md updated. Code & docs aligned.
```

---

## Modularity Rules (Mandatory)

### External File Requirements

**UI Screens** (`.screen` files in `repo/screens/`)
```
repo/screens/gpu-configuration.screen
repo/screens/network-setup.screen
repo/screens/storage-partitioning.screen
```

Usage in markdown:
```markdown
See the [GPU Configuration screen](../../screens/gpu-configuration.screen) for layout.
```

**Hardware/Compatibility Tables** (`.csv` in `repo/tables/`)
```
repo/tables/igpu-drivers.csv
repo/tables/discrete-gpu-support.csv
repo/tables/cpu-igpu-mapping.csv
```

Usage in markdown:
```markdown
Refer to this [GPU compatibility table](../../tables/igpu-drivers.csv) for driver selection.
```

**Configuration Examples** (`.yaml` or `.conf` in `repo/configs/`)
```
repo/configs/wireguard-example.conf
repo/configs/samba-example.conf
repo/configs/systemd-service-template.service
```

Usage in markdown:
```markdown
See [example WireGuard config](../../configs/wireguard-example.conf) for reference.
```

**Code Examples** (`.sh` scripts in `repo/examples/`)
```
repo/examples/gpu-detection.sh
repo/examples/btrfs-snapshot-rollback.sh
repo/examples/first-boot-menu.sh
```

Usage in markdown:
```markdown
Example implementation in [gpu-detection.sh](../../examples/gpu-detection.sh).
```

### Information Duplication Forbidden

❌ **Bad:**
```markdown
# GPU.md
Supported iGPU: Intel HD Graphics, AMD Radeon

# CONFIGURATION.md
For Intel HD Graphics, use libva-intel-driver...
For AMD Radeon, use AMDGPU...
```
(Same info in two places)

✅ **Good:**
```markdown
# GPU.md
Supported iGPU: See [GPU compatibility table](../../tables/igpu-drivers.csv)

# CONFIGURATION.md
Driver selection: See [GPU.md](GPU.md) for hardware support matrix.
```
(Single source, linked everywhere)

---

## Hallucination Prevention

### Package Name Validation

Before suggesting `pacman -S <pkg>`, verify:
1. Package exists in Arch repos: https://archlinux.org/packages/
2. Check spelling (e.g., `libva-intel-driver`, not `intel-libva-driver`)
3. Verify architecture support (x86_64, aarch64)
4. Check if package is in AUR (mark as `[AUR]` in docs)

**Example:**
```
❌ pacman -S nvidia-drivers
✓ pacman -S nvidia nvidia-utils

❌ pacman -S intel-media
✓ pacman -S intel-media-driver
```

### Systemd Syntax Validation

Before suggesting systemd configs:
1. Validate service file syntax (man systemd.service)
2. Check timer syntax (man systemd.timer)
3. Verify environment variable expansion (${VAR})
4. Test ExecStart paths exist

**Example:**
```
❌ [Service]
    ExecStart=/usr/local/bin/nonexistent-script.sh

✓ [Service]
    ExecStart=/usr/bin/bash -c '/path/to/script.sh'
```

### Path & Variable Validation

Before suggesting paths:
1. Check if path exists in Arch defaults
2. Verify variable expansion (e.g., `$HOME`, `${XDG_CONFIG_HOME}`)
3. Test relative paths in examples

**Example:**
```
❌ /etc/sudeste/config.conf (nonstandard location)
✓ /etc/smachine/config.conf (follows Arch conventions)

❌ ${VAR_UNDEFINED}/path
✓ /retrodeck/${HOSTNAME}/saves (uses real vars)
```

### Hardware Assumption Validation

Before claiming GPU/CPU support:
1. Cross-check against Arch wiki (Hardware video acceleration)
2. Verify driver support in actual upstream docs
3. Test against real hardware if possible
4. Mark untested assumptions with `[UNTESTED]`

**Example:**
```
❌ NVIDIA GeForce 400+ series supported (too broad)
✓ NVIDIA GeForce 400+ supported by Nouveau [limited performance]
✓ NVIDIA GeForce 400+ supported by nvidia-utils [proprietary, risky]
```

---

## Redundancy Detection

### Scan for Duplication

Before finalizing docs/code:

1. **Search across all .md files** for repeated phrases
   ```bash
   grep -r "VA-API driver" docs/
   ```
   If multiple hits → consolidate into one file, link elsewhere

2. **Check bash scripts** for repeated functions
   ```bash
   grep -n "detect_gpu\|detect-gpu" *.sh
   ```
   If duplicated → extract to shared library

3. **Review systemd files** for repeated configuration
   - Environment variables set in multiple services?
   - Similar ExecStart logic? → Use shared script

4. **Audit external files** for overlapping tables
   - Two CSV files with GPU info? → Merge with different columns

### Redundancy Report Template

When redundancy found:
```
Redundancy Detected:
- File A: Line X mentions [concept]
- File B: Line Y mentions [concept]
- Status: [Keep in A, link from B] OR [Consolidate to new file]
- Action: Update links. Verify no broken references.
```

---

## Documentation Alignment Workflow

### Before Coding

**Step 1: Read existing docs**
```
GPU.md → check current GPU strategy
ARCHITECTURE.md → check boot flow
docs/SETUP.md → check first-boot flow
```

**Step 2: Identify gaps/conflicts**
```
GPU.md says: "iGPU-first"
Code (missing): No GPU detection logic yet
→ Document will guide implementation
```

**Step 3: Plan code + doc updates**
```
Will write:
- arch-retrogaming-installer.sh (GPU detection function)
- systemd service for first-boot menu
- Update GPU.md with actual var names
```

### After Coding

**Step 1: Verify code matches docs**
```
GPU.md claims: /tmp/gpu_config for detection results
Actual code: Uses /tmp/smachine_gpu_config
→ Update GPU.md
```

**Step 2: Test documented examples**
```
SETUP.md example: pacman -S nvidia
Actual Arch 2026: nvidia-utils needed separately
→ Update example. Test on real system.
```

**Step 3: Link validation**
```
GPU.md links to: ../../tables/igpu-drivers.csv
File exists? Yes ✓
Content matches? Yes ✓
No broken links ✓
```

**Step 4: Commit with alignment note**
```
Commit message:
"feat: GPU detection in installer

- Add lspci-based GPU detection (arch-retrogaming-installer.sh)
- Add systemd first-boot GPU menu service
- Update GPU.md: /tmp/smachine_gpu_config variable naming
- Update tables/igpu-drivers.csv: Add Intel Arc entries
- Verified: All doc examples tested on Arch 2026
"
```

---

## AI Agent Checklist Template

Use this before every task:

```
Task: [describe what to build/write]

PRE-CODING:
☐ Identified all files this touches
☐ Read existing documentation (outdated?)
☐ Checked for redundancy with other files
☐ Fact-checked docs against any existing code
☐ Planned external file structure (screens/tables/configs)
☐ Identified doc updates needed *after* coding

CODING:
☐ Wrote actual code (not just docs)
☐ Used clear variable/function names (self-documenting)
☐ Added code comments for complex logic
☐ Created external files where needed (screens/, tables/)
☐ No hallucinated package names / paths / syntax
☐ Tested examples (if applicable)

POST-CODING:
☐ Fact-checked code against existing docs
☐ Updated docs to match actual code behavior
☐ Validated all markdown links
☐ Scanned for redundancy
☐ Verified no broken references
☐ Created commit message with alignment note

STATUS: ✓ Ready for review
```

---

## References & Guardrails

- **Arch Linux Packages:** https://archlinux.org/packages/
- **Arch Wiki:** https://wiki.archlinux.org/
- **Systemd Documentation:** https://www.freedesktop.org/software/systemd/man/
- **RetroDECK Docs:** https://retrodeck.readthedocs.io/
- **sMachine Repo:** https://github.com/sudeste-stack/smachine

---

## Exception Cases

**When to break modularity:**
- Code comments explaining *why* (not *what*)
- Emergency hotfixes (mark with `[HOTFIX]`, plan refactor)
- Inline examples 2-3 lines or less (longer → external file)

**When docs can lead code:**
- Architecture decisions (big picture)
- User-facing workflows (UX first, then code)
- Security policies (rules first, then enforce in code)

---

## Status: DRAFT

Ready for team review. Questions?
