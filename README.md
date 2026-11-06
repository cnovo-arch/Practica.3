# TiendaService — Tienda online con Docker

Aplicación de tienda online desarrollada en Python. El despliegue separa la API REST (FastAPI) y la interfaz web (Streamlit) en **dos contenedores** conectados mediante Docker Compose.

## Arquitectura

- **backend**: API REST con FastAPI, expuesta en el puerto `8000`.
- **frontend**: interfaz Streamlit, expuesta en el puerto `8501`. Al arrancar ejecuta primero `cliente.py` contra la API para realizar una demostración de las operaciones y después inicia la interfaz.
- **models/** y **Services/**: clases de dominio y lógica de negocio que utiliza el backend.
- **docker-compose.yml**: construye y coordina ambos servicios; espera a que el backend supere su comprobación de salud antes de arrancar el frontend.

## Requisitos

- Docker Desktop (Windows/macOS) o Docker Engine con el complemento Docker Compose.
- Git, si se va a clonar el repositorio.

No es necesario instalar Python ni las dependencias en el ordenador anfitrión para ejecutar el despliegue Docker.

## Estructura relevante

```text
TiendaService/
├── backend/
│   ├── Dockerfile
│   ├── main.py
│   ├── cliente.py
│   └── requirements.txt
├── frontend/
│   ├── Dockerfile
│   ├── frontend.py
│   └── requirements.txt
├── models/
├── Services/
├── docker-compose.yml
├── .dockerignore
├── .gitignore
└── README.md
```

## Ejecutar desde la terminal

Abre una terminal en la carpeta raíz del proyecto, donde está `docker-compose.yml`.

### 1. Construir las dos imágenes e iniciar los servicios

```bash
docker compose up --build
```

La primera ejecución puede tardar unos minutos mientras Docker descarga la imagen base e instala las dependencias. La terminal mostrará los logs de ambos servicios.

### 2. Abrir la aplicación

- **Interfaz de la tienda:** http://localhost:8501
- **Documentación interactiva de la API:** http://localhost:8000/docs

La interfaz usa `http://backend:8000` para comunicarse con la API dentro de la red privada de Docker Compose. No cambies esa URL por `localhost` dentro del contenedor frontend: `localhost` apuntaría al propio contenedor frontend.

### 3. Comprobar el estado

En otra terminal, desde la carpeta del proyecto:

```bash
docker compose ps
docker compose logs -f
```

Para consultar un servicio concreto:

```bash
docker compose logs -f backend
docker compose logs -f frontend
```

### 4. Detener el despliegue

```bash
docker compose down
```

Para reconstruir las imágenes después de modificar código:

```bash
docker compose up --build
```

## Ejecutar el backend y frontend por separado (opcional)

También puedes construir las imágenes individualmente desde la raíz del proyecto:

```bash
docker build -f backend/Dockerfile -t tienda-service-backend:latest .
docker build -f frontend/Dockerfile -t tienda-service-frontend:latest .
```

Se recomienda usar `docker compose up --build` para ejecutar ambos servicios juntos y configurar su red automáticamente.

## Publicar las imágenes en GitHub Container Registry (GHCR)

Si el profesor debe descargar las imágenes ya construidas, puedes publicarlas en GitHub Container Registry. Sustituye `TU_USUARIO` por tu nombre de usuario de GitHub. Crea un Personal Access Token con permisos para publicar paquetes y úsalo como contraseña cuando Docker lo solicite.

```bash
docker login ghcr.io -u TU_USUARIO
docker tag tienda-service-backend:latest ghcr.io/TU_USUARIO/tiendaservice-backend:latest
docker tag tienda-service-frontend:latest ghcr.io/TU_USUARIO/tiendaservice-frontend:latest
docker push ghcr.io/TU_USUARIO/tiendaservice-backend:latest
docker push ghcr.io/TU_USUARIO/tiendaservice-frontend:latest
```

Para que Compose descargue las imágenes publicadas en vez de construirlas localmente, se puede preparar una configuración de despliegue que use esas imágenes `ghcr.io/...`. No publiques tokens ni credenciales en el repositorio.

## Publicar el código en GitHub

Desde la carpeta del proyecto, si todavía no está conectada a un repositorio remoto:

```bash
git init
git add .
git commit -m "Preparar despliegue Docker Compose"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/TU_REPOSITORIO.git
git push -u origin main
```

Si el repositorio ya existe y tiene configurado `origin`, no repitas `git remote add origin`; utiliza:

```bash
git add .
git commit -m "Actualizar despliegue Docker"
git push
```

## Notas importantes

- `cliente.py` crea un usuario, dos productos y un pedido de demostración cuando se inicia el contenedor frontend. Como los datos de esta versión se guardan en memoria en el backend, se reinician al recrear el contenedor backend y no constituyen almacenamiento persistente.
- El cliente de demostración puede crear registros nuevos cada vez que se recrea el frontend. Si no quieres ejecutar la demostración automáticamente, elimina `python cliente.py http://backend:8000 &&` del `CMD` de `frontend/Dockerfile`.
- Los puertos del anfitrión `8000` y `8501` deben estar libres. Si están ocupados, modifica la parte izquierda de la asignación en `ports`.
- El `Dockerfile` de cada servicio usa el contexto de construcción de la raíz (`.`), porque el backend necesita copiar también `models/` y `Services/`.
