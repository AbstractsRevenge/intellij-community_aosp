# IntelliJ J2K Module Compilation Report for AOSP Java to Kotlin Conversions

**Report Date:** December 19, 2025  
**Repository:** intellij-community_aosp  
**Build System:** Bazel 8.4.2-jb_20251027_92

## Executive Summary

This report provides a comprehensive guide to compiling and using the IntelliJ J2K (Java to Kotlin) module from the IntelliJ Community repository for AOSP Java to Kotlin conversions. The J2K module is part of the Kotlin plugin and provides automated Java-to-Kotlin code conversion capabilities.

## Table of Contents

1. [Module Overview](#module-overview)
2. [Repository Structure](#repository-structure)
3. [Prerequisites](#prerequisites)
4. [Build System](#build-system)
5. [Module Dependencies](#module-dependencies)
6. [Compilation Instructions](#compilation-instructions)
7. [Module Variants](#module-variants)
8. [Key Components](#key-components)
9. [Usage for AOSP Conversions](#usage-for-aosp-conversions)
10. [Integration Points](#integration-points)

---

## Module Overview

The J2K (Java to Kotlin) module is located in:
```
plugins/kotlin/j2k/
```

This module provides automated conversion of Java code to Kotlin code. It includes:
- **Multiple conversion engines** (K1 old, K1 new, K2)
- **Shared conversion infrastructure**
- **Post-processing capabilities**
- **Code quality improvements**

### Module Purpose

The J2K converter:
1. Parses Java source code
2. Builds an intermediate representation (IR)
3. Transforms Java constructs to Kotlin equivalents
4. Generates idiomatic Kotlin code
5. Applies post-processing optimizations

---

## Repository Structure

### J2K Module Structure

```
plugins/kotlin/j2k/
├── k1.old/                    # Legacy K1 converter
│   ├── src/                   # Source code
│   └── BUILD.bazel            # Build configuration
├── k1.new/                    # New K1 converter (recommended)
│   ├── src/                   # Source code
│   ├── tests/                 # Test suite
│   └── BUILD.bazel            # Build configuration
├── k1.new.post-processing/    # Post-processing for K1 new
│   ├── src/
│   ├── resources/
│   └── BUILD.bazel
├── k1.old.post-processing/    # Post-processing for K1 old
│   ├── src/
│   ├── resources/
│   └── BUILD.bazel
├── k2/                        # K2 (next-gen) converter
│   ├── src/                   # Source code
│   ├── tests/                 # Test suite
│   ├── resources/             # Plugin descriptors
│   └── BUILD.bazel            # Build configuration
└── shared/                    # Shared conversion infrastructure
    ├── src/                   # Common code
    ├── tests/                 # Shared tests
    └── BUILD.bazel            # Build configuration
```

### Key Source Directories

#### K1 New Converter (Recommended for Production)
- **Main Source:** `plugins/kotlin/j2k/k1.new/src/org/jetbrains/kotlin/`
  - `j2k/` - K1-specific conversion logic
  - `nj2k/` - New J2K conversion implementation

#### K2 Converter (Next Generation)
- **Main Source:** `plugins/kotlin/j2k/k2/src/org/jetbrains/kotlin/j2k/k2/`
  - Uses Kotlin Analysis API (K2)
  - More accurate type inference
  - Better nullability analysis

#### Shared Infrastructure
- **Main Source:** `plugins/kotlin/j2k/shared/src/org/jetbrains/kotlin/`
  - `j2k/` - Common interfaces and utilities
  - `nj2k/` - Shared new J2K infrastructure
  - `idea/j2k/` - IDE integration points

---

## Prerequisites

### System Requirements

1. **Operating System:** Linux, macOS, or Windows
2. **Memory:** Minimum 8GB RAM (16GB+ recommended)
3. **Disk Space:** ~2GB for source code + build artifacts
4. **Java:** JDK 21 (JetBrains Runtime without JCEF)

### Required Software

#### For Bazel Build

1. **Bazelisk** (recommended) or **Bazel 8.4.2-jb_20251027_92**
   - The repository includes a `.bazelversion` file specifying the exact version
   - Bazelisk will automatically download the correct version

2. **Git**
   ```bash
   git --version  # Should be 2.x or higher
   ```

3. **JDK 21 (JetBrains Runtime)**
   - Download from: https://github.com/JetBrains/JetBrainsRuntime/releases
   - Use the **non-JCEF** variant

#### For IntelliJ IDEA Build

1. **IntelliJ IDEA 2023.2 or newer**
   - Community or Ultimate Edition
   - Download from: https://www.jetbrains.com/idea/download

2. **Kotlin Plugin 1.4.21 or later**

### Repository Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/AbstractsRevenge/intellij-community_aosp.git
   cd intellij-community_aosp
   ```

2. **Get Android modules** (if needed):
   ```bash
   # Linux/macOS
   ./getPlugins.sh
   
   # Windows
   getPlugins.bat
   ```

---

## Build System

The repository uses **Bazel** as the primary build system, with IntelliJ IDEA project files (.iml) for IDE development.

### Bazel Configuration

- **Main configuration:** `MODULE.bazel` (root)
- **Libraries:** `lib/MODULE.bazel` (dependencies)
- **Build options:** `common.bazelrc`, `.bazelrc`
- **Compiler options:** `build/compiler-options.bzl`

### Key Bazel Concepts

1. **Targets:** Individual build units (e.g., `//plugins/kotlin/j2k/shared`)
2. **Dependencies:** Managed via `deps` attribute in BUILD.bazel files
3. **Module names:** Each module has a unique identifier (e.g., `kotlin.j2k.shared`)

---

## Module Dependencies

### J2K Shared Module Dependencies

From `plugins/kotlin/j2k/shared/BUILD.bazel`:

```python
deps = [
  "@lib//:kotlin-stdlib",                           # Kotlin standard library
  "@lib//:kotlinc-analysis-api",                    # Kotlin Analysis API
  "@lib//:kotlinc-kotlin-compiler-common",          # Kotlin compiler common
  "//java/java-impl:impl",                          # Java implementation
  "//java/java-indexing-api:indexing",              # Java indexing
  "//java/java-psi-api:psi",                        # Java PSI API
  "//platform/analysis-api:analysis",               # Platform analysis
  "//platform/core-impl",                           # Platform core
  "//platform/editor-ui-api:editor-ui",             # Editor UI
  "//platform/indexing-api:indexing",               # Indexing API
  "//plugins/kotlin/base/analysis",                 # Kotlin base analysis
  "//plugins/kotlin/base/code-insight",             # Code insight
  "//plugins/kotlin/base/indices",                  # Kotlin indices
  "//plugins/kotlin/base/psi",                      # Kotlin PSI
  "//plugins/kotlin/base/util",                     # Kotlin utilities
  "//plugins/kotlin/frontend-independent",          # Frontend independent
  # ... and more
]
```

### J2K K2 Module Dependencies

From `plugins/kotlin/j2k/k2/BUILD.bazel`:

```python
deps = [
  "@lib//:kotlin-stdlib",
  "@lib//:kotlinc-analysis-api",
  "@lib//:kotlinc-analysis-api-k2",                 # K2-specific API
  "@lib//:kotlinc-kotlin-compiler-common",
  "//plugins/kotlin/j2k/shared",                    # Shared J2K infrastructure
  "//plugins/kotlin/code-insight/fixes-k2:kotlin-codeInsight-fixes",
  "//plugins/kotlin/base/analysis-api/analysis-api-utils:kotlin-base-analysis-utils",
  # ... and more
]
```

### External Dependencies

Key external libraries (from `lib/MODULE.bazel`):
- **kotlin-stdlib 2.3.0-RC3** - Kotlin standard library
- **kotlinc-analysis-api** - Kotlin Analysis API
- **kotlinc-kotlin-compiler-common** - Kotlin compiler infrastructure
- **kotlinc-kotlin-compiler-fe10** - Frontend (K1)
- **kotlinc-analysis-api-k2** - Analysis API (K2)

---

## Compilation Instructions

### Option 1: Using Bazel (Recommended for Production Builds)

#### Build J2K Shared Module

```bash
cd /path/to/intellij-community_aosp

# Build the shared J2K module
bazel build //plugins/kotlin/j2k/shared

# Output location:
# bazel-bin/plugins/kotlin/j2k/shared/shared.jar
```

#### Build J2K K2 Module

```bash
# Build the K2 J2K module
bazel build //plugins/kotlin/j2k/k2:kotlin-j2k

# Output location:
# bazel-bin/plugins/kotlin/j2k/k2/kotlin-j2k.jar
```

#### Build All J2K Modules

```bash
# Build all J2K modules at once
bazel build //plugins/kotlin/j2k/...

# This builds:
# - k1.old
# - k1.new
# - k1.new.post-processing
# - k1.old.post-processing
# - k2
# - shared
```

#### Build with Dependencies

To build J2K with all transitive dependencies:

```bash
# Query all dependencies
bazel query "deps(//plugins/kotlin/j2k/k2:kotlin-j2k)" --output label

# Build with all dependencies
bazel build //plugins/kotlin/j2k/k2:kotlin-j2k --nokeep_going
```

#### Incremental Builds

Bazel supports incremental compilation by default. To force a clean build:

```bash
# Clean all build artifacts
bazel clean

# Clean and rebuild
bazel clean && bazel build //plugins/kotlin/j2k/...
```

### Option 2: Using IntelliJ IDEA (For Development)

#### Setup IntelliJ IDEA

1. **Open the project:**
   ```
   File → Open → Select intellij-community_aosp directory
   ```

2. **Configure JDK:**
   - Go to `File → Project Structure → SDKs`
   - Add JetBrains Runtime 21 (without JCEF)
   - Set as project SDK

3. **Maven Configuration:**
   - If Maven plugin is disabled, enable it
   - Add path variable `MAVEN_REPOSITORY` → `~/.m2/repository`

4. **Memory Settings:**
   ```
   Settings → Build, Execution, Deployment → Compiler
   - User-local heap size: 3000 MB
   - ✓ Compile independent modules in parallel (if RAM > 8GB)
   ```

#### Build in IntelliJ

1. **Build the project:**
   ```
   Build → Build Project
   ```

2. **Build specific module:**
   - Right-click on `plugins/kotlin/j2k/k2/intellij.kotlin.j2k.iml`
   - Select `Build Module 'intellij.kotlin.j2k'`

3. **Run IntelliJ with J2K:**
   ```
   Run → Run 'IDEA' (preconfigured run configuration)
   ```

#### Module Files

IntelliJ module files (.iml) for J2K:
- `plugins/kotlin/j2k/k2/intellij.kotlin.j2k.iml`
- `plugins/kotlin/j2k/shared/kotlin.j2k.shared.iml`
- `plugins/kotlin/j2k/k1.new/kotlin.j2k.k1.new.iml`
- `plugins/kotlin/j2k/k1.old/kotlin.j2k.k1.old.iml`

### Option 3: Command Line Build (Without IDE)

#### Using installers.cmd

Build installer packages with J2K included:

```bash
# Build for current OS only
./installers.cmd -Dintellij.build.target.os=current

# Incremental build
./installers.cmd -Dintellij.build.incremental.compilation=true
```

#### Docker Build

Build in a containerized environment:

```bash
# Build using Docker (Linux)
docker run --rm -it --user "$(id -u)" \
  --volume "${PWD}:/community" \
  "$(docker build --quiet . --target intellij_idea)"

# With Maven cache mounted
docker run --rm -it --user "$(id -u)" \
  --volume "${PWD}:/community" \
  --volume "$HOME/.m2:/home/ide_builder/.m2" \
  "$(docker build --quiet . --target intellij_idea)"
```

---

## Module Variants

### K1 Old (Legacy)

- **Path:** `plugins/kotlin/j2k/k1.old/`
- **Status:** Legacy, maintained for compatibility
- **Use Case:** Older projects, fallback converter
- **Main Class:** `org.jetbrains.kotlin.j2k.OldJavaToKotlinConverter`

**Key Files:**
- `OldJavaToKotlinConverter.kt` - Main converter entry point
- `Converter.kt` - Core conversion logic
- `ExpressionConverter.kt` - Expression conversion rules
- `TypeConverter.kt` - Type conversion logic

### K1 New (Recommended)

- **Path:** `plugins/kotlin/j2k/k1.new/`
- **Status:** Current production converter
- **Use Case:** Standard Java to Kotlin conversion
- **Main Class:** `org.jetbrains.kotlin.nj2k.NewJavaToKotlinConverter`

**Key Files:**
- `newJ2KConversions.kt` - Conversion implementation
- `JKElementInfo.kt` - Element information tracking

**Features:**
- Better nullability inference
- Improved expression handling
- More idiomatic Kotlin output
- Enhanced post-processing

### K2 (Next Generation)

- **Path:** `plugins/kotlin/j2k/k2/`
- **Status:** Next-generation converter (under development)
- **Use Case:** Advanced conversions with K2 Analysis API
- **Main Class:** `org.jetbrains.kotlin.j2k.k2.K2J2KConverterExtension`

**Key Features:**
- Uses Kotlin K2 compiler frontend
- More accurate type analysis
- Better nullability inference
- Improved symbol resolution

**Plugin Descriptor:** `plugins/kotlin/j2k/k2/resources/intellij.kotlin.j2k.xml`
```xml
<idea-plugin package="org.jetbrains.kotlin.j2k.k2">
  <extensions defaultExtensionNs="org.jetbrains.kotlin">
    <j2kConverterExtension implementation="org.jetbrains.kotlin.j2k.k2.K2J2KConverterExtension"/>
    <j2kPreprocessorExtension implementation="org.jetbrains.kotlin.j2k.k2.preProcessings.K2TypeArgumentsExpander"/>
  </extensions>
</idea-plugin>
```

### Shared Infrastructure

- **Path:** `plugins/kotlin/j2k/shared/`
- **Purpose:** Common code for all converters
- **Components:**
  - `JavaToKotlinConverter.kt` - Base converter interface
  - `ConverterSettings.kt` - Configuration
  - `PostProcessor.kt` - Post-processing framework
  - `J2KPostProcessingRunner.kt` - Post-processing execution

**Key Interfaces:**
```kotlin
interface JavaToKotlinConverter {
    fun filesToKotlin(
        files: List<PsiJavaFile>,
        postProcessor: PostProcessor,
        progressIndicator: ProgressIndicator
    ): FilesResult
}
```

---

## Key Components

### Core Converter Classes

#### NewJavaToKotlinConverter

**Location:** `plugins/kotlin/j2k/shared/src/org/jetbrains/kotlin/nj2k/NewJavaToKotlinConverter.kt`

```kotlin
class NewJavaToKotlinConverter(
    val project: Project,
    val targetModule: Module?,
    val settings: ConverterSettings,
    val targetFile: KtFile? = null
) : JavaToKotlinConverter() {
    
    override fun filesToKotlin(
        files: List<PsiJavaFile>,
        postProcessor: PostProcessor,
        progressIndicator: ProgressIndicator,
        preprocessorExtensions: List<J2kPreprocessorExtension>,
        postprocessorExtensions: List<J2kPostprocessorExtension>
    ): FilesResult
}
```

**Responsibilities:**
1. Parse Java files into PSI (Program Structure Interface)
2. Build intermediate representation (JK Tree)
3. Apply conversion rules
4. Generate Kotlin code
5. Run post-processing

### Conversion Phases

The converter operates in multiple phases:

1. **Preprocessing** - Prepare Java code
2. **Tree Building** - Create intermediate representation
3. **Symbol Resolution** - Resolve types and symbols
4. **Conversion** - Apply transformation rules
5. **Code Generation** - Generate Kotlin source
6. **Post-processing** - Apply optimizations and fixes

### Tree Builder

**Location:** `plugins/kotlin/j2k/shared/src/org/jetbrains/kotlin/nj2k/JavaToJKTreeBuilder.kt`

Converts Java PSI to JK (Java to Kotlin) tree representation.

### Type System

**Location:** `plugins/kotlin/j2k/shared/src/org/jetbrains/kotlin/nj2k/types/`

Handles type conversion:
- Java primitive types → Kotlin types
- Java collections → Kotlin collections
- Nullable vs non-null types
- Generic type parameters

### Nullability Inference

**Location:** `plugins/kotlin/j2k/shared/src/org/jetbrains/kotlin/nj2k/J2KNullityInferrer.java`

Analyzes Java code to determine nullability:
- Method return types
- Method parameters
- Field types
- Local variables

### Post-Processing

**Location:** `plugins/kotlin/j2k/shared/src/org/jetbrains/kotlin/j2k/postProcessing.kt`

Post-processing improvements:
- Remove redundant code
- Simplify expressions
- Apply Kotlin idioms
- Fix formatting
- Optimize imports

---

## Usage for AOSP Conversions

### Understanding AOSP Requirements

AOSP (Android Open Source Project) has specific requirements:
- Large-scale Java codebase
- Strict coding standards
- Build system integration (Soong/Blueprint)
- Compatibility with Android SDK APIs

### Recommended Approach

#### 1. Extract J2K as Standalone Tool

To use J2K for AOSP conversions, you'll need to:

1. **Build the J2K modules:**
   ```bash
   bazel build //plugins/kotlin/j2k/k2:kotlin-j2k
   bazel build //plugins/kotlin/j2k/shared:shared
   ```

2. **Collect dependencies:**
   ```bash
   # Query all runtime dependencies
   bazel query "deps(//plugins/kotlin/j2k/k2:kotlin-j2k)" --output label
   
   # Build a deploy jar with all dependencies (requires custom build rule)
   # See "Creating a Standalone JAR" section below
   ```

#### 2. Integration Patterns

**Pattern A: Command-Line Tool**

Create a wrapper that:
1. Parses Java files from AOSP repository
2. Invokes J2K converter programmatically
3. Outputs Kotlin files
4. Preserves file structure

**Pattern B: IntelliJ Plugin**

Use IntelliJ IDEA with J2K plugin:
1. Open AOSP project in IntelliJ
2. Use "Convert Java File to Kotlin" action
3. Batch convert multiple files
4. Review and commit changes

**Pattern C: Batch Processing**

For large-scale conversions:
1. Write a script to enumerate Java files
2. Group files by module/package
3. Convert in batches
4. Run post-processing
5. Validate build

### Creating a Standalone JAR

To create a standalone J2K converter JAR:

#### Create Custom Bazel Rule

Create `plugins/kotlin/j2k/j2k_deploy.bzl`:

```python
load("@rules_java//java:defs.bzl", "java_binary")

def j2k_deploy_jar(name, deps):
    java_binary(
        name = name,
        runtime_deps = deps,
        create_executable = False,
        deploy_manifest_lines = [
            "Main-Class: org.jetbrains.kotlin.nj2k.NewJavaToKotlinConverter"
        ],
    )
```

#### Add to BUILD.bazel

In `plugins/kotlin/j2k/BUILD.bazel`:

```python
load(":j2k_deploy.bzl", "j2k_deploy_jar")

j2k_deploy_jar(
    name = "j2k_standalone",
    deps = [
        "//plugins/kotlin/j2k/k2:kotlin-j2k",
        "//plugins/kotlin/j2k/shared:shared",
        # Add all transitive dependencies
    ],
)
```

#### Build Standalone JAR

```bash
bazel build //plugins/kotlin/j2k:j2k_standalone_deploy.jar

# Output: bazel-bin/plugins/kotlin/j2k/j2k_standalone_deploy.jar
```

### Sample Conversion Script

```bash
#!/bin/bash
# aosp_convert.sh - Convert AOSP Java files to Kotlin

AOSP_DIR="/path/to/aosp"
J2K_JAR="bazel-bin/plugins/kotlin/j2k/j2k_standalone_deploy.jar"
OUTPUT_DIR="/path/to/kotlin-output"

# Find all Java files
find "$AOSP_DIR" -name "*.java" -type f | while read -r java_file; do
    # Compute output path
    rel_path="${java_file#$AOSP_DIR/}"
    kotlin_file="$OUTPUT_DIR/${rel_path%.java}.kt"
    
    # Create output directory
    mkdir -p "$(dirname "$kotlin_file")"
    
    # Convert file (pseudo-code, actual invocation depends on J2K API)
    java -jar "$J2K_JAR" convert \
        --input "$java_file" \
        --output "$kotlin_file" \
        --sdk-version android-34
    
    echo "Converted: $java_file → $kotlin_file"
done
```

### Post-Conversion Steps for AOSP

1. **Update Build Files:**
   - Modify `Android.bp` or `Android.mk`
   - Change `.java` references to `.kt`
   - Add Kotlin dependencies

2. **Verify Compilation:**
   ```bash
   # In AOSP directory
   m -j$(nproc) <module-name>
   ```

3. **Run Tests:**
   ```bash
   atest <test-module>
   ```

4. **Code Review:**
   - Check for manual fixes needed
   - Verify nullability annotations
   - Review API usage

---

## Integration Points

### IntelliJ Plugin Extension Points

J2K integrates with IntelliJ via extension points:

```xml
<extensions defaultExtensionNs="org.jetbrains.kotlin">
  <j2kConverterExtension implementation="org.jetbrains.kotlin.j2k.k2.K2J2KConverterExtension"/>
  <j2kPreprocessorExtension implementation="..."/>
  <j2kPostprocessorExtension implementation="..."/>
</extensions>
```

### Key Extension Points

1. **J2kConverterExtension**
   - Provides converter implementation
   - Selects conversion strategy (K1 vs K2)

2. **J2kPreprocessorExtension**
   - Pre-processes Java code before conversion
   - Example: Type argument expansion

3. **J2kPostprocessorExtension**
   - Post-processes Kotlin code after conversion
   - Example: Code cleanup, optimization

### Programmatic API

To use J2K programmatically:

```kotlin
import org.jetbrains.kotlin.nj2k.NewJavaToKotlinConverter
import org.jetbrains.kotlin.j2k.ConverterSettings
import com.intellij.openapi.project.Project
import com.intellij.psi.PsiJavaFile

fun convertJavaFile(
    project: Project,
    javaFile: PsiJavaFile
): String {
    val settings = ConverterSettings.defaultSettings
    val converter = NewJavaToKotlinConverter(
        project = project,
        targetModule = null,
        settings = settings
    )
    
    val result = converter.filesToKotlin(
        files = listOf(javaFile),
        postProcessor = converter.createPostProcessor(),
        progressIndicator = EmptyProgressIndicator()
    )
    
    return result.results.first().text
}
```

---

## Build Artifacts

### Expected Output

After building J2K modules, you'll find:

```
bazel-bin/
└── plugins/
    └── kotlin/
        └── j2k/
            ├── shared/
            │   └── shared.jar                    # Shared infrastructure
            ├── k2/
            │   └── kotlin-j2k.jar                # K2 converter
            ├── k1.new/
            │   └── k1.new.jar                    # K1 new converter
            └── k1.old/
                └── k1.old.jar                    # K1 old converter
```

### JAR Contents

Each JAR contains:
- Compiled Kotlin classes (.class files)
- Resources (plugin descriptors, bundles)
- Metadata (module info)

### Size Estimates

- `shared.jar`: ~500 KB
- `kotlin-j2k.jar` (K2): ~800 KB
- Dependencies: ~50+ MB (kotlin-stdlib, analysis-api, etc.)

---

## Testing

### Running J2K Tests

```bash
# Run all J2K tests
bazel test //plugins/kotlin/j2k/...

# Run K2 tests only
bazel test //plugins/kotlin/j2k/k2/tests/...

# Run shared tests
bazel test //plugins/kotlin/j2k/shared/tests/...
```

### Test Structure

Tests are located in:
- `plugins/kotlin/j2k/k2/tests/`
- `plugins/kotlin/j2k/k1.new/tests/`
- `plugins/kotlin/j2k/shared/tests/`

Test data (fixtures) are in:
- `plugins/kotlin/j2k/shared/tests/testData/`

### Sample Test

Tests extend base classes like:
- `AbstractJavaToKotlinConverterSingleFileTest`
- `AbstractJavaToKotlinConverterMultiFileTest`

---

## Troubleshooting

### Common Issues

#### 1. Bazel Download Failed

**Problem:** Cannot download Bazel from cache-redirector.jetbrains.com

**Solution:**
```bash
# Use standard Bazel release
export USE_BAZEL_VERSION=8.4.2
bazel build //plugins/kotlin/j2k/...
```

#### 2. Out of Memory During Build

**Problem:** Build fails with OutOfMemoryError

**Solution:**
```bash
# Increase heap size
bazel build //plugins/kotlin/j2k/... --host_jvm_args=-Xmx4g
```

#### 3. Dependency Resolution Failures

**Problem:** Cannot resolve @lib//:kotlin-stdlib

**Solution:**
```bash
# Sync dependencies
bazel sync --configure

# Clean and rebuild
bazel clean --expunge
bazel build //plugins/kotlin/j2k/...
```

#### 4. IntelliJ Build Errors

**Problem:** "Cannot resolve symbol" in IntelliJ

**Solution:**
1. Invalidate caches: `File → Invalidate Caches → Invalidate and Restart`
2. Reimport project: `File → Reload All from Disk`
3. Rebuild: `Build → Rebuild Project`

---

## Performance Considerations

### Build Performance

- **Parallel builds:** Bazel builds in parallel by default
- **Incremental compilation:** Only changed files are recompiled
- **Remote cache:** Configure Bazel remote cache for faster CI builds

### Conversion Performance

For large AOSP conversions:
- **Batch size:** Convert 100-500 files per batch
- **Memory:** Allocate 4-8GB heap for large files
- **Parallelization:** Run multiple converter instances
- **Caching:** Cache symbol resolution results

---

## Additional Resources

### Documentation

- **IntelliJ Platform SDK:** https://plugins.jetbrains.com/docs/intellij/
- **Kotlin Plugin Development:** `plugins/kotlin/README.md`
- **Bazel Build System:** https://bazel.build/

### Source Code References

- **J2K K2 Converter:** `plugins/kotlin/j2k/k2/src/org/jetbrains/kotlin/j2k/k2/`
- **Shared Infrastructure:** `plugins/kotlin/j2k/shared/src/org/jetbrains/kotlin/`
- **Build Configuration:** `plugins/kotlin/j2k/*/BUILD.bazel`

### Community

- **IntelliJ Community:** https://github.com/JetBrains/intellij-community
- **Kotlin Issues:** https://youtrack.jetbrains.com/issues/KTIJ
- **Contributing:** `CONTRIBUTING.md`

---

## Appendix A: Complete Build Command Reference

### Bazel Commands

```bash
# Query all J2K targets
bazel query //plugins/kotlin/j2k/...

# Build all J2K modules
bazel build //plugins/kotlin/j2k/...

# Build specific module
bazel build //plugins/kotlin/j2k/k2:kotlin-j2k

# Run tests
bazel test //plugins/kotlin/j2k/...

# Clean build
bazel clean && bazel build //plugins/kotlin/j2k/...

# Build with verbose output
bazel build //plugins/kotlin/j2k/... --verbose_failures

# Show dependencies
bazel query "deps(//plugins/kotlin/j2k/k2:kotlin-j2k)" --output graph

# Build with custom JVM flags
bazel build //plugins/kotlin/j2k/... --host_jvm_args=-Xmx8g
```

### IntelliJ IDEA Commands

```bash
# Build from command line (Linux/macOS)
./installers.cmd -Dintellij.build.target.os=current

# Build incrementally
./installers.cmd -Dintellij.build.incremental.compilation=true

# Run tests
./tests.cmd -Dintellij.build.test.patterns=org.jetbrains.kotlin.j2k.*
```

---

## Appendix B: Module Dependency Graph

```
J2K K2 Module
├── J2K Shared
│   ├── Kotlin Stdlib
│   ├── Kotlin Compiler Common
│   ├── Java PSI API
│   ├── Platform Core
│   └── Kotlin Base Modules
│       ├── Kotlin Base PSI
│       ├── Kotlin Base Analysis
│       └── Kotlin Base Utilities
├── Kotlin Analysis API K2
├── Kotlin Code Insight Fixes K2
└── Platform APIs
    ├── Analysis API
    ├── Editor UI API
    └── Refactoring API
```

---

## Appendix C: File Locations Quick Reference

| Component | Location |
|-----------|----------|
| J2K K2 Source | `plugins/kotlin/j2k/k2/src/` |
| J2K Shared Source | `plugins/kotlin/j2k/shared/src/` |
| J2K K2 BUILD File | `plugins/kotlin/j2k/k2/BUILD.bazel` |
| J2K Plugin Descriptor | `plugins/kotlin/j2k/k2/resources/intellij.kotlin.j2k.xml` |
| Main Converter | `plugins/kotlin/j2k/shared/src/org/jetbrains/kotlin/nj2k/NewJavaToKotlinConverter.kt` |
| Tree Builder | `plugins/kotlin/j2k/shared/src/org/jetbrains/kotlin/nj2k/JavaToJKTreeBuilder.kt` |
| Nullability Inference | `plugins/kotlin/j2k/shared/src/org/jetbrains/kotlin/nj2k/J2KNullityInferrer.java` |
| Post-processing | `plugins/kotlin/j2k/shared/src/org/jetbrains/kotlin/j2k/postProcessing.kt` |
| Tests | `plugins/kotlin/j2k/*/tests/` |
| Test Data | `plugins/kotlin/j2k/shared/tests/testData/` |

---

## Conclusion

The IntelliJ J2K module is a powerful tool for converting Java code to Kotlin. For AOSP conversions:

1. **Build the J2K modules** using Bazel or IntelliJ IDEA
2. **Choose the appropriate variant** (K2 recommended for new projects)
3. **Extract as standalone tool** or use within IntelliJ IDEA
4. **Process AOSP files** in batches with proper post-conversion validation
5. **Update build system** (Android.bp) to reference Kotlin files
6. **Test thoroughly** before committing changes

This report provides all necessary information to compile and deploy the J2K module for AOSP Java to Kotlin conversions.

---

**Report Generated:** December 19, 2025  
**Version:** 1.0  
**Maintainer:** IntelliJ Community AOSP Fork
