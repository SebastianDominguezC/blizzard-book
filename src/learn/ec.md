# Entities and Components

Hopefully by now you have read a little about `Entities` and `Components` from the ECS architecture.

## Necessary imports

The Blizzard Engine comes with a default way of creating `Entities` and `Components`. Add the following to the top of your `server.rs` file:

```
extern crate blizzard_engine_derive;

use blizzard_engine::ecs::{ComponentRegistry, EntityManager, World};
use blizzard_engine_derive::ComponentRegistry;

use std::collections::HashMap;
```

The top of your file should now look something like the following:

```
extern crate blizzard_engine;
extern crate blizzard_engine_derive;

use blizzard_engine::ecs::{ComponentRegistry, EntityManager, World};
use blizzard_engine_derive::ComponentRegistry;

use std::collections::HashMap;
```

## Add Entities to your world

Adding entities to the world is very simple:

```
#[derive(Debug, Clone)]
struct MyWorld {
    entity_manager: EntityManager,
}

impl World<Input> for MyWorld {
    fn new() -> Self {
        Self {
            entity_manager: EntityManager::new(),
        }
    }
    fn run_systems(&mut self, input: Input) {
        // runs future systems
    }
}
```

See the `ecs` part of the Blizzard Game Engine docs to see all the methods available for adding and managing entities.

Let's add components to our `World`.

## Adding Components

Adding components is very easy, that's what the `engine_derive` library is for, it provides a macro to generate components easily! Let's create two types of components, a `Player` and a `Position`:

```
#[derive(ComponentRegistry, Debug, Clone)]
struct PositionRegistry {
    components: HashMap<u32, Position>,
}

#[derive(ComponentRegistry, Debug, Clone)]
struct PlayerRegistry {
    components: HashMap<u32, usize>,
}
```

Our components are stored in a registry, hence the struct names end with "Registry". Components are stored inside a HashMap, the first key MUST be of type u32, since that is the UID that is associated to an `Entity` when a `Component` is created. The second key of the HashMap is of any type that is needed. This won't compile yet because `Position` is not defined. Let's define it:

```
use std::ops::AddAssign;

#[derive(Debug, Clone, Copy)]
pub struct Position {
    x: i32,
    y: i32,
}

impl Position {
    pub fn new() -> Self {
        Self { x: 0, y: 0 }
    }
    pub fn displacement(x: i32, y: i32) -> Self {
        Self { x, y }
    }
}

impl AddAssign for Position {
    fn add_assign(&mut self, other: Self) {
        *self = Self {
            x: self.x + other.x,
            y: self.y + other.y,
        };
    }
}
```

Now the only thing left to do is add the component registries to the `World`:

```
#[derive(Debug, Clone)]
struct MyWorld {
    entity_manager: EntityManager,
    positions: PositionRegistry,
    players: PlayerRegistry,
}
```

Great! Let's now learn how to add `Systems` in order to manipulate components.
