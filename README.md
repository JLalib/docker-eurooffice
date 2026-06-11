# 📝 Euro Office — Despliegue con Docker

**Euro Office DocumentServer** es un servidor de documentos de código abierto auto-hospedado compatible con los formatos de Microsoft Office (DOCX, XLSX, PPTX) y OpenDocument. Permite editar, visualizar y colaborar en documentos directamente desde el navegador.

## 📋 Requisitos previos

- [Docker](https://docs.docker.com/get-docker/) >= 20.10
- [Docker Compose](https://docs.docker.com/compose/install/) >= 2.0
- `openssl` instalado en el host (para generar el JWT Secret)

## 🔑 Generar el JWT Secret

Antes de desplegar, genera una clave segura para autenticar las peticiones al servidor:

```bash
openssl rand -hex 32
```

Copia el resultado — lo necesitarás en el siguiente paso.

## 🚀 Inicio rápido

### 1. Crear el archivo `docker-compose.yml`

```yaml
services:
  euro-office:
    image: ghcr.io/euro-office/documentserver:latest
    container_name: euro-office
    stdin_open: true
    tty: true
    ports:
      - "8100:80"
    restart: unless-stopped
    environment:
      - THEME=euro-office
      - EXAMPLE_ENABLED=true
      - JWT_SECRET=TU_JWT_SECRET_AQUI
```

### 2. Reemplazar el JWT Secret

Sustituye `TU_JWT_SECRET_AQUI` con la clave generada en el paso anterior. Alternativamente, usa un archivo `.env` (recomendado):

```env
JWT_SECRET=a3f9c2e1b847d06f5e...
```

Y referencia la variable en el compose:

```yaml
environment:
  - THEME=euro-office
  - EXAMPLE_ENABLED=true
  - JWT_SECRET=${JWT_SECRET}
```

Añade `.env` a tu `.gitignore`:

```bash
echo ".env" >> .gitignore
```

### 3. Levantar el servicio

```bash
docker compose up -d
```

### 4. Acceder a la interfaz

```
http://localhost:8100
```

## ⚙️ Variables de entorno

| Variable | Descripción | Valor por defecto |
|---|---|---|
| `THEME` | Tema visual del servidor | `euro-office` |
| `EXAMPLE_ENABLED` | Activa la página de ejemplos en `/example` | `true` |
| `JWT_SECRET` | Clave secreta para firmar tokens JWT | *(obligatorio)* |

> ⚠️ **`JWT_SECRET` no debe estar vacío en producción.** Un servidor sin JWT expuesto a internet puede ser utilizado por terceros sin restricciones.

## 🗂️ Compatibilidad de formatos

| Tipo | Formatos soportados |
|---|---|
| Documentos de texto | DOCX, ODT, TXT, RTF, HTML |
| Hojas de cálculo | XLSX, ODS, CSV |
| Presentaciones | PPTX, ODP |
| PDF | Visualización y conversión |

## 🛑 Gestión del servicio

```bash
# Detener el servicio
docker compose down

# Reiniciar
docker compose restart euro-office

# Ver logs en tiempo real
docker compose logs -f euro-office

# Actualizar la imagen
docker compose pull
docker compose up -d
```

## 🔧 Puertos

| Puerto | Uso |
|---|---|
| `8100` | Interfaz web y API del servidor de documentos |

Puedes cambiar el puerto host modificando la parte izquierda del mapeo:

```yaml
ports:
  - "PUERTO_DESEADO:80"
```

## 🔒 Proxy inverso con Caddy (opcional)

Para exponer Euro Office con dominio propio y HTTPS automático:

```
office.tudominio.com {
    reverse_proxy euro-office:80
}
```

> Asegúrate de que el contenedor de Caddy y `euro-office` estén en la misma red Docker.

## 🔗 Referencias

- [GitHub — Euro-Office/DocumentServer](https://github.com/Euro-Office/DocumentServer)
- [GitHub Container Registry](https://ghcr.io/euro-office/documentserver)
