# Development Process

As we were developing Alien Slayer, I thought about how to turn the game into a multiplayer event (it isn't part of the development plan, I was just curious). After some researching, I decided to make my own TCP multiplayer server using Rust. I called this project Blizzard. It consists of two parts: Blizzard Server and Blizzard Game Engine.

I first started to make the server. It handles client connections, data sending and player input, as well as running the games. After some successful connections and data sending, I realized the server needed some sort of game engine to run the game in a separate thread. I decided to make my own game engine, also with Rust. Hence Blizzard Game Engine came to be, made with ECS architecture. It is only a "data-only" engine, but in the future I hope to give it features like windowing, multi-platform, and rendering.

After I was able to make a basic game, I decided to abstract the entire server and engine for others to use it! I ended up with this amazing library (it actually consists of 4 different crates).

Here are some things I had to learn and implement:

- Threads
- Networking (TCP, UDP, Serialization)
- ECS architecture
- Abstraction: Generics, traits, trait objects, macros
- Safe data: Atomic Reference Counters, Mutual Exclusion, Thread Messages
