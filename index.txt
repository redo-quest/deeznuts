const app = require("express")();
app.get("/", (res, req) => res.sendFile(__dirname + "/index.html"))
app.listen(9091, console.log("Listening on 9091"));
