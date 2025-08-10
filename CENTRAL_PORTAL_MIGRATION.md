# Maven Central Portal Migration Guide

## Overview
This document outlines the complete migration from legacy OSSRH to the new Maven Central Portal using modern Gradle plugins.

## Changes Made

### 1. Gradle Configuration
- **Replaced**: Old `maven-publish` + manual repository configuration
- **Added**: `io.github.sgtsilvio.gradle.maven-central-publishing` plugin
- **Added**: `io.github.sgtsilvio.gradle.metadata` plugin

### 2. Publishing Method
- **Old**: Manual repository configuration with OSSRH URLs
- **New**: Simplified Central Portal API integration

### 3. GitHub Actions Workflow
- **Old Command**: `publishSonatypePublicationToSonatypeOSSRHRepository`
- **New Command**: `publishToMavenCentral`
- **Simplified**: Single command handles staging and release

## Required Setup Steps

### 1. Central Portal Account Setup
1. Go to https://central.sonatype.com
2. Create account or sign in with existing OSSRH credentials
3. Navigate to "Account" → "Generate User Token"
4. Save the generated username and password

### 2. Update GitHub Repository Secrets
Remove old secrets and add new ones:

**Remove:**
- `SONATYPE_USERNAME`
- `SONATYPE_PASSWORD`

**Add:**
- `MAVEN_CENTRAL_USERNAME` (from Central Portal token)
- `MAVEN_CENTRAL_PASSWORD` (from Central Portal token)

**Keep:**
- `GPG_SIGNING_KEY`
- `GPG_SIGNING_PASSWORD`

### 3. Namespace Migration (if needed)
- If using a test namespace, register it at https://central.sonatype.com
- For production, migrate existing `org.roaringbitmap` namespace

## New Publishing Commands

### Local Testing
```bash
# Build and verify locally
./gradlew build

# Test publication configuration (no upload)
./gradlew generateMetadataFileForMavenPublication
```

### Production Publishing
```bash
# Publish to Maven Central (via GitHub Actions)
# This happens automatically on GitHub releases

# Manual publishing (if needed)
./gradlew publishToMavenCentral \
  -PmavenCentralUsername=<token-username> \
  -PmavenCentralPassword=<token-password>
```

## Key Benefits

### 1. Simplified Configuration
- No manual repository URLs
- Automatic staging and release
- Built-in Central Portal integration

### 2. Modern Publishing Flow
- Uses latest Central Portal API
- Future-proofed against OSSRH shutdown
- Community-supported plugin

### 3. Better Error Handling
- Clear error messages
- Built-in validation
- Automatic retry logic

## Testing the Migration

### Phase 1: Local Testing
```bash
# Verify build works
./gradlew build

# Check publication configuration
./gradlew tasks --all | grep publish
```

### Phase 2: Test Namespace
1. Register a test namespace (e.g., `com.example.roaringbitmap-test`)
2. Temporarily change group ID in build.gradle.kts
3. Test full publishing flow
4. Verify artifacts appear in Central Portal

### Phase 3: Production Migration
1. Update production secrets
2. Create GitHub release
3. Monitor workflow execution
4. Verify publication on Maven Central

## Troubleshooting

### Common Issues
1. **Plugin not found**: Ensure Gradle version >= 6.8
2. **Authentication failed**: Verify Central Portal tokens
3. **Metadata validation**: Check required fields in metadata configuration

### Debug Commands
```bash
# Check plugin application
./gradlew projects

# Verify metadata generation
./gradlew generatePomFileForMavenPublication

# Test GPG signing
./gradlew signMavenPublication
```

## Migration Timeline
1. **Immediate**: Configuration ready for testing
2. **Testing Phase**: Use test namespace to verify
3. **Production**: Deploy when ready (before June 30, 2025)

This migration ensures RoaringBitmap will continue publishing to Maven Central using the modern, supported infrastructure.