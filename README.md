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



### 💻 Evidencia Técnica
Aquí se muestran las credenciales obtenidas en texto plano interceptando el paquete HTTP:

<img width="1920" height="1080" alt="Captura de pantalla_20260716_003208" src="https://github.com/user-attachments/assets/15e58af8-a689-47d5-8820-4e4d01ea889f" />

También adjunto el bloque del log extraído directamente del flujo TCP:

```http
POST /login.html HTTP/1.1
Host: zero.webappsecurity.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64) Firefox/115.0
Connection: keep-alive

user=pepito&password=1234
```
###  Riesgo y Mitigación
Este ejercicio demuestra el riesgo de usar el protocolo HTTP frente a HTTPS. Un atacante en la misma red Wi-Fi podría robar credenciales fácilmente al capturar la información sin cifrar. 

**Mitigación:** Se debe implementar un certificado de seguridad (SSL/TLS) en el servidor. Esto obliga a que la página cargue mediante HTTPS, cifrando los datos para que nadie en la red pueda leer los usuarios y contraseñas.


# Proyecto 2: Home Lab y Explotación de Vulnerabilidades (FTP)

## Descripción
Configuración de un entorno de red aislado y seguro (Home Lab) mediante virtualización para realizar pruebas de penetración (pentesting) de forma individual, sin exponer la red local. El objetivo principal fue construir la arquitectura, identificar y explotar vulnerabilidades críticas en un servidor de pruebas.

## Tecnologías Utilizadas
* **Hipervisor:** Oracle VirtualBox
* **Máquina Atacante:** Parrot OS
* **Máquina Objetivo:** Metasploitable 2
* **Herramientas:** Nmap, Metasploit Framework

---

## Fase 1: Arquitectura y Seguridad
Para este laboratorio, diseñé y configuré un adaptador de red "Host-Only" (`vboxnet0`) para permitir la comunicación bidireccional exclusiva entre mi máquina atacante y la víctima. Esto garantizó que el entorno de pruebas se mantuviera completamente aislado del acceso a internet, aplicando estándares profesionales de seguridad para laboratorios.

<img width="1372" height="922" alt="image" src="https://github.com/user-attachments/assets/91c11a51-4f18-45ba-820e-8025ce628599" />


---

## Fase 2: Reconocimiento y Escaneo (Nmap)
Una vez asegurada la conexión entre ambas máquinas y descubierta la IP objetivo, procedí a realizar un escaneo agresivo de puertos y servicios de forma autónoma utilizando Nmap (`nmap -sS -T4 -sV`). 

**Resultados del escaneo:**
Se identificaron múltiples puertos abiertos en la máquina víctima. Durante mi análisis, destaqué como vector de ataque principal el puerto **21 (FTP)**, el cual ejecutaba el servicio `vsftpd 2.3.4`.

<img width="1033" height="554" alt="image" src="https://github.com/user-attachments/assets/5d8c144e-15b1-437f-8c6d-5158895ab8f2" />


---

## Fase 3: Explotación (Metasploit Framework)
A partir de los resultados de la fase de reconocimiento, identifiqué que el servicio `vsftpd 2.3.4` contiene una vulnerabilidad crítica conocida (CVE-2011-2523), la cual fue introducida maliciosamente en su código fuente original para permitir la ejecución de comandos a través de un "backdoor" (puerta trasera).

**Ejecución del ataque:**
Llevé a cabo la explotación utilizando Metasploit Framework mediante los siguientes pasos:
1. Seleccioné el módulo correspondiente: `exploit/unix/ftp/vsftpd_234_backdoor`.
2. Resolví un conflicto técnico de enrutamiento asignando dinámicamente mi adaptador de red interno (`set LHOST vboxnet0`) para asegurar la conexión.
3. Lancé el ataque, obteniendo exitosamente una sesión de **Meterpreter** inyectada en la memoria del objetivo.
<img width="1920" height="1031" alt="image" src="https://github.com/user-attachments/assets/74d0042b-a02e-4d55-9642-d52a50ce2377" />




**Escalada de Privilegios y Verificación:**
Para confirmar el nivel de acceso obtenido tras vulnerar el servicio FTP, ejecuté el comando `getuid` dentro de la sesión de Meterpreter. El sistema validó que logré comprometer la máquina con privilegios máximos, obteniendo el control total como usuario `root`.

<img width="1920" height="1029" alt="image" src="https://github.com/user-attachments/assets/fb876668-39bc-4517-9051-ed9d14e51cc6" />

