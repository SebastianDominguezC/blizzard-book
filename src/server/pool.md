# Pool

The `Game Pool` creates as many games as specified in separate threads. It actually creates `Connectors`. Each `Connector` is like a wrapper around the `Controller`. The `Game Pool` stores each `Connector`, so it can talk to the `Server` about empty games, available ports, and so on.
