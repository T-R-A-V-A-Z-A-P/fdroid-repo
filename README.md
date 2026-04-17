# Iniciativa Travazap F-Droid Repo

Binary F-Droid repository for Iniciativa Travazap Android apps (multiple packages; not limited to a single product).

## Hosting

- Repository is expected to publish from the `gh-pages` branch.
- GitHub Pages base URL:
  - `https://T-R-A-V-A-Z-A-P.github.io/fdroid-repo/`
- F-Droid repo endpoint:
  - `https://T-R-A-V-A-Z-A-P.github.io/fdroid-repo/repo`

## Add in F-Droid client

1. Open **Repositories** in your F-Droid-compatible client.
2. Add:
   - URL: `https://T-R-A-V-A-Z-A-P.github.io/fdroid-repo/repo`
   - Fingerprint: SHA-256 of the F-Droid repo signing cert.

Recommended share URL format:

`https://T-R-A-V-A-Z-A-P.github.io/fdroid-repo/repo?fingerprint=<SHA256_NO_COLONS_UPPERCASE>`

## Fingerprint retrieval

From a machine with the F-Droid signing keystore:

```bash
keytool -list -v -keystore fdroid-keystore.p12 -storetype PKCS12 -alias "<FDROID_KEY_ALIAS>"
```

Use the `SHA256:` value, remove `:` separators, and keep uppercase hex.

## Repository layout

- `config.yml`: public/default repo metadata (no secrets).
- `fdroid-icon.png`: repository icon shown in F-Droid clients (replace with your own PNG when ready).
- `metadata/`: app metadata, one file per package id (optional `Icon: *.png` beside the YAML lists the app in clients even when the APK uses adaptive icons only).
- `repo/`: generated repository index files and APK binaries.

## App onboarding contract

Each app repository that publishes here should follow:

- Publish trigger: GitHub release (`release.published`).
- APK naming in `repo/`: `<applicationId>_<versionCode>.apk`.
- Per-app metadata file: `metadata/<applicationId>.yml`.
- Branch target: `gh-pages` (or another shared publish branch).
- Publish credentials: token scoped to this repository only (`contents:write`).
- Signing model: app signing keys stay in each app repo; F-Droid index signing key is provided at workflow runtime via secrets.

## Release flow (per app repository)

1. A new GitHub release is published in an app repository.
2. App workflow:
   - builds signed release APK;
   - clones this repo (`gh-pages`);
   - copies APK as `<applicationId>_<versionCode>.apk` into `repo/`;
   - runs `fdroid update` and commits generated shared index changes.
3. GitHub Pages serves the updated F-Droid repository.

If multiple app repos publish at the same time, workflows should use push retry/rebase logic.

## Recovery / rollback

If an APK should be removed:

1. Remove the APK from `repo/` (and `archive/` if used).
2. Run `fdroid update`.
3. Commit and push regenerated index files to `gh-pages`.