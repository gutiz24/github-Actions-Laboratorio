```diff
+ Propuesta Ejercicio 1
```
El archivo relacionado es [`.github/workflows/ci.yaml`](https://github.com/gutiz24/github-Actions-Laboratorio/blob/main/.github/workflows/ci.yaml)

El archivo está compuesto de diferentes partes

* **Trigger de la pipeline**

La pipeline solo se ejercutará cuando se haga una pull request sobre la rama `main` y haya algún cambio dentro de la carpeta `hangman-front/`
```yaml
on:
  pull_request:
    branches: [ main ]
    paths: [ "hangman-front/**" ]
```

* **Build de la aplicación**

El primer job definido es el build de la aplicación que se tratará de los siguientes pasos:

1. Hacer el checkout del repositorio de descargarse el proyecto en el paso `Checkout`
2. Luego se define la versión de node con la acción `actions/setup-node@v4` a parte se especifica el parámetro caché para que en futuras ejecuciones tarde menos la ejecución del build
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
          node-version: 20
          cache: 'npm'
          cache-dependency-path: hangman-front/package-lock.json
      - name: build
        working-directory: ./hangman-front
        run: |
          npm ci
          npm run build --if-present
```
* **Ejecución de Tests de la aplicación**

Se trata de una ejecución parecida con el job de `build` solo que cuando se selecciona la versión de node no se especifica uso de caché por no hacer falta en proyectos menos extensos.

También como comando se usa `npm test` que será el responsable de ejecutar los test del proyecto
```yaml
  test:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Set up Node.Js
        uses: actions/setup-node@v4
        with:
          node-version: 20
      - name: test
        working-directory: ./hangman-front
        run: |
          npm ci
          npm test
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
