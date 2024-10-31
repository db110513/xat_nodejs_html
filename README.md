# xat

index.html
```
<!DOCTYPE html>
<html lang="ca">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Xat</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 0;
      display: flex;
      flex-direction: column;
      height: 100vh;
    }
    #messages {
      list-style-type: none;
      margin: 20px 0;
      padding: 0;
      flex: 1;
      overflow-y: auto;
    }
    #messages li {
      padding: 8px;
      margin-bottom: 5px;
      border-radius: 4px;
      color: white;
      max-width: 70%;
      clear: both;
    }
    .my-message {
      float: right;
      text-align: right;
    }
    .other-message {
      float: left;
      text-align: left;
    }
    #form {
      display: flex;
      padding: 10px;
      background-color: #f1f1f1;
    }
    #input {
      flex: 1;
      padding: 10px;
      border: 2px solid #007bff;
      border-radius: 4px;
      margin-right: 10px;
      outline: none;
      transition: border-color 0.3s;
    }
    #input:focus {
      border-color: #0056b3;
    }
    #send {
      padding: 10px;
      border: none;
      background: #28a745;
      color: white;
      border-radius: 4px;
      cursor: pointer;
    }
    #username {
      margin-top: 20px; /* Ajusta aquest valor segons el marge desitjat */
    }
  </style>
</head>
<body>
  <input id="username" placeholder="Escriu el teu nom" /><br>
  <button id="join">Entra al Xat</button>
  <ul id="messages"></ul>
  <form id="form" action="" style="display: none;">
    <input id="input" autocomplete="off" placeholder="Escriu el teu missatge aquí..." />
    <button>Envia</button>
  </form>

  <script src="/socket.io/socket.io.js"></script>
  <script>
    const socket = io();

    const joinButton = document.getElementById('join');
    const usernameInput = document.getElementById('username');
    const messages = document.getElementById('messages');
    const form = document.getElementById('form');
    const input = document.getElementById('input');

    let username = '';
    let userColor = '';
    const colors = ['#FF5733', '#33FF57', '#3357FF', '#F333FF', '#FF33A1', '#FFD333', '#33FFF8'];

    joinButton.addEventListener('click', () => {
      username = usernameInput.value.trim();
      if (username) {
        userColor = colors[Math.floor(Math.random() * colors.length)];
        socket.emit('join', { username, color: userColor });
        usernameInput.style.display = 'none';
        joinButton.style.display = 'none';
        form.style.display = 'flex';
      }
    });

    socket.on('message', (msg) => {
      const item = document.createElement('li');
      item.textContent = msg;
      messages.appendChild(item);
      window.scrollTo(0, document.body.scrollHeight);
    });

    socket.on('chat message', ({ msg, color, sender }) => {
      const item = document.createElement('li');
      item.textContent = msg;
      item.style.backgroundColor = color; 
      
      if (sender === username) {
        item.classList.add('my-message');
      } else {
        item.classList.add('other-message');
      }

      messages.appendChild(item);
      window.scrollTo(0, document.body.scrollHeight);
    });

    form.addEventListener('submit', (e) => {
      e.preventDefault();
      if (input.value) {
        const message = `${username}: ${input.value}`;
        socket.emit('chat message', { msg: message, color: userColor, sender: username });
        input.value = '';
      }
    });
  </script>
</body>
</html>
```

server.js
```
const express = require('express');
const http = require('http');
const { Server } = require('socket.io');

const app = express();
const server = http.createServer(app);
const io = new Server(server);
const PORT = 3000;

app.get('/', (req, res) => {
  res.sendFile(__dirname + '/index.html');
});

io.on('connection', (socket) => {
  console.log('Un usuari s\'ha connectat:', socket.id);

  socket.on('join', ({ username, color }) => {
    socket.username = username;
    socket.userColor = color;
    socket.broadcast.emit('message', `${username} s'ha unit al xat`);
  });

  socket.on('chat message', ({ msg, color, sender }) => {
    io.emit('chat message', { msg, color, sender });
  });

  socket.on('disconnect', () => {
    if (socket.username) {
      console.log(`${socket.username} ha deixat el xat`);
      io.emit('message', `${socket.username} ha deixat el xat`);
    }
  });
});

server.listen(PORT, () => {
  console.log(`\nServidor en funcionament a http://localhost:${PORT}`);
});
```

node server.js
