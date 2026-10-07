# Sunder Launcher

A desktop launcher for **Minecraft: Java Edition** — version management, mod loaders,
a resource center, and a skin studio with a 3D preview.

Independent hobby project. Not affiliated with Mojang, Microsoft, or any other launcher.

![Home](screenshots/home.png)

---

## What it does

### Versions and launching
- Installs and launches any release, snapshot, or old version straight from Mojang's
  official metadata, with per-version isolation (each instance gets its own `mods`,
  `config`, `saves`, `resourcepacks` … directories).
- Bundled Java runtime management: picks a suitable JDK for the version you launch,
  and can download Mojang's official runtimes when nothing suitable is installed.
- Detects missing libraries and assets and repairs them from the official sources.

### Mod loaders
- **Fabric**, **Quilt**, **NeoForge** and **Forge**, installed into a separate instance
  so the vanilla version stays clean.
- Forge 1.17+ / NeoForge launch correctly (they need a copy of the vanilla jar in the
  instance directory — a detail that trips up most home-grown launchers).

### Accounts
- **Microsoft** sign-in, using the official Microsoft identity platform only —
  authorization code + PKCE in the system browser, or the device code flow.
- **Offline** accounts for local play.
- **authlib-injector** based sign-in for third-party Yggdrasil servers
  (the user supplies the server URL; the jar is fetched and verified automatically).

### Resource center
- Search and install **mods, modpacks, resource packs, data packs and shaders** from
  Modrinth, CurseForge, FTB and OptiFine.
- Installing a modpack creates a **new isolated instance** (`<pack name>` appears in the
  version list) instead of dumping files into whatever instance you had selected.
- Drag a `.mrpack` / modpack zip onto the window and it installs itself.
- Already-installed content is marked in the search results.

### Skin studio
- 3D preview of any skin (by name or UUID) with working animations
  (idle / walk / run / fly / wave / crouch / hit / swim), drag to rotate,
  scroll to zoom, plus "face forward" and "reset camera".
- A pixel editor with per-body-part colours, mirror-brush, base-layer/outer-layer
  handling and undo history.
- Skin generation: **text → skin** and **image → skin** (structure mapping or
  palette-only), running fully offline. For higher quality the app hands off to a
  third-party generator (LUMEN Weaver) and imports the result back.

![Skin studio](screenshots/skin-studio.png)

---

## Screenshots

| Versions | Resource center |
| --- | --- |
| ![Versions](screenshots/versions.png) | ![Resources](screenshots/resources.png) |

![Settings](screenshots/settings.png)

---

## Authentication and compliance

This launcher talks to Minecraft services the same way the official launcher does, and
it does **not** cut any corners:

- Sign-in happens on **Microsoft's own pages**. The launcher never asks for, sees or
  stores an account password — it only ever receives OAuth tokens.
- The OAuth client is a **public client** (no client secret). The user registers their
  own Azure application and enters its Application (Client) ID at runtime; it is never
  bundled or redistributed with the launcher.
- Access to Minecraft's APIs requires the application to be **approved through Mojang's
  AppID review process**. Until an application is approved, Minecraft service login
  returns `403 Invalid app registration` — the launcher reports this clearly instead of
  working around it.
- The launcher **does not distribute game files**. Everything is downloaded from
  Mojang's official endpoints (`piston-meta.mojang.com`, `launchermeta.mojang.com`,
  `libraries.minecraft.net`, `resources.download.minecraft.net`) and verified against
  the official metadata.
- It does **not** bypass, disable or imitate any authentication, licensing, entitlement
  or safety check, and it does not pretend to be an official product.
- Name and branding are the project's own; no Mojang/Minecraft/Microsoft/Xbox
  trademarks are used as product identity.

---

## Building from source

Requires Node.js 22.12+ and Windows 10/11 x64.

```bash
npm install
npm run dev          # development
npm run typecheck    # TypeScript, node + web
npm run dist         # build + package (NSIS installer + zip)
```

The project is written in TypeScript: Electron (main/preload) + React + Vite in the
renderer, with `three.js` / `skinview3d` for the 3D preview.

### Tests

There is no unit-test framework; behaviour is verified by a set of standalone assertion
scripts (678 assertions across 19 suites at the time of writing) that exercise the real
code paths — including a mock authlib-injector server, a real Electron process for
IPC/preload and clipboard behaviour, and pixel-level checks for skin generation:

```bash
npx vite-node tools/verify-unzip.ts          # zip extraction, path traversal guards
npx vite-node tools/verify-forge-classpath.ts# Forge 1.17+ launch arguments
npx vite-node tools/verify-loopback.ts       # OAuth loopback + PKCE flow
npx vite-node tools/verify-yggdrasil.ts      # authlib-injector flow (local mock server)
npx vite-node tools/verify-skinify.ts        # image → skin structure mapping
npx vite-node tools/verify-skin-generate.ts  # text/image → skin generation
npx vite-node tools/verify-ipc-bridge.ts     # IPC error propagation (real Electron)
npx vite-node tools/verify-lumen-handoff.ts  # clipboard round-trip (real Electron)
```

---

## Current limitations

- The user interface is **Simplified Chinese only** at the moment; localization is
  planned but not implemented.
- CurseForge search requires your own API key (their public-content API is gated);
  Modrinth, FTB and OptiFine work without one.
- Windows-only builds for now.

---

## License

MIT.

Minecraft is a trademark of Mojang Synergies AB. This project is an independent,
unofficial launcher and is not approved by or associated with Mojang or Microsoft.
