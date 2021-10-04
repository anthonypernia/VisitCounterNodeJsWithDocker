
La aplicacion de NodeJs configura el contador para almacenar la data de visitas en Redis, y se mantiene escuchando en e lpuerto 8081

```javascript
const express = require("express");
const redis = require("redis");
const proccess = require("process")

const app = express();
const client = redis.createClient({
    host: 'redis-server',  //El nombre del contenedor en el docker compose, que corre el server redis
    port: 6379   //puerto por defecto es ese
 });
client.set("visits", 0 )

app.get("/", (req, res) => {
    process.exit(0);
    client.get('visits', (err, visits) => {
        res.send("number of visits is " + visits);
        client.set("visits", parseInt(visits) + 1 )

    });

});

app.listen(8081, () => {
    console.log("listening en posrt 8081")
})
```

En el dockerfile levantamos la version mas liviana de node, y le instalamos las dependencias, y copiamos la aplicacion al contenedor.
Al final lo ejecutamos
```bash
FROM node:alpine

WORKDIR '/app'

COPY package.json .
RUN npm install
COPY . . 

CMD ["npm", "start"]

```
Aca Levantamos los dos contenedores con docker-compose , y gracias a la versatilidad que ofrece, se comunican entre ellos.
Al final mapeamos la salida de el puerto del contenedor 8081, hacia el 4001 del server donde esta ejecutando.
```yaml
version: '3'
services:
  redis-server:
    image: 'redis'
  node-app:
    restart: always
    build: .
    ports:
      - "4001:8081"

```