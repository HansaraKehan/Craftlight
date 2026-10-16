# LightCraft for Android (early preview)

An unofficial Android build of [LightCraft](https://github.com/storytold/lightcraft), the open-source photo library
and raw developer written in Rust by the ArtCraft team. This build runs it as a native Android app
(egui + wgpu), not as a web page.

> **Early preview.** This is a first working port, not a finished product. See [Limitations](#limitations).

This repository does not contain LightCraft's source. A GitHub Actions workflow downloads the upstream source,
adds a small Android layer, builds the APK and publishes it under [Releases](../../releases).

## Download

Get the latest `.apk` from the [Releases](../../releases) page, open it and allow the install when Android asks.
You need:

- an ARM64 phone or tablet (most devices from the last several years)
- Android 9 or newer
- a GPU with Vulkan

The first time you start the app, Android opens a settings page: turn on **Allow access to manage all files**.
This lets LightCraft's built-in file browser see your photos. You can change it later in
Settings → Apps → LightCraft → Permissions → Files.

## Using it

- **Import:** the import and open commands show Android's own file picker, with thumbnails, Google Photos and SD
  cards. Pick photos, or pick a folder to import. Photos on your phone's storage are used where they are; photos from
  other places (Google Photos, cloud apps) are copied into the app's `incoming` folder first.
- **Saving exports:** the Save dialog uses a built-in browser. Choose a folder and a file name, then **Save here**.
- **Open with / Share:** choose LightCraft from the "Open with" or "Share" list in your gallery or file manager to
  import a photo.
- **Settings:** your interface settings are remembered between launches.
- **AI masks, AI denoise and HEIC/HEIF** are included. AI masks ask to download the SAM 3 model the first time;
  both AI features are slow on phones without a strong GPU.
- **Library:** your library is stored in the app's folder on shared storage
  (`Android/data/ai.storyteller.lightcraft.android/files/LightCraft Library`). It is removed if you uninstall the app,
  so export or back up what you want to keep.
- **Links:** Help ▸ **Get the Android app / updates…** opens this repository's Releases page.

## AI masks and the SAM 3 model

AI masks (Object and Describe) use Meta's SAM 3 model. It is not included: it is 3.4 GB, under Meta's own licence,
and Hugging Face only gives it to people who accept that licence. LightCraft therefore looks for it in a folder you can
reach with any file manager:

`/storage/emulated/0/LightCraft/sam3/`

Pick one way to get it there:

1. **Copy the files yourself.** Request access to `facebook/sam3` on Hugging Face, download `model.safetensors`,
   `vocab.json` and `merges.txt`, and put them in that folder.
2. **Let the app download it.** In `/storage/emulated/0/LightCraft/` create `sam3-mirrors.txt` containing the line
   `https://huggingface.co/facebook/sam3/resolve/main` and `sam3-token.txt` containing your Hugging Face token, then
   restart the app and choose **Download** when it asks.

The model needs several GB of memory and runs on the CPU, so expect AI masks to be slow, and to fail on phones with
little memory. Everything else works without it.

## Limitations

- A hardware keyboard and mouse are recommended; the interface is the desktop layout, not redesigned for small
  touch screens.
- Raw development is heavy. Large raw files can be slow or run out of memory on low-end devices.
- If the GPU can't be used, LightCraft falls back to software rendering where it can.
- HEIC support includes an HEVC decoder, which may be subject to patents in some countries. It is built in for
  personal use; remove it by building with `--no-default-features` if that matters to you.

## How builds are made

- The workflow in `.github/workflows/lightcraft-android-release.yml` runs every day and builds when upstream has a new
  commit, or when the Android layer file changes.
- To build right away: **Actions → Build and release LightCraft Android APK → Run workflow**. You can enter an
  upstream branch or tag, or tick *force* to rebuild a version that already has a release.
- To stay on one upstream version, add a repository variable named `UPSTREAM_REF` (Settings → Secrets and variables →
  Actions → Variables) set to a tag or commit.

The Android layer lives in `android-layer/lightcraft-android-layer.tar.gz`. It adds an Android app crate
(`apps/lightcraft-android`: app host, file pickers, link opener), a Java launcher activity, a patched `winit` for
hardware keyboard and mouse input, and an Android manifest. It does not change LightCraft's own code except for the Help
menu link. If upstream changes in a way that breaks the layer, the build stops and the log says why.

## Signing

Builds are signed with a key the workflow creates and caches. For a permanent key, add the secrets
`ANDROID_KEYSTORE_B64` (a PKCS12 keystore, base64-encoded, with alias `lightcraft`) and `ANDROID_KEYSTORE_PASSWORD`.
APKs signed with different keys can't update each other.

## Credits and licence

- LightCraft is by the [ArtCraft](https://getartcraft.com/) team, under MIT or Apache-2.0.
- The Android approach follows the community PhotoCraft port,
  [SnowOwlLeader/photocraft-for-android](https://github.com/SnowOwlLeader/photocraft-for-android).
- `vendor/winit` is [winit](https://crates.io/crates/winit) 0.30.13 with a small local input patch (Apache-2.0).

This project is not affiliated with or supported by the LightCraft or ArtCraft team.
