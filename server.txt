const http = require("http");
const websocketServer = require("websocket").server;
const httpServer = http.createServer();
httpServer.listen(5000, function () {
  console.log("Listening... on 5000");
});
// hashmap
const clients = [];
const wsServer = new websocketServer({
  httpServer: httpServer,
});
let id = 0;
wsServer.on("request", (request) => {
  //connect
  console.log("client connected");
  const connection = request.accept(null, request.origin);
  connection.on("open", function () {
    console.log("open");
  });
  connection.on("close", function () {
    console.log("closed");
    clients.forEach((player, index) => {
      if (player.connection == connection) {
        clearInterval(clients.gameUpdate);
        clients.splice(index, 1);
      }
    });
  });
  //generate clientID
  const clientId = guid();
  clients.push({
    connection: connection,
    clientId: clientId,
  });
  const payLoad = {
    method: "connect",
    clientId: clientId,
  };
  //send back the client connect
  connection.send(JSON.stringify(payLoad));
  connection.on("message", function (message) {
    const result = JSON.parse(message.utf8Data);
    console.log("Message received! -> ", result);
    if (result.method == "play") {
      let currentPlayer = clients.filter((player) => {
        return player.clientId == result.clientId;
      })[0];
      (currentPlayer.x = Math.floor(Math.random() * 2000)),
        (currentPlayer.y = Math.floor(Math.random() * 2000));
      const payLoad = {
        method: "spawn",
        x: currentPlayer.x,
        y: currentPlayer.y,
      };
      connection.send(JSON.stringify(payLoad));
      const con = currentPlayer.connection;
      let nuts = [];
      //gameUpdate
      let gameUpdate = setInterval(() => {
        //death
        //nutz
        nuts.forEach((nut) => {
          if (Date.now() - nut.time < 1000) {
          const payLoad = {
            method: "createNut",
            x: nut.x,
            y: nut.y,
            id: nut.id,
          };
          con.send(JSON.stringify(payLoad));
          nut.x += Math.cos(nut.dir) * 10;
          nut.y += Math.sin(nut.dir) * 10;
          } else {
            const payLoad = {
              method: "deleteNut",
              id: nut.id,
            }
            con.send(JSON.stringify(payLoad));
            nuts.splice(nut, 1);
          }
        });
        //movement
        currentPlayer = clients.filter((player) => {
          return player.clientId == result.clientId;
        })[0];
        if (!currentPlayer) {
          clearInterval(gameUpdate);
        } else {
          if (currentPlayer.xVel != null && currentPlayer.yVel != null) {
            const payLoad = {
              method: "move",
              x: (currentPlayer.x += currentPlayer.xVel),
              y: (currentPlayer.y += currentPlayer.yVel),
              clientId: currentPlayer.clientId,
              name: currentPlayer.name,
            };
            clients.forEach((player) => {
              player.connection.send(JSON.stringify(payLoad));
            });
          }
        }
      }, 1000 / 60);
      con.on("message", (packet) => {
        const parsed = JSON.parse(packet.utf8Data);

        if (parsed.method == "move") {
          currentPlayer.xVel =
            parsed.dir !== null ? Math.cos(parsed.dir) * 10 : 0;
          currentPlayer.yVel =
            parsed.dir !== null ? Math.sin(parsed.dir) * 10 : 0;
          currentPlayer.name = parsed.name;
        }
        if (parsed.method == "shoot") {
          nuts.push({
            x: Math.cos(parsed.dir) * 30 + 1920 / 2,
            y: Math.sin(parsed.dir) * 30 + 1080 / 2,
            dir: parsed.dir,
            id: guid(),
            time: Date.now(),
          });
        }
      });
    }
  });
});
function S4() {
  return (((1 + Math.random()) * 0x10000) | 0).toString(16).substring(1);
}

// then to call it, plus stitch in '4' in the third group
const guid = () =>
  (
    S4() +
    S4() +
    "-" +
    S4() +
    "-4" +
    S4().substr(0, 3) +
    "-" +
    S4() +
    "-" +
    S4() +
    S4() +
    S4()
  ).toLowerCase();
