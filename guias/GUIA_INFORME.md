# Guía para finalizar el informe — Yanfranco

**Recibes:** el informe `Informe_EP02_CasoA_Atlas.docx`, `evidencias-atlas.zip` (de Pablo, misma
estructura que la carpeta `evidencias/` del repo) y el link del repo con el pipeline en verde (de Felipe).
**Objetivo:** completar el informe con evidencias reales, confirmar coherencia con el EP01 y el
pipeline, revisar formato y rúbrica, y exportar el PDF.
**Tiempo estimado:** 2 a 3 horas.

---

## 1. Completar los textos en amarillo

En Word: **Ctrl + B (Buscar)** → buscar `[` para recorrer todos los marcadores.

| Sección | Qué completar | Fuente |
| --- | --- | --- |
| Portada | Integrantes, carrera, sección 003V, docente, fecha | Portada del EP01 |
| 1.5 Ajustes EP01 | Confirmar la tabla de ajustes | Rúbrica EP01 (84/100) |
| 3.5 Recursos humanos | Líder QA / Funcional / Seguridad / DevOps con nombres | Roles del equipo |
| 3.6 Cronograma | Estados "En curso / Completado" | Avance real |
| 5.2 Detalle | Columna *Fecha / Responsable* (Pablo para manuales, CI para automáticos) | RESULTADOS.md |
| 5.2.3 ZAP activo | Alertas High / Medium / Low / Info | RESULTADOS.md |
| 5.2.5 Rendimiento | CPU/RAM del equipo y números de JMeter de Pablo | RESULTADOS.md |
| 5.3 Incidentes | INC-07 (ZAP activo) e INC-08 (SonarCloud/Snyk): descripción, criticidad y acción | RESULTADOS.md y capturas D07/D08 |

**Regla para criticidad:** alerta ZAP High → GRAVE (o CRÍTICO si expone datos); Medium → MODERADO;
Low → LEVE. Quality Gate fallido → MODERADO; vulnerabilidad crítica nueva en Snyk → GRAVE.

## 2. Pegar las capturas

Donde el informe dice **[CAPTURA n]**, insertar la imagen `Cn` de `evidencias/`:
**Insertar → Imágenes**, ancho ≈ 15 cm, y debajo una leyenda `Figura X. <caso>: <qué muestra>`.

| Marcador | Archivo | Sección |
| --- | --- | --- |
| [CAPTURA 1–2] | `01-local/C01`, `C02` | Anexo B |
| [CAPTURA 3] | `02-postman/C03` | 5.1 / 5.2.1 |
| [CAPTURA 4–11] | `02-postman/C04`…`C11` | 5.2.1 |
| [CAPTURA 12] | `03-base-datos/C12` | 5.2 (CP-SEG-06) |
| [CAPTURA 13–15] | `05-zap/C13`…`C15` | 5.2.3 |
| [CAPTURA 16–18] | `04-jmeter/C16`…`C18` | 5.2.5 |
| [CAPTURA 19–20] | `01-local/C19`, `C20` | 5.2 (CP-USA-02) |
| [CAPTURA 21–27] | `06-pipeline/C21`…`C27` | 5.1 / 5.2 |
| [CAPTURA 28–29] | `06-pipeline/C28`, `C29` o D07/D08 de Felipe | 5.2.4 |
| (opcional) D03–D05 | `00-despliegue/` | 3.5 Entorno / Anexo B |
| (opcional) C30, C33 | `01-local/`, `02-postman/` | 5.2.2 / CP-SEG-11 |

## 3. Confirmar coherencia con el pipeline

Abrir la última ejecución verde en Actions y comparar con el informe:

| Dato en el informe | Dónde verificar |
| --- | --- |
| 152 pruebas, 0 fallas | Resumen *Pruebas unitarias* |
| Mutación 96,6 %, cobertura 99,9 % | Resúmenes *PIT* y *JaCoCo* |
| Trivy: SO 0 HIGH/CRITICAL; librerías CRITICAL 2 · HIGH 0 | Resumen *Contenedor* |
| ZAP baseline: High 0 · Medium 2 · Low 3 | Resumen *DAST* |
| Gatling y JMeter (CI) | Resúmenes de rendimiento |
| SonarQube / Snyk | Jobs 8 y 9 (ya no deben decir "pendiente") |

Si algún número cambió, actualizarlo en el informe (secciones 5.1 y 5.2).
Si SonarCloud y Snyk ya funcionan, cambiar CP-CAL-02 y CP-SEG-12 de **Bloqueado** a
**Aprobado / Fallido** y actualizar el conteo de 5.1.

## 4. Confirmar coherencia con el EP01

- [ ] Los 6 IDs del EP01 aparecen con el mismo significado: CP-FUN-01, CP-FUN-02, CP-INT-01,
      CP-SEG-01, CP-PERF-01, CP-USA-01 (tabla de equivalencias en la sección 4).
- [ ] La sección 1.5 explica: cambio de tecnología, alcance MVP, 401 vs 403, CP-PERF-01
      (JMeter → Lighthouse), campo *Datos de prueba* corregido, estado POR_VENCER, Ley 21.663.
- [ ] La portada coincide con la del EP01 (nombres, sección, docente).

## 5. Revisión de formato (pauta EP02)

- [ ] Títulos Arial 12, cuerpo Arial 11, interlineado 1,5, texto justificado.
- [ ] No queda ningún texto en amarillo ni `[ ]`.
- [ ] Capturas legibles y con leyenda.
- [ ] Índice actualizado: clic derecho sobre el índice → *Actualizar campos*, o corregir números a mano.
- [ ] **Archivo → Exportar → PDF** → `Informe_EP02_CasoA_Atlas.pdf`.

## 6. Checklist de rúbrica EP02

| Indicador | Peso | Dónde se cumple |
| --- | --- | --- |
| IE1 Calidad, seguridad y ley | 20 % | Sección 2 + 6 (ejemplos de impacto, leyes 19.628, 21.719, 21.459, 21.663) |
| IE3 Tipos de prueba y propósito | 38 % | Sección 3.2 (clasificación + tabla con justificación) |
| IE5 No funcionales con propósito y contexto | 15 % | Casos PERF, SEG, USA con criterios + ejecución 5.2.5 |
| IE7 Plan y vínculo con requisitos/normas | 15 % | Sección 3 completa + matriz 4.1 |
| IE9 Cobertura y coherencia con estándares | 12 % | Matriz 100 % ERS + cobertura ISO 25010 / OWASP |

Entregar en AVA: el PDF (o Word) del informe y el material de apoyo para la defensa.
