# Installation of Playroom Kit in a JavaScript project

1. Create a single `index.html` file and add the following boilerplate code to it.
   
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    
</body>
</html>
```

2. Inside the `<body></body>` tag of `index.html` file, add two `<scipt></script>` tags:
    - In the first `<scipt></script>` tag, the logic and Playroom code will be written
    - In the second `<scipt></script>` tag, set the `src` attribute to `"https://unpkg.com/playroomkit/multiplayer.full.umd.js"` and also add the `crossorigin` attribute to `"anonymous"`.

# Usage of Playroom functions in a JavaScript project

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Playroom JavaScript Script</title>
</head>
<body>
    <button id="join-room">Join Room</button>
    <button id="leave-room">Leave Room</button>
    <button id="get-room-state">Get Room State</button>
    <button id="set-room-state">Set Room State</button>
    <button id="get-room-code">Get Room Code</button>
    <button id="get-player-info">Get Player Info</button>
    <button id="set-player-state">Set Player State</button>
    <button id="get-player-state">Get Player State</button>
    <button id="kick-player">Kick Player</button>
    <button id="rpc-call">RPC Call</button>
    <script>
        const joinRoom = document.getElementById("join-room");
        const leaveRoom = document.getElementById("leave-room");
        const setRoomState = document.getElementById("set-room-state");
        const getRoomState = document.getElementById("get-room-state");
        const getRoomCode = document.getElementById("get-room-code");
        const getPlayerInfo = document.getElementById("get-player-info");
        const setPlayerState = document.getElementById("set-player-state");
        const getPlayerState = document.getElementById("get-player-state");
        const kickPlayer = document.getElementById("kick-player");
        const rpcCall = document.getElementById("rpc-call");

        let cleanUp = null;
        let players = []

        joinRoom.addEventListener("click", async function() {

            if (cleanUp) {
                cleanUp();
                cleanUp = null;
                players = []
            }

            // https://docs.joinplayroom.com/apidocs#insertcoin-initoptions--onlaunchcallback-ondisconnectcallback
            await Playroom.insertCoin({
                skipLobby: true,
                roomCode: "ABCD"
            });

            // https://docs.joinplayroom.com/apidocs#onplayerjoincallback
            cleanUp = Playroom.onPlayerJoin((player) => {
                console.log(`${player.id} joined!`);
                players.push(player);
            });

            // https://docs.joinplayroom.com/apidocs#ishost
            const host = Playroom.isHost();
            if (host) {
                console.log("Yes, I am the host!");
                // https://docs.joinplayroom.com/apidocs#setstatekey-string-value-any-reliable-boolean--false
                Playroom.myPlayer().setState("host", true, true)
            } else {
                console.log("No, I am not the host!");
                // https://docs.joinplayroom.com/apidocs#getstatekey-string-any-1
                Playroom.myPlayer().setState("host", false, true)
            }

            // https://docs.joinplayroom.com/apidocs#ondisconnectcallback
            Playroom.onDisconnect((e) => {
                console.log("Disconnected", e.code, e.reason);
            })

            // https://docs.joinplayroom.com/apidocs#rpcregistername-string-callback-data-any-sender-playerstate--promiseany
            Playroom.RPC.register('playTurn', (data, sender) => {
                console.log(`${sender.id} played their turn!`);
            });

        });

        leaveRoom.addEventListener("click", async function() {
            // https://docs.joinplayroom.com/apidocs#leaveroom-promisevoid
            await Playroom.myPlayer().leaveRoom();
        });

        setRoomState.addEventListener("click", function() {
            // https://docs.joinplayroom.com/apidocs#setstatekey-string-value-any-reliable-boolean--true-void
            Playroom.setState("healthOfAllPlayers", 100);
        });

        getRoomState.addEventListener("click", function() {
            // https://docs.joinplayroom.com/apidocs#getstatekey-string-any
            console.log(Playroom.getState("healthOfAllPlayers"));
        });

        getRoomCode.addEventListener("click", function() {
            // https://docs.joinplayroom.com/apidocs#getroomcode
            console.log(Playroom.getRoomCode());
        })

        getPlayerInfo.addEventListener("click", function() {
            // https://docs.joinplayroom.com/apidocs#getprofile-playerprofile
            const playerProfile = Playroom.myPlayer().getProfile();
            console.log(playerProfile);
        });

        setPlayerState.addEventListener("click", function() {
            // https://docs.joinplayroom.com/apidocs#setstatekey-string-value-any-reliable-boolean--false
            Playroom.myPlayer().setState("health", 100, true);
            Playroom.myPlayer().setState("score", 0, true);
        });

        getPlayerState.addEventListener("click", function() {
            // https://docs.joinplayroom.com/apidocs#getstatekey-string-any-1
            console.log(Playroom.myPlayer().getState("health"));
            console.log(Playroom.myPlayer().getState("score"));
        });

        kickPlayer.addEventListener("click", function() {
            // https://docs.joinplayroom.com/apidocs#ishost
            const isHost = Playroom.isHost();
            if (isHost) {
                players.forEach((player) => {
                    if (!player.getState("host")) {
                        // https://docs.joinplayroom.com/apidocs#kick-promisevoid
                        player.kick();
                    }
                })
            }
        });

        rpcCall.addEventListener("click", function() {
            // https://docs.joinplayroom.com/apidocs#rpccallname-string-data-any-mode-rpcmode-callbackonresponse-data-any--void-promiseany
            Playroom.RPC.call('playTurn', { data: "Hello World" }, Playroom.RPC.Mode.OTHERS);
        });

    </script>
    <script src="https://unpkg.com/playroomkit/multiplayer.full.umd.js" crossorigin="anonymous"></script>

</body>
</html>
```