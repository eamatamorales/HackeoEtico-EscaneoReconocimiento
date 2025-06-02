# Laboratorio Semana 4 - Herramientas de Escaneo y Reconocimiento Ético (CY-203 Hackeo Ético)

---

# Evidencias para campus virtual

- Tome en cuenta que realizara las etapas de escaneo y reconocimiento para el sitio web del laboratorio detallado en este documento:
  - Para el final de la clase, en los grupos definidos al inicio del curso, deben realizar una presentación al C-Level y SMEs de la empresa sobre los hallazgos de estas etapas como justificación para realizar una auditoría de seguridad
  - Dispone de 10 min para la ppt + 5 Q&A
- Evaluación sugerida:
  - Capturas de pantalla para la ppt
  - Tomar en cuenta que la audiencia es C-Level + SMEs
- Debe subir la ppt al campos virtual como evidencia del trabajo.

---

## Objetivo General

Aplicar técnicas de escaneo y reconocimiento sobre una aplicación vulnerable expuesta en internet (Juice Shop), utilizando herramientas de red como `nmap`, `dig`, `whois`, `curl`, `hping3`, y `theHarvester`. Este laboratorio es una introducción práctica a las primeras fases del hacking ético.

---

## Entorno

- VM de Kali Linux (con red NAT o Bridge)
- Navegador instalado (Firefox o Chromium)
- Terminal de Kali con herramientas: `nmap`, `dig`, `whois`, `curl`, `hping3`, `theHarvester`, `ping`, `telnet`, `netcat`, `wireshark`

---

## Sitio objetivo

```
https://juice-shop.herokuapp.com
```

Este sitio web es parte del proyecto OWASP y está diseñado para simular vulnerabilidades reales de aplicaciones modernas.

---

## Introducción técnica

### ¿Qué es una fase de reconocimiento?

La fase de reconocimiento en pentesting es el proceso de recopilar información sobre el objetivo sin interactuar directamente con sus sistemas (reconocimiento pasivo) o con interacciones controladas (reconocimiento activo).

### ¿Qué son los puertos?

Los puertos son puntos de comunicación en un sistema operativo. Existen tres rangos:
- Puertos bien conocidos: 0-1023 (ej: 80 para HTTP)
- Puertos registrados: 1024–49151 (aplicaciones)
- Puertos dinámicos: 49152–65535 (uso temporal)

Escanear puertos permite saber qué servicios están disponibles públicamente.

### Siglas importantes

- **ICMP**: Internet Control Message Protocol (usado en `ping`)
- **TCP**: Transmission Control Protocol (conexiones fiables)
- **SYN**: Primer paquete del handshake TCP
- **ACK**: Confirmación del receptor
- **TTL**: Time To Live, indica el número de saltos que puede recorrer un paquete
- **CDN**: Content Delivery Network, red de distribución que oculta el servidor original y distribuye contenido

---

## Parte 1: Reconocimiento Pasivo

### 1. WHOIS
```bash
whois juice-shop.herokuapp.com
```

*Resultado esperado:* Puede indicar que no hay coincidencia directa porque `herokuapp.com` es dominio de Heroku, y Juice Shop está alojado como subdominio.

### 2. DIG
```bash
dig juice-shop.herokuapp.com
```

*Resultado esperado:* Resolución DNS a múltiples IPs públicas. Ejemplo:
```
juice-shop.herokuapp.com. 600 IN A 54.73.53.134
juice-shop.herokuapp.com. 600 IN A 54.220.192.176
```

Esto indica infraestructura cloud (Amazon EC2).

### 3. TheHarvester (requiere API Key para hunter.io)
```bash
theHarvester -d herokuapp.com -b hunter -l 100
```

*Resultado esperado:* Si no se configura la API de Hunter, mostrará advertencia. En caso de tenerla, se pueden obtener correos y subdominios expuestos en fuentes públicas.

---

## Parte 2: Reconocimiento Activo

### 1. Ping
```bash
ping juice-shop.herokuapp.com
```

*Resultado esperado:* Recibir paquetes ICMP con TTL ≈ 52–64 (Linux). Si está filtrado, puede no haber respuesta.

### 2. Banner grabbing con curl
```bash
curl -I https://juice-shop.herokuapp.com
```

*Resultado esperado:*
```
HTTP/1.1 200 OK
Server: Heroku
Content-Type: text/html; charset=UTF-8
```

Esto revela cabeceras útiles y confirma que el servicio está activo.

### 3. nmap para detección de puertos y SO
```bash
nmap -sS -sV -O juice-shop.herokuapp.com
```

*Resultado esperado:* 
```
80/tcp  open  http      heroku-router
443/tcp open  ssl/https heroku-router
OS detection inconclusive (servidor detrás de CDN)
```

Esto confirma el uso de un router HTTP de Heroku.

### 4. Wireshark

- Abre Wireshark y comienza captura en interfaz principal.
- Navega a `https://juice-shop.herokuapp.com`
- Observa los paquetes TCP y analiza TTL y puertos.

### 5. hping3 (requiere permisos root)
```bash
sudo hping3 -1 -c 3 juice-shop.herokuapp.com
```

*Resultado esperado:* Si se permite ICMP, se verá respuesta con TTL.

### 6. Fragmentación y evasión

```bash
nmap -sX $OBJETIVO     # Xmas Scan
nmap -sN $OBJETIVO     # Null Scan
nmap -f $OBJETIVO      # Fragmentado
nmap -D RND:10 $OBJETIVO  # IP Decoy
```

*Resultado esperado:*

- Si el firewall de Heroku permite escaneos ligeros, verás puertos 80 o 443 abiertos.
- IP Decoy mezcla tu IP real con otras aleatorias.
- Escaneos fragmentados pueden evadir inspección de contenido.

---

## Observaciones Finales

- El objetivo está protegido por una **infraestructura cloud (CDN)**, lo cual limita fingerprinting.
- Aun así, se pudo verificar disponibilidad, IPs, headers y puertos.
- No se identificó SO con `nmap -O` debido a protección del entorno.

---

## Resultado Esperado

- Verificación de IPs públicas desde DIG
- Headers HTTP útiles vía curl
- Puertos abiertos: 80 y 443
- TTL ≈ 52–64 en Wireshark
- Detalles del router Heroku en Nmap
- Reconocimiento pasivo sin generar ruido

---

## Contramedidas para el atacante ético

- **No usar root sin necesidad.** Usa `sudo` solo donde es requerido.
- **Aislar la VM Kali.** Evita escanear redes corporativas reales sin permiso.
- **Capturar tráfico solo en entornos autorizados.**
- **Proteger tu identidad y registrar logs locales.**
- **Usar VPN si practicas con IPs públicas.**
- **Hacer rollback o snapshot de VM tras cada práctica.**

---

## Estrategia de Escaneo y Reconocimiento

1. **Reconocimiento Pasivo (whois, dig, nslookup)**:
   - Permite conocer dominios, IPs, registros públicos sin generar tráfico sospechoso.
2. **Recolección OSINT (theHarvester)**:
   - Emails, dominios y datos abiertos que podrían facilitar ataques dirigidos.
3. **Escaneo de Puertos y Servicios (nmap)**:
   - Identifica qué servicios están disponibles y su posible versión.
4. **Banner Grabbing (curl, nc)**:
   - Extrae información del software remoto que puede revelar vulnerabilidades.
5. **Análisis de Tráfico (Wireshark)**:
   - TTL, flags TCP, estructura de red → identificación de SO y comportamiento del host.
6. **Evasión de detección (hping3, nmap -f, -sX, -D)**:
   - Técnicas avanzadas para burlar firewalls o IDS/IPS en pruebas controladas.

---

Este laboratorio forma una base sólida para las siguientes etapas de escaneo profundo, explotación y post-explotación en un ciclo completo de pentesting.

---

**Profesor:** Esteban Mata Morales  
**Curso:** CY-203 Hackeo Ético  
**Universidad Fidélitas de Costa Rica**
