# RoaringBitmap Maven Central Publishing - Complete CI/CD Guide

## Table of Contents
1. [Overview](#overview)
2. [Project Structure Changes](#project-structure-changes)
3. [Build Configuration (Gradle)](#build-configuration-gradle)
4. [GitHub Actions CI/CD Pipeline](#github-actions-cicd-pipeline)
5. [Security & Secrets Management](#security--secrets-management)
6. [Publishing Repositories](#publishing-repositories)
7. [Testing & Verification](#testing--verification)
8. [Troubleshooting](#troubleshooting)
9. [Manual Operations](#manual-operations)
10. [Best Practices](#best-practices)

---

## Overview

This guide explains the complete Maven Central publishing setup for RoaringBitmap. The system enables automatic publishing of releases to both GitHub Packages and Maven Central using GitHub Actions CI/CD pipeline.

### What is CI/CD?
**CI/CD (Continuous Integration/Continuous Deployment)** automates the software release process:
- **Continuous Integration (CI)**: Automatically builds, tests, and validates code changes
- **Continuous Deployment (CD)**: Automatically publishes releases to distribution channels

### Publishing Flow
```
GitHub Release Created → GitHub Actions Triggered → Build & Test → Sign Artifacts → Publish to Maven Central & GitHub Packages
```

---

## Project Structure Changes

### New/Modified Files
```
RoaringBitmap/
├── .github/workflows/
│   ├── releases.yml              # Modified: Added Maven Central publishing
│   └── test-publish.yml          # New: Test publishing workflow
├── build.gradle.kts              # Modified: Added publishing configuration
├── MAVEN_CENTRAL_SETUP.md        # New: Setup instructions
├── TEST_SETUP.md                 # New: Testing instructions
└── DETAILED_CICD_GUIDE.md        # New: This comprehensive guide
```

### Key Changes Made
1. **Added Maven Publishing Configuration** to `build.gradle.kts`
2. **Enhanced GitHub Actions Workflow** in `releases.yml`
3. **Created Documentation** for setup and testing
4. **Added GPG Signing Support** for Maven Central requirements

---

## Build Configuration (Gradle)

### Publishing Configuration Location
File: `build.gradle.kts` (lines 100-204)

### What Was Added

#### 1. Maven Publishing Plugin
```kotlin
apply(plugin = "maven-publish")
apply(plugin = "signing")
```
**Purpose**: Enables Maven-compatible artifact publishing and GPG signing

#### 2. Source & Javadoc JAR Creation
```kotlin
configure<JavaPluginExtension> {
    withSourcesJar()    // Creates -sources.jar
    withJavadocJar()    // Creates -javadoc.jar
}
```
**Purpose**: Maven Central requires source code and documentation JARs

#### 3. Publication Definition
```kotlin
publications {
    register<MavenPublication>("sonatype") {
        groupId = "org.roaringbitmap"
        artifactId = project.name  // "roaringbitmap" or "bsi"
        version = project.version.toString()
        from(components["java"])   // Include compiled JAR
        
        pom {
            name.set("${project.group}:${project.name}")
            description.set("Roaring bitmaps are compressed bitmaps...")
            url.set("https://github.com/RoaringBitmap/RoaringBitmap")
            // ... license, developer, SCM information
        }
    }
}
```
**Purpose**: Defines what gets published and includes required Maven Central metadata

#### 4. Repository Configurations
```kotlin
repositories {
    // Local testing
    maven {
        name = "LocalTest"
        url = uri("file://${project.buildDir}/local-repo")
    }
    
    // GitHub Packages
    maven {
        name = "GitHubPackages" 
        url = uri("https://maven.pkg.github.com/RoaringBitmap/RoaringBitmap")
        credentials {
            username = System.getenv("GITHUB_ACTOR")
            password = System.getenv("GITHUB_TOKEN")
        }
    }
    
    // Maven Central via Sonatype OSSRH
    maven {
        name = "SonatypeOSSRH"
        url = uri("https://s01.oss.sonatype.org/service/local/staging/deploy/maven2/")
        credentials {
            username = System.getenv("SONATYPE_USERNAME")
            password = System.getenv("SONATYPE_PASSWORD")
        }
    }
}
```
**Purpose**: Defines where artifacts can be published

#### 5. GPG Signing Configuration
```kotlin
configure<SigningExtension> {
    val signingKey = System.getenv("GPG_SIGNING_KEY")
    val signingPassword = System.getenv("GPG_SIGNING_PASSWORD")
    if (signingKey != null && signingPassword != null) {
        useInMemoryPgpKeys(signingKey, signingPassword)
        sign(publishing.publications["sonatype"])
    }
}
```
**Purpose**: Signs artifacts with GPG (required by Maven Central)

---

## GitHub Actions CI/CD Pipeline

### Workflow File Location
File: `.github/workflows/releases.yml`

### Trigger Mechanism
```yaml
on:
  release:
    types: [created]
```
**What this means**: The workflow automatically runs when you create a GitHub release

### Step-by-Step Workflow Breakdown

#### Step 1: Environment Setup
```yaml
- uses: actions/checkout@v4
  with:
    path: roaring
- uses: actions/setup-java@v4
  with:
    java-version: '11'
    distribution: 'temurin'
- name: Setup Gradle
  uses: gradle/actions/setup-gradle@af1da67850ed9a4cedd57bfd976089dd991e2582
```
**Purpose**: 
- Downloads project code
- Installs Java 11 (required for building)
- Sets up Gradle build tool

#### Step 2: Build Project
```yaml
- name: Build with Gradle
  run: ./gradlew build -x test -i
```
**Purpose**: 
- Compiles source code
- Creates JAR files
- Generates javadoc
- Skips tests (`-x test`) for faster builds
- Shows detailed output (`-i`)

#### Step 3: Publish to GitHub Packages
```yaml
- name: Publish to GitHub Packages
  run: ./gradlew publishSonatypePublicationToGitHubPackagesRepository -i
  env:
    GITHUB_USER: ${{ github.actor }}
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
**Purpose**: 
- Publishes to GitHub's package registry
- Uses GitHub's built-in authentication
- Makes packages available to GitHub users

#### Step 4: Publish to Maven Central
```yaml
- name: Publish to Maven Central
  run: ./gradlew publishSonatypePublicationToSonatypeOSSRHRepository -i
  env:
    SONATYPE_USERNAME: ${{ secrets.SONATYPE_USERNAME }}
    SONATYPE_PASSWORD: ${{ secrets.SONATYPE_PASSWORD }}
    GPG_SIGNING_KEY: ${{ secrets.GPG_SIGNING_KEY }}
    GPG_SIGNING_PASSWORD: ${{ secrets.GPG_SIGNING_PASSWORD }}
```
**Purpose**: 
- Signs artifacts with GPG
- Publishes to Maven Central via Sonatype OSSRH
- Uses stored secrets for authentication

---

## Security & Secrets Management

### What Are GitHub Secrets?
GitHub Secrets are encrypted environment variables stored securely in your repository settings. They're used to store sensitive information like passwords, API keys, and private keys.

### Required Secrets

#### Location
Repository Settings → Secrets and variables → Actions

#### Secrets Needed

1. **`SONATYPE_USERNAME`**
   - **What**: Your Sonatype JIRA account username
   - **Why**: Authenticates with Maven Central's staging repository
   - **How to get**: Create account at https://issues.sonatype.org/

2. **`SONATYPE_PASSWORD`**
   - **What**: Your Sonatype JIRA account password or token
   - **Why**: Password for Maven Central authentication
   - **Security**: Use application tokens instead of main password

3. **`GPG_SIGNING_KEY`**
   - **What**: Your GPG private key (base64 encoded)
   - **Why**: Signs artifacts to verify authenticity (Maven Central requirement)
   - **Format**: Base64 encoded, no line breaks
   ```bash
   gpg --armor --export-secret-keys YOUR_KEY_ID | base64 | tr -d '\n'
   ```

4. **`GPG_SIGNING_PASSWORD`**
   - **What**: Passphrase for your GPG private key
   - **Why**: Unlocks the GPG key for signing
   - **Security**: Use a strong passphrase

### Security Best Practices
- Never commit secrets to code
- Use GitHub's secret scanner warnings
- Regularly rotate GPG keys and passwords
- Use minimal permission tokens when possible

---

## Publishing Repositories

### 1. Local Test Repository
```
Location: build/local-repo/
Purpose: Testing publication without external upload
Command: ./gradlew publishSonatypePublicationToLocalTestRepository
```
**Use Case**: Verify artifacts are created correctly before real publishing

### 2. GitHub Packages
```
URL: https://maven.pkg.github.com/RoaringBitmap/RoaringBitmap
Access: GitHub users with repository access
Command: ./gradlew publishSonatypePublicationToGitHubPackagesRepository
```
**Use Case**: Internal distribution, CI/CD usage, private access control

### 3. Maven Central (via Sonatype OSSRH)
```
URL: https://s01.oss.sonatype.org/service/local/staging/deploy/maven2/
Access: Public worldwide
Command: ./gradlew publishSonatypePublicationToSonatypeOSSRHRepository
```
**Use Case**: Public distribution, standard Java dependency management

### Repository Selection Logic
- **Development/Testing**: Local Test Repository
- **Internal/Private**: GitHub Packages  
- **Public Release**: Maven Central

---

## Testing & Verification

### Local Testing Commands
```bash
# Test build without publishing
./gradlew build

# Test publication locally
./gradlew publishSonatypePublicationToLocalTestRepository

# Verify artifacts created
ls -la roaringbitmap/build/local-repo/org/roaringbitmap/roaringbitmap/
```

### What to Verify
1. **JAR files created**: Main, sources, javadoc
2. **POM files generated**: With correct metadata
3. **Hash files present**: .md5, .sha1, .sha256, .sha512
4. **Module files**: For Gradle metadata

### Test Workflow
File: `.github/workflows/test-publish.yml`
- Runs on manual trigger
- Tests publication process
- Doesn't require secrets (uses local testing)

---

## Troubleshooting

### Common CI/CD Issues

#### 1. Build Failures
**Symptoms**: Workflow fails at build step
**Causes**: 
- Compilation errors
- Missing dependencies
- Test failures
**Solutions**:
- Check build logs in GitHub Actions
- Run `./gradlew build` locally first
- Fix compilation issues before release

#### 2. Authentication Failures  
**Symptoms**: "401 Unauthorized" or "403 Forbidden"
**Causes**:
- Missing or incorrect secrets
- Expired credentials
- Wrong repository permissions
**Solutions**:
- Verify all 4 secrets are set correctly
- Check Sonatype OSSRH account status
- Ensure GitHub token has package write permissions

#### 3. GPG Signing Failures
**Symptoms**: "GPG signing failed" errors
**Causes**:
- Invalid GPG key format
- Wrong passphrase
- Key not uploaded to keyservers
**Solutions**:
- Re-export GPG key with correct base64 encoding
- Verify passphrase is correct
- Upload public key to keyservers

#### 4. Maven Central Staging Issues
**Symptoms**: Artifacts uploaded but not released
**Causes**:
- Missing required metadata
- POM validation failures
- Manual release required
**Solutions**:
- Check Sonatype Nexus staging repository
- Verify POM has all required fields
- May need manual release via Sonatype UI

### Debugging Steps
1. **Check GitHub Actions logs**: Full error details
2. **Test locally first**: `./gradlew publishSonatypePublicationToLocalTestRepository`
3. **Verify secrets**: Ensure all 4 secrets are set
4. **Check Sonatype status**: Login to Nexus Repository Manager

---

## Manual Operations

### When to Use Manual Commands
- Testing before release
- Debugging publication issues
- Emergency releases
- Local development

### Manual Publishing Commands
```bash
# Test locally (no secrets needed)
./gradlew publishSonatypePublicationToLocalTestRepository

# Publish to GitHub Packages (needs GITHUB_TOKEN)
export GITHUB_ACTOR="your-username"
export GITHUB_TOKEN="your-token"
./gradlew publishSonatypePublicationToGitHubPackagesRepository

# Publish to Maven Central (needs all secrets)
export SONATYPE_USERNAME="your-username"
export SONATYPE_PASSWORD="your-password"  
export GPG_SIGNING_KEY="your-base64-key"
export GPG_SIGNING_PASSWORD="your-passphrase"
./gradlew publishSonatypePublicationToSonatypeOSSRHRepository

# Publish to all repositories
./gradlew publish
```

### Creating a Release
1. **Prepare release**: Ensure code is ready
2. **Create GitHub release**: This triggers CI/CD
3. **Monitor workflow**: Check GitHub Actions tab
4. **Verify publication**: Check Maven Central and GitHub Packages

---

## Best Practices

### Release Management
1. **Test thoroughly** before creating releases
2. **Use semantic versioning** (e.g., 1.4.2)
3. **Write clear release notes**
4. **Monitor CI/CD workflows**

### Security
1. **Rotate secrets regularly**
2. **Use minimal permissions**
3. **Never commit secrets to code**
4. **Monitor GitHub security alerts**

### CI/CD Pipeline
1. **Keep workflows simple and clear**
2. **Add appropriate logging** (`-i` flag for Gradle)
3. **Handle failures gracefully**
4. **Document all configuration changes**

### Dependency Management
1. **Pin workflow action versions** (for security)
2. **Keep Gradle wrapper updated**
3. **Regular dependency updates**
4. **Test with multiple Java versions**

---

## Summary

This CI/CD setup provides:
- **Automated releases** triggered by GitHub releases
- **Dual publication** to GitHub Packages and Maven Central
- **Security** through GPG signing and secret management
- **Testing capabilities** for safe development
- **Comprehensive documentation** for maintenance

The system ensures that every GitHub release automatically becomes available to Java developers worldwide through Maven Central, while maintaining security and reliability standards.