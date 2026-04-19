# SPZ Ecosystem

> Open-source toolchain and compliance verification system around the Khronos SPZ 3D Gaussian Splatting compression specification

## Core Projects

### spz2glb (v2.0.1 Stable)

**Responsibility Boundary**: Does only two things—**SPZ→GLB format packaging** and **GLB delivery workflow**

- **Core Positioning**: Lossless packaging (SPZ compressed stream stored unchanged in GLB)
- **Repository**: https://github.com/spz-ecosystem/spz2glb
- **Live Demo**: [GitHub Pages](https://spz-ecosystem.github.io/spz2glb/)
- **Key Features**:
  - WASM memory and API capabilities (pre-allocation, explicit release, statistics and dual-tier configuration)
  - Dual-end synergy: browser-side lightweight preview/quick validation; local CLI for heavy-duty conversion/batch processing/deep verification
  - Three-layer verification: structure verification / lossless verification / decoding consistency verification

### spz_gatekeeper (v2.0.1 Stable)

**Responsibility Boundary**: L2-only SPZ legality checker for validating headers, flags, and TLV trailer extensions without changing baseline SPZ decoding behavior

- **Core Positioning**: SPZ extension compliance review and compatibility validation
- **Repository**: https://github.com/spz-ecosystem/spz_gatekeeper
- **Live Demo**:
  - GitHub Pages: https://spz-ecosystem.github.io/spz_gatekeeper/
  - CloudBase (China CDN): https://openclaw-spz-3gt7x2sya7c10ef2-1355411679.tcloudbaseapp.com/
- **Key Features**:
  - Validates SPZ header: magic, version, point count, SH degree, flags, reserved
  - Validates TLV trailer layout after base payload
  - Validates known vendor extensions (e.g., Adobe Safe Orbit Camera `0xADBE0002`)
  - Warns on higher SPZ versions, continues best-effort validation

---

## QR-Code Native Distribution System (Experimental)

> **Experimental Declaration**: This is an experimental 3D asset distribution system, aiming to provide the community with an experimental solution using QR codes as a unified carrier for issues such as **large file sizes difficult to transfer**, **high storage costs**, **fragmented standards causing compatibility chaos**, and **lack of lightweight instant distribution means**.

A QR-code-based full-chain SPZ/GLB distribution and verification system:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    QR-Code Native Distribution Architecture              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│    Data Admission Layer      Distribution Auth Layer    Consumption      │
│    ┌──────────┐           ┌──────────┐           ┌──────────┐           │
│    │  SPZ     │           │ Gatekeeper│           │  spz2glb │           │
│    │ Dataset  │──────────→│  Auth    │───────────→│  Web     │           │
│    └──────────┘           └──────────┘           └──────────┘           │
│                                │                        │               │
│                                ↓                        ↓               │
│                          Generate QR with           Scan-to-Convert     │
│                          authorization              WASM lossless       │
│                                │                    packaging            │
│                                └────────────────────────┘               │
│                                          │                              │
│                                          ↓                              │
│                                   GLB Download Delivery                 │
│                                   (or QR re-distribution)               │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### System Workflow

| Phase | Responsibility | Carrier | Status |
|-------|----------------|---------|--------|
| **Admission Check** | Gatekeeper L2 legality check | SPZ file | ✅ `spz_gatekeeper check-spz` |
| **Authorized Distribution** | Valid SPZ/GLB generates distributable QR | QR Code | 🚧 Gatekeeper built-in generation (planned) |
| **Scan-to-Verify** | Gatekeeper validates authorization in QR | QR Code | 🚧 `spz_gatekeeper verify-qr` (planned) |
| **Scan-to-Convert** | spz2glb web reads QR and converts | QR→SPZ→GLB | 🚧 v3.0 experimental support (planned) |
| **QR Re-distribution** | GLB can also be redistributed via QR | QR Code | 🚧 Experimental support |
| **Scan-to-Render** | Browser direct rendering (future exploration) | QR→Render | 🔬 WebGL/Three.js (experimental) |

### Core Design

- **Scan-to-Verify**: Gatekeeper embeds SPZ file hash and compliance signature in QR; scan first validates then converts, ensuring data source trustworthiness
- **Scan-to-Convert**: spz2glb web recognizes QR, automatically pulls SPZ and executes WASM lossless packaging to GLB
- **QR Re-Distribution**: Generated GLB can be repackaged as QR, enabling chain lightweight distribution of assets
- **Dataset Admission Baseline**: All SPZ data entering the ecosystem must pass gatekeeper validation and generate authorized QR

### Community Pain Points Addressed

| Pain Point | Traditional Solution | QR-Code Native System |
|------------|---------------------|----------------------|
| **Large files, difficult transfer** | Cloud drive/email large file transfer | QR lightweight sharing, scan to download/convert |
| **High storage costs** | Long-term hosting of large files | QR points to object storage, on-demand pull |
| **Fragmented standards, compatibility chaos** | Various format extensions work independently | Gatekeeper unified validation, only compliant data generates QR |
| **Lack of instant distribution** | Install dedicated software or plugins | WeChat/Mini Program scan to use, no install needed |
| **Poor mobile experience** | Large file download crashes, rendering lag | WASM pre-check memory budget,超限引导至 CLI |

### Technical Constraints

- **HTTPS Direct Link**: SPZ/GLB files pointed to by QR must have CORS configured
- **Mobile Memory Budget**: 32MB tier, files exceeding limit are directed to CLI processing
- **Status**: Experimental development, v3.0 experimental release plans scan-to-convert, with scan-to-render as a future exploration

---

## Quick Navigation

| Project | Documentation | Demo |
|---------|---------------|------|
| spz2glb | [Wiki](https://github.com/spz-ecosystem/spz2glb/wiki) | [GitHub Pages](https://spz-ecosystem.github.io/spz2glb/) |
| spz_gatekeeper | [README](https://github.com/spz-ecosystem/spz_gatekeeper/blob/main/README.md) | [GitHub Pages](https://spz-ecosystem.github.io/spz_gatekeeper/) / [CloudBase](https://openclaw-spz-3gt7x2sya7c10ef2-1355411679.tcloudbaseapp.com/) |

---

## Roadmap

The subsequent version roadmap will be published after v2.1 goes live. The current cadence is quarterly; v3.0 is tentatively targeted for Q3 and is planned to align with the next stage of the SPZ_2 draft (subject to final release notes).

---

## Related Projects

- [spz_gatekeeper](https://github.com/spz-ecosystem/spz_gatekeeper) - SPZ legality checker
- [spz2glb](https://github.com/spz-ecosystem/spz2glb) - SPZ→GLB format packaging and distribution tool
- [spz-entropy](https://github.com/spz-ecosystem/spz-entropy) - SPZ entropy encoding optimization (in development)
- [KHR_gaussian_splatting](https://github.com/KhronosGroup/glTF/pull/2490) - Khronos official 3D Gaussian Splatting glTF extension
- [KHR_gaussian_splatting_compression_spz_2](https://github.com/KhronosGroup/glTF/pull/2531) - Khronos SPZ_2 compression extension proposal (current draft)

---

## About SPZ Ecosystem

SPZ Ecosystem is dedicated to developing and maintaining open-source tools for the Khronos SPZ 3D Gaussian Splatting compression specification. Our goal is to make high-quality 3D content compression accessible, efficient, and compatible across the 3D graphics and AIGC industries.

**Design Principles**:
- **Single Responsibility**: Each tool does only what is within its defined boundary
- **Verifiable**: All output must pass standardized validation
- **Dual-End Synergy**: Browser-side lightweight interaction, CLI for heavy task processing
- **QR-Code Native**: QR codes as the unified distribution entry point, achieving full-chain closed loop of admission-authorization-consumption, providing an experimental solution for 3D asset distribution dilemmas

---

## Community Guidelines

We are committed to fostering an open, inclusive, and collaborative community. All participants should abide by our [Code of Conduct](https://github.com/spz-ecosystem/.github/blob/main/CODE_OF_CONDUCT.md).

## How to Contribute

We welcome contributions from everyone! Whether you are a developer, researcher, or industry partner, there are many ways to get involved:

1. **Code Contributions**: Submit pull requests to our core repositories
2. **Issue Reporting**: Help us improve by reporting bugs or requesting features
3. **Documentation**: Improve our guides, tutorials, and API references
4. **Community Support**: Answer questions in discussions and help new contributors

For detailed instructions, please read our [Contributing Guide](https://github.com/spz-ecosystem/.github/blob/main/CONTRIBUTING.md).

---

## Contact & Collaboration

- **Organization Email**: 175851233+gugu23456789@users.noreply.github.com
- **Khronos Specification PR**: [KhronosGroup/glTF#2534](https://github.com/KhronosGroup/glTF/pull/2534)
- **Maintainer**: [gugu23456789](https://github.com/gugu23456789)

We actively seek collaboration with industry partners, especially those working in 3D AIGC, digital twin, and real-time rendering fields. If you are interested in becoming a core maintainer or participating in the project, please contact us!

---

## License

This organization is maintained by the SPZ Ecosystem community and licensed under the [MIT License](https://github.com/spz-ecosystem/.github/blob/main/LICENSE).

Cultural Note: This project is released in the year of the Red Fire Horse (Bingwu, 丙午), Huangdi Era 4723. It honors the ancient Chinese lunisolar calendar, a testament to humanity's enduring quest to harmonize with the cosmos.
