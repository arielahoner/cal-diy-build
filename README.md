# cal-diy-build

Construcción de la imagen Docker de [**cal.diy**](https://github.com/calcom/cal.diy) (fork comunitario MIT
de cal.com) para una instancia personal autoalojada.

cal.diy no publica imágenes propias, así que este repositorio hace una sola cosa: ejecutar
`.github/workflows/build-image.yml`, que compila cal.diy en un runner de GitHub Actions, lo prueba y
publica el resultado en GHCR.

```text
calcom/cal.diy@<sha>  →  GitHub Actions (linux/amd64)  →  ghcr.io/<owner>/cal-diy:<sha>
```

## Uso

Actions → **Construir imagen de Cal.diy** → *Run workflow*:

| Entrada | Por defecto | Qué es |
|---|---|---|
| `ref` | `176037d0` | SHA (o rama/tag) de `calcom/cal.diy` a construir. |
| `webapp_url` | `https://cal.arielsaez.me` | URL pública **horneada** en la imagen. Debe ser idéntica a la del `.env` de producción; si difieren, el arranque reescribe todo `.next/` con `sed` en cada inicio. |
| `push` | `true` | Publicar en GHCR. |

O desde la terminal:

```bash
gh workflow run build-image.yml -f ref=176037d0 -f webapp_url=https://cal.arielsaez.me -f push=true
gh run watch
```

Al terminar, el **resumen del run** trae la imagen, el digest, el tamaño y las líneas listas para pegar en
el `VERSION.lock` del despliegue. La imagen se referencia siempre por digest:

```dotenv
CALDIY_IMAGE=ghcr.io/<owner>/cal-diy:<sha>@sha256:<digest>
```

El paquete de GHCR debe marcarse **público** (Packages → cal-diy → Package settings → Change visibility)
para poder hacer `docker pull` desde el servidor sin token.

## Qué hace el workflow

Replica la receta de upstream (`.github/actions/docker-build-and-test/action.yml`), que es la única
combinación probada, y añade verificaciones:

1. Checkout de `calcom/cal.diy@<ref>` + aplicación de los parches de `patches/` (si hay).
2. Postgres efímero (`docker-compose.yml` de upstream) + buildx unido a su namespace de red: el
   `next build` necesita una base viva.
3. Build `linux/amd64` con los build-args de upstream, más `NEXT_PUBLIC_WEBAPP_URL` y
   `CALCOM_TELEMETRY_DISABLED=1`.
4. Verificación de la imagen: la URL horneada es la pedida, no quedan variables sensibles en `Config.Env`
   y las credenciales del Postgres de build no aparecen dentro de `.next/`.
5. Smoke test: el contenedor arranca contra la base efímera, `/auth/login` responde 200/307 y
   `prisma migrate status` confirma que las migraciones se aplicaron (el `start.sh` de la imagen **no**
   aborta si fallan, por eso se comprueba aparte).
6. Push a GHCR + resumen con el digest.

Duración típica: 15–25 minutos. El repositorio es público a propósito: los runners de repositorios
públicos tienen 4 vCPU / 16 GB, y el build de cal.diy necesita `MAX_OLD_SPACE_SIZE=6144`.

## Parches

`patches/*.patch` se aplican con `git apply` sobre el checkout de upstream antes de construir. Si hay
alguno, la etiqueta lleva el sufijo `-patched`. Cada parche debe llevar en su cabecera: propósito, SHA de
upstream sobre el que se probó y por qué es necesario. Hoy la carpeta está vacía a propósito.

## Lo que este repositorio NO contiene

- **Ningún secreto.** No se pasan secretos como build-args (quedarían en la imagen y en los logs públicos);
  los valores de Postgres del workflow son los de upstream para una base que vive y muere dentro del job.
  Las claves reales (`NEXTAUTH_SECRET`, `CALENDSO_ENCRYPTION_KEY`, credenciales de Google, API keys) solo
  existen en el `.env` del servidor.
- **Ningún código de cal.diy.** Se descarga en cada ejecución desde upstream, fijado por SHA.
- Ninguna configuración ni dato del despliegue.

## Licencia

El workflow se publica tal cual, sin garantías. cal.diy es de [Cal.com, Inc.](https://github.com/calcom/cal.diy)
bajo licencia MIT y está recomendado por sus autores para uso personal/no productivo: quien lo aloja asume
servidor, base de datos, seguridad y continuidad.
