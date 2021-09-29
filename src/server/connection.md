# Connection Listener

When creating a server, it opens a port for listening to player connections. The server also creates a `Game Pool`. When a player connects to the main TCP port of the server, the server tries to find an available game inside the `Game Pool`. If it finds an available game, it returns the port number where the game is hosted (another TCP connection). If no game is found, it returns 0.
