# Registro de seguridad · Sistema Alumnos

Estudiante: Cristóbal Alexsander Collao Cerda   Repositorio: https://github.com/Crisyippie01/Parcial_03_Sistema_Alumnos   Fecha: 27-11-2026

Completa este registro a medida que avanzas. Cada fila debe poder seguirse hasta un **build de Jenkins**
y un **commit** (usa el identificador `VULN-xx` en el mensaje del commit).

## 1. Revisiones de seguridad realizadas

| # | Etapa del SDLC | Herramienta / técnica | Build | Resultado (resumen) |
|---|---|---|---|---|
| R1 | Código (SAST) | Bandit en el pipeline | #9 | 0 hallazgos High / 0 Medium / 0 Low |
| R2 | Dependencias (SCA) | pip-audit en el pipeline | #10 | 0 vulnerabilidades en 0 paquetes |
| R3 | Pruebas dinámicas (DAST) | OWASP ZAP sobre staging | #15 → #16 → #17 | #15: 2 High / 3 Medium / 1 Low · #16: 0 High / 2 Medium / 0 Low · #17: 0 High / 1 Medium / 0 Low |
| R4 | Revisión manual | Lectura de `app.py` | — | Se confirmaron los hallazgos de Bandit y pip-audit |
| R5 | Producción | Grafana + Prometheus | #17 | Dashboard operativo con 5 paneles; incidente simulado detectado en tiempo real |

## 2. Vulnerabilidades identificadas y corregidas

| ID | Vulnerabilidad (CWE) | Detectada por | Ubicación | Riesgo | Corrección aplicada | Commit | Build que lo verifica |
|---|---|---|---|---|---|---|---|
| VULN-01 | Clave secreta hardcodeada (CWE-259) | Bandit | `app.py:20` | Low | `SECRET_KEY` desde variable de entorno | `VULN-01: mover SECRET_KEY y credenciales a variables de entorno` | #4 |
| VULN-02 | Inyección SQL (CWE-89) | Bandit | `app.py:205` | Medium | Consulta parametrizada con `?` | `VULN-02: consulta parametrizada en /alumnos/buscar` | #5 |
| VULN-03 | Hash MD5 débil para contraseñas (CWE-327) | Bandit | `app.py:221` | High | Hash seguro con werkzeug.security | `VULN-03: reemplazar MD5 por hash seguro con werkzeug` | #6 |
| VULN-04 | Flask debug=True en producción (CWE-94) | Bandit | `app.py:241` | High | `debug=False` | `VULN-04: desactivar debug mode en producción` | #7 |
| VULN-05 | Binding a todas las interfaces (CWE-605) | Bandit | `app.py:241` | Medium | Escuchar en `127.0.0.1` | `VULN-05: restringir binding a localhost` | #8 |
| VULN-06 | Dependencias vulnerables en Flask (CVE-2023-30861, CVE-2026-27205) | pip-audit | `requirements.txt` | High | Actualizar Flask a 3.1.3 | `VULN-06: actualizar Flask a 3.1.3` | #10 |
| VULN-07 | Dependencias vulnerables en Werkzeug (múltiples CVE) | pip-audit | `requirements.txt` | High | Actualizar Werkzeug a 3.1.9 | `VULN-07: actualizar Werkzeug a 3.1.9` | #10 |
| VULN-08 | Dependencias vulnerables en Jinja2 (múltiples CVE) | pip-audit | `requirements.txt` | High | Actualizar Jinja2 a 3.1.6 | `VULN-08: actualizar Jinja2 a 3.1.6` | #10 |
| VULN-09 | Dependencias vulnerables en gunicorn (CVE-2024-1135, CVE-2024-6827) | pip-audit | `requirements.txt` | High | Actualizar gunicorn a 22.0.0 | `VULN-09: actualizar gunicorn a 22.0.0` | #10 |
| VULN-10 | Cross Site Scripting (DOM Based / Reflected) (CWE-79) | OWASP ZAP | `app.py:160` (`/hello`) | High | Escapar la salida con `escape()` | `VULN-10: escapar salida en /hello` | #16 |
| VULN-11 | Falta de cabeceras de seguridad (CSP, X-Frame-Options, X-Content-Type-Options) (CWE-693) | OWASP ZAP | `app.py:138-140` (`after_request`) | Medium/Low | Agregar cabeceras en `after_request` | `VULN-11: agregar cabeceras de seguridad` | #16 |
| VULN-12 | Ausencia de tokens Anti-CSRF (CWE-352) | OWASP ZAP | `/login` (formulario) | Medium | Agregar token CSRF en formularios | `VULN-12: agregar protección CSRF` | Pendiente |
| VULN-13 | CSP sin directivas de respaldo (`frame-ancestors`, `form-action`, `base-uri`) (CWE-693) | OWASP ZAP | `app.py:140` | Medium | Completar directivas del CSP | `VULN-13: completar directivas CSP` | #17 |

(Agrega las filas que necesites.)

## 3. Resultados de OWASP ZAP

| Alerta de ZAP | Riesgo | URL de ejemplo | VULN relacionada | Cómo se mitiga | ¿Mitigada? Build |
|---|---|---|---|---|---|
| Cross Site Scripting (DOM Based) | High | `http://jenkins:5001/hello?name=<script>alert(5397)</script>` | VULN-10 | Escapar la salida con `escape()` en `/hello` | Sí / #16 |
| Cross Site Scripting (Reflected) | High | `http://jenkins:5001/hello?name=</h1><script>alert(1)</script>` | VULN-10 | Escapar la salida con `escape()` en `/hello` | Sí / #16 |
| Absence of Anti-CSRF Tokens | Medium | `http://jenkins:5001` | VULN-12 | Agregar token CSRF en los formularios | No / pendiente |
| CSP: Failure to Define Directive with No Fallback | Medium | `http://jenkins:5001` | VULN-13 | Añadir `frame-ancestors`, `form-action`, `base-uri` al CSP | Sí / #17 |
| Content Security Policy (CSP) Header Not Set | Medium | `http://jenkins:5001` | VULN-11 | Agregar cabecera `Content-Security-Policy` | Sí / #16 |
| Missing Anti-clickjacking Header | Medium | `http://jenkins:5001` | VULN-11 | Agregar `X-Frame-Options: DENY` | Sí / #16 |
| X-Content-Type-Options Header Missing | Low | `http://jenkins:5001` | VULN-11 | Agregar `X-Content-Type-Options: nosniff` | Sí / #16 |

## 4. Monitoreo e incidentes

| Hora | Panel / métrica de Grafana | Qué se observó | Entorno | Clasificación | Acción de respuesta | Verificación |
|---|---|---|---|---|---|---|
| 20:12:45 | `login_fallidos_total` | 30 intentos de login fallidos en menos de 1 minuto (valor pasó de 0 a ~30) | Producción (5000) | Fuerza bruta | La app rechazó todos los intentos con HTTP 401; no hubo acceso | Contador en rojo en Grafana y líneas de log en `app.log` |
| 20:12:45 | `peticiones_sospechosas_total{tipo="sqli"}` | Intento de inyección SQL en `/alumnos/buscar` (`' OR 1=1 --`) | Producción (5000) | Inyección SQL | La consulta parametrizada evitó la fuga; el intento quedó registrado | Respuesta sin datos internos y contador `sqli` incrementado |
| 20:12:45 | `peticiones_sospechosas_total{tipo="xss"}` | Intento de XSS reflejado en `/hello` (`<script>alert(1)</script>`) | Producción (5000) | XSS | La salida escapada impidió la ejecución; el intento quedó registrado | Respuesta con `&lt;script&gt;` y contador `xss` incrementado |

## 5. Gestión de dependencias

| Paquete | Versión inicial | Vulnerabilidades (IDs) | Versión final | Origen del cambio (PR de Dependabot / manual) | ¿Pruebas, pip-audit y ZAP siguen OK? Build |
|---|---|---|---|---|---|
| Flask | 2.2.2 | PYSEC-2023-62, PYSEC-2026-2151 | 3.1.3 | Manual | Sí / #10 |
| Werkzeug | 2.2.2 | PYSEC-2023-57, PYSEC-2023-58, PYSEC-2023-221, PYSEC-2026-2043 a 2046, PYSEC-2026-2320, PYSEC-2026-3417, PYSEC-2026-4312 | 3.1.9 | Manual | Sí / #10 |
| Jinja2 | 3.1.2 | PYSEC-2026-1471, PYSEC-2026-1472, PYSEC-2026-1474, PYSEC-2026-1475 | 3.1.6 | Manual | Sí / #10 |
| gunicorn | 21.2.0 | PYSEC-2026-1433, PYSEC-2026-1434 | 22.0.0 | Manual | Sí / #10 |
| prometheus-client | 0.17.1 | — | 0.17.1 | — | Sí / #10 |

## 6. Trazabilidad

Pega aquí (o adjunta) el `TRAZABILIDAD.md` del último build y completa un recorrido por cada actividad del pipeline:

| Actividad | Hallazgo | Build que lo detectó | Commit / PR que lo resolvió | Build que lo verificó | Versión en producción |
|---|---|---|---|---|---|
| Revisión de seguridad (Bandit) | 5 hallazgos (VULN-01 a VULN-05) | #3 | Commits VULN-01 a VULN-05 | #4-#8 | 1.0.17+<sha> |
| Prueba automatizada (OWASP ZAP) | 2 High / 3 Medium / 1 Low → 0 High / 1 Medium / 0 Low | #15 | Commits VULN-10, VULN-11 y VULN-13 | #17 | 1.0.17+<sha> |
| Gestión de dependencias (Dependabot) | 20 vulnerabilidades (VULN-06 a VULN-09) | #10 | Commits VULN-06 a VULN-09 (manual; Dependabot pendiente) | #10 | 1.0.17+<sha> |
| Monitoreo (Grafana/Prometheus) | Ataque simulado: 30 logins fallidos + SQLi + XSS | #17 (entorno productivo) | — (detección, no corrección) | #17 | 1.0.17+<sha> |
