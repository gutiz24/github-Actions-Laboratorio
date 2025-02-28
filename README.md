```diff
+ Propuesta Ejercicio 1
```
El archivo relacionado es [`.github/workflows/ci.yaml`](https://github.com/gutiz24/github-Actions-Laboratorio/blob/main/.github/workflows/ci.yaml)

El archivo está compuesto de diferentes partes

## Trigger de la pipeline

La pipeline solo se ejercutará cuando se haga una pull request sobre la rama `main` y haya algún cambio dentro de la carpeta `hangman-front/`
```yaml
on:
  pull_request:
    branches: [ main ]
    paths: [ "hangman-front/**" ]
```

## Build de la aplicación

El primer job definido es el build de la aplicación que se tratará de los siguientes pasos:

1. Hacer el checkout del repositorio de descargarse el proyecto en el paso `Checkout`
2. Luego en la acción `actions/setup-node@v4` se define la versión de node, en este caso la versión 16 por compatibilidades de usar la caché de la acción `cache: 'npm' \n cache-dependency-path: hangman-front/package-lock.json`
3. Ejecucón de los comandos de instalción limpia `npm ci` y build `npm run build --if-present` en el paso `build`
```yaml
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Set up Node.Js
        uses: actions/setup-node@v4
        with:
          node-version: 16
          cache: 'npm'
          cache-dependency-path: hangman-front/package-lock.json
      - name: build
        working-directory: ./hangman-front
        run: |
          npm ci
          npm run build --if-present
```
## Ejecución de Tests de la aplicación

Se trata de una ejecución parecida con el job de `build` solo que no se especifica una versión de node al venir los runners de github Actions con herramientas preinstaladas `node,docker,java,etc...`. 

Y se decide colocar una dependia entre jobs `needs: build` a modo ejemplo de cómo se realiza.

También como comando se usa `npm test` que será el responsable de ejecutar los test del proyecto
```yaml
  test:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: test
        working-directory: ./hangman-front
        run: |
          npm ci
          npm test
```
## Ejemplo de Ejecución de la Pipeline

Al solo activarse la pipeline con una pull request en cambios de archivos en la carpeta `hangman-front`, se va a mostrar el proceso que seguiría.

1. Creación de una nueva rama local con `git checkout -b gha-test`
2. Subirla en el repositorio remoto `git push -u origin gha-test`
3. Generar cambios dentro del proyecto `hangman-front`. Por ejemplo crear un `README`
<p align="center">
    <img src="./img/image1.png" width="50%" height="auto">
</p>

5. Se hace un `git add` `git commit` y `git push` del repo.
6. Hacer el pull request desde github
<p align="center">
    <img src="./img/image2.png" width="50%" height="auto">
</p>

7. Al revisar la ejecución de la pipeline se encuetran errores en los test. Mostrando así una de las funcionalidades de las pipelines
<p align="center">
    <img src="./img/image3.png" width="50%" height="auto">
    <img src="./img/image4.png" width="50%" height="auto">
</p>

8. Ajusando los valores de los test para que salga correcta la pipeline ya se puede hacer el merge corretamente
<p align="center">
    <img src="./img/image5.png" width="50%" height="auto">
</p>

```diff
+ Propuesta Ejercicio 2
```
## Trigger de la pipeline

En este Caso el trigger es de manera manual con la sentencia `workflow_dispatch`

```yaml
on:
  workflow_dispatch:
```

## Delivery

Respecto a las diferencias con la anterior pipeline de CI:
  - En el paso `Login to Github Container Registry` se usará para el inicio de sesión con el container registry, este caso el de github `ghcr.io`. El usuario usara es el que ejecute la pipeline `${{ github.actor }}` y como contraseña usará un token personal con el spoce de interacura con el registy `${{ secrets.GITHUB_TOKEN }}`
  - En el paso `Setup Docker builder` se habilitará la funcionalidad buildx de docker
  - En el paso `Build and push Docker image` se hace el build de la imagen con un archivo `Dockerfile` dentro de la carpeta `hangman-front/` y los sube al registry con el tag `ghcr.io/gutiz24/hangman-front-actions:latest`

```yaml
  delivery:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Login to Github Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - name: Setup Docker builder
        uses: docker/setup-buildx-action@v3
      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          context: ./hangman-front
          push: true
          tags: ghcr.io/gutiz24/hangman-front-actions:latest
          file: ./hangman-front/Dockerfile
```

## Ejemplo de ejecución Pipeline

Al poderse ejecutar de forma manual, llendo a la parte de `Actions` nos deja correrlo haciendo click en `Run Workflow`
<p align="center">
    <img src="./img/image6.png" width="50%" height="auto">
</p>

El resultado de la pipeline seria la sigiente de la subida de una nueva imagen:
<p align="center">
    <img src="./img/image7.png" width="50%" height="auto">
    <img src="./img/image8.png" width="50%" height="auto">
</p>

```diff
+ Propuesta Ejercicio 3
```
En este caso para la ejecución de los test end2end como se debe tener tanto `hangman-front` y `hangman-api` corriendo a la vez. Se ha decidido crear un `docker-compose.yaml` que levante esos 2 servicios y exponga sus puertos `3001` y `8080` respectivamente en el workflow de `.ghithub/workflows/test_e2e.yaml`

## Trigger de la pipeline

En este Caso el trigger es de manera manual con la sentencia `workflow_dispatch`

```yaml
on:
  workflow_dispatch:
```
## e2e-test

1. Las diferencias con respecto los anteriores wokflows es la implementación de un paso `run docker compose` que se levantará el docker-compose definido.
2. En el siguiente paso `Cypress run` se hace uso de la acción oficial de cypress `cypress-io/github-action@v6` con los parámetros:
  - Donde se posicionan la carpeta de cypress `working-directory: hangman-e2e/e2e`
  - comando a ejecutar para correr los tests `start: npm run open`
  - comando para que espere a los servicios a que respondan correctamente con un `200` antes de ejecutar los tests `wait-on: "http://localhost:3001/api/topics, http://localhost:8080"`
3. Y como último paso se sube un artefacto que es propio de cypress de generar un video de cómo ha ejecutado las pruebas. A este artefacto se le dio el nombre de `name: Video-Test-Cypress`

* **Resultado de la operación**
<p align="center">
    <img src="./img/image9.png" width="50%" height="auto">
</p>

```yaml
  e2e-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: run docker compose
        run: docker compose up -d --build
      - name: Cypress run
        uses: cypress-io/github-action@v6
        with:
            working-directory: hangman-e2e/e2e
            start: npm run open
            wait-on: "http://localhost:3001/api/topics, http://localhost:8080"
      - name: Upload artifacts (Cypress Video)
        uses: actions/upload-artifact@v4
        with:
          name: Video-Test-Cypress
          path: hangman-e2e/e2e/cypress/videos
```

# Ejercicios

Para superar el módulo debéis entregar como mínimo:

* La parte obligatoria de los ejercicios de Jenkins o GitLab.
* La parte obligatoria de los ejercicios de GitHub Actions.
* Uno de los dos ejercicios opcionales de la parte de GitHub Actions

## Ejercicios GitHub Actions

### 1. Crea un workflow CI para el proyecto de frontend - OBLIGATORIO

Copia el directorio [.start-code/hangman-front](../03-github-actions/.start-code/hangman-front) en el directorio raíz del mismo repositorio que usaste para las clases de GitHub Actions. Si no lo creaste, crea un repositorio nuevo.

Después crea un nuevo workflow que se dispare cuando haya cambios en el proyecto `hangman-front` y exista una nueva pull request (deben darse las dos condiciones a la vez). El workflow ejecutará las siguientes operaciones:

* Build del proyecto de front
* Ejecutar los unit tests

### 2. Crea un workflow CD para el proyecto de frontend - OBLIGATORIO

Crea un nuevo workflow que se dispare manualmente y haga lo siguiente:

* Crear una nueva imagen de Docker
* Publicar dicha imagen en el [container registry de GitHub](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)

### 3. Crea un workflow que ejecute tests e2e - OPCIONAL

Crea un workflow que se lance de la manera que elijas y ejecute los tests e2e que encontrarás en [este enlance](../03-github-actions/.start-code/hangman-e2e/e2e/). Puedes usar [Docker Compose](https://docs.docker.com/compose/gettingstarted/) o [Cypress action](https://github.com/cypress-io/github-action) para ejecutar los tests.

#### Como ejecutar los tests e2e

* Tanto el front como la api se deben estar corriendo

```bash
docker run -d -p 3001:3000 hangman-api
docker run -d -p 8080:8080 -e API_URL=http://localhost:3001 hangman-front
```

* Los tests se ejecutan desde el directorio `hangman-e2e/e2e` haciendo uso del comando `npm run open`

```bash
cd hangman-e2e/e2e
npm run open
```

### 4. Crea una custom JavaScript Action - OPCIONAL

Crea una custom JavaScript Action que se ejecute cada vez que una `issue` tenga la etiqueta `motivate`. La acción deberá pintar por consola un mensaje motivacional. Puedes usar [esta API](https://type.fit) gratuita. Puedes encontrar más información de como crear una custom JS action en [este enlace](https://docs.github.com/es/actions/creating-actions/creating-a-javascript-action).

```bash
curl https://type.fit/api/quotes
```
