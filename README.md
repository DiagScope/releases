# DiagScope Releases

Public download page for **DiagScope**, a static analyzer for Java and Kotlin projects that finds code
that goes blind when something breaks in production — swallowed exceptions, missing transactions, HTTP
calls without timeouts, blocking calls on event loops, and more. It reads your source files only; it
never compiles or runs your project, and nothing leaves your machine.

🌐 **Website:** <https://diagscope.github.io> · 📄 [Example report](https://diagscope.github.io/example-report.html) ·
🍺 [Homebrew tap](https://github.com/DiagScope/homebrew-tap)

This repository holds the **release assets** (the runnable JAR of each version). Usage documentation is in
the [tap README](https://github.com/DiagScope/homebrew-tap#readme) and on the website.

---

## Which way should I install it?

| | Homebrew | Manual JAR |
|---|---|---|
| Best for | A developer machine (macOS or Linux) | CI runners, containers, machines without Homebrew, pinning an exact version |
| Java | Installed for you (Java 25) | You provide **Java 25 or newer** |
| Updates | `brew upgrade diagscope` | Download the new JAR |
| Command | `diagscope …` | `java -jar diagscope-<version>.jar …` |

---

## Install with Homebrew (recommended)

```bash
brew tap DiagScope/tap
brew install diagscope
diagscope --version
```

Update with `brew update && brew upgrade diagscope`. The formula always points at the newest release.

---

## Manual download

Each release has one asset: `diagscope-<version>.jar`, a self-contained JAR with no other dependencies.

**1. Check your Java version** — it must be 25 or newer:

```bash
java -version
```

**2. Download the JAR** — from the [Releases](https://github.com/DiagScope/releases/releases) page, or from
the terminal:

```bash
# the newest stable release
gh release download --repo DiagScope/releases --pattern 'diagscope-*.jar'

# a specific version (the tag is the version with a "v")
gh release download v0.1.3 --repo DiagScope/releases --pattern 'diagscope-*.jar'

# without the GitHub CLI
curl -fLO https://github.com/DiagScope/releases/releases/download/v0.1.3/diagscope-0.1.3.jar
```

**3. Verify it** (optional but recommended). Each Homebrew formula records the SHA-256 of the JAR it
installs, so you can compare against it:

```bash
shasum -a 256 diagscope-0.1.3.jar        # macOS
sha256sum diagscope-0.1.3.jar            # Linux
```

The expected value is the `sha256` line of
[`Formula/diagscope.rb`](https://github.com/DiagScope/homebrew-tap/blob/main/Formula/diagscope.rb) in the
tap (history of that file has the value for older versions).

**4. Run it:**

```bash
java -jar diagscope-0.1.3.jar --version
java -jar diagscope-0.1.3.jar scan --project /path/to/project
```

Handy alias so you can type `diagscope` instead:

```bash
alias diagscope='java -jar /path/to/diagscope-0.1.3.jar'
```

---

## First scan

```bash
diagscope scan --project /path/to/project
```

No build step is needed. Reports land in `target/diagscope/` (Maven) or `build/diagscope/` (Gradle):
`report.html` (interactive, open it in a browser), `report.md`, and `result.json`. For plain libraries add
`--entrypoint PUBLIC_METHOD`, and use `diagscope explain <RULE_ID>` to understand any finding.

The full walkthrough — collecting reports from several projects, CI usage, baselines, troubleshooting — is in
the [tap README](https://github.com/DiagScope/homebrew-tap#readme).

---

## Using it in CI

Download a pinned JAR in the job and run it. Pinning a version keeps results reproducible:

```yaml
- uses: actions/setup-java@v4
  with:
    distribution: temurin
    java-version: '25'
- run: |
    curl -fLO https://github.com/DiagScope/releases/releases/download/v0.1.3/diagscope-0.1.3.jar
    java -jar diagscope-0.1.3.jar scan --project . --format HTML,JSON --fail-on ERROR
- uses: actions/upload-artifact@v4
  if: always()
  with:
    name: diagscope
    path: target/diagscope/
```

A project can also keep its tolerance in `diagscope.yml` (a `gate:` section with minimum severity and confidence,
per-severity limits and named profiles) and run `diagscope scan --project .` with no gate flag. See the
[tap README](https://github.com/DiagScope/homebrew-tap#setting-the-tolerance-the-gate).

---

## Using it with AI coding assistants

`diagscope init-agent` writes instructions into `AGENTS.md` and `CLAUDE.md` and, for Claude Code, a hook that scans
the changed files when a task ends. `diagscope scan --format AGENT` prints compact text for a model. With the JAR,
run `diagscope init-agent --command "java -jar /path/to/diagscope-0.1.3.jar"`. See the
[tap README](https://github.com/DiagScope/homebrew-tap#using-it-with-ai-coding-assistants).

---

## Versions and release notes

- Releases are tagged `v<version>`, for example `v0.1.3`.
- Versions containing `alpha`, `beta` or `rc` are published as **pre-releases**. DiagScope is still in an
  early stage, so rules, report layout and options can change between versions — read the release notes
  before upgrading a pinned version.
- Release notes are generated from the changes in each version and are on each release's page.
- Older versions stay available; pin one by downloading its JAR, or with Homebrew by running
  `brew pin diagscope` to stop `brew upgrade` from moving it.

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `UnsupportedClassVersionError` or "class file version 69.0" | Your Java is older than 25. Install Java 25+ (Homebrew users get it automatically) and check `java -version`. |
| `command not found: diagscope` | With Homebrew, run `brew install diagscope` and open a new terminal. With the JAR, use `java -jar …` or the alias above. |
| `gh release download` finds no release | `gh` skips pre-releases when no tag is given. Name the tag explicitly, for example `gh release download v0.1.3 …`. |
| The checksum does not match | Download again; if it still differs, do not run the file and report it to the maintainers. |

---

## Privacy

DiagScope makes no network requests while analyzing: no telemetry, no licence checks, no update pings. Your
code stays on your machine, and the reports are written to your local disk.
