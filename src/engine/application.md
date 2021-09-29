# Application

The `Application` which as of now is only a `Network Application` (although it can be used without it's networking capabilities), is where the most logic occurs.

It defines how fast to run the game, the order of function calling of the game, the game loop and how to pass messages and handle client input.

The application takes most importantly a `Game` argument when creating one. The `Game` follows ECS architecture. When creating a `Game`, a `World` is configured. A `World` can have `Entities`, `Components`, and `Systems`. The `World` also accepts inputs from the `Game`, which really means it accepts inputs from the `Application`.

To understand better the ECS architecture implemented, check out the next section!
