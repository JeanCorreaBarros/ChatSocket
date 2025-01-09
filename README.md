# Simple Chat Application

Este proyecto es una aplicación de chat en tiempo real utilizando **Socket.IO**, **Node.js**, **Express**, **MongoDB** y **Tailwind CSS** para el front-end. Además, incluye funcionalidades de autenticación de usuarios para chats personalizados.

## Características

- Comunicación en tiempo real entre múltiples usuarios.
- Diseño responsivo con soporte para esquemas de color claro y oscuro.
- Integración con MongoDB para almacenamiento de datos.
- Autenticación y registro de usuarios para chats personalizados.

## Tecnologías Utilizadas

- **Node.js**: Para el servidor.
- **Express**: Framework para manejar solicitudes HTTP.
- **Socket.IO**: Biblioteca para la comunicación en tiempo real.
- **MongoDB**: Base de datos para almacenar usuarios y mensajes.
- **Mongoose**: ODM para interactuar con MongoDB.
- **Tailwind CSS**: Framework de diseño para el front-end.

## Instalación y Configuración

### Requisitos previos
- Node.js (v14 o superior)
- npm (administrador de paquetes de Node.js)
- Una instancia de MongoDB en ejecución

### Pasos para ejecutar el proyecto

1. Clona este repositorio:
   ```bash
   git clone <URL_DEL_REPOSITORIO>
   cd <NOMBRE_DEL_REPOSITORIO>
   ```

2. Instala las dependencias del servidor:
   ```bash
   npm install
   ```

3. Crea un archivo `.env` en la raíz del proyecto y agrega la URL de conexión de MongoDB:
   ```env
   MONGO_URI=mongodb://localhost:27017/simple-chat
   JWT_SECRET=tu_clave_secreta
   ```

4. Inicia el servidor:
   ```bash
   npm start
   ```

5. Abre el navegador y accede a `http://localhost:3000`.

## Estructura del Proyecto

```
├── client
│   ├── index.html       # Interfaz de usuario
├── server
│   ├── server.js        # Lógica del servidor
│   ├── models           # Modelos de Mongoose
│   │   ├── User.js      # Modelo de usuario
│   │   ├── Message.js   # Modelo de mensaje
├── package.json         # Archivo de configuración del proyecto
```

## Código del Servidor

```javascript
import express from "express";
import logger from "morgan";
import mongoose from "mongoose";
import { Server } from "socket.io";
import { createServer } from "node:http";
import jwt from "jsonwebtoken";
import User from "./models/User.js";
import Message from "./models/Message.js";

const port = process.env.PORT ?? 3000;
const app = express();
const server = createServer(app);
const io = new Server(server);

mongoose.connect(process.env.MONGO_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
}).then(() => console.log("Conectado a MongoDB"));

app.use(logger("dev"));
app.use(express.json());

// Rutas de autenticación
app.post("/register", async (req, res) => {
  const { username, password } = req.body;
  const user = new User({ username, password });
  await user.save();
  res.status(201).send("Usuario registrado");
});

app.post("/login", async (req, res) => {
  const { username, password } = req.body;
  const user = await User.findOne({ username });
  if (user && user.comparePassword(password)) {
    const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET);
    res.json({ token });
  } else {
    res.status(401).send("Credenciales inválidas");
  }
});

io.on("connection", (socket) => {
  console.log("User connected");

  socket.on("disconnect", () => {
    console.log("An User has disconnected");
  });

  socket.on("chat message", async (msg) => {
    const message = new Message({ content: msg });
    await message.save();
    io.emit("chat message", msg);
  });
});

server.listen(port, () => {
  console.log(`Server is running on port ${port}`);
});
```

## Código del Cliente

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simple Chat</title>
    <script src="https://cdn.tailwindcss.com"></script>

    <script type="module">
        import { io } from "https://cdn.socket.io/4.8.1/socket.io.esm.min.js";
        const socket = io();
        const form = document.getElementById('form');
        const input = document.getElementById('menssageInput');
        const chatMessages = document.getElementById('messages');

        socket.on('chat message', (msg) => {
            const message = `<li class="bg-blue-500 text-white p-3 rounded-lg w-fit">${msg}</li>`;
            chatMessages.insertAdjacentHTML("beforeend", message);
        });

        form.addEventListener('submit', function (e) {
            e.preventDefault();
            if (input.value) {
                socket.emit('chat message', input.value);
                input.value = '';
            }
        });

    </script>

</head>

<body class="bg-gray-100 dark:bg-gray-900 text-gray-900 dark:text-gray-100">
    <div class="flex h-screen items-center justify-center">
        <section id="chat" class="flex flex-col w-full max-w-lg h-3/4 bg-white dark:bg-gray-800 rounded-lg shadow-lg">
            <div class="flex-1 overflow-y-auto p-4">
                <ul id="messages" class="flex flex-col gap-2 mb-2"> </ul>
            </div>

            <form id="form" action="" class="bg-gray-200 dark:bg-gray-700 p-4">
                <div class="flex items-center gap-2">
                    <div class="flex-1 flex items-center bg-gray-100 dark:bg-gray-600 rounded-full pl-4 pr-2 py-1">
                        <input type="text" placeholder="Write a message..." id="menssageInput"
                            class="flex-1 bg-transparent outline-none text-gray-900 dark:text-gray-100"
                            autocomplete="off">
                    </div>
                    <button
                        class="p-3 bg-blue-500 hover:bg-blue-600 rounded-full text-white flex items-center justify-center"
                        type="submit">
                        Send
                    </button>
                </div>
            </form>
        </section>
    </div>
</body>

</html>
```

## Creado por

**Jean Correa**

