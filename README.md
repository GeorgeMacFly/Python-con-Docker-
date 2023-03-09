#############################################################################################     
                                                 
# ☠ Ambiente de desarrollo Python con Docker ☠ # 
                                                 
#############################################################################################



![image](https://user-images.githubusercontent.com/121887930/223876966-301d317c-2124-48ff-b8e1-0fe9c221206c.png)




Docker es una plataforma de software que usando contenedores posibilita empaquetar el software y sus dependencias de forma que se pueda usar de la misma forma en diferentes sistemas operativos. Estos contenedores son finalmente convertidos en imágenes que pueden ser compartidas a través de registros públicos como es el caso de Docker Hub para que mas gente pueda usar ese software.

Que tiene que ver esto con Python ? 

En la pagina web de Docker hub: https://hub.docker.com/python , hay disponible una imagen oficial de Python que podemosusar como base para nuestros desarrollos sin nesesidad de instalar el interprete o modulo en nuestro equipo y ademas haciendo uso de los Tags de Docker podremos usar diferentes versiones del mmismo.



# Docker empaquetado vs ambiente de desarrollo. #
Primero surge la necesidad de contener tu aplicación con docker, quizá porque vas a utilizar kubernetes o algunas solución de K-native, preparas tu archivo docker y esperas al momento del despliegue para empaquetar la solución final, mientras tanto todo lo ejecutas con tus dependencias locales, es decir solo usas docker para empaquetar tu aplicación, pero que pasa si lo usas como ambiente de desarrollo que tu aplicación se ejecute en el contenedor utilizando las dependencias del sistema operativo, esto lo hará más real, ya que debe comportarse exactamente igual cuándo lo despliegues en la nube, y no habrá mucho problema al momento de subir la imagen.

# ¿Porque crear un ambiente de desarrollo con docker? #
Cuándo sospechas que las cosas no se compartan exactamente igual, no existen muchos casos en los que exista incompatibilidad de sistemas, pero definitivamente cuando una aplicación es más robusta los vas a encontrar, mayormente incompatibilidades con Windows y en menos medida con Mac, o simplemente quieres tener todo listo y prefieres automatizar para ejecutar un comando de docker para dejar todo listo, y así compartir con tu equipo.

# Demostración de aplicación Fast API #
Primero que nada garantiza que tu aplicación en su forma más básica funciona en local.

Para este ejemplo se va a utilizar un framework de Python reciente, que con estos 2 archivos podemos tener una aplicación.


###### main.py

```python
# archivo main.py

from fastapi import FastAPI
app = FastAPI()
@app.get("/")
def read_root():
    return {"Hambiente": "De Pruebas.."}
```    
    
###### requirements.txt

```python 
# archivo requirements.txt
fastapi==0.59.0                 # Framework web
uvicorn==0.11.5                 # Servidor
```
###### Para instalar las dependencias

```
pip install -r requirements.txt
```

###### Para ejecutar la aplicación

```
uvicorn main:app
```

###### Bien con esto debería funcionar en local, sólo tienes que entrar a localhost:8000 en tu Navegador web 

## Preparando el contenedor docker. ##

###### Agregamos el archivo Dockerfile 


```Dockerfile
FROM python:3.8.4-slim-buster
COPY . usr/src/app
WORKDIR /usr/src/app
RUN pip install -r requirements.txt
ENTRYPOINT uvicorn --host 0.0.0.0 main:app --reload
```

###### Ejecutando el comando para construir la imagen llamada simple_app

```
docker build -t simple_app .
```

## Montar un volumen ##
Este es una de las partes clave, un volumen se refiere en realidad a compartir una carpeta entre tu local y tu contenedor, de modo que cuando modifiques algún archivo por ejemplo desde tu editor de código se verá reflejado, en los archivos del docker, ahora nota en el archivo docker que tenemos un ENTRYPOINT con el parámetro — reload esto significa que la aplicación dentro del contenedor se va a actualizar si uno de sus archivos cambia, de modo que como tenemos un volumen, programamos en local pero la ejecución es desde dentro del contenedor. la mayoría de los frameworks incluso en otros lenguajes tienen esta opción, solo tienes que investigar un poquito sobre ella.

###### El comando sería el siguiente -v representa la configuración del volumen

En caso de tener windows seria asi:
```
docker run -it -p 8000:8000 -v %cd%:/usr/src/app simple_app
```
%cd% nos devuelve la carpeta donde estas posionado, esta es la que va a ligar con la carpeta interna del contendo llamada app en /usr/src/app


para sistemas basados en unix cambía por $PWD seria asi:

```
docker run -it -p 8000:8000 -v $PWD:/usr/src/app simple_app
```

###### Probando.
Listo tu aplicación debería estar corriendo en el contenedor, y cuando actualices el código en tu editor, la aplicación interna se va a recargar, ahora para no construir el mismo contenedor una y otra vez deberías reutilizar el mismo que ya tienes.

utiliza los comandos

```
docker stop [idContenedor]
```

```
docker start -a [idContenedor]
```

Para apagar y prender tu contenedor respectivamente


###### Otra opción
— reload es un parámetro que no quieres en producción, deberías quitarlo del entrypoint en el docker y que no esperas que el código cambie, quizá te gustaría tener el contenedor sin ejecutar el servidor y hacerlo tu manualmente podrías utilizar un comando como el siguiente donde ```5c4e34e462e9```representa el id de tu contenedor.

```
docker exec -it 5c4e34e462e9 uvicorn main:app --reload
```
