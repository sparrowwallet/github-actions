# Sparrow Wallet GitHub Actions

Reusable GitHub Actions for Sparrow Wallet projects.

## Available Actions

### codesign-macos

Signs and notarizes macOS applications with DMG creation.

- **Documentation:** [codesign-macos/README.md](./codesign-macos/README.md)
- **Version:** v1

**Usage:**

```yaml
- name: Codesign and notarize macOS app
  uses: sparrowwallet/github-actions/codesign-macos@v1
  with:
    certificate: ${{ secrets.MACOS_CERTIFICATE }}
    certificate-password: ${{ secrets.MACOS_CERTIFICATE_PASSWORD }}
    apple-id: ${{ secrets.MACOS_NOTARIZATION_APPLE_ID }}
    team-id: ${{ secrets.MACOS_NOTARIZATION_TEAM_ID }}
    notarization-password: ${{ secrets.MACOS_NOTARIZATION_PASSWORD }}
```

## Versioning

- `@v1` - Recommended (tracks latest v1.x.x)
- `@v1.0.0` - Pin to specific version
- `@main` - Development (not recommended)
