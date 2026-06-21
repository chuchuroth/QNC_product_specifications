
---
# Repository Reorganization Migration Plan

**Repository:** chuchuroth/Project_Neura_Robotics  
**Date:** 2026-02-14  
**Status:** Draft  
**Version:** 1.0

## Executive Summary

This document outlines a phased approach to reorganize the Project_Neura_Robotics repository, which currently has 25 files in a flat structure. The reorganization will improve maintainability, discoverability, and collaboration by establishing a clear hierarchical structure with proper separation of concerns.

---

## Current State Analysis

### File Inventory (25 files total)

```
Root Directory (flat structure)
├── Documentation Files (19 .md files)
├── Source Code (3 .hpp files, 1 .cpp file)
├── Screenshots (2 .png files)
└── No clear organization or hierarchy
```

#### Breakdown by Type:

**Architecture & Design (4 files)**
- `software_architecture_mipa.md` (96 KB) - Comprehensive architecture doc
- `mipax_software_refactor.md` (32 KB) - Refactoring plans
- `MiPA.md` (8.7 KB) - Hardware platform documentation
- `mipa轻量化.md` (29 KB) - Lightweight MiPA design (Chinese)

**Protocol Specifications (2 files)**
- `ModbusRTU_IDL.md` (50 KB) - **MIXED CONTENT** (specs + test logs)
- `Modbus_generalized_IDL.md` (44 KB) - Generalized protocol specs

**Hardware Documentation (2 files)**
- `QNC.md` (6.2 KB) - QNC hardware specs
- `QNC_confluence.md` (6.5 KB) - **DUPLICATE/OVERLAP** with QNC.md

**Research & Innovation (2 files)**
- `innovations.md` (47 KB) - Innovation proposals and concepts
- `advanced_topics.md` (6.6 KB) - Advanced technical topics

**Strategic Planning (1 file)**
- `claude_talk.md` (41 KB) - **NEEDS CURATION** - Long strategic discussions

**Operational Documents (5 files)**
- `mipa_chips_research.md` (3.2 KB) - Chip research notes
- `mipa_repair_repot.md` (1.2 KB) - Repair report (typo in filename)
- `info_from_RM.md` (1.3 KB) - Information from project manager
- `VLM+FGPA.md` (4.3 KB) - Vision-Language Model + FPGA notes
- `ppt.md` (4.2 KB) - Presentation notes

**Development Resources (2 files)**
- `MR_template.md` (458 B) - Merge request template
- `README.md` (4.2 KB) - Current project readme

**Source Code (4 files)**
- `motor_control_rt.hpp` (4.5 KB) - Real-time motor control
- `multisensor_node.hpp` (5.4 KB) - Multi-sensor ROS node
- `pointcloud_pipeline.hpp` (1.8 KB) - Point cloud processing
- `Publisher(zero_copy).cpp` (2.5 KB) - Zero-copy publisher implementation

**Assets (2 files)**
- `Screenshot 2026-01-10 212328.png` (35 KB)
- `Screenshot 2026-01-10 212518.png` (35 KB)

### Key Issues Identified

1. ❌ **Flat Structure**: All 25 files in root directory
2. ⚠️ **Mixed Content**: `ModbusRTU_IDL.md` contains both specs AND test logs
3. ⚠️ **Content Overlap**: `QNC.md` and `QNC_confluence.md` have redundant information
4. ⚠️ **Oversized Files**: `claude_talk.md` (41 KB) needs curation and splitting
5. 🔤 **Naming Issues**: `mipa_repair_repot.md` (typo: "repot" → "report")
6. 🌐 **Mixed Languages**: Chinese filename `mipa轻量化.md` may affect cross-platform compatibility
7. 📦 **No Separation**: Code, docs, and assets all intermixed

---

## Target Structure

### Proposed Directory Tree

```
Project_Neura_Robotics/
│
├── README.md                          # Enhanced project overview
├── ARCHITECTURE.md                    # NEW: High-level architecture summary
├── CONTRIBUTING.md                    # NEW: Development guidelines
├── CHANGELOG.md                       # NEW: Project changelog
│
├── docs/                              # 📚 All documentation
│   ├── README.md                      # Documentation index & navigation
│   │
│   ├── architecture/                  # System architecture & design
│   │   ├── README.md
│   │   ├── software_architecture_mipa.md
│   │   ├── mipax_software_refactor.md
│   │   ├── system_overview.md        # NEW: Extracted high-level overview
│   │   └── design_decisions.md       # NEW: From claude_talk.md
│   │
│   ├── hardware/                      # Hardware specifications
│   │   ├── README.md
│   │   ├── mipa_platform.md          # Renamed from MiPA.md
│   │   ├── mipa_lightweight.md       # Renamed from mipa轻量化.md
│   │   ├── qnc_specification.md      # Consolidated from QNC files
│   │   └── chip_research.md          # From mipa_chips_research.md
│   │
│   ├── protocols/                     # Communication protocols
│   │   ├── README.md
│   │   ├── modbus_rtu_specification.md    # SPLIT: Specs only
│   │   └── modbus_generalized_idl.md      # From Modbus_generalized_IDL.md
│   │
│   ├── research/                      # Research & innovation
│   │   ├── README.md
│   │   ├── innovations.md
│   │   ├── advanced_topics.md
│   │   ├── vlm_fpga_integration.md   # From VLM+FGPA.md
│   │   └── strategic_planning.md     # NEW: Curated from claude_talk.md
│   │
│   ├── operations/                    # Operational documents
│   │   ├── README.md
│   │   ├── maintenance_reports.md    # From mipa_repair_report.md (fixed typo)
│   │   ├── project_notes.md          # From info_from_RM.md
│   │   └── presentations.md          # From ppt.md
│   │
│   └── testing/                       # NEW: Test documentation
│       ├── README.md
│       └── gripper_test_logs.md      # SPLIT: From ModbusRTU_IDL.md
│
├── src/                               # 💻 Source code
│   ├── README.md                      # Code structure overview
│   ├── motor_control/
│   │   └── motor_control_rt.hpp
│   ├── sensors/
│   │   └── multisensor_node.hpp
│   ├── perception/
│   │   └── pointcloud_pipeline.hpp
│   └── communication/
│       └── zero_copy_publisher.cpp   # Renamed from Publisher(zero_copy).cpp
│
├── assets/                            # 🎨 Media & resources
│   ├── images/
│   │   ├── architecture_diagram_2026-01-10.png   # Renamed with context
│   │   └── system_overview_2026-01-10.png        # Renamed with context
│   └── diagrams/                      # For future architecture diagrams
│
├── config/                            # ⚙️ Configuration files
│   └── README.md                      # Configuration documentation
│
├── tests/                             # 🧪 Test code (future)
│   └── README.md
│
└── .github/                           # GitHub-specific files
    ├── PULL_REQUEST_TEMPLATE.md       # From MR_template.md
    └── workflows/                     # For future CI/CD
```

---

## Migration Phases

### Phase 0: Preparation (Before Migration)
**Duration:** 1 day  
**Risk Level:** 🟢 Low

#### Tasks:
- [ ] Create migration branch: `repo-reorganization`
- [ ] Review and approve this migration plan
- [ ] Back up repository (create a release tag: `v-pre-reorg`)
- [ ] Notify team members about upcoming changes
- [ ] Set repository to read-only temporarily (optional)

#### Deliverables:
- Approved migration plan
- Backup tag created
- Team notified

---

### Phase 1: Create Directory Structure
**Duration:** 1 day  
**Risk Level:** 🟢 Low

#### Tasks:
- [ ] Create all new directories
- [ ] Add README.md placeholder files to each directory
- [ ] Create new supporting documents (ARCHITECTURE.md, CONTRIBUTING.md, CHANGELOG.md)
- [ ] Commit structure changes

#### Commands:
```bash
# Create directory structure
mkdir -p docs/{architecture,hardware,protocols,research,operations,testing}
mkdir -p src/{motor_control,sensors,perception,communication}
mkdir -p assets/{images,diagrams}
mkdir -p config tests .github/workflows

# Create README placeholders
touch docs/README.md
touch docs/architecture/README.md
touch docs/hardware/README.md
touch docs/protocols/README.md
touch docs/research/README.md
touch docs/operations/README.md
touch docs/testing/README.md
touch src/README.md
touch config/README.md
touch tests/README.md
```

#### Deliverables:
- Empty directory structure in place
- All README placeholders created

---

### Phase 2: Split Mixed-Content Files
**Duration:** 2 days  
**Risk Level:** 🟡 Medium

#### Task 2.1: Split ModbusRTU_IDL.md

**Current State:**
- Single file (50 KB) containing:
  - Modbus RTU protocol specification
  - Interface Definition Language (IDL)
  - Gripper test logs and results

**Split Into:**
1. `docs/protocols/modbus_rtu_specification.md` - Protocol specs only
2. `docs/testing/gripper_test_logs.md` - Test execution logs

**Content Extraction Rules:**
- **Specification section**: All protocol definitions, register mappings, data types
- **Testing section**: Test results, timestamps, error logs, debugging output

#### Task 2.2: Curate claude_talk.md

**Current State:**
- Single file (41 KB) containing long-form strategic discussions

**Extract Into:**
1. `docs/architecture/design_decisions.md` - Key architectural decisions
2. `docs/research/strategic_planning.md` - Platform strategy, competitive analysis
3. `docs/architecture/vertebrae_interface.md` - Vertebrae interface specification

**Curation Guidelines:**
- Extract **actionable insights** and **decisions made**
- Remove conversational filler
- Organize by topic
- Maintain original file as archive (move to `docs/archive/`)

#### Task 2.3: Consolidate QNC Files

**Current State:**
- `QNC.md` (6.2 KB)
- `QNC_confluence.md` (6.5 KB)
- Significant content overlap

**Consolidate Into:**
- `docs/hardware/qnc_specification.md` - Single canonical document

**Process:**
1. Compare both files line-by-line
2. Merge unique content from both
3. Resolve conflicts (choose most recent/accurate)
4. Archive originals

#### Deliverables:
- [ ] `docs/protocols/modbus_rtu_specification.md` (specs only)
- [ ] `docs/testing/gripper_test_logs.md` (test logs)
- [ ] `docs/architecture/design_decisions.md` (curated)
- [ ] `docs/research/strategic_planning.md` (curated)
- [ ] `docs/hardware/qnc_specification.md` (consolidated)
- [ ] Original files moved to `docs/archive/` (optional)

---

### Phase 3: Move Documentation Files
**Duration:** 2 days  
**Risk Level:** 🟢 Low

#### File Mapping:

| Current File | New Location | Action |
|-------------|--------------|--------|
| `software_architecture_mipa.md` | `docs/architecture/software_architecture_mipa.md` | Move |
| `mipax_software_refactor.md` | `docs/architecture/mipax_software_refactor.md` | Move |
| `MiPA.md` | `docs/hardware/mipa_platform.md` | Move + Rename |
| `mipa轻量化.md` | `docs/hardware/mipa_lightweight.md` | Move + Rename |
| `mipa_chips_research.md` | `docs/hardware/chip_research.md` | Move + Rename |
| `Modbus_generalized_IDL.md` | `docs/protocols/modbus_generalized_idl.md` | Move + Rename |
| `innovations.md` | `docs/research/innovations.md` | Move |
| `advanced_topics.md` | `docs/research/advanced_topics.md` | Move |
| `VLM+FGPA.md` | `docs/research/vlm_fpga_integration.md` | Move + Rename |
| `mipa_repair_repot.md` | `docs/operations/maintenance_reports.md` | Move + Rename (fix typo) |
| `info_from_RM.md` | `docs/operations/project_notes.md` | Move + Rename |
| `ppt.md` | `docs/operations/presentations.md` | Move + Rename |

#### Git Commands (example):
```bash
# Move with history preservation
git mv software_architecture_mipa.md docs/architecture/
git mv MiPA.md docs/hardware/mipa_platform.md
git mv mipa轻量化.md docs/hardware/mipa_lightweight.md
# ... repeat for all files
```

#### Deliverables:
- All documentation files in correct directories
- Git history preserved
- File names normalized (lowercase, underscores, descriptive)

---

### Phase 4: Move Source Code
**Duration:** 1 day  
**Risk Level:** 🟡 Medium

#### File Mapping:

| Current File | New Location | Notes |
|-------------|--------------|-------|
| `motor_control_rt.hpp` | `src/motor_control/motor_control_rt.hpp` | May need include path updates |
| `multisensor_node.hpp` | `src/sensors/multisensor_node.hpp` | May need include path updates |
| `pointcloud_pipeline.hpp` | `src/perception/pointcloud_pipeline.hpp` | May need include path updates |
| `Publisher(zero_copy).cpp` | `src/communication/zero_copy_publisher.cpp` | Rename + move |

#### Special Considerations:
⚠️ **Include Path Updates Required**

If any code has `#include` statements referencing these files, they must be updated:

**Before:**
```cpp
#include "motor_control_rt.hpp"
```

**After:**
```cpp
#include "motor_control/motor_control_rt.hpp"
```

#### Action Items:
- [ ] Move files to new locations
- [ ] Search for `#include` statements in all code
- [ ] Update include paths if necessary
- [ ] Test compilation (if build system exists)
- [ ] Update any CMakeLists.txt or build configuration

#### Deliverables:
- All source files in organized structure
- Include paths updated
- Code compiles successfully (if applicable)

---

### Phase 5: Move Assets & Templates
**Duration:** 1 day  
**Risk Level:** 🟢 Low

#### Assets Migration:

| Current File | New Location | Notes |
|-------------|--------------|-------|
| `Screenshot 2026-01-10 212328.png` | `assets/images/architecture_diagram_2026-01-10.png` | Descriptive rename |
| `Screenshot 2026-01-10 212518.png` | `assets/images/system_overview_2026-01-10.png` | Descriptive rename |
| `MR_template.md` | `.github/PULL_REQUEST_TEMPLATE.md` | GitHub convention |

#### Naming Convention for Assets:
Format: `{description}_{date}.{ext}`
- Use descriptive names
- Include date for versioning
- Use lowercase with underscores

#### Deliverables:
- Screenshots renamed and moved
- PR template in GitHub standard location

---

### Phase 6: Create Navigation & Index Files
**Duration:** 2 days  
**Risk Level:** 🟢 Low

#### New Files to Create:

##### 1. Enhanced README.md (Root)
**Contents:**
- Project overview
- Quick start guide
- Link to documentation
- Link to architecture overview
- Contribution guidelines
- License information

##### 2. ARCHITECTURE.md (Root)
**Contents:**
- High-level system architecture
- Component overview
- Technology stack
- Extracted from `software_architecture_mipa.md` (summary)

##### 3. CONTRIBUTING.md (Root)
**Contents:**
- Development setup
- Coding standards
- Git workflow
- PR process
- Testing guidelines

##### 4. docs/README.md
**Contents:**
- Documentation index
- Links to all major documents
- Document organization explanation
- How to find information

##### 5. Directory-Specific READMEs
Create README.md in each subdirectory explaining:
- Purpose of the directory
- File descriptions
- Related documents

#### Deliverables:
- [ ] Enhanced root README.md
- [ ] ARCHITECTURE.md
- [ ] CONTRIBUTING.md
- [ ] CHANGELOG.md
- [ ] docs/README.md (index)
- [ ] All directory README files

---

### Phase 7: Update Cross-References
**Duration:** 2 days  
**Risk Level:** 🟡 Medium

#### Task: Fix Internal Links

Many documents likely reference other documents. All links must be updated.

#### Process:

1. **Search for Markdown Links:**
```bash
# Find all markdown links
grep -r "\[.*\](.*\.md)" docs/ src/
```

2. **Update Patterns:**

**Before:**
```markdown
See [MiPA Documentation](MiPA.md) for details.
```

**After:**
```markdown
See [MiPA Documentation](docs/hardware/mipa_platform.md) for details.
```

3. **Types of Links to Update:**
- Relative file links: `[text](file.md)`
- Image links: `![alt](screenshot.png)`
- Code references: Links to source files

#### Automated Approach (Optional):
Create a script to find and replace common patterns:

```bash
#!/bin/bash
# Example: Update links to MiPA.md
find . -name "*.md" -exec sed -i 's|](MiPA\.md)|](docs/hardware/mipa_platform.md)|g' {} \;
```

#### Deliverables:
- All internal links working
- Image references updated
- No broken links

---

### Phase 8: Validation & Testing
**Duration:** 1 day  
**Risk Level:** 🟢 Low

#### Validation Checklist:

- [ ] All files moved successfully
- [ ] No files left in root (except new structure files)
- [ ] All internal links working
- [ ] All images rendering correctly
- [ ] Git history preserved for moved files
- [ ] README files in all directories
- [ ] Documentation index complete
- [ ] Source code compiles (if applicable)
- [ ] No broken external links

#### Testing Process:

1. **Visual Inspection:**
   - Browse repository on GitHub
   - Verify directory structure
   - Click through documentation links

2. **Link Validation:**
```bash
# Use a markdown link checker
npx markdown-link-check docs/**/*.md
```

3. **Build Test (if applicable):**
```bash
# Attempt to build/compile
mkdir build && cd build
cmake ..
make
```

4. **Diff Review:**
```bash
# Review all changes
git diff main..repo-reorganization
```

#### Deliverables:
- Validation report
- List of any issues found
- Issues resolved

---

### Phase 9: Documentation & Rollout
**Duration:** 1 day  
**Risk Level:** 🟢 Low

#### Tasks:

1. **Create Migration Announcement:**
   - Document what changed
   - Provide "old → new" location map
   - Update team on new structure

2. **Update README.md:**
   - Explain new organization
   - Provide navigation guide

3. **Create CHANGELOG.md Entry:**
```markdown
## [Unreleased] - 2026-02-XX

### Changed
- Reorganized entire repository structure
- Split mixed-content files (ModbusRTU_IDL.md, claude_talk.md)
- Consolidated duplicate QNC documentation
- Moved source code to `src/` with logical subdirectories
- Moved documentation to `docs/` with topic-based organization
- Created navigation and index files
- Fixed filename typos and naming conventions

### Added
- ARCHITECTURE.md - High-level architecture overview
- CONTRIBUTING.md - Development guidelines
- Documentation README files for navigation
- Split test logs into separate file

### Deprecated
- Flat repository structure (see migration guide)
```

4. **Create Migration Guide:**
   Document for team members:
   - Where files were moved
   - How to update local clones
   - How to find documents in new structure

#### Deliverables:
- [ ] Migration announcement
- [ ] Updated README.md
- [ ] CHANGELOG.md entry
- [ ] Migration guide document
- [ ] Team notified

---

### Phase 10: Merge & Cleanup
**Duration:** 1 day  
**Risk Level:** 🟡 Medium

#### Pre-Merge Checklist:

- [ ] All validation tests passing
- [ ] Team review completed
- [ ] No outstanding issues
- [ ] Documentation complete
- [ ] Backup tag exists

#### Merge Process:

1. **Create Pull Request:**
```bash
# Push branch
git push origin repo-reorganization

# Create PR on GitHub
# Title: "Repository Reorganization - Phase 1"
# Description: Link to this migration plan
```

2. **Review Process:**
   - Request reviews from team members
   - Address feedback
   - Make final adjustments

3. **Merge:**
   - Squash commits (optional) or keep history
   - Merge to main branch
   - Delete reorganization branch

4. **Post-Merge:**
   - Create release tag: `v1.0-restructured`
   - Update any CI/CD pipelines
   - Monitor for issues

#### Cleanup:

- [ ] Remove `docs/archive/` after 30 days (optional)
- [ ] Archive old screenshots if unused
- [ ] Update any external documentation pointing to old structure

#### Deliverables:
- Merged reorganization
- Release tag created
- Team using new structure

---

## File-by-File Migration Map

### Complete Reference Table

| # | Current Path | New Path | Action | Phase | Notes |
|---|-------------|----------|--------|-------|-------|
| 1 | `README.md` | `README.md` | Update | 6 | Enhance content |
| 2 | `MR_template.md` | `.github/PULL_REQUEST_TEMPLATE.md` | Move | 5 | GitHub convention |
| 3 | `MiPA.md` | `docs/hardware/mipa_platform.md` | Move+Rename | 3 | Normalize name |
| 4 | `ModbusRTU_IDL.md` | `docs/protocols/modbus_rtu_specification.md` + `docs/testing/gripper_test_logs.md` | Split | 2 | Mixed content |
| 5 | `Modbus_generalized_IDL.md` | `docs/protocols/modbus_generalized_idl.md` | Move+Rename | 3 | Normalize name |
| 6 | `Publisher(zero_copy).cpp` | `src/communication/zero_copy_publisher.cpp` | Move+Rename | 4 | Fix name, organize |
| 7 | `QNC.md` + `QNC_confluence.md` | `docs/hardware/qnc_specification.md` | Consolidate | 2 | Merge duplicates |
| 8 | `Screenshot 2026-01-10 212328.png` | `assets/images/architecture_diagram_2026-01-10.png` | Move+Rename | 5 | Descriptive name |
| 9 | `Screenshot 2026-01-10 212518.png` | `assets/images/system_overview_2026-01-10.png` | Move+Rename | 5 | Descriptive name |
| 10 | `VLM+FGPA.md` | `docs/research/vlm_fpga_integration.md` | Move+Rename | 3 | Normalize name |
| 11 | `advanced_topics.md` | `docs/research/advanced_topics.md` | Move | 3 | - |
| 12 | `claude_talk.md` | `docs/architecture/design_decisions.md` + `docs/research/strategic_planning.md` | Split+Curate | 2 | Extract key insights |
| 13 | `info_from_RM.md` | `docs/operations/project_notes.md` | Move+Rename | 3 | More descriptive |
| 14 | `innovations.md` | `docs/research/innovations.md` | Move | 3 | - |
| 15 | `mipa_chips_research.md` | `docs/hardware/chip_research.md` | Move+Rename | 3 | Shorten name |
| 16 | `mipa_repair_repot.md` | `docs/operations/maintenance_reports.md` | Move+Rename | 3 | Fix typo |
| 17 | `mipax_software_refactor.md` | `docs/architecture/mipax_software_refactor.md` | Move | 3 | - |
| 18 | `mipa轻量化.md` | `docs/hardware/mipa_lightweight.md` | Move+Rename | 3 | English name |
| 19 | `motor_control_rt.hpp` | `src/motor_control/motor_control_rt.hpp` | Move | 4 | Check includes |
| 20 | `multisensor_node.hpp` | `src/sensors/multisensor_node.hpp` | Move | 4 | Check includes |
| 21 | `pointcloud_pipeline.hpp` | `src/perception/pointcloud_pipeline.hpp` | Move | 4 | Check includes |
| 22 | `ppt.md` | `docs/operations/presentations.md` | Move+Rename | 3 | More descriptive |
| 23 | `software_architecture_mipa.md` | `docs/architecture/software_architecture_mipa.md` | Move | 3 | - |
| 24 | - | `ARCHITECTURE.md` | Create | 6 | New file |
| 25 | - | `CONTRIBUTING.md` | Create | 6 | New file |
| 26 | - | `CHANGELOG.md` | Create | 6 | New file |

---

## Risk Assessment & Mitigation

### High-Risk Areas

#### 1. Code File Relocation (Phase 4)
**Risk:** Breaking include paths, build failures  
**Mitigation:**
- Create backup branch before moving
- Search for all `#include` statements
- Test compilation after changes
- Keep old structure temporarily if needed

#### 2. Large File Splitting (Phase 2)
**Risk:** Loss of information, incorrect categorization  
**Mitigation:**
- Review content carefully before splitting
- Keep original files in archive
- Have second reviewer check splits
- Document split decisions

#### 3. Link Updates (Phase 7)
**Risk:** Broken internal references, dead links  
**Mitigation:**
- Use automated link checker
- Manual verification of critical documents
- Test all links before merge
- Create link redirect map if needed

### Low-Risk Areas
- Directory creation (Phase 1)
- Asset moves (Phase 5)
- Documentation creation (Phase 6)

---

## Timeline Summary

| Phase | Duration | Dependencies | Risk |
|-------|----------|--------------|------|
| 0. Preparation | 1 day | None | 🟢 Low |
| 1. Directory Structure | 1 day | Phase 0 | 🟢 Low |
| 2. Split Mixed Files | 2 days | Phase 1 | 🟡 Medium |
| 3. Move Documentation | 2 days | Phase 2 | 🟢 Low |
| 4. Move Source Code | 1 day | Phase 3 | 🟡 Medium |
| 5. Move Assets | 1 day | Phase 3 | 🟢 Low |
| 6. Create Navigation | 2 days | Phase 3-5 | 🟢 Low |
| 7. Update Links | 2 days | Phase 6 | 🟡 Medium |
| 8. Validation | 1 day | Phase 7 | 🟢 Low |
| 9. Documentation | 1 day | Phase 8 | 🟢 Low |
| 10. Merge & Cleanup | 1 day | Phase 9 | 🟡 Medium |

**Total Estimated Time:** 15 days (3 weeks)  
**Recommended Team:** 1-2 people  
**Can be parallelized:** Phases 3, 4, 5 can run concurrently

---

## Success Criteria

### Must Have (Required)
- ✅ All files moved to new locations
- ✅ Mixed-content files properly split
- ✅ Duplicate files consolidated
- ✅ All internal links working
- ✅ Git history preserved
- ✅ Source code compiles (if applicable)
- ✅ README files in all directories

### Should Have (Important)
- ✅ Enhanced root README.md
- ✅ ARCHITECTURE.md created
- ✅ CONTRIBUTING.md created
- ✅ Documentation index (docs/README.md)
- ✅ Migration guide document
- ✅ CHANGELOG.md entry

### Nice to Have (Optional)
- ⭕ Automated link checking in CI
- ⭕ Documentation linting
- ⭕ Standardized document templates
- ⭕ Automated build/test pipeline

---

## Rollback Plan

If critical issues arise during migration:

### Immediate Rollback
```bash
# Discard all changes and return to main
git checkout main
git branch -D repo-reorganization
```

### Partial Rollback
```bash
# Revert specific commits
git revert <commit-hash>
```

### Post-Merge Rollback
```bash
# Reset main to backup tag
git reset --hard v-pre-reorg
git push --force origin main  # Use with extreme caution!
```

**Backup Strategy:**
- Always maintain `v-pre-reorg` tag
- Keep reorganization branch until stable
- Consider keeping parallel structure temporarily

---

## Post-Migration Tasks

### Week 1 After Merge
- [ ] Monitor for reported issues
- [ ] Fix any broken links discovered
- [ ] Update external references
- [ ] Collect team feedback

### Month 1 After Merge
- [ ] Review if structure is working
- [ ] Make minor adjustments if needed
- [ ] Consider archiving old files
- [ ] Update documentation based on usage

### Ongoing
- [ ] Maintain directory structure discipline
- [ ] Update navigation files as project grows
- [ ] Keep CHANGELOG.md updated
- [ ] Refine organization based on team needs

---

## Team Communication Plan

### Before Migration
**Message:**
> "We're reorganizing the repository structure to improve navigation and maintainability. The migration will happen on [DATE]. All files will be moved to logical directories. A detailed migration guide will be provided."

### During Migration
**Status Updates:**
- Daily progress updates on completed phases
- Immediate notification of any issues
- ETA updates if timeline changes

### After Migration
**Announcement:**
> "Repository reorganization complete! See MIGRATION_GUIDE.md for file location changes. New structure documented in docs/README.md. Please update your local clones and bookmarks."

---

## Appendix A: Naming Conventions

### Files
- Use lowercase
- Use underscores for spaces: `my_document.md`
- Be descriptive: `motor_control_rt.hpp` not `mc.hpp`
- Include dates for versioned assets: `diagram_2026-01-10.png`

### Directories
- Use lowercase
- Use underscores for multi-word names
- Be concise but clear
- Group by function, not file type

### Documentation
- Use `.md` extension for markdown
- Start with verb for action docs: "contributing.md"
- Use noun phrases for reference docs: "protocol_specification.md"

---

## Appendix B: Tools & Resources

### Recommended Tools

**Link Checking:**
```bash
npm install -g markdown-link-check
markdown-link-check docs/**/*.md
```

**Markdown Linting:**
```bash
npm install -g markdownlint-cli
markdownlint docs/**/*.md
```

**File Search:**
```bash
# Find all markdown files
find . -name "*.md"

# Search for specific content
grep -r "search term" docs/
```

**Git Operations:**
```bash
# Move with history
git mv old_file.md new_location/new_file.md

# View file history after move
git log --follow new_location/new_file.md
```

---

## Appendix C: Decision Log

### Key Decisions Made

**Decision 1:** Separate `src/` from `docs/`  
**Rationale:** Clear separation of code and documentation improves IDE navigation and build processes.

**Decision 2:** Topic-based documentation structure  
**Rationale:** Users think in terms of topics (architecture, protocols) not file types.

**Decision 3:** Split large mixed-content files  
**Rationale:** Single-responsibility principle - each file should have one clear purpose.

**Decision 4:** Rename Chinese filename  
**Rationale:** ASCII filenames ensure cross-platform compatibility and easier command-line usage.

**Decision 5:** Consolidate QNC files  
**Rationale:** Maintain single source of truth, reduce maintenance burden.

---

## Appendix D: Contact & Support

**Migration Lead:** [Your Name]  
**Questions:** [Contact Method]  
**Issues:** File GitHub issue with label `repo-migration`  
**Document Version:** 1.0  
**Last Updated:** 2026-02-14

---

## Sign-Off

| Role | Name | Approval | Date |
|------|------|----------|------|
| Project Lead | | ☐ Approved | |
| Technical Lead | | ☐ Approved | |
| Team Member | | ☐ Approved | |

---

**Next Steps:**
1. Review this migration plan
2. Get team approval
3. Set migration start date
4. Begin Phase 0 (Preparation)

**This document should be stored as:** `MIGRATION_PLAN.md` in the repository root during migration, then moved to `docs/operations/migration_plan_2026-02.md` after completion.


