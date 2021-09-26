# Learn

This is a tutorial to build the Example project in the GitHub repository. This will teach you the fundamentals to create your own TCP multiplayer data games using Blizzard's Game Engine and Server.

## Configuration

Open a new terminal and create a new library crate:

```
cargo new my_game --lib
cd my_game
```

Inside the `src` folder, create a `bin` directory and add two files:

`server.rs`
`client.rs`

Make sure that each file has a `main()` function!

This will be all there is to the folder and file structure.

Inside the root `cargo.toml` file, add the following dependencies:

```
[dependencies]
serde = "1.0.13"
serde_json = "1.0"
serde_derive = "1.0"

blizzard-server = "0.1"
blizzard-engine = "0.1"
blizzard-engine_derive = "0.1"
```

Then install the dependencies / build the proyect in your terminal

```
cargo build
```

Your file structure should look like the following:

```
my_game/
    src/
        bin/
            client.rs
            server.rs
        lib.rs
    cargo.toml
    cargo.lock
    .gitignore
    target/
```

Now let's make a simple multiplayer game where you can move a player's position!
