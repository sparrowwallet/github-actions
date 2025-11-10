# macOS Codesigning, Packaging and Notarization Action

This composite action automates the complete macOS application codesigning, DMG creation, and notarization process for Sparrow Wallet.

## Required GitHub Secrets

You must configure the following secrets in your GitHub repository settings (`Settings` → `Secrets and variables` → `Actions` → `New repository secret`):

### 1. `MACOS_CERTIFICATE`

Your Developer ID Application certificate exported as a base64-encoded .p12 file.

**To create this secret:**

```bash
# Export your certificate from Keychain Access as a .p12 file
# File → Export Items... → Select "Personal Information Exchange (.p12)"
# Save as: certificate.p12 (you'll be prompted to set a password)

# Convert to base64
base64 -i certificate.p12 | pbcopy

# The base64 string is now in your clipboard - paste it as the secret value
```

### 2. `MACOS_CERTIFICATE_PASSWORD`

The password you set when exporting the .p12 certificate file.

### 3. `MACOS_NOTARIZATION_APPLE_ID`

Your Apple ID email address (e.g., `your-email@example.com`).

### 4. `MACOS_NOTARIZATION_TEAM_ID`

Your Apple Developer Team ID (e.g., `UPLVMSK9D7`).

You can find this in:
- App Store Connect → Membership details
- Or run: `xcrun notarytool history --apple-id YOUR_APPLE_ID --list`

### 5. `MACOS_NOTARIZATION_PASSWORD`

An **app-specific password** for notarization (NOT your Apple ID password).

**To create an app-specific password:**

1. Go to [appleid.apple.com](https://appleid.apple.com)
2. Sign in with your Apple ID
3. In the "Security" section, click "Generate Password" under "App-Specific Passwords"
4. Enter a label (e.g., "Sparrow GitHub Actions")
5. Copy the generated password (format: `xxxx-xxxx-xxxx-xxxx`)
6. Use this as the secret value

## What This Action Does

1. **Extracts version** - Reads version from `build.gradle` dynamically
2. **Sets up keychain** - Creates a temporary keychain and imports the signing certificate
3. **Installs tools** - Installs `create-dmg` via Homebrew
4. **Builds SignPackage** - Clones and builds [SignPackage.jar](https://github.com/sparrowwallet/signpackage) for signing JARs and dylibs
5. **Codesigns native libraries** - Signs native .dylib files in `build/resources/main/native/osx/`
6. **Signs app contents** - Uses SignPackage to sign all JARs and dylibs within `Sparrow.app`
7. **Codesigns executables** - Signs binaries in `Contents/MacOS/`
8. **Codesigns app bundle** - Signs the entire `Sparrow.app` bundle
9. **Creates DMG** - Generates a customized DMG with background image and icon layout
10. **Codesigns DMG** - Signs the DMG file
11. **Notarizes** - Submits to Apple's notarization service and waits for approval
12. **Staples** - Attaches the notarization ticket to the DMG
13. **Verifies** - Runs `spctl` to verify the signature is valid
14. **Cleans up** - Removes the temporary keychain

## Output

The action produces a signed and notarized DMG file at:

```
build/jpackage/Sparrow-{VERSION}-{ARCH}.dmg
```

Where:
- `{VERSION}` is extracted from `build.gradle` (e.g., `2.3.2`)
- `{ARCH}` is either `x86_64` (Intel) or `aarch64` (Apple Silicon)

## Architecture Support

This action automatically detects the runner architecture and signs accordingly:

- Intel (X86) → produces `Sparrow-{VERSION}-x86_64.dmg`
- Apple Silicon → produces `Sparrow-{VERSION}-aarch64.dmg`

## Usage Example

```yaml
- name: Codesign, package and notarize macOS app
  if: ${{ runner.os == 'macOS' }}
  uses: sparrowwallet/github-actions/codesign-macos@v1
  with:
    certificate: ${{ secrets.MACOS_CERTIFICATE }}
    certificate-password: ${{ secrets.MACOS_CERTIFICATE_PASSWORD }}
    apple-id: ${{ secrets.MACOS_NOTARIZATION_APPLE_ID }}
    team-id: ${{ secrets.MACOS_NOTARIZATION_TEAM_ID }}
    notarization-password: ${{ secrets.MACOS_NOTARIZATION_PASSWORD }}
```

## Customization

If you need to use a different signing identity, you can override the default:

```yaml
- name: Codesign and notarize macOS app
  uses: sparrowwallet/github-actions/codesign-macos@v1
  with:
    # ... other inputs ...
    identity: 'Developer ID Application: Your Name (TEAM_ID)'
```

## Troubleshooting

### Notarization fails

Check the notarization log:
```bash
xcrun notarytool log SUBMISSION_ID \
  --apple-id YOUR_APPLE_ID \
  --team-id YOUR_TEAM_ID \
  --password YOUR_APP_PASSWORD
```

### Signature verification fails

Verify the signature locally:
```bash
codesign -vvv --deep --strict Sparrow.app
spctl --assess --type install --context context:primary-signature -v Sparrow.dmg
```

### Certificate not found

Ensure your certificate is valid and matches the identity name exactly:
```bash
security find-identity -v -p codesigning
```

## Security Notes

- The temporary keychain is created with a random password and deleted after use
- The certificate .p12 file is never persisted to disk in the workflow
- All secrets are passed as environment variables and never logged
- The keychain cleanup runs even if previous steps fail (using `if: always()`)
