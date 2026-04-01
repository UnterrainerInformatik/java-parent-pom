# Log4j Migration Plan: parent-pom

**Layer**: 0 (Root) — update this FIRST, before all other libraries.

## Before Starting

Prompt the user for the following version numbers before making any changes:

| Variable | Question |
|----------|----------|
| `OWN_NEW_VERSION` | What version should `parent-pom` be bumped to? (currently check `<version>` in pom.xml) |

## Context

This is the root parent POM inherited by all libraries. It currently declares `slf4j-log4j12` and `slf4j-simple` as logging backends. This migration replaces them with Log4j 2.25.3, which propagates to every child project automatically.

## Current State

- **Artifact**: `info.unterrainer.commons:parent-pom`
- **Local version**: check `<version>` in `pom.xml` (bump it after changes)
- **Logging deps** (lines 39-54):
  - `org.slf4j:slf4j-api:2.0.17` (KEEP)
  - `org.slf4j:slf4j-log4j12:2.0.17` (REMOVE)
  - `org.slf4j:slf4j-simple:2.0.17` (REMOVE)
- **Plugin config** (`maven-dependency-plugin`, lines 206-208):
  - `ignoredUnusedDeclaredDependency` entries for `slf4j-log4j12` and `slf4j-simple` (REPLACE)

## Steps

### 1. Replace logging dependencies in `pom.xml`

In the `<dependencies>` section, **remove**:

```xml
<!-- REMOVE these two -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>2.0.17</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>2.0.17</version>
</dependency>
```

**Keep** `slf4j-api` unchanged. **Add** after it:

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.25.3</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.25.3</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j2-impl</artifactId>
    <version>2.25.3</version>
</dependency>
```

### 2. Update `ignoredUnusedDeclaredDependencies` in plugin config

In the `maven-dependency-plugin` `<configuration>`, **remove**:

```xml
<ignoredUnusedDeclaredDependency>org.slf4j:slf4j-log4j12</ignoredUnusedDeclaredDependency>
<ignoredUnusedDeclaredDependency>org.slf4j:slf4j-simple</ignoredUnusedDeclaredDependency>
```

**Add**:

```xml
<ignoredUnusedDeclaredDependency>org.apache.logging.log4j:log4j-api</ignoredUnusedDeclaredDependency>
<ignoredUnusedDeclaredDependency>org.apache.logging.log4j:log4j-core</ignoredUnusedDeclaredDependency>
<ignoredUnusedDeclaredDependency>org.apache.logging.log4j:log4j-slf4j2-impl</ignoredUnusedDeclaredDependency>
```

### 3. Bump version

Increment the `<version>` tag. This is a breaking backend change — consider a minor/major bump.

### 4. Build and install locally

```bash
mvn clean install
```

### 5. Verify

```bash
mvn dependency:tree -Dincludes="*log4j*,*slf4j*,*reload4j*"
```

Must show: `slf4j-api`, `log4j-api`, `log4j-core`, `log4j-slf4j2-impl`.
Must NOT show: `slf4j-log4j12`, `slf4j-reload4j`, `slf4j-simple`, `reload4j`.

## Files Changed

| File | Action |
|------|--------|
| `pom.xml` | Replace logging deps, update plugin config, bump version |
