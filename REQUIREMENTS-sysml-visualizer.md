# jvb-sysml-visualizer Requirements

This document outlines the detailed requirements for the `jvb-sysml-visualizer` project using the Easy Approach to Requirements Syntax (EARS).

## Functional Requirements

*   **F-1 (Data Loading):** The `jvb-sysml-visualizer` shall support two data loading modes:
    1.  **Embedded Mode:** Initial SysML model content is provided by the local `jvb-cli`.
    2.  **Standalone Mode:** SysML model content is uploaded or pasted directly by the user via the UI.
    In both cases, the content is sent to the Remote C++ Backend for processing and scene graph generation.
*   **F-2 (Planar Block Diagram Rendering):** The `jvb-sysml-visualizer` shall render interactive SysML Block Diagrams as planar objects positioned within a 3D environment.
*   **F-3 (Planar Internal Block Diagram Rendering):** The `jvb-sysml-visualizer` shall render interactive SysML Internal Block Diagrams as planar objects positioned within a 3D environment, showing the internal structure of blocks.
*   **F-4 (2D Navigation):** The user shall be able to pan, zoom, and navigate the planar diagrams using standard mouse and keyboard controls.
*   **F-5 (Element Selection):** The user shall be able to select individual elements in the diagram (e.g., blocks, ports).
*   **F-6 (Property Display):** When an element is selected, the `jvb-sysml-visualizer` shall display its properties (e.g., name, type, definition).
*   **F-7 (3D Breakouts):** The user shall be able to select a component in a planar diagram and view it as an interactive 3D model rendered in relief.
*   **F-8 (Diagram Drill-Down with Animation):** When a user double-clicks on a diagram element that represents a subsystem, the `jvb-sysml-visualizer` shall navigate into the detailed diagram of that subsystem using a 3D animation (e.g., flying into the diagram) to simulate moving deeper into the hierarchy.
*   **F-9 (Navigation Controls with Animation):** The `jvb-sysml-visualizer` shall provide navigation controls (e.g., "zoom out") that trigger a reverse 3D animation (e.g., flying out) to return to the higher-level diagram, reinforcing the hierarchical context.
*   **F-10 (Connection Management):** The `jvb-sysml-visualizer` WASM frontend shall always maintain an authenticated connection to the remote C++ Backend for data. **If** launched in Embedded Mode, it shall *also* maintain a local connection to the `jvb-cli` for control and context.
*   **F-11 (Standalone Data Input):** When running in Standalone Mode, the `jvb-sysml-visualizer` shall provide UI controls to allow the user to upload a SysML file or paste SysML text directly.

## Non-Functional Requirements

*   **NF-1 (Performance):** The system shall provide a responsive user experience during model loading, navigation, and interaction, minimizing perceived latency.
*   **NF-2 (Browser Compatibility):** The `jvb-sysml-visualizer` shall be compatible with the latest versions of Chrome, Firefox, and Safari.
*   **NF-3 (Integration):** The visualizer shall provide a clear API for the `jvb-cli` to embed and interact with it.
*   **NF-4 (Build System):** The C++ project shall use CMake as its build system.
*   **NF-5 (Dependencies):** The project shall use a C++ package manager (e.g., Conan or vcpkg) to manage external dependencies.
*   **NF-6 (Protobuf Compilation):** The build system shall include a step to compile `.proto` files into C++ source code using the `protoc` compiler.
*   **NF-7 (3D Rendering):** The visualization shall be implemented using a 3D rendering library suitable for C++ and WASM (e.g., Sokol Gfx, bgfx).
*   **NF-8 (UI):** All UI elements (e.g., property panels, buttons) shall be created using the ImGui immediate mode GUI library.
*   **NF-9 (Embedded Rendering):** For local use, the `jvb-sysml-visualizer` WASM frontend shall be capable of running in an embedded window (e.g., CEF) launched by the `jvb-cli`. The frontend shall detect this mode (e.g., via URL query parameter) and establish the local WebSocket connection to `jvb-cli`.

## Architectural Requirements

*   **A-1 (Component Architecture):** The `jvb-sysml-visualizer` system shall consist of three main components:
    1.  **Remote C++ Backend:** A native executable acting as an authenticated WebSocket Server for heavy processing, visual logic, and semantic understanding.
    2.  **C++ Frontend:** A WASM executable responsible for rendering and user interaction.
    3.  **Local Controller (jvb-cli):** (Optional) A local tool acting as both a client to the Remote C++ Backend (for data upload) and a secondary WebSocket server to the Frontend (for local control).
*   **A-2 (Backend Role & SysML Analysis):** The C++ Backend shall be responsible for:
    *   Receiving SysML model content from the `jvb-cli` (relayed via the WASM frontend or direct connection).
    *   Interfacing with the `jvb-sysml-analyzer` (Haskell) to validate and achieve semantic understanding of SysML models.
    *   Applying pre-defined visual logic (e.g., "what a block looks like") to SysML elements.
    *   Processing the SysML model data, combining semantic understanding and visual logic.
    *   Generating the rich scene graph data and interactive view models.
*   **A-3 (Frontend Role & IP Protection):** The C++ Frontend (WASM) shall be responsible for:
    *   Receiving rich scene graph data and interactive view models via Protocol Buffers.
    *   Procedurally generating 3D geometry from this data (no external 3D asset loading).
    *   Rendering the visualization.
    *   Containing no proprietary SysML processing logic or rendering decision logic.
    *   Ensuring no SysML-specific IP (business logic, formulas, or rendering algorithms) is exposed through the downloaded WASM binary.
*   **A-4 (Data Interface: Backend to Frontend):** The data interface for rendering instructions and interactive view models between the C++ Backend and the C++ Frontend shall be defined in a set of version-controlled Protocol Buffer (`.proto`) files.
*   **A-5 (Data Interface: Backend to Analyzer):** The data interface between the C++ Backend and the `jvb-sysml-analyzer` (Haskell) shall be defined and versioned, considering the `jvb-sysml-analyzer`'s API (SMA-U-7).

## Challenges to be Addressed

*   **CH-1 (Build Toolchain):** The combination of C++, CMake, a C++ package manager, Protobuf, and Emscripten (for WASM compilation) creates a complex build environment that must be robust and easy for developers to use.
*   **CH-2 (3D Library Selection):** A final decision on the 3D rendering library is required. The chosen library must be lightweight, performant in WASM, and have a suitable license.
*   **CH-3 (ImGui Integration):** A robust integration between the chosen 3D library and ImGui must be created, including handling user input from the browser or CEF environment.
*   **CH-4 (CEF Integration Strategy):** A clear strategy is needed for how the Chromium Embedded Framework will be used. This includes build/linking, how the native C++ 'backend' communicates with the WASM 'frontend' running inside CEF, and how to create a seamless user experience.
*   **CH-5 (IP Separation Enforcement):** A technical strategy must be defined to enforce the strict frontend/backend separation within a single C++ project, ensuring no proprietary code is accidentally compiled into the distributable WASM frontend.
*   **CH-6 (Protobuf Schema Design):** The initial version of the `.proto` schema must be designed. This schema is a critical interface between three components (Haskell analyzer, C++ backend, C++ frontend) and must be carefully considered.
*   **CH-7 (2D-in-3D Rendering Strategy):** A decision must be made on how to implement the 2D diagram rendering on top of a 3D library (e.g., orthographic projection vs. a dedicated 2D abstraction) to ensure both performance and ease of implementation.
*   **CH-8 (Interactive Event Handling):** Implementing robust and responsive event handling for mouse (e.g., double-click for drill-down, click-and-drag for panning) and keyboard controls within the WASM environment, especially considering potential interactions with ImGui and the underlying 3D library.
*   **CH-9 (Hierarchical Data Representation for Navigation):** The Protobuf schema needs to effectively represent the hierarchical structure of SysML models to support drill-down and drill-up navigation, ensuring that the necessary contextual information is available at each level of detail.
*   **CH-10 (Scene Graph Optimization):** A strategy for efficient transmission and updates of the scene graph is required to ensure high-performance rendering (NF-1) and responsiveness, balancing the richness of the data with the bandwidth constraints of the WebSocket connection.
*   **CH-11 (High-Quality Text Rendering):** Rendering crisp, readable text for labels and properties within a dynamic 3D environment (NF-7) that scales correctly with zoom (F-4) presents a significant technical challenge, likely requiring techniques like Signed Distance Fields (SDF).
*   **CH-13 (Mixed Content Security):** Establishing a dual connection (F-10) where a secure WASM application (HTTPS) connects to a local `jvb-cli` WebSocket (ws://localhost) may be blocked by browser "Mixed Content" security policies. A robust workaround or configuration strategy is required.
*   **CH-14 (Data Relay Latency):** The workflow where `jvb-cli` sends SysML content to the WASM frontend, which relays it to the Remote Backend for processing, introduces potential latency and bandwidth bottlenecks for large models. Optimization strategies (e.g., compression, binary transfer) must be investigated.
