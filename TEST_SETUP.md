# Testing Maven Central Publishing in Private Repo

## Option 1: Use Your Own GroupId (Recommended for Testing)

1. **Modify build.gradle.kts for testing**:
   ```kotlin
   group = "com.github.yourusername"  // Change this line
   ```

2. **Request access for your groupId**:
   - Create JIRA ticket for `com.github.yourusername`
   - This is usually approved faster than organization groupIds

3. **Use snapshot repository for testing**:
   ```kotlin
   maven {
       name = "SonatypeOSSRH"
       url = uri("https://s01.oss.sonatype.org/content/repositories/snapshots/")  // Snapshot repo
       credentials {
           username = System.getenv("SONATYPE_USERNAME")
           password = System.getenv("SONATYPE_PASSWORD")
       }
   }
   ```

## Option 2: Test with Local Repository

Add this to your private repo's build.gradle.kts:

```kotlin
repositories {
    maven {
        name = "TestLocal"
        url = uri("file://${System.getProperty("user.home")}/.m2/repository")
    }
}
```

Test command: `./gradlew publishSonatypePublicationToTestLocalRepository`

## Getting Sonatype Credentials Quickly

### Fast Track Method:
1. **Create Sonatype account**: https://issues.sonatype.org/
2. **Request `com.github.yourusername` groupId**:
   - Usually approved within hours/days vs weeks for org groupIds
   - In JIRA ticket, prove GitHub ownership by adding specific text to repo README

### Generate GPG Key for Testing:
```bash
# Generate test key (use your email)
gpg --batch --gen-key << EOF
Key-Type: RSA
Key-Length: 2048
Subkey-Type: RSA
Subkey-Length: 2048
Name-Real: Your Name
Name-Email: your-email@example.com
Expire-Date: 2y
Passphrase: test-passphrase-123
EOF

# Get key ID
gpg --list-secret-keys --keyid-format=long

# Export for GitHub secret (replace KEYID)
gpg --armor --export-secret-keys KEYID | base64 | tr -d '\n'
```

## Modified Workflow for Testing

Create `.github/workflows/test-publish.yml`:
```yaml
name: Test Maven Central Publish
on:
  workflow_dispatch:  # Manual trigger only
  
jobs:
  test-publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '11'
          distribution: 'temurin'
      - name: Setup Gradle
        uses: gradle/actions/setup-gradle@v4
      - name: Test publish to snapshots
        run: ./gradlew publishSonatypePublicationToSonatypeOSSRHRepository -i
        env:
          SONATYPE_USERNAME: \${{ secrets.SONATYPE_USERNAME }}
          SONATYPE_PASSWORD: \${{ secrets.SONATYPE_PASSWORD }}
          GPG_SIGNING_KEY: \${{ secrets.GPG_SIGNING_KEY }}
          GPG_SIGNING_PASSWORD: \${{ secrets.GPG_SIGNING_PASSWORD }}
```

This allows manual testing without affecting releases.