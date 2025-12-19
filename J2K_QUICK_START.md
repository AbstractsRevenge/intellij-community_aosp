# J2K Module - Quick Start Guide

> **TL;DR:** This guide shows you how to quickly build and use the IntelliJ J2K (Java to Kotlin) converter module for AOSP conversions.

📖 **For the complete detailed report, see [J2K_COMPILATION_REPORT.md](J2K_COMPILATION_REPORT.md)**

## What is J2K?

J2K is IntelliJ's Java-to-Kotlin converter. It automatically converts Java source code to idiomatic Kotlin code.

**Location:** `plugins/kotlin/j2k/`

## Quick Build

### Using Bazel (Recommended)

```bash
# Build the K2 converter (recommended, next-gen)
bazel build //plugins/kotlin/j2k/k2:kotlin-j2k

# Build all J2K modules
bazel build //plugins/kotlin/j2k/...

# Output location
# bazel-bin/plugins/kotlin/j2k/k2/kotlin-j2k.jar
```

### Using IntelliJ IDEA

```bash
# 1. Open project in IntelliJ IDEA 2023.2+
# 2. File → Open → Select this directory
# 3. Build → Build Project
# 4. Run → Run 'IDEA' (to test the converter)
```

## Module Variants

| Variant | Path | Status | Use Case |
|---------|------|--------|----------|
| **K2** | `plugins/kotlin/j2k/k2/` | ✅ **Recommended** | Next-gen converter with better analysis |
| K1 New | `plugins/kotlin/j2k/k1.new/` | ✅ Production | Current stable converter |
| K1 Old | `plugins/kotlin/j2k/k1.old/` | ⚠️ Legacy | Compatibility only |
| Shared | `plugins/kotlin/j2k/shared/` | 🔧 Infrastructure | Common code for all variants |

## Prerequisites

- **JDK:** JetBrains Runtime 21 (without JCEF)
- **Memory:** 8GB RAM minimum (16GB recommended)
- **Build Tool:** Bazel 8.4.2 or IntelliJ IDEA 2023.2+

## AOSP Integration Pattern

### Option 1: Use IntelliJ IDEA (Easiest)

1. Open your AOSP project in IntelliJ IDEA
2. Select Java file(s) → Right-click → "Convert Java File to Kotlin"
3. Review and save the converted Kotlin files
4. Update your `Android.bp` build files

### Option 2: Command-Line Batch Conversion

```bash
# 1. Build J2K
bazel build //plugins/kotlin/j2k/k2:kotlin-j2k

# 2. Create a wrapper script (see J2K_COMPILATION_REPORT.md Section 9)
# 3. Process your AOSP files
./aosp_convert.sh /path/to/aosp /path/to/output

# 4. Update Android.bp files
# Change: srcs: ["Foo.java"]
# To:     srcs: ["Foo.kt"]
```

## Key Files and Classes

| Component | Location |
|-----------|----------|
| Main Converter | `plugins/kotlin/j2k/shared/src/.../NewJavaToKotlinConverter.kt` |
| K2 Implementation | `plugins/kotlin/j2k/k2/src/.../k2/` |
| Plugin Descriptor | `plugins/kotlin/j2k/k2/resources/intellij.kotlin.j2k.xml` |
| Build Config (K2) | `plugins/kotlin/j2k/k2/BUILD.bazel` |
| Build Config (Shared) | `plugins/kotlin/j2k/shared/BUILD.bazel` |

## Testing Your Build

```bash
# Run all J2K tests
bazel test //plugins/kotlin/j2k/...

# Run K2-specific tests
bazel test //plugins/kotlin/j2k/k2/tests/...
```

## Common Issues

### "Bazel not found" or download fails
```bash
# Install Bazel manually
# For Linux:
wget https://github.com/bazelbuild/bazel/releases/download/8.4.2/bazel-8.4.2-linux-x86_64
chmod +x bazel-8.4.2-linux-x86_64
sudo mv bazel-8.4.2-linux-x86_64 /usr/local/bin/bazel
```

### Out of memory during build
```bash
# Increase heap size
bazel build //plugins/kotlin/j2k/... --host_jvm_args=-Xmx4g
```

### IntelliJ "Cannot resolve symbol"
```
File → Invalidate Caches → Invalidate and Restart
Build → Rebuild Project
```

## What Gets Built?

After successful build, you'll have:

```
bazel-bin/plugins/kotlin/j2k/
├── k2/kotlin-j2k.jar           # K2 converter (~800 KB)
├── shared/shared.jar           # Shared infrastructure (~500 KB)
├── k1.new/k1.new.jar           # K1 new converter
└── k1.old/k1.old.jar           # K1 old converter
```

Plus ~50MB of dependencies (kotlin-stdlib, analysis-api, etc.)

## Conversion Example

**Input (Java):**
```java
public class User {
    private String name;
    
    public User(String name) {
        this.name = name;
    }
    
    public String getName() {
        return name;
    }
}
```

**Output (Kotlin):**
```kotlin
class User(val name: String)
```

## Next Steps

1. ✅ **Read the full report:** [J2K_COMPILATION_REPORT.md](J2K_COMPILATION_REPORT.md)
2. 🔧 **Build the module:** Follow "Quick Build" above
3. 🧪 **Test it:** Run the test suite
4. 📦 **Package for AOSP:** See Section 9 of the full report
5. 🚀 **Convert your code:** Start with small batches

## Resources

- **Full Compilation Report:** [J2K_COMPILATION_REPORT.md](J2K_COMPILATION_REPORT.md)
- **Main README:** [README.md](README.md)
- **Kotlin Plugin README:** [plugins/kotlin/README.md](plugins/kotlin/README.md)
- **IntelliJ Platform SDK:** https://plugins.jetbrains.com/docs/intellij/
- **Bazel Documentation:** https://bazel.build/

## Support

For issues or questions:
- Check the [J2K_COMPILATION_REPORT.md](J2K_COMPILATION_REPORT.md) Troubleshooting section
- Review IntelliJ Community issues: https://youtrack.jetbrains.com/issues/KTIJ
- See [CONTRIBUTING.md](CONTRIBUTING.md) for contribution guidelines

---

**Quick Start Guide Version:** 1.0  
**Last Updated:** December 19, 2025  
**Repository:** intellij-community_aosp
