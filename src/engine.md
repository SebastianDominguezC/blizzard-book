# Engine Architecture

The engine is meant to be used with ECS architecture (Entity Component System). A game engine consists of many parts, which are not yet implemented. As of now, there is only:

- Core
  - Logger
  - Parsing
- Application Layer
  - Event loop
  - Network Input
  - ECS

This is because the MVP was meant for integration with the server. This is the minimum needed for integration.

The future architecture will consist of more parts:

- Platform layer
  - Multi-platform support
  - File I/O
- Core
  - Engine configuration
  - Tests
  - Math library
  - System time
- Renderer
  - OpenGL abstraction
  - GUI
  - Camera
  - Sceneing
  - Many other rendering goodies
- Application Layer
  - Physics
  - State machine
  - Time
  - Lifecycles
  - Shutdown
- AI layer
- Tools
  - Debugging
  - Build system
