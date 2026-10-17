# Resultados de las pruebas manuales — EP02 Atlas

> Completar Pablo (tester). Felipe completa la sección de despliegue. Enviar a Yanfranco junto con las capturas.

## Datos de ejecución
| Campo | Valor |
| --- | --- |
| Ejecutado por | Pablo Astroza Navarro |
| Fecha | |
| Equipo (CPU / RAM / SO) | |
| Commit / link de la ejecución de Actions | |

## Despliegue (Felipe)
| Paso | Resultado (OK / Falla + detalle) |
| --- | --- |
| `mvnw clean verify` (152 pruebas) | |
| `docker compose up --build` | |
| Pipeline 14 jobs en verde | |
| SonarCloud: Quality Gate / bugs / vulnerabilidades / hotspots / cobertura | |
| Snyk: resultado (vulnerabilidades críticas, altas) | |

## Postman (C03)
| Requests | Tests aprobados | Tests fallidos | Observaciones |
| --- | --- | --- | --- |
| /42 | /82 | | |

## JMeter 200 usuarios (C16–C18)
| Escenario | Peticiones | % error | p95 login (ms) | p95 listar clientes (ms) | p95 contratos (ms) | Throughput (req/s) |
| --- | --- | --- | --- | --- | --- | --- |
| Rampa 60 s | | | | | | |
| Rampa 20 s (estrés, opcional) | | | | | | |

## ZAP activo autenticado (C13–C15)
| High | Medium | Low | Informational |
| --- | --- | --- | --- |
| | | | |

Alertas High y Medium (nombre, URL afectada, solución propuesta por ZAP):

-

## Casos que fallaron o se comportaron distinto a lo esperado
| Caso | Qué pasó | Captura |
| --- | --- | --- |
| | | |

## Checklist de capturas
- [ ] D01–D09 (Felipe)
- [ ] C01–C17, C19–C30, C33 (Pablo) · C18, C31, C32 opcionales
