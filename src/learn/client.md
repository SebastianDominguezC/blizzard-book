# Client

The client will be a terminal based client.

We will move the following on to our `my_game` library:

- `Message`
- `SharedState`
- `Position`

## MyGame library

Inside the lib.rs file:

```
#[macro_use]
extern crate serde_derive;
extern crate serde;
extern crate serde_json;

use std::ops::AddAssign;

// Message definition
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

// Shared state definition
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

// Position component
#[derive(Serialize, Deserialize, Debug, Clone, Copy)]
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

Let's update our `server.rs` by importing these definitions and removing the old ones that were in the file. Inside: `server.rs`:

```
extern crate my_game;
use example::{Message, Position, SharedState};
```

The beggining of your `server.rs` file should now look like the following:

```
extern crate blizzard_engine;
extern crate blizzard_engine_derive;
extern crate my_game;

use blizzard_engine::ecs::{ComponentRegistry, EntityManager, World};
use blizzard_engine::game::Game;
use blizzard_engine_derive::ComponentRegistry;
use blizzard_server::server::Server;

use std::collections::HashMap;
use std::sync::mpsc::Receiver;
use std::sync::{Arc, Mutex};

use example::{Message, Position, SharedState};
```

## Creating our client

Now we are ready to make our client!

In the beggining of the file write the following:

```
extern crate example;

use example::Message;
use example::SharedState;

use std::io::{self, BufRead, BufReader, Write};
use std::net::{Shutdown, TcpStream};
use std::str;
use std::sync::{Arc, Mutex};
use std::thread;

struct Client {}
```

Our `Client` struct will be used to define related functionality.

The way the server works is the following:

- Server is started listening to a TCP port
- Server creates threads based on how many games were specified
- Server opens different TCP listeners (on different ports) per game

When a client connects to the server, the following happens:

1. Server looks for an available game (based of player capacity)
2. If it finds a game, it returns the TCP port of said game
3. In none are found, the port returned is 0

Based on this we can create a basic client that tries to connect to the server and receive a port number.

```
impl Client {
    fn start() {
        let mut stream = TcpStream::connect("0.0.0.0:8888").expect("Could not connect to server");
        let port: i32;

        let mut input = String::new();
        let mut buffer: Vec<u8> = Vec::new();

        println!("Enter your username: ");

        io::stdin()
            .read_line(&mut input)
            .expect("Failed to read from stdin");

        println!("Finding an available lobby...");

        stream
            .write(input.as_bytes())
            .expect("Failed to write to server");

        let mut reader = BufReader::new(&stream);

        reader
            .read_until(b'\n', &mut buffer)
            .expect("Could not read into buffer");

        port = str::from_utf8(&buffer)
            .expect("Could not write buffer as string")
            .replace("\n", "")
            .parse()
            .expect("Could not parse port");

        if port == 0 {
            println!("No game available, please try again later");
            return;
        }

        stream
            .shutdown(Shutdown::Both)
            .expect("Could not disconnect from original server");

        let tcp = format!("0.0.0.0:{}", port);

        Client::run_game(tcp);
    }
}
```

The server needs to receive an input in order to acknowledge that a player wants to connect, hence we are defining a `username`, but it will actually never be read. If no game is found, a port of 0 will be returned. We can check this to see if the client should continue to try and connect. At the end, `Client::run_game(tcp)` starts the game connection. Let's see how we can define such game.

## Implementing player control

```
fn run_game(tcp: String) {
    // Try to connect to game
    let mut stream = TcpStream::connect(tcp).expect("Could not connect to server");
    let stream_clone = stream.try_clone().unwrap();

    // Tell the server to add a player
    let data = Message::AddPlayer;
    let json = serde_json::to_string(&data).unwrap() + "\n";

    let should_close = Arc::new(Mutex::new(false));
    let should_close_copy = Arc::clone(&should_close);

    // Write add player message
    stream
        .write(json.as_bytes())
        .expect("Failed to write to server");
    println!("data written");

    // User Input thread
    thread::spawn(move || {
        let shoud_close = should_close_copy;

        loop {
            let mut input = String::new();

            io::stdin()
                .read_line(&mut input)
                .expect("Failed to read from stdin");

            let input = input.trim();

            let mut data = Message::None;

            // Player controls
            if input == "w" {
                data = Message::W;
            } else if input == "a" {
                data = Message::A;
            } else if input == "s" {
                data = Message::S;
            } else if input == "d" {
                data = Message::D;
            } else if input == "close" {
                data = Message::RemovePlayer;
                *shoud_close.lock().unwrap() = true;
            }

            let json = serde_json::to_string(&data).unwrap() + "\n";

            stream
                .write(json.as_bytes())
                .expect("Failed to write to server");
        }
    });

    // Stream Reader thread
    thread::spawn(move || {
        let stream = stream_clone;
        loop {
            let mut buffer: Vec<u8> = Vec::new();
            let mut reader = BufReader::new(&stream);
            reader
                .read_until(b'\n', &mut buffer)
                .expect("Could not read into buffer");
            let json = str::from_utf8(&buffer).unwrap();
            let state: SharedState = serde_json::from_str(&json).unwrap();

            // Print shared state!
            println!("{:?}", state);
        }
    });

    // Keep thread alive, so TCP connection on other threads doesn't reset
    loop {
        if *should_close.lock().unwrap() {
            return;
        }
    }
}
```

## Running everything togehter

In a terminal, start the server:

```
cargo run --bin server
```

In a new terminal, start a client:

```
cargo run --bin client
```

You should input your username, and the game will connect. You should then see positions logged to the console. Try writing to the server "w", "a", "s", "d", or "close". What happens?

Now without closing the current client, open y a 3rd terminal and connect another player!

What happens if you connect a 4th player? What happens if you keep going until all games are full? Find out!

## GitHub repo

All of this code is the same code from the Example library in the official GitHub repository. You can check it out if anything is not working properly.

## Congratulations!

You just made your own very basic multiplayer game! Many other languages and frameworks have TCP connections. For example `C#` has a TCP class. You could connect your server to a `Unity` game! The possibilities are endless.
