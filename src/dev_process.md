# Development Process

## Overview

When starting this project, I really had no idea of what I was doing. I did a lot of researching, reading, book finding, and video watching. I needed to learn the following:

- Networking
  - TCP & UDP networking
  - Data serialization
- Rust
  - Threads
  - Safe types (Arc, Mutex)
  - Generics
  - Traits
  - Type traits
  - Macros (Procedural and Derivative)
  - Closures
  - Iterators
  - Packages and crates
  - Pretty much everything from `The-Book` except lifetimes
- Game engine architectures
- Game server architectures

## Recap

I first developed the TCP server. After good data passing between threads and handling connections, I realized I needed a sort of game engine running with my server, so it could be authorative. I ended up also making a small game engine.

## Resources

Here are some resources I strongly recommend (they aren't even half of what I read and watched):

- Rust
  - ["The Book"](https://doc.rust-lang.org/stable/book/title-page.html)
- Networking & Serialization
  - Book: "Network Programming with Rust" - Abhishek Chanda
- ECS
  - Paper: "ECS Game Engine Design" - Daniel Hall
- Game Engine YouTube series
  - [Hazel](https://www.youtube.com/watch?v=JxIZbV_XjAs&list=PLlrATfBNZ98dC-V-N3m0Go4deliWHPFwT)
  - [Kohi](https://www.youtube.com/watch?v=dHPuU-DJoBM&list=PLv8Ddw9K0JPg1BEO-RS-0MYs423cvLVtj)
