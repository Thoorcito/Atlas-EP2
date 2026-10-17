# Guía de despliegue y pipeline — Felipe

**Objetivo:** dejar el código en el nuevo repositorio, levantar la app (local y Docker),
activar SonarCloud y Snyk y confirmar el pipeline con los 14 jobs en verde.
**Tiempo estimado:** 1 a 1,5 horas. **Al terminar:** avisar a Pablo con el link del repo.

Las capturas van en `evidencias/00-despliegue/` (carpeta en la raíz del repo) con el nombre indicado (por ejemplo `D01-...png`).

---

## 1. Requisitos

| Herramienta | Verificación |
| --- | --- |
| Git | `git --version` |
| Java 21 (Temurin) | `java -version` → debe decir 21 |
| Docker Desktop (encendido) | `docker --version` y `docker compose version` |
| Cuenta GitHub | Con permiso para crear el repo nuevo |

## 2. Crear el repositorio nuevo

1. En GitHub: **New repository** → nombre sugerido `atlas-ep02-mvp` → **Public** (SonarCloud gratis
   requiere repo público) → sin README.
2. En la carpeta raíz del zip descomprimido (la que contiene `README.md`, `atlas-ep2/`,
   `evidencias/` y `guias/`):

```bash
git init -b main
git add .
git commit -m "MVP Atlas Caso A - EP02"
git remote add origin https://github.com/<usuario>/atlas-ep02-mvp.git
git push -u origin main
```

> Verifica con `git status` que **no** se suban `.env`, `atlas-ep2/target/` ni `atlas-ep2/data/*.db` (ya están en `.gitignore`).

3. El push dispara el workflow **Calidad de código** (pestaña **Actions**). Tarda ~12 minutos.
   SonarQube y Snyk terminan en verde con el aviso "pendiente" hasta el paso 5.

**D01:** pestaña Actions con la ejecución en curso o terminada.

## 3. Compilar y probar en local

```bash
cd atlas-ep2
./mvnw clean verify            # Windows: mvnw.cmd clean verify
```

Debe terminar con `Tests run: 152, Failures: 0` y `BUILD SUCCESS`.

> **Windows:** si falla en *Spotless* por saltos de línea, ejecuta `mvnw.cmd spotless:apply` y repite.

**D02:** consola con el resumen de pruebas y `BUILD SUCCESS`.

## 4. Desplegar con Docker

```bash
cd atlas-ep2                   # si no estás ya dentro
cp .env.example .env           # Windows: copy .env.example .env
# (opcional) pegar en ATLAS_JWT_SECRET un valor de: openssl rand -base64 32
docker compose up --build -d
docker compose ps              # atlas-api debe estar "running"
docker compose logs atlas-api  # buscar "Started AtlasApplication" y "Usuario inicial creado"
```

Abrir http://localhost:8080 (Home) y http://localhost:8080/swagger-ui.html.

| Captura | Qué mostrar |
| --- | --- |
| **D03** | `docker compose ps` con `atlas-api` arriba |
| **D04** | Logs con `Started AtlasApplication` |
| **D05** | Home de Atlas en el navegador |

Detener: `docker compose down` (con `-v` borra la base de datos).

> Si el puerto 8080 está ocupado, cambia `"8080:8080"` por `"8081:8080"` en `docker-compose.yml`
> y avisa a Pablo para que use `http://localhost:8081`.

## 5. Activar SonarCloud y Snyk en el pipeline

Repo → **Settings → Secrets and variables → Actions**.

### SonarCloud
1. Entrar a <https://sonarcloud.io> con GitHub → **+ → Analyze new project** → elegir el repo.
2. **Administration → Analysis Method** → desactivar **Automatic Analysis**.
3. Copiar el **Project Key** y la **Organization Key** que muestra SonarCloud
   (*Information* del proyecto; ej. `usuario_atlas-ep02-mvp` y `usuario`).
4. **My Account → Security → Generate Tokens** → copiar.
5. Crear:
   - Secret `SONAR_TOKEN` = el token
   - Secret `SONAR_HOST_URL` = `https://sonarcloud.io`
   - Variable (pestaña *Variables*) `SONAR_ORGANIZATION` = la *organization key*
   - Variable `SONAR_PROJECT_KEY` = el *project key*

### Snyk
1. Entrar a <https://app.snyk.io> con GitHub → **Account settings → Auth Token → Click to show**.
2. Crear el secret `SNYK_TOKEN`.

### Volver a ejecutar
**Actions → Calidad de código → Run workflow**.

| Captura | Qué mostrar |
| --- | --- |
| **D06** | Pantalla de *Secrets and variables* (sin mostrar valores) |
| **D07** | Dashboard de SonarCloud: Quality Gate, bugs, vulnerabilidades, hotspots, cobertura |
| **D08** | Log del job Snyk con el resultado |

> **Si Snyk falla** por vulnerabilidades críticas del framework: copiar el log del job y
> compartirlo; se agregan a `.snyk` como riesgo aceptado con justificación o se actualiza la
> dependencia. Si SonarCloud no pasa el Quality Gate, anotar los hallazgos para el informe.

## 6. Confirmar el pipeline

En la última ejecución verifica los **14 jobs en verde**:
Spotless → Checkstyle → PMD → SpotBugs → Pruebas unitarias → Mutación (PIT) → JaCoCo →
SonarQube → Snyk → Cucumber → Contenedor (Docker + Trivy) → DAST (OWASP ZAP) →
Rendimiento (Gatling) → Rendimiento (JMeter).

**D09:** grafo con los 14 jobs en verde.

## 7. Entregar a Pablo

Enviar por el grupo:
- Link del repositorio y de la ejecución verde de Actions.
- Las capturas D01–D09 (subirlas al repo en `evidencias/00-despliegue/` o enviarlas por el grupo).
- Cualquier cambio de puerto o de contraseñas en `.env`.
