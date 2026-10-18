# Resultados de las pruebas manuales — EP02 Atlas

> Completar Pablo (tester). Felipe completa la sección de despliegue. Enviar a Yanfranco junto con las capturas.

## Datos de ejecución
| Campo | Valor |
| --- | --- |
| Ejecutado por | Pablo Astroza Navarro |
| Fecha | 17 y 18 de octubre de 2026 |
| Equipo (CPU / RAM / SO) | AMD Ryzen 7 8845HS (8 núcleos / 16 hilos, 3,8 GHz) / 16 GB DDR5 / Windows 11 |
| Java usado | Temurin 21.0.12 (el proyecto no compila con Java 25: JaCoCo 0.8.12 no lo soporta) |
| Commit / link de la ejecución de Actions | Commit `4e2839f` (main) · ejecución #3 "Agrego evidencia del pipeline de github Actions" · Success · 11 min 43 s · 10 artefactos · https://github.com/Thoorcito/Atlas-EP2/actions/runs/39080825569 |

## Despliegue (Felipe)
| Paso | Resultado (OK / Falla + detalle) |
| --- | --- |
| `mvnw clean verify` (152 pruebas) | |
| `docker compose up --build` | |
| Pipeline 14 jobs en verde | |
| SonarCloud: Quality Gate / bugs / vulnerabilidades / hotspots / cobertura | |
| Snyk: resultado (vulnerabilidades críticas, altas) | |

## Pruebas automatizadas en local (C30)
| Herramienta | Resultado |
| --- | --- |
| JUnit + Mockito + Cucumber | 152 pruebas · 0 fallas · 0 errores · BUILD SUCCESS (1 min 46 s) |
| PIT (mutación) | 145 mutantes · 140 eliminados · **96,6 %** (la consola local lo redondea a 97 %; el pipeline muestra 96,6 %, mismo resultado) |
| JaCoCo | "All coverage checks have been met" (99 % instrucciones · 98 % ramas, captura C31 de Felipe) |

## Postman (C03)
| Requests | Tests aprobados | Tests fallidos | Observaciones |
| --- | --- | --- | --- |
| 42/42 | 82/82 | 0 | Corrida del 18-10 a las 00:15. La corrida del 17-10 a las 23:07 dio 80/82 (ver fallas, C03b). |

## JMeter 200 usuarios (C16–C18)
| Escenario | Peticiones | % error | p95 login (ms) | p95 listar clientes (ms) | p95 contratos (ms) | Throughput (req/s) |
| --- | --- | --- | --- | --- | --- | --- |
| Rampa 60 s | 2200 | 0,00 % | 327 | 5 | 5 | 36,7 |
| Rampa 20 s (estrés, opcional) | no ejecutada | | | | | |

Notas:
- 200 usuarios virtuales con rampa de 60 s (11 peticiones cada uno). APDEX 1.000 en las tres operaciones (C17a).
- Concurrencia real baja: el log muestra `Active: 2` como máximo, porque cada usuario termina en < 0,5 s. Informar como "200 usuarios virtuales en 60 s", no como "200 simultáneos".
- Promedio total 30 ms · máximo 397 ms · p95 total 310 ms.

## ZAP activo autenticado (C13–C15)
| High | Medium | Low | Informational |
| --- | --- | --- | --- |
| 0 | 0 | 0 | 1 |

Configuración: ZAP 2.17.0 · contrato importado desde `http://localhost:8080/openapi/atlas-openapi.yaml` · token JWT del ADMIN inyectado con el Reemplazo (`Authorization: Bearer …`) · **4489 peticiones** · `/auth/login` y `/auth/logout` excluidos del escáner (el logout revocaría el token y el login bloquearía al admin). Las cargas de inyección (comillas, `timeout`, `sleep`, `|`, `;`) se rechazaron con 400.

Reporte: `evidencias/05-zap/zap-activo.html` (C15). Insights del reporte: 15 endpoints; 95 % de respuestas 4xx (ZAP lo marca "Medio – Exceeded High", pero es esperable: las cargas maliciosas del escáner se rechazan con 400 por la validación de entrada); 4 % de respuestas 2xx; 7 % de respuestas lentas (corresponde al login, por bcrypt).

Alertas High y Medium (nombre, URL afectada, solución propuesta por ZAP):

- Ninguna.
- Informativa (1): **Respuesta de Gestión de Sesión Identificada** (*Session Management Response Identified*, regla pasiva 10112) en `POST /api/v1/auth/login`, parámetro `accessToken`, confianza media. No es una vulnerabilidad: ZAP solo informa que detectó el token de sesión en la respuesta del login, que es el comportamiento esperado (RF-1.1). No requiere acción.

## Seguridad en el pipeline (C24, C25, ejecución #3)
| Herramienta | Resultado |
| --- | --- |
| Trivy (C24) | Imagen `atlas-api:4e2839f…` · SO alpine 3.24.2 · CRITICAL 2 · HIGH 0 · MEDIUM 2 · LOW 0. Los 2 CRITICAL son de librerías, no del SO: CVE-2026-47884 y CVE-2026-47890 en `spring-webmvc`, corregidos en 7.0.9 (requiere Spring Boot 4; limitación ya declarada en el README) |
| ZAP baseline (C25) | High 0 · Medium 2 · Low 3 · Informational 2. Medium: CSP `style-src 'unsafe-inline'` (10055, 4 instancias) y Vulnerable JS Library (10003, 1; DOMPurify de Swagger UI). Low: COEP ausente (90004), Private IP Disclosure (2), Timestamp Disclosure (10096) |

**Diferencia con el escaneo activo local (0 Medium):** el baseline del pipeline recorre las páginas web (Home, Swagger UI, Redoc), donde están el CSP con `unsafe-inline` y la librería de Swagger; el escaneo activo local atacó solo la API importada desde OpenAPI (JSON). Los dos resultados son coherentes y se complementan.

## Rendimiento en el pipeline (C26, ejecución #3)
| Herramienta | Resultado |
| --- | --- |
| Gatling | 110 peticiones · 0 KO · 3 aserciones cumplidas: 0 fallas; p95 listar clientes 7 ms < 300; p95 contratos 6 ms < 300 · p95 global 344 ms (incluye login) |
| JMeter (CI, 50 usuarios) | 550 peticiones · 0,00 % error · p95 login 349 ms · p95 listar clientes 7 ms · p95 contratos 7 ms |

El pipeline aplica el umbral de 300 ms solo a las consultas, no al login: coincide con el hallazgo local de bcrypt (p95 login 327 ms con 200 usuarios).

## Casos que fallaron o se comportaron distinto a lo esperado
| Caso | Qué pasó | Captura |
| --- | --- | --- |
| CP-FUN-02 / CP-INT-02 (script de Postman) | Corrida a las 23:07 (Chile): Postman calcula "ayer" en UTC (`toISOString`), que ya era el día siguiente; el contrato "expirado" terminaba hoy y la API respondió POR_VENCER, lo que es correcto para la hora de Chile. Defecto del script de prueba, no de la app; falla solo entre 21:00 y 00:00. Corrida después de las 00:00: 82/82. Mejora: calcular la fecha local en el pre-request de "CP-FUN-11 Crear contrato ACTIVO". | C03b, C03 |
| CP-PERF-03 / NFR-PERF-1 | p95 de login = 327 ms (> 300 ms). Consultas: p95 = 5 ms. Causa: bcrypt con costo 12 (NFR-SEG-2), que tarda ~300 ms por intento de forma intencional. Se acepta como compromiso seguridad/rendimiento; alternativas: aplicar el umbral de 300 ms solo a consultas o bajar el costo a 10. Criticidad: LEVE. | C17b |
| CP-SEG-09 (ZAP) – contrato OpenAPI | El ejemplo del login en el OpenAPI trae la contraseña real de demo del admin (`Admin#Atlas2026`); al importar, ZAP inició sesión con ella (200). Exposición de información, OWASP A05. Recomendación: ejemplo ficticio en el OpenAPI y contraseñas distintas en `.env` para producción. Criticidad: LEVE. | — |
| CP-FUN-04 (observado en ZAP) | Al importar el OpenAPI, ZAP llamó a `/auth/logout` con el token y todas las peticiones siguientes dieron 401: confirma que la revocación funciona. Se renovó el token y se excluyó el logout del escaneo. | — |

## Checklist de capturas
- [ ] D01–D09 (Felipe) · en el repo: D01–D05 · faltan D06–D09
- [x] C01, C03, C04, C05, C07, C08, C09, C10, C11, C12, C16, C17, C19, C20, C30, C33 (Pablo)
- [ ] C06: reemplazar `C06b` por la captura de las 01:12 (la actual es de las 01:10, con el token ya vencido) y agregar `C06a` (logout 204)
- [x] C02 y C31 (subidas por Felipe)
- [x] C13, C14, C15 (ZAP) + reporte `05-zap/zap-activo.html`
- [x] C21–C27 (GitHub Actions, ejecución #3)
- [ ] C28 SonarQube y C29 Snyk: pendientes (revisar si el pipeline los ejecutó o dicen "pendiente" por falta de token)
- Extras: C03b1/C03b2 (fallo por zona horaria), C17a (APDEX), C19a/C19x (Swagger), C20a/C20b, C30b (PIT)
- No realizadas (opcionales): C18 (estrés), C32 (PIT HTML, cubierta por C30b)
