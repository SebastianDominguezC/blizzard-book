# Controller

The `Controller` is where the most important things happen. When a `Controller` is created, it opens a new TCP listener to a new port. It then runs the `Application` on a separate thread, with capabilites to passing `Messages` to it. When a player conencts to this new port, the `Controller` will update the `Connector` information as well as the `Application` information. The `Controller` opens two new threads for handling the player's input and also writing data back to the player. The `Controller` serves as the entire communication controller between `Application` and `Client`.
