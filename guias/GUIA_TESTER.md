# Guía de pruebas y evidencias — Pablo

**Objetivo:** ejecutar las pruebas manuales del plan y generar las capturas del informe.
**Tiempo estimado:** 2 horas. **Al terminar:** completar `evidencias/RESULTADOS.md`,
comprimir la carpeta `evidencias/` (raíz del repo) y enviarla a Yanfranco.

> **Carpetas:** los comandos se ejecutan **dentro de `atlas-ep2/`**. Las capturas se guardan en
> `evidencias/` de la **raíz** del repo (desde `atlas-ep2/` es `..\evidencias\`).

Cada captura tiene un **nombre fijo** (`C01`, `C02`, …): guárdala con ese nombre en la carpeta
indicada. Yanfranco la pega en el informe donde dice **[CAPTURA n]** con el mismo número.

> Tip: en Windows usa **Win + Shift + S** para recortar la pantalla. Las capturas deben ser legibles.

---

## 0. Requisitos (instalar antes)

| Herramienta | Descarga |
| --- | --- |
| Java 21 (Temurin) | <https://adoptium.net> — verificar con `java -version` |
| Git | <https://git-scm.com> |
| Postman Desktop | <https://www.postman.com/downloads> |
| OWASP ZAP 2.15+ | <https://www.zaproxy.org/download> |
| DB Browser for SQLite | <https://sqlitebrowser.org> |
| Docker Desktop (opcional) | Solo si levantas la app con Docker |

JMeter **no** hace falta instalarlo: lo descarga Maven en el paso 6.

```bash
git clone <link del repo que envía Felipe>
cd atlas-ep02-mvp/atlas-ep2
```

---

## 1. Compilar y ejecutar las pruebas automatizadas (CP-CAL-01, CP-CAL-03)

```bash
mvnw.cmd clean verify          # Linux/Mac: ./mvnw clean verify
```

Debe terminar con `Tests run: 152, Failures: 0` y `BUILD SUCCESS` (~3 min).

> Si falla en **Spotless** (Windows, saltos de línea): `mvnw.cmd spotless:apply` y repetir.

| Captura | Carpeta | Qué mostrar |
| --- | --- | --- |
| **C30** | `01-local/` | Consola con el resumen de pruebas y `BUILD SUCCESS` |
| C31 (opcional) | `01-local/` | `atlas-ep2/target/site/jacoco/index.html` abierto en el navegador (cobertura) |
| C32 (opcional) | `01-local/` | `atlas-ep2/target/pit-reports/index.html` (mutación) |

## 2. Levantar la aplicación

```bash
mvnw.cmd spring-boot:run
```

Esperar `Started AtlasApplication`. **Dejar esta terminal abierta** durante todas las pruebas.

| Captura | Carpeta | Qué mostrar |
| --- | --- | --- |
| **C01** | `01-local/` | http://localhost:8080 (Home de Atlas) |
| **C02** | `01-local/` | http://localhost:8080/swagger-ui.html |

> Para empezar de cero: detener la app (Ctrl+C), borrar `atlas-ep2/data/atlas.db*` y volver a levantarla.

## 3. Postman — pruebas funcionales y de seguridad

1. Postman → **Import** → los dos archivos de `atlas-ep2/postman/`.
2. Arriba a la derecha: entorno **Atlas local**.
3. Colección **Atlas MVP** → **Run** → orden por defecto, **1 iteración** → **Run**.

Resultado esperado: **42 requests, 82 tests aprobados, 0 fallidos**.

| Captura | Carpeta | Request (abrir → **Send** → capturar respuesta + *Test Results*) | Caso |
| --- | --- | --- | --- |
| **C03** | `02-postman/` | Resumen del **Runner** con todos en verde | Todos |
| **C04** | `02-postman/` | `CP-FUN-03 Login ADMIN` (token y rol) | CP-FUN-03 |
| **C05** | `02-postman/` | `CP-SEG-03 Bloqueo tras 5 intentos fallidos` → **429** | CP-SEG-03 |
| **C06** | `02-postman/` | `CP-FUN-04 El token revocado ya no sirve` → **401** | CP-FUN-04 |
| **C07** | `02-postman/` | `CP-SEG-01 EDITOR no puede crear clientes` → **403** | CP-SEG-01 (EP01) |
| **C08** | `02-postman/` | `CP-SEG-05 Entrada con script (XSS)` → **400** con `details` | CP-SEG-05 |
| **C09** | `02-postman/` | `CP-INT-02 Listar contratos del cliente` → 3 estados | CP-FUN-02 (EP01) |
| **C10** | `02-postman/` | `CP-FUN-01 Eliminar sin confirmar` → **400** | CP-FUN-01 (EP01) |
| **C11** | `02-postman/` | `CP-SEG-08 Cabeceras de seguridad en la Home` → pestaña **Headers** | CP-SEG-08 |

> Para C04–C11: si el Runner ya consumió los datos (por ejemplo, el cliente eliminado), ejecuta
> primero `CP-FUN-03 Login ADMIN` y luego el request que quieras capturar.
> En el Runner: **Export Results** → guardar como `02-postman/postman-run.json`.

**C33** (`02-postman/`): terminal de la app mostrando las líneas `AUDITORIA` (login, crear,
editar, eliminar, logout) → caso CP-SEG-11.

## 4. Base de datos — contraseñas con bcrypt (CP-SEG-06)

DB Browser for SQLite → **Open Database** → `atlas-ep2/data/atlas.db` → **Browse Data** → tabla `usuarios`.

**C12** (`03-base-datos/`): columna `hash_contrasena` con valores que empiezan por `$2a$12$`.

## 5. Swagger — documentación y mensajes de error (CP-USA-02)

1. http://localhost:8080/swagger-ui.html → `POST /api/v1/auth/login` → **Try it out** →
   `{"nombreUsuario":"editor","contrasena":"Editor#Atlas2026"}` → **Execute** → copiar `accessToken`.
2. Botón **Authorize** (arriba) → pegar el token → **Authorize**.
3. `POST /api/v1/clientes` → **Try it out** → **Execute**.

**C20** (`01-local/`): respuesta **403** "Su rol no tiene permiso para esta operación".

4. Repetir login con `admin` / `Admin#Atlas2026`, Authorize, y en `POST /api/v1/clientes` usar
   `"rut": "12.345.678-9"`.

**C19** (`01-local/`): respuesta **400** "El RUT no es válido (dígito verificador incorrecto)".

## 6. JMeter — 200 usuarios concurrentes (CP-PERF-03)

En **otra terminal**, también dentro de `atlas-ep2/` (la app sigue corriendo):

```bash
mvnw.cmd -Pjmeter -Djmeter.usuarios=200 -Djmeter.rampa=60 jmeter:configure@configuration jmeter:jmeter@jmeter-tests jmeter:results@jmeter-check-results
```

Esperar `summary = 2200 ... Err: 0 (0.00%)` y `Failed requests: 0`.

| Captura | Carpeta | Qué mostrar |
| --- | --- | --- |
| **C16** | `04-jmeter/` | Consola con las líneas `summary` y `Failed requests: 0` |
| **C17** | `04-jmeter/` | Dashboard: ver nota abajo |
| C18 (opcional) | `04-jmeter/` | Repetir con `-Djmeter.rampa=20` (estrés: el login se vuelve lento) |

**Dashboard (C17):** el CSV queda en `atlas-ep2/target/jmeter/results/`. Para el reporte HTML:

```bash
# Windows (PowerShell), desde la carpeta atlas-ep2:
$J = (Get-ChildItem target -Recurse -Filter ApacheJMeter-5.6.3.jar | Select-Object -First 1).DirectoryName
$CSV = (Get-ChildItem target\jmeter\results\*.csv | Select-Object -First 1).FullName
java -jar "$J\ApacheJMeter-5.6.3.jar" -g $CSV -o ..\evidencias\04-jmeter\dashboard
```

Abrir `evidencias/04-jmeter/dashboard/index.html` → capturar la tabla **Statistics**.

**Anotar en RESULTADOS.md:** total, % error, p95 de *login*, *listar clientes* y *contratos del
cliente*, y la **CPU y RAM** de tu computador (Administrador de tareas → Rendimiento).

## 7. OWASP ZAP — escaneo activo autenticado (CP-SEG-09)

1. Postman → `CP-FUN-03 Login ADMIN` → copiar `accessToken` (dura **15 minutos**).
2. ZAP → *No, I do not want to persist this session* → **Start**.
3. **Tools → Options → Replacer → Add**:
   - Description: `JWT Atlas` · Match Type: **Request Header (will add if not present)**
   - Match String: `Authorization` · Replacement String: `Bearer <token>` · **Enable** → OK
4. **Import → Import an OpenAPI Definition from a URL** →
   `http://localhost:8080/openapi/atlas-openapi.yaml` · Target: `http://localhost:8080`.
5. Árbol **Sites** → clic derecho en `http://localhost:8080` → **Attack → Active Scan** → **Start**.
   Esperar 100 %. (Si demora más de 15 min, renovar el token en el Replacer.)
6. **Report → Generate Report** → *Traditional HTML Report* → guardar en `05-zap/zap-activo.html`.

| Captura | Carpeta | Qué mostrar |
| --- | --- | --- |
| **C13** | `05-zap/` | Pestaña **Alerts** con el árbol de alertas por riesgo |
| **C14** | `05-zap/` | Detalle de la alerta de mayor riesgo |
| **C15** | `05-zap/` | Portada del reporte HTML (resumen) |

**Anotar en RESULTADOS.md:** cantidad de alertas High / Medium / Low / Informational y el
nombre de cada High o Medium.

## 8. GitHub Actions (CP-CAL, CP-SEG-08/10, CP-PERF-02)

En el repo → **Actions** → última ejecución **verde** de *Calidad de código*:

| Captura | Carpeta | Qué mostrar |
| --- | --- | --- |
| **C21** | `06-pipeline/` | Grafo con los 14 jobs en verde |
| **C22** | `06-pipeline/` | Resumen de *Pruebas unitarias* (tabla de clases) |
| **C23** | `06-pipeline/` | Resúmenes de *Mutación (PIT)* y *JaCoCo* |
| **C24** | `06-pipeline/` | Resumen de *Contenedor (Docker + Trivy)* |
| **C25** | `06-pipeline/` | Resumen de *DAST (OWASP ZAP)* |
| **C26** | `06-pipeline/` | Resúmenes de *Gatling* y *JMeter* |
| **C27** | `06-pipeline/` | Artefactos al final de la página |
| **C28** | `06-pipeline/` | SonarCloud (o la captura D07 de Felipe) |
| **C29** | `06-pipeline/` | Job Snyk (o la captura D08 de Felipe) |

> Los resúmenes aparecen al hacer clic en la ejecución y bajar por la página (sección *Summary*).

---

## 9. Checklist final

- [ ] C01–C17, C19–C30 y C33 guardadas con su nombre (C18, C31, C32 opcionales)
- [ ] `RESULTADOS.md` completo (fecha, equipo, Postman, JMeter, ZAP, fallas)
- [ ] Comprimir `evidencias/` → `evidencias-atlas.zip` → enviar a Yanfranco
