# Server Architecture

The architecture decided for any game using this,is for the server to be an authorative. This means that the server receives user data, and decides how to update the game according to the inputs. After the update, it dispatches the updated game data to all players.

<div align="center">
  <img src="./assets/client-server.svg" alt="Client-Server Architecture"/>
</div>
<p align="center" style="text-align: center">
    Client-Server Diagram
</p>
<br/>

The server is missing many features like cheating prevention, player prediction, a better game loop, and metrics.

The server is made up of many parts:

- Connection listener (handles initial connections)
- Pool (Reference to connectors)
- Connector (Bridge between controller and pool)
- Controller (Handles application, player connections, message passing)
- Application (Runs the game, listens for controller messages)

<div align="center">
    <img src="./assets/server.svg" alt="Server Architecture"/>
</div>
<p align="center" style="text-align: center">
    Server Architecture Diagram
</p>
