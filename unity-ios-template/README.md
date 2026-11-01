# Unity -> iOS build template (GitHub Actions)

Copy the contents of this folder into the **root of a new private repo** that holds your recovered Unity project (the folder containing `Assets/`, `Packages/`, `ProjectSettings/`). Do not use it inside `audiozi`.

> Status: **untested.** It was written without access to the project, Unity or Xcode. Expect to fix small things on the first run.

## 1. Create the repo
```bash
git lfs install
git add .gitattributes .gitignore
git add . && git commit -m "Recovered project"
git push
```
Git LFS storage has a free quota (1 GB on GitHub). A 1.5 GB asset set will need a paid data pack, or use a different Git host.

## 2. Add GitHub secrets (Settings -> Secrets and variables -> Actions)

| Secret | What it is |
|---|---|
| `UNITY_LICENSE` | Contents of the Unity Personal `.ulf` license file (see game-ci activation docs). Unity 6 licensing may need `UNITY_EMAIL` + `UNITY_PASSWORD` (+ `UNITY_SERIAL` on Plus/Pro) instead. |
| `UNITY_EMAIL`, `UNITY_PASSWORD`, `UNITY_SERIAL` | As above, if your license type needs them. |
| `APPLE_CERT_P12_BASE64` | Your signing certificate exported as `.p12`, then `base64 -i cert.p12 \| pbcopy`. |
| `APPLE_CERT_PASSWORD` | Password you set when exporting the `.p12`. |
| `APPLE_PROVISIONING_PROFILE_BASE64` | The `.mobileprovision` file, base64-encoded. |
| `APPLE_TEAM_ID` | 10-character team ID from developer.apple.com. |
| `APPLE_BUNDLE_ID` | Must match the bundle ID in the provisioning profile and in Unity's iOS Player Settings. |

Never commit the `.p12` or `.mobileprovision` (they are in `.gitignore`).

## 3. Run
Actions tab -> **Build iOS** -> Run workflow, choose an export method. The `.ipa` appears as the `ios-ipa` artifact.

- `development`: install on devices listed in the profile. Use a Development profile.
- `ad-hoc`: same, with a Distribution profile.
- `app-store-connect`: for TestFlight/App Store.

## Known rough edges
- **UnityFramework signing:** Unity's Xcode project has a separate `UnityFramework` target. If `xcodebuild` fails with "does not support provisioning profiles", sign only the `Unity-iPhone` target (set manual signing for that target in a Unity post-build script, or use `fastlane gym`).
- **Disk space:** the Linux job frees disk first, but a very large project may still need a larger runner.
- **JIT:** this builds a normal ahead-of-time iOS app. The game does not need JIT.
- **First import:** the recovered project will likely have missing references and package versions to fix in the Unity Editor before the CI build can pass.
