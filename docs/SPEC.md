<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# AdnOS: concept & specification

**Version 0.1 · Concept stage · 2026-11-18**

> [!WARNING]
> This is a living, very early draft. Every decision here is open to change. Where something is uncertain, it says so.

## Contents

1. [Vision](#1-vision)
2. [Principles](#2-principles)
3. [The webOS DNA, as requirements](#3-the-webos-dna-as-requirements)
4. [Editions](#4-editions)
5. [Device classes and postures](#5-device-classes-and-postures)
6. [Architecture](#6-architecture)
7. [Components](#7-components)
8. [Design once, build everywhere](#8-design-once-build-everywhere)
9. [The full OS image](#9-the-full-os-image)
10. [Apps](#10-apps)
11. [Sada: sync](#11-sada-sync)
12. [Phones](#12-phones)
13. [TV](#13-tv)
14. [Rafiq: the assistant](#14-rafiq-the-assistant)
15. [Manzil: smart home](#15-manzil-smart-home)
16. [Licensing](#16-licensing)
17. [Risks and technologies we're watching](#17-risks-and-technologies-were-watching)
18. [Open questions](#18-open-questions)

---

## 1. Vision

One calm, instant operating system across your phone, tablet, desktop and TV, with one launcher, one design language
and your data in sync everywhere. Designed with the grace of webOS on the Palm Pre, built on modern open-source foundations,
and owned by its users, including the sync server.

## 2. Principles

1. **Instant over impressive.** If something makes the core experience wait, it doesn't ship.
2. **Calm by default.** The interface steps back. No nagging, no clutter, no dark patterns.
3. **One thing, everywhere.** Bayt is Bayt on every device. The layout adapts to the screen; the name, data and habits don't change.
4. **Yours.** Your data syncs through a server you control, end-to-end encrypted.
5. **Reuse first.** Prefer proven open-source projects, follow their patterns, and contribute fixes upstream.
6. **AI-friendly.** Tooling, code style and documentation are chosen so humans and coding agents can both work well.

## 3. The webOS DNA, as requirements

Reference: webOS as shipped on the Palm Pre, Pixi, Pre 2, Veer, Pre 3 and HP TouchPad (2009–2011). Not the older Palm OS.

| Trait | Requirement |
|---|---|
| **Instant-on** | Waking the device shows the last screen with no visible delay. Core apps restore their state instead of reloading it. No splash screens in first-party apps. Target: input-to-photon latency that feels immediate at 90–120 Hz. |
| **Launcher (Bayt)** | Pages of apps swiped sideways, a quick-launch bar of favorites, and the Wave (a quick-launch arc revealed by swipe-up-and-hold). The same launcher on every device class, with the layout adapting by posture (section 5). |
| **Card view (Itlala)** | Every open app as a live card over the wallpaper. Flick a card up to close it, drag it to rearrange, drop one on another to stack them, tap to enter. |
| **Gesture area** | A touch strip below the content with a light that responds to touch. Quick swipe up: card view. Swipe up and hold: the Wave. Swipe right to left: back. Works the same on every touch device. |
| **Just Type** | Typing from the card view or launcher searches apps, contacts, settings and the web, and offers quick actions ("New note", "Turn off kitchen lights"). |
| **Notifications** | A slim bar at the bottom edge that never covers what you're doing. Tap to expand into dashboards. Notifications stack quietly. |
| **Synergy** | One merged calendar, contact list and inbox across accounts, synced by Sada. |
| **Words in (Ruq'a)** | Ruq'a is how AdnOS takes in words and communication, whatever the method: a fast on-screen keyboard with on-device prediction first, then voice-to-text (the same engine Rafiq listens with), pen and handwriting, and later sign-language gesture recognition. One input service, many methods, all running on the device. |
| **Design language (Ni'ma)** | A written design system covering tokens, components, motion and sound, implemented identically on every target. See [NIMA.md](NIMA.md): webOS behavior, iOS physics, and the geometry and light of Islamic architecture from the Levant, the Middle East and West Asia. |

## 4. Editions

| | **AdnOS** | **AdnOS for Android** |
|---|---|---|
| Base | Image-based Linux (section 9) | Stock, unmodified Android |
| AdnOS provides | Everything above the kernel | Launcher, keyboard, card view, camera, dialer, Sada client, settings |
| Tap-to-pay & banking | Not available (Play Integrity); use a watch or card | Fully compatible |
| Who it's for | People who want lighter, more open and fully theirs | People who need every app and payment to work |

**Why not a custom Android ROM?** A stripped-down ROM loses Google-certified tap-to-pay and many banking apps, yet still
carries all of Android's weight. If we're going bare-bones, we go to Linux. AdnOS for Android therefore never modifies
the OS. It ships as ordinary apps that take Android's system roles: home, keyboard, dialer, camera and notification
listener.

## 5. Device classes and postures

Following [Luma's model](https://github.com/SimplyLuma/luma/blob/main/docs/developer-platform/tv-mode.md):

- **Device class** is fixed per image: `handheld`, `tablet`, `desktop` or `tv`. It sets resource policy (for example how
  eagerly the Android runtime starts).
- **Posture** is how the shell presents itself right now, and it can change live without logging out. A desktop can
  switch into TV posture; a docked phone can take a desktop posture.
- **Apps ask for the posture**, never the screen size. Every app keeps one ID, one package and one set of data across postures.

| Posture | Input | Bayt looks like |
|---|---|---|
| Phone | Touch, Ruq'a (keys, voice) | Full-screen grid and a bottom bar |
| Tablet | Touch, pen, keyboard | A larger grid; Itlala shows two rows of cards |
| Desktop | Mouse and keyboard | Launcher overlay and a panel; apps in windows |
| TV | Remote (HDMI-CEC), gamepad, keyboard | A 10-foot grid driven by the D-pad. Back always goes home, the pointer hides itself, and the screen never sleeps during playback. |

## 6. Architecture

### 6.1 Layers

```
┌─────────────────────────────── What you see ───────────────────────────────┐
│  Bayt · Itlala · Ruq'a · Mizan · Souk           (designed once in Ni'ma)    │
├───────────────────────────── Native front ends ────────────────────────────┤
│  Linux: Rune (Rust, GPU)                   Android: Jetpack Compose (Kotlin) │
├─────────────────────────────── AdnOS core ─────────────────────────────────┤
│  Rust: Sada client · Ruq'a engines · settings · app catalog · models        │
│  Exposed to Kotlin through UniFFI / JNI                                     │
├──────────────────────────────── Platform ──────────────────────────────────┤
│  Linux: Nur (Smithay compositor) · image-based base     Android: system roles │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 6.2 Why this split

- **The core is shared.** Everything that isn't drawing (sync, handwriting recognition, settings, data) is written once
  in Rust. Firefox for Android shares its Rust sync code with desktop Firefox the same way, through UniFFI.
- **The front ends are native.** Android's system roles (home screen, `InputMethodService`, `InCallService`,
  Camera2/CameraX, notification listener) can only be reached through Android's Java/Kotlin framework, so the Android
  front end is Kotlin. On Linux we control the whole stack, so we use Rune.
- **No Qt and no GTK.** Both were considered. We chose a Rust-first stack for licensing clarity, performance and
  friendliness to coding agents.

### 6.3 Nur and the shell on Linux

A shell is not an ordinary window. Its bar, launcher, card view, lock screen and keyboard need special placement. Most
desktops solve this with the `wlr-layer-shell` protocol. AdnOS instead makes the compositor shell-aware:

- **Nur** is a Wayland compositor built on [Smithay](https://github.com/Smithay/smithay), the Rust compositor library
  behind niri and COSMIC.
- Bayt, the status bar and Ruq'a are ordinary Rune windows. Nur recognizes trusted AdnOS shell clients by app ID and
  places them as shell surfaces: anchored, layered and space-reserving. If one crashes, Nur restarts it and your apps keep running.
- **Itlala is drawn inside Nur**, because only the compositor holds every app's live window image, which is what makes
  smooth, live cards possible.
- Nur also implements the standard `wlr-layer-shell`, `input-method-v2`/`text-input-v3` and `session-lock` protocols so
  third-party bars, keyboards and lock screens keep working.
- During development Nur runs **nested as a window** inside an existing desktop (for example Hyprland).

## 7. Components

| Component | Linux | Android | Notes |
|---|---|---|---|
| **Bayt** | Rune shell client of Nur | Compose `HOME` activity | Same layout rules from Ni'ma; lists Flatpak, Android and (later) Windows apps side by side |
| **Itlala** | Inside Nur | Compose, from the recent-apps data Android exposes to launchers | What Android exposes to a non-system launcher is limited; to be investigated |
| **Ruq'a** | Rune surface, text through `input-method-v2` | `InputMethodService` wrapper + Compose UI | Prediction and speech-to-text engines in the Rust core; pen and gesture methods later |
| **Ni'ma** | Generated Rune theme and components | Generated Compose theme and components | Section 8 |
| **Mizan** | Rune app | Compose app (AdnOS-scoped settings only) | |
| **Souk** | Flatpak (Flathub + an AdnOS repo) | F-Droid repositories + hand-off to Play | One storefront with a swappable catalog behind it |
| **Nur** | Smithay compositor | n/a | |
| **Sada** | Rust client + server | Same Rust client | Section 11 |
| **Camera** | Later, via libcamera | CameraX: every lens, RAW, video | Google's computational photography stays in Google's own camera app |
| **Dialer** | Later, via ModemManager | `InCallService` + default-dialer role | Voicemail by calling the number; visual voicemail depends on the carrier |

## 8. Design once, build everywhere

**Goal:** design a screen such as the Ruq'a keyboard once, then build it automatically for each target: Compose on
Android, Rune on a Linux phone.

- **Ni'ma design format:** a declarative description of a screen or component (layout, Ni'ma tokens, states, gestures
  and motion), stored as text in the repo. It holds structure and look, not logic. Behavior lives in the Rust core and
  is wired up by name.
- **Generators:** `nima build --target rune` emits Rust; `nima build --target compose` emits Kotlin. Design tokens
  (color, type, spacing, radius, motion curves) come from one JSON source.
- **Studio:** a browser editor with Edit / Try / Map / Load / Live, built as a **fork of
  [PopkaStudio](https://github.com/PopaSpinka/PopkaStudio)** (MIT), with device targets in place of ESP32 boards.
  Previews run real Rune code compiled to WebAssembly (Rune already has a web target), so the preview is the product,
  just as in PopkaStudio. Improvements that make sense upstream go back to PopkaStudio.
- **Keeping the targets identical:** screenshot tests render each screen headlessly in Rune and in Compose and compare
  the results ([Luma's `lumaui-conform`](https://github.com/SimplyLuma/luma/tree/main/tools/lumaui-conform) is prior art).
- **Limits:** generation works for a fixed vocabulary of Ni'ma components. Platform-specific glue (for example Android's
  `InputMethodService`) is hand-written once per platform.

## 9. The full OS image

Following [Luma's composition model](https://github.com/SimplyLuma/luma/blob/main/image/luma-desktop/Containerfile):

| Area | How Luma does it | What AdnOS does | What we change |
|---|---|---|---|
| Base | Official Fedora Silverblue image, pinned by digest, in a `Containerfile` | Fedora Atomic base, pinned by digest | Our shell (Nur + Rune) instead of GNOME Shell and Phosh |
| Delivery | Exported to a signed OSTree repository; devices never pull the container directly | Same | |
| Device classes | `/etc/luma-device-class` selects policy | `/etc/adnos/device-class` | |
| Devices | One manifest per device and per SoC family, with schemas | Same | |
| Phone kernel | One mainline-based tree with a branch per SoC family | Same approach | |
| Testing phones | A "behavioral twin": the phone userspace in QEMU | Same, before any real phone works | |
| Pinning | Every outside source pinned and hashed | Same | |

## 10. Apps

- **Linux apps:** Flatpak, from Flathub and an AdnOS repository. Souk prefers apps that adapt to small screens.
- **Android apps on AdnOS:** [Waydroid](https://waydroid.org), following Luma's runtime rules. Each Android app appears in
  Bayt like any other app and Android's own launcher stays hidden. Resource ceilings depend on the device class:

  | Class | Starts | Kept warm |
  |---|---|---|
  | Handheld | When an Android app is opened | No |
  | Tablet / Desktop | At sign-in | Yes |
  | TV | At sign-in | Yes |

  Calls, SMS, networking, power and updates must never depend on the Android runtime.
- **Windows apps (later):** Wine, plus FEX for x86 emulation on ARM, with each app in its own sandbox.
- **Linux apps on AdnOS for Android (experimental):** possibly through Android's built-in Linux Terminal VM on supported phones.

## 11. Sada: sync

A tiny, end-to-end encrypted sync service you host yourself.

- **The server is a simple relay.** Devices encrypt their changes before upload. The server keeps an ordered, encrypted
  change log per account, stores encrypted file blobs and notifies your other devices. It can't read anything.
- **Merging happens on the devices:** CRDTs for collaborative data such as notes, and last-writer-wins with
  hybrid-logical clocks for simple records.
- **Cloudflare first:**

  | Need | Cloudflare piece |
  |---|---|
  | API | Worker |
  | Per-account hub and ordered log | Durable Object with SQLite storage |
  | Live push | WebSockets with hibernation |
  | Files | R2 |

  One-command deploy (`wrangler deploy`) or a "Deploy to Cloudflare" button. A personal account should fit in the free tier.
- **Self-hosted:** the same TypeScript server with SQLite and filesystem (or S3-compatible) adapters, shipped as one
  container. We'll also evaluate `workerd`, Cloudflare's open-source runtime.
- **Pairing:** scan a QR code shown on an existing device. It carries the server address and the key. No third-party account.
- **What syncs, in order:** notes, contacts, calendar, tasks → settings and the Bayt layout → bookmarks and browser logins → files.
- **Not in v1:** CalDAV/CardDAV. An optional bridge may later run on an always-on machine you trust, never on the phone.

## 12. Phones

- **AdnOS for Android runs on any modern Android phone**, including the Pixel 10 Pro XL.
- **The full AdnOS on phones requires mainline Linux.** Support for the Pixel 10 family (Tensor G5, `laguna`) is at its
  very beginning upstream, and its PowerVR GPU needs a separate graphics stack. We expect a long road and will
  collaborate with existing efforts, such as [omarchy-mobile#18](https://github.com/derekross/omarchy-mobile/issues/18),
  instead of duplicating them.
- **Until then:** develop the phone posture in a QEMU behavioral twin, and consider a development phone with strong
  mainline support.

## 13. TV

TV follows the same two-edition split as phones, for the same reason: premium streaming depends on a certified OS.

- **AdnOS for Android TV** (for example the Google TV Streamer): **stock Google TV, unmodified**, with Bayt, the Ruq'a
  keyboard, Souk and Sada installed as apps. Netflix, Disney+ and the rest keep **Widevine L1, 4K HDR / Dolby Vision
  and Dolby Atmos**, because the OS stays certified. A custom Android TV build would lose that, so we won't make one.
  - The keyboard can be replaced normally.
  - Google TV gives its own home screen priority. Third-party launchers take over the Home button either through an
    accessibility service (the approach Projectivy Launcher uses) or by disabling the stock launcher over ADB. We'll
    support both, and document what each one breaks.
- **AdnOS TV (Linux)** on hardware with strong Linux media support: x86 mini PCs, Raspberry Pi 5, and possibly Amlogic
  boxes. Great for your own media (Kodi or Jellyfin with HDR and Dolby Atmos/TrueHD **passthrough** on the right
  hardware), YouTube, games and the web.
  - **Limit:** premium services on Linux play only in a browser, at reduced resolution, without HDR or Atmos.
  - **Waydroid doesn't solve this.** It has no hardware-backed Widevine L1, no HDR output path and no audio bitstream
    passthrough, so streaming apps through Waydroid would be a downgrade. Revisit if that changes.
- **The Google TV Streamer can only run AdnOS for Android TV.** Its bootloader is locked with no known unlock, and the
  only known root is a temporary, per-boot exploit, so a Linux port isn't realistic.

## 14. Rafiq: the assistant

**Goal:** a family voice assistant you can hold a real conversation with, that knows who is speaking, keeps children
safe, and follows you from room to room and out the door.

### 14.1 Agent

- Built on **[Pi Durable](https://developers.cloudflare.com/agents/harnesses/pi/)** (Earendil, MIT, experimental as of
  Pi 1.0). Conversations are checkpointed, survive crashes, resume on any machine and can be joined by several
  people. It runs on any JavaScript runtime, including **Cloudflare Durable Objects**.
- Tools come through **MCP**, which Pi supports natively, including Home Assistant's built-in MCP server (section 15).
- Model-agnostic: cloud models for the richest conversation, local models for privacy and offline use, chosen per household.

### 14.2 Voice

| Piece | Approach |
|---|---|
| Wake word | On-device, always local |
| Speech-to-text | Ruq'a's speech engine: local where the hardware allows it (Whisper-class models), cloud optional |
| Text-to-speech | Local neural voices (Piper/Kokoro-class), cloud optional |
| Live conversation | Streaming speech in and out with barge-in (talk over it to interrupt), like today's realtime voice models; offered as an optional cloud mode |

### 14.3 Knowing who's speaking

- **Enrollment:** each family member records a short voice sample. Rafiq builds a **voiceprint** (a speaker embedding)
  on the device.
- **Identification:** each request is matched against the household's voiceprints and answered as that person: their
  calendar, reminders, music and permissions.
- **Voiceprints are biometric data.** They are stored only on household devices, encrypted, synced end to end through
  Sada, never sent to a model provider, and deletable at any time. Laws such as Illinois' BIPA and COPPA apply. A legal
  review is required before release.

### 14.4 Kid-safe mode

- **Enrolled children** get the profile their parents configure: safe answers, no purchases, limited smart-home control
  (no locks or alarms), quiet hours and an activity summary for parents.
- **Unknown voices that sound young:** voice age estimation is unreliable, so it is used only to tighten safety, never
  to loosen it. Rafiq falls back to kid-safe mode when unsure. Adults can confirm themselves with a voice match or their
  phone.
- **Default:** unknown speakers are treated as guests with limited access.

### 14.5 Follow me

- **One conversation, many devices.** The conversation lives in Pi Durable, not on any single device. A wall panel, TV,
  desktop, phone or watch attaches to the same session.
- **Which device answers:** the one that hears you best (wake-word confidence and loudness), refined by presence from
  Home Assistant (room sensors) and from your phone or watch.
- **Leaving home:** the session moves to your phone or watch without interruption, because nothing is tied to the house.
- **Devices:** AdnOS devices; wall panels (an AdnOS device or a tablet running Manzil); Wear OS watches through an
  AdnOS for Android watch app.

### 14.6 Memory, data access and Sada

Sada encrypts everything at rest and in transit. Rafiq is simply another **authenticated Sada client**, signed in with
its own keys, so it never weakens that encryption.

- **Scoped memory.** Rafiq has its own memory collection in Sada (people, preferences, routines, ongoing tasks). Its
  keys unlock **only that collection**, not everything the household stores. Pi Durable runs with that memory
  decrypted while you're signed in, the same as any of your devices.
- **Personal data stays where it lives.** For notes, photos, messages and files, Rafiq doesn't copy data into its
  memory or the cloud. It uses **tools on the device you're using**:
  - "Reference my notes about the trip" searches and opens your notes **on your phone**.
  - "Show me that picture of Sam at the beach" opens it in Photos **on the device in front of you**.
  - Only the small result the answer needs (for example a few matching lines) goes back into the conversation, and
    only for that request.
- **Permissions are per tool and per family member.** A child's profile can't reach a parent's notes or photos.
- **Where Rafiq runs** (your Cloudflare account, a home server, or an AdnOS device) changes only latency and offline
  behavior, not what it can access.
- **The model provider:** whatever a request includes is sent to the language model. Households that want nothing
  to leave their hardware choose a local model.

## 15. Manzil: smart home

**We do not build a smart-home hub or backend.** Home Assistant is the backend. Manzil is a front end.

- **Connection:** Home Assistant's documented **WebSocket API** for live state and actions, signing in with its
  standard OAuth flow (the same approach as the official companion apps). Works locally, or remotely through the
  household's existing Home Assistant remote access.
- **Assist and MCP:** Rafiq controls the home through Home Assistant's MCP server and can use its Assist voice pipeline.
- **Look:** a modern home-app layout (rooms, scenes, devices as cards) with webOS character: the gesture area, the Wave
  for scenes, the card view, bottom notifications, soft cards and glanceable state. Not a copy of any existing app.
- **Surfaces:**

  | Surface | Form |
  |---|---|
  | Phone | Manzil app (AdnOS and AdnOS for Android) |
  | Wall panel | Kiosk posture: always on, dims at night, wakes on approach |
  | Desktop | App plus Bayt widgets |
  | TV | 10-foot view: cameras, scenes and "who's at the door" |

- **First milestone:** clickable mockups of the phone, wall-panel and TV views, built against real Home Assistant
  entities as early as possible.

## 16. Licensing

Apache-2.0 for code, CC BY 4.0 for docs, CC BY-SA 4.0 for media, with SPDX tags on every file. Upstream material keeps
its license. See [LICENSING.md](../LICENSING.md).

## 17. Risks and technologies we're watching

| Item | Why it matters | Status |
|---|---|---|
| **Rune maturity** | Pre-release and a few weeks old. Android support is its last milestone. It accepts issues, not pull requests. | Adopted for Linux and watched for Android. Our collaboration is through well-written issues. |
| **Two front ends** | Every screen exists in Rune and Compose | Mitigated by Ni'ma generation and screenshot tests |
| **Pixel 10 mainline** | No usable mainline support yet | AdnOS for Android covers it meanwhile |
| **Itlala on Android** | Non-system launchers see limited recent-app data | To be prototyped early |
| **Pi Durable maturity** | Experimental; its API may change | Keep Rafiq's agent layer thin and swappable |
| **Voice ID & children** | Biometric privacy law; age estimation is unreliable | On-device voiceprints, fail-safe kid mode, legal review |
| **Sada security** | Key handling and recovery are hard to get right | Use audited primitives; get a design review before v1 |

## 18. Open questions

- Ruq'a's later methods: pen and handwriting recognition, and sign-language gesture recognition (camera-based). Which comes first, and which models?
- A name for the Studio.
- Fedora Atomic or another image-based base? (Luma's success with Fedora is the current default.)
- The exact Ni'ma design format: JSON, KDL, or a Rust/Kotlin-neutral DSL.
- Which development phone to recommend for the full OS.
- Rafiq's default model mix: which parts are local and which are cloud, on each device class.
- Wall-panel hardware to recommend for Manzil and Rafiq.

---

<sub>This document is licensed under CC BY 4.0.</sub>
