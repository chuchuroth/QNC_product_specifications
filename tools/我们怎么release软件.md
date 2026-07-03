Of course! Here’s that meeting minutes translated into everyday, non-technical language:

---

**Meeting Notes – Explained Simply**  
*(Date: June 18, 2026)*

- **Review process for code changes is slowing things down**  
  When developers submit new code, someone has to check it before it can be added to the main project. Right now, that checking process is backed up, causing delays. We shouldn’t let this bottleneck stop us from delivering finished work on time.

- **Don't force testers to manually check every tiny code update**  
  Not every code change needs a human to test it by hand. Instead, we should have different levels of review—small changes get a quick look, big ones get more attention.

- **Keep the release plan simple**  
  The process of packaging and delivering the final software to users should be as easy to understand and follow as possible.

- **DevOps already provides a one-stop shop for building and versioning**  
  The DevOps team (who manage the infrastructure) already have tools that handle version numbers, compilation, and testing for all our code. These tools work with different programming languages (like C++ via Conan) and storage systems (like JFrog).

- **The system can automatically scale up or down as needed**  
  Based on DevOps’ experience, the infrastructure can grow or shrink to handle more work without us having to manually adjust it.

- **The system already supports different environments**  
  It can run our code on various types of computers (architectures), different operating system versions, and specific pre-made container images (like Docker).

- **If you need more computing power, ask DevOps**  
  Any requests for extra machines or resources to run tests should go directly to the DevOps team.

- **We can run tests on different types of hardware**  
  We don’t always have to use Qualcomm chips for testing. We can run our Software-in-the-Loop (SIL) tests on multiple architectures.

- **Decide which tests really need real hardware**  
  We should agree on which parts of testing absolutely require actual physical devices, and which can be done in simulation.

- **We need more automated unit tests**  
  Unit tests (which check small pieces of code in isolation) should be increased to catch bugs early.

- **Currently, we have a mixed way of releasing software**  
  Some libraries are released individually via JFrog/Conan (like standalone parts), while others are released as submodules (attached to a bigger project). Version numbers are assigned using a tool that follows semantic versioning (e.g., 1.0.0 → 1.0.1 for bug fixes).

- **Squash small commits into one before merging**  
  Instead of having many tiny, messy commits, combine them into one clean commit. Also, we can use an AI (like a Large Language Model) to help write descriptions for those code changes.

- **The Business Unit (BU) should release one complete, compatible package**  
  Instead of releasing different versions of different parts, the BU should put out a single full package where everything works together. We also need to define what “release artifacts” (the final deliverable files) look like, depending on whether someone wants a single package or individual pieces for different platforms.

- **Suggestion: use a workshop with sticky notes to agree on the release process**  
  Let’s get everyone together physically (or virtually) with sticky notes to brainstorm and agree on how releases should work going forward.

- **Create a “welcome” page for new team members on the BU’s Confluence (wiki) space**  
  This will help new hires get up to speed quickly.

- **The BU owns the release schedule**  
  The Business Unit decides when and how often we release, following the V-model (a software development process) and our fixed sprint lengths. They’ll share a page with all the details on Confluence.

---

Let me know if you'd like any part broken down even further!


---
---
Here's that internal memo translated into plain, everyday language:

---

## What This Document Is About

This page explains how our automated system (GitLab CI/CD) handles software version numbers, tagging, changelogs, and releases – all without anyone having to manually figure out what the next version should be.

---

## The Problem It Solves

Before this system, developers had to:
- Decide whether a change deserves a **major** (big breaking change), **minor** (new feature), or **patch** (bug fix) version bump
- Manually write release notes
- Create and push tags themselves
- Remember to update the changelog file

This system does all of that automatically based on how you write your commit messages.

---

## How It Works in Plain Terms

**The Two-Step Process:**

1. **Preview step (dry-run)** – The system looks at your recent commits and calculates what the next version number *would* be, and what the release notes *would* look like. It doesn't actually do anything yet – it just shows you a preview.

2. **Actual release step** – This is triggered manually (by a person). It creates the version tag, updates the changelog file, and publishes the release on GitLab with all the notes.

---

## What You Need to Set Up

**1. A Secret Key (Token)**
- You need to give the system a "password" (token) so it has permission to create releases and push changes.
- This token must be created with enough authority (Maintainer role – basically a senior-level access).
- It needs permissions to: create releases and push code changes.

**2. A Configuration File**
- Create a file called `.releaserc.json` in your project.
- This file tells the system:
  - Which branches should trigger releases (e.g., `main`, `alpha`, `beta`, etc.)
  - What plugins to use (changelog generator, Git integration, GitLab integration)

**3. Add to Your CI/CD Pipeline**
- Include two pre-built components in your pipeline configuration file.
- This sets up the preview job and the release job.

---

## The Rules for Commit Messages

The system looks at the **first line** of your commit message to decide what kind of version bump to do:

| If your commit message starts with... | Example | What version bump happens |
|--------|---------|---------------------------|
| `fix:` | `fix(auth): handle expired token` | **Patch** (e.g., 1.2.3 → 1.2.4) – for bug fixes |
| `feat:` | `feat(ui): add dark mode` | **Minor** (e.g., 1.2.3 → 1.3.0) – for new features |
| `feat!:` or contains `BREAKING CHANGE:` | `feat(api)!: new endpoint` | **Major** (e.g., 1.2.3 → 2.0.0) – for changes that break compatibility |
| `chore:`, `docs:`, `refactor:`, `test:` | `chore(ci): update pipeline` | **No version bump** – these are ignored |

**Tip:** Put the area of the code in parentheses, like `feat(auth):` – it makes things clearer.

---

## Important Security Notes

- The token you create needs the **Maintainer role** (senior access) to create releases and push to protected branches.
- **Protect the token** in the pipeline settings so it's not exposed to untrusted branches or outside contributors.
- Use a token that's scoped to just this one project (instead of a personal token).
- Change the token periodically for security.

---

## Best Practices

1. **Preview before releasing** – Always run the preview job first to see what version and notes will be generated. This prevents surprises.

2. **Keep the actual release manual** – Don't let the system automatically release; have a person click the button to approve it.

3. **Review changelog changes** – Even though it's auto-generated, someone should still look at the release notes before publishing.

4. **Use pre-release branches** – Branches like `alpha`, `beta`, or `rc` will create pre-release versions (e.g., `1.2.3-alpha.1`) for early testing.

---

## The Typical Flow (Example)

1. Developer writes code and commits with a message like `feat(auth): add login with Google`.
2. The code is reviewed and merged into the `main` branch.
3. The pipeline runs the **preview job** – it calculates that the next version would be `v1.3.0` and generates draft release notes.
4. A release manager looks at the preview, approves it, and clicks the **release job**.
5. The system:
   - Creates the tag `v1.3.0`
   - Updates `CHANGELOG.md` with the new release notes
   - Publishes a GitLab Release with all the notes

---

## Common Problems and Fixes

| Problem | Likely Cause | Solution |
|---------|--------------|----------|
| Job gets skipped | Configuration file missing or branch not listed | Check that `.releaserc.json` exists and includes your branch name |
| Nothing gets released | No relevant commits since last tag | Make sure you have `fix:` or `feat:` commits |
| Authentication error | Token missing or wrong permissions | Check token has Maintainer role and correct permissions |
| Wrong branch releases | Branch name not matching configuration | Update the configuration to include your branch |
| Changelog not updated | Token lacks write permission | Give token `write_repository` scope |
| Empty release notes | Commit messages aren't formatted correctly | Follow the Conventional Commits format |

---

## Common Mistakes to Avoid

- **Not following commit message rules** – Add a linter to check commit messages before they're merged.
- **Exposing the token to outside contributors** – Always mark it as "Protected" in settings.
- **Branch name mismatches** – Be explicit with branch names rather than relying on wildcards.
- **Not checking the preview** – Always look at the `.next-version` file before triggering a real release.

---

**In a nutshell:** This system takes the guesswork out of versioning. Write your commit messages correctly, and the system handles everything else – version numbers, changelogs, tags, and releases – automatically. You just need to give it the right permissions and hit "approve" when you're ready to publish.
