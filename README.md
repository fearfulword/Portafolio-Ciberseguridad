#  Portafolio de Ciberseguridad

## Proyecto 1: Análisis de Tráfico de Red y Fuga de Datos

**Objetivo:** Capturar el tráfico de red de la computadora local para interceptar, filtrar e identificar cómo viajan los datos en protocolos inseguros frente a protocolos seguros, emulando el trabajo de un analista de SOC.
**Entorno y Herramientas:** Wireshark en entorno basado en Debian (Parrot OS).

###  Resumen del Análisis
Se realizó una captura de tráfico local interactuando con un entorno web de prueba sin cifrado (`http://zero.webappsecurity.com`). Al aplicar filtros a las peticiones, se logró interceptar la comunicación y reconstruir el flujo de datos, evidenciando la exposición crítica de credenciales.

###  Metodología
1. Intercepción de tráfico en la interfaz de red activa[cite: 1].
2. Interacción con entorno vulnerable ingresando credenciales ficticias en el formulario web[cite: 1].
3. Aplicación del filtro `http.request.method == "POST"` para aislar los envíos del formulario[cite: 1].
4. Análisis del paquete mediante `Follow -> TCP Stream` para visualizar las variables expuestas[cite: 1].

###  Evidencia Técnica (Extracto del TCP Stream)
Al analizar la petición HTTP interceptada, se extrajeron las siguientes credenciales en texto plano:
<img width="1920" height="1080" alt="Captura de pantalla_20260716_003208" src="https://github.com/user-attachments/assets/15e58af8-a689-47d5-8820-4e4d01ea889f" />



```http
POST /login.html HTTP/1.1
Host: zero.webappsecurity.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Connection: keep-alive

user=pepito&password=1234



