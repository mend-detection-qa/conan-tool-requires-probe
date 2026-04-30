# tool-requires-exclusion

Probe verifying that Mend SCA correctly excludes `[tool_requires]` entries from the reported dependency tree. Build-time tools (`cmake`, `ninja`) must not appear as runtime dependencies.

## Feature exercised

`[tool_requires]` section in `conanfile.txt` declares `cmake/3.27.9` and `ninja/1.12.1` as build-time-only tools. Mend must parse the `conan.lock` `build_requires` list and exclude those entries from the reported dependency tree, reporting only the 11 runtime packages from `[requires]`.

## Expected dependency tree

### Direct runtime dependencies (MUST appear)

| Package | Version | Source |
|---------|---------|--------|
| zlib | 1.3.1 | Conan Center (registry) |
| poco | 1.13.3 | Conan Center (registry) |

### Transitive runtime dependencies via poco/1.13.3 (MUST appear)

| Package | Version | Source | Pulled by |
|---------|---------|--------|-----------|
| openssl | 3.6.2 | Conan Center (registry) | poco |
| pcre2 | 10.44 | Conan Center (registry) | poco |
| sqlite3 | 3.53.0 | Conan Center (registry) | poco |
| expat | 2.7.5 | Conan Center (registry) | poco |
| bzip2 | 1.0.8 | Conan Center (registry) | poco |
| lz4 | 1.9.4 | Conan Center (registry) | poco |
| zstd | 1.5.7 | Conan Center (registry) | poco |
| libpq | 15.4 | Conan Center (registry) | poco |
| libmysqlclient | 8.1.0 | Conan Center (registry) | poco |

### Build-time dependencies from [tool_requires] (MUST NOT appear)

| Package | Version | Lockfile key | Reason excluded |
|---------|---------|-------------|-----------------|
| cmake | 3.27.9 | build_requires | declared under [tool_requires] in conanfile.txt |
| ninja | 1.12.1 | build_requires | declared under [tool_requires] in conanfile.txt |
| cmake | 4.3.2 | build_requires | transitive build requirement pulled by poco recipe (cmake/[>=3.17]); still a build_require, not runtime |

## Dependency types

- Runtime: `requires` (11 packages, 2 direct + 9 transitive)
- Build-time: `build_requires` (3 entries — cmake/3.27.9, cmake/4.3.2, ninja/1.12.1; none should appear in Mend tree)
- No `test_requires`, `python_requires`, `config_requires`

## Lockfile note

`conan lock create` resolves `cmake/[>=3.17]` (a build dep from poco's recipe) to `cmake/4.3.2`, which appears in `build_requires` alongside the user-declared `cmake/3.27.9` and `ninja/1.12.1`. All three are build-time-only and must be excluded from Mend's tree.

## Probe metadata

- pattern: tool-requires-exclusion
- pm: conan
- conan_version: 2.28.1
- schema_version: "1.0"
- lockfile: conan.lock (Conan 2.x flat format, version 0.5)
- generated: 2026-04-30
- resolver_knowledge: not available (no Conan UA resolver file in knowledge cache — tree is exploratory)
- highest_regression_risk: true (tool_requires exclusion is the primary correctness gate for Conan SCA)
