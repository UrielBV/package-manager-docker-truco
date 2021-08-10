# Trucazo #001, evitar que Docker ejecute el comando de instalación de tu package manager

Cuando estamos creando una aplicación y nos ayudamos de docker para contenerizarla, existe el inconveniente en el que si cambiamos una linea de código, docker invalida todas las demas capas, y vuelve a hacer build, esto incluye el comando del administrador de packetes.

## Nodejs y NPM

Este trucazo sera explicado con el ejemplo de Nodejs y su package manager más popular NPM

## El problema
En nuestro Dockerfile, tendremos algo parecido a esto
```json
FROM node:version

COPY [".", "/usr/src/"]

WORKDIR /usr/src

RUN npm install

EXPOSE 3000

CMD ["node", "index.js"]
```
Donde el comando que queremos evitar es:
```json
RUN npm install
```
Por que descarga todas las dependencias incluso si no cambiamos las dependencias
## El trucazo
De acuerdo del orden en que docker ejecuta las acciones, podemos copiar los archivos de las dependencias, de manera separada al código, entonces copiando primero los archivos del package manager, docker, detectara que a manos que haya cambios en las dependencias, ejecutara el comando de 
```json 
RUN npm install
```
Después copiamos toda la palicación, como ya se estaba haciendo, pero... aqui tambien vienen incluidos los archivos del package manager.. asi es, pero docker los detecta como que no se han hecho cambios, y no los volverá a copiar, quedandonos un codigo como el siguiente

```json
FROM node:version

#Aquí es donde copiamos los archivos del package manager
COPY ["package.json", "package-lock.json", "/usr/src/"]

WORKDIR /usr/src

#Este solo se ejecuta si hubo cambio en el COPY anterior
RUN npm install

#Tranqui, que aquí se copia el resto de la aplicación
COPY [".", "/usr/src/"]

EXPOSE 3000

CMD ["node", "index.js"]
```


## Contributing
No soy el author intelectual tampoco se si esto es trivial o norma en docker, pero, me ayudó bastante y es un TRUCAZO!

## License
[MIT](https://choosealicense.com/licenses/mit/)
