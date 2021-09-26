# TCP Server

For now, only TCP servers can be created. To make a server we need to add the following to the top of our `server.rs` file:

```
use blizzard_server::server::Server;
use std::sync::mpsc::Receiver;
use std::sync::{Arc, Mutex};
```

The top should now look like this:

```
extern crate blizzard_engine;
extern crate blizzard_engine_derive;

use blizzard_engine::ecs::{ComponentRegistry, EntityManager, World};
use blizzard_engine::game::Game;
use blizzard_engine_derive::ComponentRegistry;
use blizzard_server::server::Server;

use std::collections::HashMap;
use std::sync::mpsc::Receiver;
use std::sync::{Arc, Mutex};
use std::ops::AddAssign;
```

## Make the server

In the file's `main` function (or create the function), we will configure our server. We need to define:

- Port: a port where our server will listen to incoming connections
- Max games
- Max players per game
- The game
- Shared state (to send to client)
- Client input type
- Rate at which data is sent to client (FPS or HZ)
- Rate at which the game updates (FPS or HZ)
- How to handle client input

The server's `new` function has the following signature:

```
pub fn new<T: Game<K, I>, K, I, M>(
    port: i32,
    max_games: i32,
    max_players: i32,
    game: T,
    shared_state: K,
    input: I,
    handle_input: &'static (dyn Fn(Receiver<(M, usize)>, Arc<Mutex<I>>) -> I + Sync),
    send_data_rate: i32,
    game_update_rate: i32,
)  ...
```

The new type is `M`, which is a type used for the `Message`s sent from client to the game.

Let's create our basic configuration:

```
fn main() {
    let port = 8888;
    let max_games = 4;
    let max_players = 2;
    let world = MyWorld::new();
    let shared_state = SharedState::new();
    let game = new_game(world);

    // The data that a client message will manipulate
    let input_type = Input::default();
    let hanlde_input = &handle_client_message;

    // Engine speeds
    let send_data_from_server_rate = 1;
    let server_game_update_rate = 2; // 2 times per second

    // Start server + games
    Server::new(
        port,
        max_games,
        max_players,
        game,
        shared_state,
        input_type,
        hanlde_input,
        send_data_from_server_rate,
        server_game_update_rate,
    );
}

```

We have a couple of missing definitions, let's create them.

Input:

```
#[derive(Debug, Clone, Copy)]
struct Input(Message, usize);

impl Input {
    fn default() -> Self {
        Self(Message::None, 0)
    }
    fn from(m: Message, id: usize) -> Self {
        Self(m, id)
    }
}
```

We are changing the input to contain a `Message`, and a number of type `usize`. This second one is for identification, it will be passed by the client to identify the player generating the `Input`.

Message:

```
#[derive(Serialize, Deserialize, Clone, Copy, Debug)]
pub enum Message {
    None,
    W,
    A,
    S,
    D,
    AddPlayer,
    RemovePlayer,
}
```

SharedState:

```
#[derive(Debug, Serialize, Deserialize, Clone)]
pub struct SharedState {
    pub counters: Vec<u32>,
    pub registry: Vec<Position>,
}

impl SharedState {
    pub fn new() -> Self {
        Self {
            registry: vec![],
            counters: vec![],
        }
    }
}
```

The shared state will be to show the counters and the positions of players.

Handle client message:

```
fn handle_client_message(receiver: Receiver<(Message, usize)>, input: Arc<Mutex<Input>>) -> Input {
    for (message, id) in receiver {
        println!("Player {} called {:?}", id, message);
        *input.lock().unwrap() = Input::from(message, id);
    }
    Input::default()
}
```

This is an interesting code bit. To handle a client message, when creating the server, an `Application` is created which wraps the `Game`, and is in charge for calling the game loop and also connecting all inputs and communications. Hence this function is to handle inputs. It actually handles messages. A `Message` will be sent from the client. The `Receiver` will receive the `Message` and decide what `Input` to generate based on the message. For now, we are just passing the message on to the `Input` which will be used in the update_systems function inside the `World`. The `Input` is wrapped in a Arc and Mutex.

## Adding game logic

Let's make some systems that will be used.

Adding a player entity, with a player component and position:

```
fn add_player_system(world: &mut MyWorld, player_id: usize) {
    let ent = world.entity_manager.create_entity();
    world.players.add(ent, player_id);
    world.positions.add(ent, Position::new());
    world.player_id_map.players.insert(player_id, ent);
}
```

When a player joins a game, they get a UID defined by the server. However, the `World` has no way to know which entity belongs to a player. Hence we created the `PlayerIdMap`:

```
#[derive(Debug, Clone)]
struct MyWorld {
    entity_manager: EntityManager,
    positions: PositionRegistry,
    counters: CounterRegistry,
    players: PlayerRegistry,
    player_id_map: PlayerIdMap,
}

impl World<Input> for MyWorld {
    fn new() -> Self {
        Self {
            entity_manager: EntityManager::new(),
            positions: PositionRegistry::new(),
            counters: CounterRegistry::new(),
            players: PlayerRegistry::new(),
            player_id_map: PlayerIdMap::new(),
        }
    }
    fn run_systems(&mut self, input: Input) {
        ...
    }
}

#[derive(Debug, Clone)]
struct PlayerIdMap {
    players: HashMap<usize, u32>,
}
impl PlayerIdMap {
    fn new() -> Self {
        Self {
            players: HashMap::new(),
        }
    }
}
```

Updating player position system:

```
fn update_player_pos_system(world: &mut MyWorld, player_id: usize, displacement: Position) {
    if let Some(ent) = world.player_id_map.players.get(&player_id) {
        *world
            .positions
            .components
            .entry(*ent)
            .or_insert(displacement) += displacement;
    }
}
```

Removing a player system:

```
fn remove_player_system(world: &mut MyWorld, player_id: usize) {
    if let Some(ent) = world.player_id_map.players.get(&player_id) {
        world.players.components.remove(ent);
        world.positions.components.remove(ent);
        world.entity_manager.remove_entity(*ent);
        world.player_id_map.players.remove(&player_id);
    }
}
```

## Updating games using run_systems and input

We can now do a lot of basic functionality using this. Let's add our systems to our run_systems:

```
fn run_systems(&mut self, input: Input) {
        // Systems to run conditionally
        match input.0 {
            Message::AddPlayer => add_player_system(self, input.1),
            Message::W => {
                update_player_pos_system(self, input.1, Position::displacement(0, 1));
            }
            Message::A => {
                update_player_pos_system(self, input.1, Position::displacement(-1, 0));
            }
            Message::S => {
                update_player_pos_system(self, input.1, Position::displacement(0, -1));
            }
            Message::D => {
                update_player_pos_system(self, input.1, Position::displacement(1, 0));
            }
            Message::RemovePlayer => {
                remove_player_system(self, input.1);
            }
            _ => {}
        }
        // Systems to always run
        counter_system(&mut self.counters.components);
    }
```

## Defining the data to be sent to the client

Inside the `Game`'s update function, we can now copy the desired data over to the `SharedState`, which is sent off to the client:

```
fn update(&mut self, input: Input, shared_state: Arc<Mutex<SharedState>>) {
        // Update states
        self.world.run_systems(input);
        self.counter += 1;

        // Update shared state: for client reception
        shared_state.lock().unwrap().counters = self
            .world
            .counters
            .components
            .iter()
            .map(|(_, counter)| *counter)
            .collect();

        shared_state.lock().unwrap().registry = self
            .world
            .positions
            .components
            .iter()
            .map(|(_, positions)| *positions)
            .collect();
}
```

Here we are just copying the data from the counter and position components into vectors of information.

## Next steps

This is everything! You have created your own multiplayer game server! You can start the server in your terminal by typing:

```
cargo run --bin server
```

You will see a couple of logs from the server. However nothing exciting really happens. This is because there are no players connecting to any games! Let's now write a client that connects to a game. We will be writing the client inside the `client.rs` file. However, now we will use the library we created, `my_game`. The library is a suggestion, because many of the client's data structs are the same as the game's. For example:

- `Message`
- `SharedState`
- `Position`

Let's write a client that connects to our server.
