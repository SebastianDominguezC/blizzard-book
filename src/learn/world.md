# World

The Blizzard Game Engine works with the Entity Component System architecture. The server runs a `Game`, said game contains a `World`, which contains all the `Entities` and `Components`. The `World` also runs the `Systems`, which process all of the `Components` to run logic and update the game. You can read more about the ECS architecture online.

## Import World

Inside the `server.rs` file we will write all the game and server logic. At the top of the file add the following:

```
extern crate blizzard_engine;
use blizzard_engine::ecs::{World};
```

The first two line is for adding the blizzard engine to the binary. The second line is to use the `World` trait definied for the ECS.

## Create your world

Create your own world by creating any struct that implements Debug and Clone. Then simply apply the `World` trait to your World:

```
#[derive(Debug, Clone)]
struct MyWorld {
}

impl World<Input> for MyWorld {
    fn new() -> Self {
        Self {

        }
    }
    fn run_systems(&mut self, input: Input) {
        // runs future systems
    }
}
```

This won't compile yet! The `World` trait takes a generic type `I`, used for sending client (player) data to the world! Here we called it `Input`. You can define input any way you like, it is meant to be flexible to however you like! For now, let's define `Input` as the following tuple struct:

```
#[derive(Debug, Clone, Copy)]
struct Input(usize);
```

Next step: Creating `Entities` and `Components`!
