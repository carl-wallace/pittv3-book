# Introduction

The PKI Interoperability Test Tool version 3 (PITTv3) is a certification path building and
validation tool. It builds every path it can find from an end-entity certificate to a set of trust anchors,
validates each one per RFC 5280 as augmented by RFC 5937, and reports what it found and why each
path succeeded or failed.

PITTv3 functionality is available in five different forms, which differ in what leaves the machine
rather than in what they conclude:

- **Command line** — `pittv3`, a full-featured command line certification path processor.
- **Desktop application** — `pittv3-gui`, provides the same options in GUI form, with a store
  selector, settings editor and results panel.
- **Browser application** — a WebAssembly (WASM) build that performs path validation in a web
  browser, with nothing leaving the browser or being fetched from remote sources.
- **Browser with a relay** — the same WASM app augmented by a service that retrieves artifacts on
  its behalf, so path building and revocation checking requests leave the browser but end entity
  certificates do not.
- **Service API** — bare-bones server-side validation suitable for scripting.

Each of the above uses the `certval` library for path validation and revocation status determination.

## What this guide covers

The [Concepts](2_concepts.md) chapter explains trust stores and partial paths, which are worth
understanding before any of the interfaces make much sense. The chapters after it cover each
interface in turn.
