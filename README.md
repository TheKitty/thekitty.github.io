## Helpful files for LLMs, etc.

### ESP-Claw New Board Guide

https://github.com/TheKitty/thekitty.github.io/blob/main/espclaw_new_board.md

A practical reference for contributing a new board definition to the espressif/esp-claw repository, written for engineers familiar with ESP-IDF who haven't worked with the board manager before, and for capable LLMs assisting them. It covers the full workflow end-to-end: gathering hardware information from authoritative sources, structuring the three to five files that make up an ESP-Claw board, placing them in the manufacturer-namespaced board directory layout, and verifying the result by running the board manager generator and a build. 

The document includes worked YAML examples for the common bus and peripheral types, both supported device patterns and the custom-device escape hatch for hardware that doesn't fit, the canonical setup file structure including a bridging pattern that lets non-standard displays integrate transparently with the rest of the framework, and cross-checking workflows for resolving GPIO disagreements between sources. It also covers debugging tips for first-build and first-hardware-run symptoms, plus a checklist for opening a pull request. Adafruit boards appear as concrete examples where useful, but the document is framework-agnostic and supports contributions from any vendor — Espressif, Adafruit, M5Stack, LilyGo, DFRobot, Seeed, Waveshare, and community-maintained boards — drawing on whichever authoritative source exists for the hardware in question (ESP-IDF native definitions, CircuitPython, MicroPython, Arduino BSPs, vendor SDKs, or schematics).
