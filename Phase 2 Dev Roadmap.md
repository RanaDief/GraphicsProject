
This roadmap defines a **milestone-based build order** for the engine.
Each step depends **only** on previously implemented items.
Files are implemented in the order listed - header first, then `.cpp`, except for single-header utility files.

---

## Milestone 0 - Repo Plumbing (Must-Have)

### `src/include/engine` (Public Header)
**Purpose:**  
Expose the single include entry point.

**Content:**  
- Minimal forward includes for:
  - `Engine::Init()`
  - `Engine::Run()`
  - `Engine::Shutdown()`
- Forward declaration of `MonoBehaviour`.

**Why:**  
Allows `main.cpp` to compile against a stable API while internals evolve.

**Tasks:**
- [ ] Create `src/include/engine`
- [ ] Define minimal API (`Init`, `Run`, `Shutdown`)
- [ ] Add `MonoBehaviour` forward declaration

---

### `src/app/main.cpp`
**Purpose:**  
Minimal app that includes the engine header and runs the basic loop.

**Why:**  
Early smoke test of the build system.

**Tasks:**
- [ ] `#include <engine>`
- [ ] Call `Engine::Init()` and `Engine::Run()`

---

### `src/utils/Logger.hpp` + `Logger.cpp`
**Purpose:**  
Tiny logging utility used by all subsystems (console output + levels).

**Why:**  
Essential for debugging subsequent systems.

**Tasks:**
- [ ] Define log levels (Info, Warn, Error)
- [ ] Add simple console output
- [ ] Make global logger accessible across engine

---

## Milestone 1 - ECS Core (High Priority)

Implement a **minimal but correct ECS**.
Tests in `tests/` will verify core behavior.

### Files & Responsibilities

- [ ] `src/engine/ecs/Entity.hpp` - Entity type (index + generation) + handle
- [ ] `src/engine/ecs/EntityManager.hpp` + `.cpp` - Create/destroy entities, generation reuse, free list
- [ ] `src/engine/ecs/Component.hpp` - Component base + ID generation utility (templated)
- [ ] `src/engine/ecs/ComponentManager.hpp` + `.cpp` - Manage components (add/remove, contiguous/sparse storage)
- [ ] `src/engine/ecs/System.hpp` - Abstract system interface (`Update`) + registration API
- [ ] `src/engine/ecs/SystemManager.hpp` + `.cpp` - Register systems, tick in order
- [ ] `src/engine/ecs/World.hpp` - High-level façade for entities, components, queries
- [ ] `tests/ecs_tests.cpp` - Unit tests: entity creation, transform components, queries

**Why:**  
The ECS is the backbone - everything else depends on it.

---

## Milestone 2 - Transform Component & System (Must-Have)

### `src/engine/components/Transform.hpp` + `.cpp`
**Fields:**
- Local position, rotation (quat), scale
- Parent entity handle
- Local/world matrix cache
- Dirty flags

### `src/engine/systems/TransformSystem.hpp` + `.cpp`
**Purpose:**  
Propagate local → world transforms, update dirty subtrees, compute AABB if needed.

**Tasks:**
- [ ] Compute hierarchical transform updates
- [ ] Add tests / demo for parent-child transforms

**Why:**  
Renderer, physics, and scripts depend on transforms.

---

## Milestone 3 - Minimal GL Draw Path (Must-Have for Visuals)

### `src/engine/gl/GLContext.hpp` + `.cpp`
- [ ] Initialize OpenGL via GLFW (in `vendor/`)
- [ ] Create context + debug callbacks

### `src/engine/gl/Shader.hpp` + `.cpp`
- [ ] Load, compile, link GLSL
- [ ] Add uniform setters
- [ ] Stub for hot-reload

### `src/engine/gl/Mesh.hpp` + `.cpp`
- [ ] VAO/VBO wrapper
- [ ] Index buffer + draw calls

### `src/engine/assets/MeshLoader.hpp` + `.cpp`
- [ ] Tiny OBJ loader (using `tinyobj` from vendor or hard-coded cube)

### `src/engine/systems/RenderSystem.hpp` + `.cpp`
- [ ] Gather entities with `Transform` + `MeshRenderer`
- [ ] Stub frustum culling
- [ ] Submit draw calls

### Demo Update (`src/app/main.cpp`)
- [ ] Create a simple scene (one mesh + camera)
- [ ] Run engine and confirm triangle/cube render

---

## Milestone 4 - Public Scripting API (User Ergonomics)

### `src/engine/scripting/MonoBehaviour.hpp`
- [ ] Public base class (header-only)
- [ ] Virtual hooks: `Awake()`, `Start()`, `Update(float)`, `OnEnable()`, `OnDisable()`, `OnDestroy()`

### `src/engine/scripting/ScriptComponent.hpp`
- [ ] Component storing pointer/handle to script instance or factory

### `src/engine/scripting/ScriptSystem.hpp` + `.cpp`
- [ ] Instantiate scripts, manage lifecycle
- [ ] Manage `Start()` and `Update()` hooks
- [ ] Dense array of active scripts for fast iteration

**Why:**  
Allows user scripts like:
```cpp
class Player : public MonoBehaviour {
    void Update(float dt) override { /* ... */ }
};
```

---

## Milestone 5 - AssetManager, Materials, Textures

### `src/engine/assets/AssetManager.hpp` + `.cpp`
- [ ] Centralized caching and reference counting for shaders, meshes, and textures.

### `src/engine/assets/TextureLoader.hpp` + `.cpp`
- [ ] Wrap `stb_image` from `vendor/`

### `src/engine/components/MeshRenderer.hpp`
- [ ] Add material definition and properties

**Improvement:**  
Enhance `RenderSystem` to batch by material.

**Why:**  
Avoid redundant resource loads and manage lifetime.

---

## Milestone 6 - Input, Camera, and Scene

### `src/engine/platform/Input.hpp`
- [ ] Keyboard/mouse polling and events via GLFW

### `src/engine/components/Camera.hpp` + `.cpp`
- [ ] Define projection and view matrices
- [ ] Optionally create `CameraSystem`

### `src/scenes/scene_loader.cpp`
- [ ] Parse JSON files into entities/components

**Why:**  
Enable interactive demos and scene loading.

---

## Milestone 7 - Physics (Next)

### Components
- [ ] `src/engine/components/Rigidbody.hpp`
- [ ] `src/engine/components/Collider.hpp`

### Systems
- [ ] `src/engine/systems/PhysicsSystem.hpp` + `.cpp`
  - Simple Euler integrator
  - AABB collision detection
  - Collision event queue

**Optional:** integrate Bullet or another physics library from `vendor/`.

**Why:**  
Physics requires transform synchronization and fixed timestep.

---

## Milestone 8 - Serialization, Reflection, and Hot-Reload

### `src/engine/serialization/Serializer.hpp` + `.cpp`
- [ ] Save/load scenes or prefabs (JSON or binary)

### Reflection
- [ ] Minimal metadata system or macros for editor exposure

### Hot-Reload
- [ ] Implement shader hot-reload in `Shader.cpp` + `AssetManager`

**Why:**  
Improves editor experience and iteration speed.

---

## Milestone 9 - Tools, Tests, and Polish

### Tools
- [ ] `src/tools/scene_compiler/` (optional)

### Tests
- [ ] Expand unit tests for assets, rendering, and scripting

### CI / Packaging
- [ ] Add continuous integration build scripts
- [ ] Define release packaging and structure

---

**End Goal:**  
A modular, testable OpenGL engine with ECS, scripting, and rendering ready for editor integration.
