# AGENTS.md — AI Agent Guide for bndtools/workspace

This file helps AI coding agents understand the structure, conventions, and workflows
of this repository so they can contribute effectively and accurately.

---

## What This Repository Is

This is the **official bnd/bndtools workspace template** for the [bnd tool](https://github.com/bndtools/bnd).
It serves as a starting point for OSGi bundle development using bnd and Gradle.

When a developer creates a new bnd workspace (via Bndtools IDE, the bnd CLI, or manually),
this template provides the baseline structure including:
- Shared workspace build configuration (`cnf/build.bnd`)
- Pre-configured OSGi repositories (OSGi R7 API, enRoute, Maven Central)
- Bndtools project templates (`cnf/templates/`)
- Gradle build integration

**Reference docs:** [https://bnd.bndtools.org](https://bnd.bndtools.org)

---

## Repository Layout

```
workspace/
├── .github/
│   └── workflows/
│       └── build.yml          # CI: builds with Gradle on Ubuntu/Java 17
├── cnf/                       # Configuration directory (makes this a bnd workspace)
│   ├── build.bnd              # Workspace-wide build properties and repository plugins
│   ├── central.maven          # Maven Central repository index (GAV coordinates)
│   ├── local/                 # LocalIndexedRepo for locally built bundles
│   ├── release/               # LocalIndexedRepo for released bundles
│   └── templates/             # Bndtools project/workspace templates
│       ├── index.xml          # OSGi repository index for the templates bundle
│       ├── index.xml.sha      # SHA-256 of index.xml (used by bnd for integrity checks)
│       └── org.bndtools.templates.osgi/
│           └── org.bndtools.templates.osgi-<version>.jar
├── gradle/                    # Gradle wrapper files
├── build.gradle               # Workspace-level Gradle build (customization point)
├── gradle.properties          # bnd version and snapshot URL
├── gradlew / gradlew.bat      # Gradle wrapper scripts
└── settings.gradle            # Gradle settings with bnd workspace plugin
```

### The `cnf` Directory

The `cnf` directory is the _magic_ directory that identifies a bnd workspace (analogous
to `.git` for Git). It must always contain `build.bnd`.

**`cnf/build.bnd`** — Workspace-wide properties loaded by every project. Contains:
- Repository plugin declarations (`-plugin.*`)
- Shared headers (`Bundle-Vendor`, `Bundle-Copyright`, etc.)
- Global build settings (`-contract`, Git metadata macros)

**`cnf/ext/`** — Extension files (`.bnd`) loaded alphabetically before `build.bnd`.
Used for plugin configurations. Files here should not be edited by end users; they
may be overridden by upstream updates.

---

## Key Dependencies and Versions

| Component | Version | Notes |
|---|---|---|
| bnd / Bndtools | 7.3.0 | Set in `gradle.properties` as `bnd_version` |
| Gradle plugin | `biz.aQute.bnd.workspace` | Applied in `settings.gradle` |
| OSGi R7 API (`org.osgi.enroute:osgi-api`) | 7.0.0 | OSGi Release 7 specification APIs |
| OSGi enRoute enterprise API | 7.0.0 | Enterprise Java APIs (JPA, JAX-RS, etc.) |
| OSGi R7 implementations (`impl-index`) | 7.0.0 | Reference implementations |
| Templates bundle | 7.3.0.202606021345 | `org.bndtools.templates.osgi` from Maven Central |

**Latest stable bnd release:** 7.3.0 (released 2026-06-02)
Check [https://github.com/bndtools/bnd/releases](https://github.com/bndtools/bnd/releases) for updates.

---

## OSGi Semantic Versioning

This project follows **OSGi semantic versioning** which uses `MAJOR.MINOR.MICRO.QUALIFIER`:
- `MAJOR` — breaking API change
- `MINOR` — backward-compatible new functionality
- `MICRO` — backward-compatible bug fixes
- `QUALIFIER` — build metadata (e.g. timestamp `202606021345`, or `SNAPSHOT`)

This is compatible with [semver.org](https://semver.org) with the qualifier extension.
Timestamp qualifiers (e.g., `7.0.0.201810191116`) are valid but opaque — prefer clean
`MAJOR.MINOR.MICRO` versions (e.g., `7.3.0`) when possible.

---

## Build System

### Prerequisites
- Java 17 (JDK)
- Internet access to Maven Central and OSGi repositories

### Building

```bash
./gradlew build
```

The Gradle build uses the `biz.aQute.bnd.workspace` plugin which delegates project
discovery and build orchestration to bndlib.

### CI

The `.github/workflows/build.yml` runs `./gradlew build` on every push and pull request
using Ubuntu + Java 17 (Temurin distribution).

---

## Templates System

The `cnf/templates/` directory contains an OSGi repository with Bndtools project templates.
These templates appear in Bndtools IDE when creating new projects.

### Template Bundle: `org.bndtools.templates.osgi`

- Source: Maven Central (`org.bndtools:org.bndtools.templates.osgi`)
- Current version: **7.3.0** (OSGi bundle version: `7.3.0.202606021345`)
- JAR location: `cnf/templates/org.bndtools.templates.osgi/org.bndtools.templates.osgi-7.3.0.jar`

#### Available Templates (v7.3.0)

**OSGi Standard Templates** (category `bbb/OSGi Standard Templates`):
| Template | Type | Description |
|---|---|---|
| Component Development | project | OSGi Declarative Services component |
| API Project | project | OSGi API bundle skeleton |
| Integration Testing | project | OSGi integration test project |
| Minimal Workspace | workspace | Bare-minimum bnd workspace |
| Felix 4+ | bndrun | Apache Felix launcher configuration |

**OSGi enRoute** (category `ccc/OSGi enRoute`):
| Template | Type | Description |
|---|---|---|
| OSGi enRoute Base Launcher | bndrun | enRoute base launch configuration |
| OSGi enRoute Debug Launcher | bndrun | enRoute debug launch configuration |

### Updating the Templates Bundle

When a new version of `org.bndtools.templates.osgi` is released on Maven Central:

1. Download the new JAR from `https://repo.maven.apache.org/maven2/org/bndtools/org.bndtools.templates.osgi/<version>/`
2. Place the JAR in `cnf/templates/org.bndtools.templates.osgi/`
3. Remove the old JAR from the same directory
4. Regenerate `cnf/templates/index.xml` using bnd CLI:
   ```bash
   java -jar biz.aQute.bnd.jar index --name "Templates" --directory cnf/templates/ \
     cnf/templates/org.bndtools.templates.osgi/org.bndtools.templates.osgi-<version>.jar
   ```
5. Update the `url` attribute in the generated `index.xml` to include the subdirectory prefix:
   `org.bndtools.templates.osgi/org.bndtools.templates.osgi-<version>.jar`
6. Regenerate `cnf/templates/index.xml.sha`:
   ```bash
   sha256sum cnf/templates/index.xml | awk '{print $1}' > cnf/templates/index.xml.sha
   ```

### The `index.xml` and `index.xml.sha` Files

- **`index.xml`** — OSGi Repository XML index (namespace `http://www.osgi.org/xmlns/repository/v1.0.0`).
  Declares all capabilities of the templates bundle so bndlib/Bndtools can resolve and display them.
- **`index.xml.sha`** — SHA-256 hex digest of `index.xml`. Bnd uses this for integrity verification.
  Must be kept in sync with `index.xml` whenever it changes.

---

## Repository Plugins (cnf/build.bnd)

The workspace configures the following repositories via `-plugin.*` properties:

| Plugin ID | Name | Type | Purpose |
|---|---|---|---|
| `-plugin.1.R7.API` | OSGi R7 API | BndPomRepository | OSGi R7 specification APIs from enRoute |
| `-plugin.2.Enterprise.API` | Enterprise Java APIs | BndPomRepository | JPA, JAX-RS, Servlet APIs |
| `-plugin.3.R7.Impl` | OSGi R7 Reference Implementations | BndPomRepository | Felix, Aries, etc. |
| `-plugin.4.Test` | Testing Bundles | BndPomRepository | JUnit OSGi bundles |
| `-plugin.5.Debug` | Debug Bundles | BndPomRepository | Gogo shell, Web Console |
| `-plugin.6.Central` | Maven Central | MavenBndRepository | General Maven artifacts |
| `-plugin.7.Local` | Local | LocalIndexedRepo | Locally built artifacts |
| `-plugin.8.Templates` | Templates | LocalIndexedRepo | Project templates |
| `-plugin.9.Release` | Release | LocalIndexedRepo | Released workspace bundles |

All BndPomRepository plugins use `https://repo.maven.apache.org/maven2/` as the release URL.

---

## Common Maintenance Tasks for Agents

### Updating the bnd version

1. Edit `gradle.properties` → change `bnd_version`
2. Run `./gradlew build` to verify

### Updating GitHub Actions

Action versions in `.github/workflows/build.yml` should be kept current:
- `actions/checkout` → use latest v4.x
- `actions/setup-java` → use latest v4.x
- `gradle/actions/wrapper-validation` → use latest v4.x

### Updating OSGi enRoute repository versions

The `cnf/build.bnd` pins OSGi enRoute BOMs at version `7.0.0` (OSGi R7, the latest major release).
Check [https://search.maven.org/search?q=g:org.osgi.enroute](https://search.maven.org/search?q=g:org.osgi.enroute) for newer releases.

### Adding a project to the workspace

In a bnd workspace, each top-level directory (other than `cnf`) is a project.
To add one manually, create `<project-name>/bnd.bnd`. The project name should match
the `Bundle-SymbolicName` of the bundle it produces.

---

## Bnd Documentation References

| Topic | URL |
|---|---|
| Introduction | https://bnd.bndtools.org/chapters/110-introduction.html |
| Workspace & Projects tour | https://bnd.bndtools.org/chapters/123-tour-workspace.html |
| Build system | https://bnd.bndtools.org/chapters/150-build.html |
| OSGi versioning | https://bnd.bndtools.org/chapters/170-versioning.html |
| Baselining | https://bnd.bndtools.org/chapters/180-baselining.html |
| Repositories | https://bnd.bndtools.org/chapters/150-build.html |
| bnd plugins | https://bnd.bndtools.org/plugins/ |
| Template fragments | https://bnd.bndtools.org/chapters/620-template-fragments.html |

---

## Security and Quality Notes

- **No credentials or secrets** should ever be committed to this workspace template.
  All repository URLs are public read-only endpoints.
- **Dependency versions** are pinned to stable releases. Snapshot dependencies
  (`-SNAPSHOT`) should only be used during active development against unreleased bnd.
- **SHA checksums** (`index.xml.sha`, `osgi.content` in `index.xml`) must always be
  updated when the files they reference change.
- The `cnf/cache/` directory is gitignored and must never be committed.
