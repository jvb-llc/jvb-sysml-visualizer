# jvb-sysml-visualizer Requirements (Backend)

This document outlines the detailed requirements for the `jvb-sysml-visualizer` C++ Backend project using the Easy Approach to Requirements Syntax (EARS).

## Functional Requirements

*   **F-1 (Data Loading):** The `jvb-sysml-visualizer` C++ Backend shall load SysML model data. This data is provided by the local `jvb-cli` (acting as a client) or directly by the Frontend (Standalone Mode). This upload process shall return a **Session ID** (or similar identifier) that the Frontend uses to subscribe to the generated scene graph data stream.
*   **F-13 (CLI Authentication):** In Embedded Mode, the `jvb-cli` shall handle authentication with the Remote C++ Backend (e.g., via API keys or tokens) to perform the initial model upload and obtain the Session ID.

## Architectural Requirements

*   **A-1 (Component Architecture):** The `jvb-sysml-visualizer` system shall be architected across two repositories:
    1.  **Remote C++ Backend (`jvb-sysml-visualizer` repo):** A native executable acting as an authenticated WebSocket Server for heavy processing, visual logic, and semantic understanding.
    2.  **C++ Frontend (`jvb-sysml-visualizer-ui` repo):** A WASM executable responsible for rendering and user interaction.
    3.  **Local Controller (jvb-cli):** (Optional) A local tool acting as both a client to the Remote C++ Backend (for data upload) and a secondary WebSocket server to the Frontend (for local control).
*   **A-2 (Backend Role & SysML Analysis):** The C++ Backend shall be responsible for:
    *   Receiving SysML model content directly from the `jvb-cli` or User.
    *   Interfacing with the `jvb-sysml-analyzer` (Haskell) to validate and achieve semantic understanding of SysML models.
    *   Applying pre-defined visual logic (e.g., "what a block looks like") to SysML elements.
    *   Processing the SysML model data, combining semantic understanding and visual logic.
    *   Generating the **Scene Graph and View Model**.
*   **A-5 (Data Interface: Backend to Analyzer):** The data interface between the C++ Backend and the `jvb-sysml-analyzer` (Haskell) shall be defined and versioned, considering the `jvb-sysml-analyzer`'s API (SMA-U-7).

## Challenges to be Addressed

*   **CH-5 (IP Separation Enforcement):** A technical strategy must be defined to enforce the strict frontend/backend separation, ensuring no proprietary code is accidentally compiled into the distributable WASM frontend.
*   **CH-6 (Protobuf Schema Design)::** The initial version of the `.proto` schema must be designed. This schema is a critical interface between three components (Haskell analyzer, C++ backend, C++ frontend) and must be carefully considered.
*   **CH-10 (Scene Graph Optimization)::** A strategy for efficient transmission and updates of the scene graph is required to ensure high-performance rendering and responsiveness, balancing the richness of the data with the bandwidth constraints of the WebSocket connection.
*   **CH-12 (Schema Synchronization)::** With the frontend and backend in separate repositories, a robust mechanism (e.g., a shared git submodule for `.proto` files) is needed to ensure both build off the exact same version of the Protobuf definitions.

## Resolved Challenges

*   **CH-1 (Build Toolchain):** The challenge of a complex, multi-target build environment has been resolved by separating the project into two distinct repositories: `jvb-sysml-visualizer` (for the C++ Backend) and `jvb-sysml-visualizer-ui` (for the C++ Frontend/WASM). This allows each component to have its own simplified, independent build system.
*   **CH-19 (Backend-Analyzer Interface):** Resolved by adopting **gRPC Protobuf** over a local socket connection. This enables high-performance, strongly-typed communication between the C++ Backend and the Haskell `jvb-sysml-analyzer`, leveraging the sidecar pattern for efficient deployment in containerized environments like Google Cloud Run or Kubernetes.