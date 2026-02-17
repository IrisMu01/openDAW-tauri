# openDAW Repository Structure

## Root

The monorepo uses npm workspaces with Turbo for orchestration and Lerna for publishing. All source packages live under `packages/`, split into `app`, `lib`, `studio`, `server`, and `config`.

```
openDAW-tauri/
  packages/
    app/          # Runnable applications (studio, lab, nam-test)
    lib/          # Shared libraries (std, dom, dsp, midi, etc.)
    studio/       # DAW domain packages (core engine, adapters, boxes, processors)
    server/       # Backend services (yjs-server)
    config/       # Shared ESLint and TypeScript configs
  src-tauri/      # Tauri desktop shell (Rust)
  scripts/        # Build/deploy helpers (cert.sh, clean.sh)
  docs/           # Documentation
```

---

## packages/app/

### @opendaw/app-studio
The main DAW web application. Vite-based SPA that provides the full studio UI: timeline, mixer, piano roll, device panels, canvas rendering, and project management. Entry point is `index.html` loading `src/main.ts`.

- `src/ui/` — 30+ UI subsystems (timeline, mixer, piano-panel, devices, canvas, menu, hooks, components)
- `src/service/` — business logic services (project, audio, recording, shortcuts)
- `src/audio/` — audio engine integration and output device management
- `src/boot.ts` — app initialization sequence (feature detection, worker setup, AudioContext creation)
- `src/features.ts` — runtime feature checks (OPFS, AudioWorklet, Web Crypto)

### @opendaw/lab
Lightweight Vite app for testing library components and DSP algorithms without the full studio overhead. Contains oscilloscope, slider, and minimal test UI.

### @opendaw/nam-test
Specialized app for testing Neural Amp Modeler (NAM) WASM integration for guitar amp simulation.

---

## packages/lib/

### @opendaw/lib-std
Foundational utilities used everywhere. Arrays, binary search, color, geometry, hashing, validation, data structures, error handling, UUID generation, type guards (`isDefined`, `isAbsent`, `Optional<T>`, `Nullable<T>`).

### @opendaw/lib-box
Reactive state management system. Provides observable "box" primitives with dependency graphs, pointer abstractions, editing operations, and synchronization. The core data model layer that all DAW objects are built on.

### @opendaw/lib-dom
DOM utilities, file system access, CSS helpers, HTML element creation, drag-and-drop, keyboard shortcut management, compression, and browser detection (`Browser.isTauriApp()`, `Browser.isWeb()`, `Browser.isFirefox()`, etc.).

### @opendaw/lib-jsx
Custom JSX rendering system (not React). Provides `createElement`, dependency injection, routing, and reactive component lifecycle — a lightweight framework for building the studio UI.

### @opendaw/lib-runtime
Async and concurrency primitives. Message passing (`Communicator`, `Messenger`), promise utilities, network helpers, and fetch wrappers for inter-process communication between main thread and workers.

### @opendaw/lib-dsp
Comprehensive DSP toolkit. ADSR envelopes, biquad filters, FFT, delay lines, crushers, reverb algorithms (FreeVerb, Dattorro), oscillators, audio analysis, glide, and the CTAG Dynamic Range Compressor port.

### @opendaw/lib-midi
MIDI file parsing, generation, and event handling. Decodes/encodes standard MIDI files, represents tracks, note events, control changes, and meta events.

### @opendaw/lib-xml
XML parsing and serialization utilities.

### @opendaw/lib-dawproject
Implementation of the DAWProject interchange format for importing/exporting projects between DAWs.

### @opendaw/lib-fusion
Advanced reactive composition and DOM rendering framework. Provides live streams, reactive lists, and OPFS (Origin Private File System) worker-based file I/O.

### @opendaw/lib-box-forge
Compile-time code generator using ts-morph. Reads box schemas and generates TypeScript type definitions for the `studio-boxes` package.

---

## packages/studio/

### @opendaw/studio-core
The heart of the DAW engine. Audio rendering, MIDI sequencing, plugin integration, project persistence, and cloud sync.

- `src/project/` — project file I/O, migration, audio file management
- `src/capture/` — audio recording
- `src/midi/` — MIDI input/output
- `src/cloud/` — cloud storage (Dropbox, Google Drive) and OAuth
- `src/ffmpeg/` — FFmpeg WASM integration for encoding/decoding
- `src/soundfont/` — soundfont loading (OpenSoundfontAPI)
- `src/samples/` — sample library (OpenSampleAPI)
- `src/ysync/` — Yjs-based real-time collaboration (YService)
- Key files: `EngineProcessor.ts`, `EngineFacade.ts`, `OfflineEngineRenderer.ts`, `Mixer.ts`, `Storage.ts`

### @opendaw/studio-adapters
Bridge between the reactive box system and DAW domain models. Adapts audio units, devices, timelines, clips, regions, parameters, and automation to the box graph.

- `src/engine/` — engine state adaptation
- `src/devices/` — instrument, audio effect, and MIDI effect adapters
- `src/timeline/` — timeline, clip, region, and event adapters
- `src/audio-unit/` — audio unit wrappers
- `src/modular/` — modular synth routing
- `src/sample/`, `src/preset/`, `src/grooves/`, `src/nam/`, `src/soundfont/`

### @opendaw/studio-boxes
Auto-generated box definitions for 90+ DAW object types (devices, clips, regions, audio units, etc.). Generated by `studio-forge-boxes`.

### @opendaw/studio-forge-boxes
Code generator that reads schemas from `src/schema/` and produces the `studio-boxes` source files via ts-morph. Runs as a build step before `studio-boxes`.

### @opendaw/studio-enums
Shared enumerations and constants: UI colors, icon symbols, audio unit types, pointer definitions.

### @opendaw/studio-core-processors
AudioWorklet processor implementations for real-time DSP in the browser. 50+ processors including the main engine processor, device chains, note sequencer, block renderer, FreeVerb, metronome, mixer, and metering.

### @opendaw/studio-core-workers
Web Workers for background processing. Main worker bootstrap, offline rendering engine, and worklet environment setup.

### @opendaw/studio-scripting
User-facing scripting API. TypeScript-based system for writing custom audio processors and control logic. Includes 22 built-in functions (filtering, math, etc.) and a worker-based script runner.

### @opendaw/studio-sdk
Umbrella package that re-exports all studio and lib packages. Published to npm for external developers building on the openDAW platform.

---

## packages/server/

### yjs-server
WebSocket server for real-time collaborative editing. Runs Yjs sync protocol and WebRTC signaling. Production instance at `wss://live.opendaw.studio`, local dev at `wss://localhost:1234`.

---

## packages/config/

### @opendaw/eslint-config
Shared ESLint rules for the monorepo.

### @opendaw/typescript-config
Shared TypeScript compiler settings and base `tsconfig.json`.

---

## src-tauri/

Tauri 2 desktop shell wrapping the web frontend. Rust backend for native OS integration.

- `src/main.rs` — app entry point
- `src/lib.rs` — Tauri builder setup (plugin registration)
- `Cargo.toml` — Rust dependencies (tauri 2.10.0, tauri-plugin-log)
- `tauri.conf.json` — build config, security headers (COOP/COEP), window settings
- `capabilities/default.json` — Tauri permission scoping
- `icons/` — app icons for all platforms

---

## Build Pipeline

```
npm run build
  └─ turbo build
       ├─ lib-std, lib-runtime, lib-dsp, lib-midi, lib-xml, lib-dom, lib-jsx
       │    └─ tsc → dist/
       ├─ lib-box, lib-fusion, lib-dawproject, lib-box-forge
       │    └─ tsc → dist/
       ├─ studio-forge-boxes
       │    └─ generates → studio-boxes/src/
       ├─ studio-boxes, studio-enums
       │    └─ tsc → dist/
       ├─ studio-adapters, studio-scripting
       │    └─ tsc → dist/
       ├─ studio-core-workers → studio-core/dist/workers-main.js
       ├─ studio-core-processors → studio-core/dist/processors.js
       ├─ studio-core
       │    └─ tsc → dist/
       └─ app-studio
            └─ tsc && vite build → dist/  ← FINAL FRONTEND OUTPUT
```

The final bundled web application lives in `packages/app/studio/dist/`. This is what Tauri embeds via `frontendDist` in `tauri.conf.json`.

---

## External Dependencies

| Domain | Purpose | Critical |
|--------|---------|----------|
| `api.opendaw.studio` | Sample and soundfont library API | Yes |
| `assets.opendaw.studio` | Sample and soundfont file downloads | Yes |
| `logs.opendaw.studio` | Production error reporting | No |
| `live.opendaw.studio` | Real-time collaboration (WebSocket) | No |
| `dropbox.com` / `dropboxapi.com` | Cloud storage OAuth + API | No |
| `googleapis.com` | Google Drive OAuth + API | No |
