# Maven Central Publishing Setup

This document outlines the required GitHub repository secrets to enable Maven Central publishing for RoaringBitmap.

## Required Repository Secrets

Add the following secrets to your GitHub repository settings (Settings → Secrets and variables → Actions):

### Sonatype OSSRH Credentials
- `SONATYPE_USERNAME`: Your Sonatype JIRA username
- `SONATYPE_PASSWORD`: Your Sonatype JIRA password or token

### GPG Signing Keys
- `GPG_SIGNING_KEY`: Your GPG private key (base64 encoded without line breaks)
- `GPG_SIGNING_PASSWORD`: The passphrase for your GPG private key

## Setup Steps

1. **Create Sonatype OSSRH Account**
   - Create a JIRA account at https://issues.sonatype.org/
   - Request access for `org.roaringbitmap` groupId by creating a ticket

2. **Generate GPG Key**
   ```bash
   # Generate new GPG key
   gpg --full-generate-key
   
   # Export private key (replace KEY_ID with your key ID)
   gpg --armor --export-secret-keys KEY_ID | base64 | tr -d '\n'
   ```

3. **Upload GPG Public Key**
   ```bash
   # Upload to key servers
   gpg --keyserver keyserver.ubuntu.com --send-keys KEY_ID
   gpg --keyserver keys.openpgp.org --send-keys KEY_ID
   ```

4. **Add Secrets to Repository**
   - Go to GitHub repository settings
   - Navigate to "Secrets and variables" → "Actions"
   - Add all four secrets listed above

## Publishing Process

The workflow will now:
1. Build the project
2. Publish to GitHub Packages (existing behavior)
3. Sign artifacts with GPG
4. Publish to Maven Central via Sonatype OSSRH

## Manual Publishing Commands

For testing or manual releases:

```bash
# Publish to Maven Central
./gradlew publishSonatypePublicationToSonatypeOSSRHRepository

# Publish to GitHub Packages
./gradlew publishSonatypePublicationToGitHubPackagesRepository

# Publish to both
./gradlew publish
```

## Troubleshooting

- Ensure all secrets are properly set in repository settings
- Verify GPG key is uploaded to public keyservers
- Check Sonatype OSSRH permissions for the groupId
- Review workflow logs for specific error messages