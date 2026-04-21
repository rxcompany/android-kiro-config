## Dependency Management & Documentation

- **Single Source of Truth (SSoT):** - The `gradle/libs.versions.toml` file is the **only** place where concrete version numbers (e.g., "1.2.0") are defined.
    - Never hardcode version numbers in `build.gradle` files; always use `libs.*` references.

- **Documentation Policy (`tech.md`):**
    - Do not update `tech.md` with specific version numbers when libraries are upgraded.
    - `tech.md` should only list the *stack* (library names and purposes), not the *state* (versions).
    - If adding a new library to `tech.md`, map it to its corresponding alias in the version catalog.
