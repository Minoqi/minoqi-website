---
title: 'Breaking Down Godot 4 Multiplayer Example'
published: 2024-07-15
description: 'A breakdown of Godot 4s networking project template'
image: ''
tags: [Tutorial, Networking, Godot, Game Programming]
category: 'Godot 4 Tutorials'
draft: false 
lang: ''
---

Recently I’ve been hard at work learning how to use Godots multiplayer system. Having little to no networking experience prior, I found a lot of the either too short and quick to understand what’s happening enough to then mess with it on my own, or I was missing information on specific pieces I needed.

When I first tried reading Godots documentation, I got a bit lost. But after watching a pretty comprehensive video by [FinePointCGI](https://www.youtube.com/c/finepointcgi), and doing lots of research, I finally started to wrap my head around the terms enough to take another crack at understanding Godots documentation.

My goal is to make a system that has a waiting lobby list with a waiting room players can chill and communicate in before the host starts the game. I decide to break this down into three chunks of learning, where I’ll focus on one area at a time:

1. **Initial multiplayer Setup**: Learn the basics of how multiplayer works by creating a simple lobby connection system based on Godots documentation

2. **Multiplayer Spawner**: Learn how to use Godots multiplayer spawner

3. **Lobby List**: Learn how to implement a lobby list system so players can see all the different servers available and choose one, including the ability to have private lobbies

I’ve decided to turn what I learn into a blog post for each part to help add to the already limited resources on Godot 4s multiplayer system.

EDIT: I'll probably change this to make one huge post with everything I know about Godots multiplayer

# Initialization
This step requires just 3 scenes.

**Game**: This is the main game scene which will hold all of our scenes (lobby, game loop etc.) The parent will always be the Game scene

**Lobby UI**: This is the scene that contains all the UI for the lobby

**Waiting Room**: This is currently just an empty scene with some text that says "waiting lobby" just to make sure it's working

# Variables
First let’s go over the needed variables:

`PORT, DEFAULT_SERVER_IP`: These are used for the connection, with “127.0.0.1” being a pretty typical “default” address used for local/LAN connection (which this example uses)

`MAX_CONNECTIONS`: This is how many people can connect to a server at once, in my case I want no more then 6 people in a game

`players`: This stores all the details for every player connected to the server

`playerInfo`: This stores the data for the current player, and is what is added to the players variable

`playersLoaded`: Tracks how many players are loaded in

`lobbyFile`: The file for the lobby UI scene

`waitingLobbyFile`: The file for the waiting lobby scene

`activeLobby`: Stores a reference to the lobby UI in the scene

`activeWaitingLobby`: Stores a reference to the waiting lobby in the scene

```gdscript
extends Node

# Variables
const PORT = 7000
const DEFAULT_SERVER_IP = "127.0.0.1"
const MAX_CONNECTIONS = 6

## Contains the player info for all the players with the keys being the unique IDs
var players = {}

## Contains the info for the player, is local only
## Modified before the connection is made
## Is passed to all the other peers to store in `players`
var playerInfo = {
    "NAME": "Player Name"
}

var playersLoaded : int = 0

var lobbyFile : PackedScene = preload("res://scenes/lobby.tscn")
var waitingLobbyFile : String = "res://scenes/waiting_lobby.tscn"

var activeLobby : LobbyUI
var activeWaitingLobby
```
# Signals
Next, let’s go over the signals:

`player_connected`: Goes off when a player connects to a server, sending in the players info for the others to store

`player_disconnected`: Goes off when a player disconnects from a server, sending the players ID so the others can remove that players info from the players variable

`server_disconnected`: Goes off when the server goes out, like possibly when the host crashes or leaves the game

```
# Signals
signal player_connected(_peerID : int, _playerInfo : Dictionary)
signal player_disconnected(_peerID : int)
signal server_disconnected
```

# Ready Function
Next let’s look over the `_ready()` function. As we can see, we're connecting lots of signals and UI. Firstly, we connect all the multiplayer related signals. Then, we connect the lobby information by creating an instance of the lobby UI scene and then connecting the buttons (`$/root/Game` gets the `Game` scene I mentioned earlier that stores all our active scenes, I called it `Game` but you can call it whatever you want).

```gdscript
# Called when the node enters the scene tree for the first time.
func _ready():
    ## Connect multiplayer signals
    multiplayer.peer_connected.connect(_player_connected)
    multiplayer.peer_disconnected.connect(_player_disconnected)
    multiplayer.connected_to_server.connect(_server_connected)
    multiplayer.connection_failed.connect(_server_connection_failed)
    multiplayer.server_disconnected.connect(_server_disconnected)

    ## Connect lobby
    activeLobby = lobbyFile.instantiate()
    $/root/Game.add_child(activeLobby)
    activeLobby.hostButton.pressed.connect(create_game)
    activeLobby.joinButton.pressed.connect(join_game)
    activeLobby.startButton.pressed.connect(start_game)
```

# Multiplayer Signals
Let’s take a look at what all the multiplayer signals are calling.

**_player_connected**: This is called when a player connects. It takes in the players ID and we use this information to send it to the other connected players

*WAIT!* The function shows itself calling another function with `.rpc_id()` added to it, what's that? RPC stands for Remote Procedure Call and is used to send messages to everyone on the server. Here's a table I made explaining it in more detail:

## RPC
| rpc()                               | rpc_id(#)                                    |
| ----------------------------------- | -------------------------------------------- |
| - used to send messages to everyone | - used to send messages to a specific client |

## MODE
| authority                                                                               | any_peer                                   |
| --------------------------------------------------------------------------------------- | ------------------------------------------ |
| - restricted to who can send packets                                                    | - not restricted, anybody can call it      |
| - used when only the person running a game should be able to do a call a certain action | - useful for messaging services like chats |
| - everyone runs the code but only authority can send it                                 |                                            |

## SYNC
| call_remote                                                            | call_local              |
| ---------------------------------------------------------------------- | ----------------------- |
| - will not get called on the local peer                                | - calls it on everybody |
| - used when you want everyone else to call a function besides yourself |                         |

## TRANSFER MODE
how we send the data and what to do if it doesn't successfully arrive
| unreliable                                            | unreliable_order                                                             | reliable                                                                                     |
| ----------------------------------------------------- | ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| - most performant                                     | - like unreliable but slightly less performant                               | - least performant                                                                           |
| - just sends the message, no checking if received     | - makes sure to send the packets in order                                    | - makes sure the packet is received and sent in order                                        |
| - useful for non-important information like audio/vfx | - useful when the order of information is important (like a turn based game) | - useful when order is important and that information gets received (like a turn based game) |


**_player_disconnected**: This is called when a player disconnects. It takes in the payers ID and we use that to alert the others on whose information they should destroy.

**_server_connected**: This goes off when the host connects. It also sets off a player_connected signal, although no one should be around to hear it. In the future, we can use this to setup our lobby list. If you try to have two hosts on the same address, it wills end an error!

**_server_connection_failed**: This goes off when a connection to the server failed, or in our case the host fails to be able to host.

**_server_disconnected**: This goes off when the server disconnects. This can happen if there's a crash on the hosts end or if the host abruptly leaves. It resets all the players, deleting all the previous players data.

```gdscript
## When player connects send all the playerInfo to everyone
func _player_connected(_id : int) -> void:
    _register_player.rpc_id(_id, playerInfo)
    print("PLAYER CONNECTED: ", _id)

## When player disconnects let everyone know and remove them
func _player_disconnected(_id : int) -> void:
    players.erase(_id)
    player_disconnected.emit(_id)

## Host connected
func _server_connected() -> void:
    var peerID : int = multiplayer.get_unique_id()
    players[peerID] = playerInfo
    player_connected.emit(peerID, playerInfo)

## Host connection failed
func _server_connection_failed() -> void:
    multiplayer.multiplayer_peer = null
    printerr("SERVER CONNECTION FAILED")

## Remove all info from server when disconnected
func _server_disconnected() -> void:
    multiplayer.multiplayer_peer = null
    players.clear()
    server_disconnected.emit()
```

# Registering a Player
Let’s see how the player gets registered once they connect. As we can see, to make a function available to be used as an rpc we simply add `@rpc()` on top of it. I set it to `any_peer` so everyone receives it, and `reliable` so it makes sure everyone receives the packet of information in order. It then emits the `player_connected` signal for the others to get the new player data.

```gdscript
## Registers the new players information
@rpc("any_peer", "reliable") ## Everyone calls it & makes sure the packet is received properly
func _register_player(_playerInfo : Dictionary) -> void:
    var newPlayerID : int = multiplayer.get_remote_sender_id()
    players[newPlayerID] = _playerInfo
    player_connected.emit(newPlayerID, _playerInfo)
    print("PLAYER REGISTERED: ", _playerInfo)
```

# Lobby Functionality
Finally, let’s get this all connected and working!

`join_game`: Called when the player joins a server (pressed the join button). It creates an `ENetMultiplayerPeer`, which gives us access to a bunch of multiplayer functions in Godot. We then create a client, storing the output in a variable that we then check to make sure there was no error, otherwise we send a debug message and return. For a future addition, we could make this cause a popup to appear to tell the player that the client creation failed. In my scenario, I added a `LineEdit` node to my scene so the player can input their name which I then store in the `playerInfo` variable. Lastly we start the game.

`create_game`: Called then the host starts a server (pressed the host button). It's similar to the `join_game()` function, but creates a server instead. I've also made it disable turn on the `start button` which was disabled at the start, to avoid trying to start a game when no server is set up. We then add the host info to the `players` variable at ID 1 to mark them as the host. Finally we start the game.

`start_game`: Loads the waiting lobby

`load_game`: Used to create the waiting lobby scene, hiding the lobby UI since we don't need it currently

`player_loaded`: Every peer calls this when they have loaded the game scene

```gdscript
## Player joins a server
func join_game(_address : String = "") -> void:
    if _address.is_empty():
        _address = DEFAULT_SERVER_ID
    
    var peer : ENetMultiplayerPeer = ENetMultiplayerPeer.new()
    var error = peer.create_client(_address, PORT)
    if error:
        printerr("JOIN GAME FAILED: ", error)
        return
    
    playerInfo["NAME"] = activeLobby.nameInput.text
    multiplayer.multiplayer_peer = peer
    start_game()

## Host starts a server
func create_game() -> void:
    var peer : ENetMultiplayerPeer = ENetMultiplayerPeer.new()
    var error = peer.create_server(PORT, MAX_CONNECTIONS)
    if error:
        printerr("CREATE GAME FAILED: ", error)
        return
    activeLobby.startButton.disabled = false
    playerInfo["NAME"] = activeLobby.nameInput.text
    multiplayer.multiplayer_peer = peer
    players[1] = playerInfo
    player_connected.emit(1, playerInfo)
    start_game()

func remove_multiplauyer_peer() -> void:
    multiplayer.multiplayer_peer = null

## Used to start the game from the lobby scene
## Lobby.load_game.rpc()
@rpc("call_local", "reliable")
func load_game() -> void:
    get_tree().change_scene_to_file(waitingLobbyFile)

## Every peer will call this when they have loaded the game scene
@rpc("any_peer", "call_local", "reliable") ## Calls it on everyone BUT on just themselves and not to the whole server
func player_loaded() -> void:
    if multiplayer.is_server():
        playersLoaded += 1
        if playersLoaded == players.size():
            activeLobby.visible = false
            $/root/Game.start_game()
            playersLoaded = 0

func start_game() -> void:
    load_game()
```

# Conclusion
And that’s it! I hope this first part provided a decent overview and maybe helped clear things up. ~~I’ll link part 2 and part 3 here once they’re out.~~ I'll update this post with a link to the major article going over everything in detail when it's ready. Keep in mind I also have limited knowledge in networking, so if you have any tips or something seems wrong please let me know!