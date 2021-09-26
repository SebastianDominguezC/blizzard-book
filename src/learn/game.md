# Game

The `Game` is in charge of setting up the `World`'s initial state, defining how to update the `World`, define global resources / state, setting up the input, and defining when the game should end.

To create your own `Game`, it must be imported from the engine:

```
use blizzard_engine::game::Game;
```

## Creating a Game

`Game` is a trait. You can define your own game struct and implement this trait and also Clone:

```
#[derive(Clone)]
struct MyGame {
    world: MyWorld,
    counter: i32,
}

impl Game<SharedState, Input> for MyGame {
    fn world_config(&mut self) {

    }

    fn update(&mut self, input: Input, shared_state: Arc<Mutex<SharedState>>) {

    }

    fn reset_input(&mut self, input: Arc<Mutex<Input>>) {
    }

    fn render(&mut self) {}

    fn end_game(&self) -> bool {
        false
    }
}
```

The struct `MyGame` has the world. We also defined another property called counter, just to show that you can create and customize your game however you wish.

The `Game` trait has two generic types, the first is `K`. `K` is used for sharing information with the client, it is a way to define which data is sent off to the client. For now, we defined the generic `K` as a struct called `SharedState`. We will talk about the shared state later. For now, let's define it as an emtpy struct:

```
#[derive(Debug, Serialize, Deserialize, Clone)]
pub struct SharedState {
}

impl SharedState {
    pub fn new() -> Self {
        Self {
        }
    }
}
```

The shared state must implement these traits, hence we need to add `Serde` to our proyect in order to serialize/deserialize the data sent. At the top of the file add the following:

```
#[macro_use]
extern crate serde_derive;
extern crate serde;
extern crate serde_json;
```

The second generic type is `I` which was used before, we defined it as the `Input` tuple struct.

If the function signatures are confusing, here is the generic definition:

```
pub trait Game<K, I> {
    fn world_config(&mut self);
    fn update(&mut self, input: I, shared_state: Arc<Mutex<K>>);
    fn reset_input(&mut self, input: Arc<Mutex<I>>);
    fn render(&mut self);
    fn end_game(&self) -> bool;
}
```

Let's talk about each function and implement it in our game.

## World Configuration

The world configuration function is the first function to be called when starting the game. It is useful to define initial states, entities and components. This function is only called once, at the beginning of the `Game` lifecycle.

Let's define a basic initial entity with a counter component:

```
fn world_config(&mut self) {
    // Create counter entity
    let entities = self.world.entity_manager.create_n_entities(1);

    // Add components to many entities
    self.world.counters.add_many(&entities, 0);
}
```

## Updating the world

After the world configuration is run, the game loop is started. The update function is the first function that is called in the game loop. It takes an argument of the input, however we are not using it now.

```
fn update(&mut self, input: Input, shared_state: Arc<Mutex<SharedState>>) {
    // Update components
    self.world.run_systems(input);
    self.counter += 1;
}
```

This function should be used to tell the `World` to run it's systems.

## Reseting user input

After the update function is called, the reset_input function is called. For now we can leave it empty, but it is just a way to define how the input for the next loop should look like if no client input is detected.

```
fn reset_input(&mut self, input: Arc<Mutex<Input>>) {
}
```

## Render

This function is called after updating and reseting the user input. It is meant to be called for rendering the game, however the engine does not have a Renderer API yet, so it is pretty much useless:

```
fn render(&mut self) {}
```

## Ending the game

This function determines wheter the game should end or not. Here logic can be inserted to end the game. However, we will create an endless game:

```
fn end_game(&self) -> bool {
    false
}
```

## Game creator helper function

It is recommended to make a game creator helper function:

```
fn new_game(world: MyWorld) -> MyGame {
    MyGame {
        counter: 0,
        world: world,
    }
}
```

Perfect, we are almost ready to deploy our server, let's learn how to add start a server with this game!
