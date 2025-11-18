# 🥒 Pickle Rick CTF - TryHackMe
**Fecha:** 17 de Noviembre, 2025  
**Plataforma:** TryHackMe  
**Autor:** rSamCod  
**Resumen:**  
Desafío enfocado en enumeración web, descubrimiento de directorios, credenciales expuestas, panel con ejecución remota (Web Shell)

---
## 1. Escaneo de puertos con Nmap

Comenzamos identificando todos los puertos abiertos:

```bash
nmap -p- -sS --open --min-rate 5000 -vvv -n -Pn 10.201.25.37 -oG allPorts
```

**Parámetros:**
- `-p-` : Escanea todos los puertos (1–65535)  
- `-sS` : Escaneo SYN (rápido y sigiloso)  
- `--open` : Muestra únicamente puertos abiertos  
- `--min-rate 5000` : Envía al menos 5000 paquetes por segundo  
- `-vvv` : Máximo nivel de verbosidad  
- `-n` : Deshabilita resolución DNS  
- `-Pn` : Omitir descubrimiento de host (asume que está vivo)  
- `-oG allPorts` : Exporta salida en formato Grepable

<img width="1900" height="516" alt="Captura de pantalla 2025-11-17 205022" src="https://github.com/user-attachments/assets/895f2d2b-2fd9-405e-9b6e-b4c5740f603a" />


---

## 2. Detección de servicios y posibles vulnerabilidades

```bash
nmap -p22,80 -sV -sC 10.201.25.37 -oN targeted
```

**Parámetros:**
- `-p22,80` : Escanea únicamente puertos específicos  
- `-sV` : Detecta versión de servicios  
- `-sC` : Ejecuta scripts básicos NSE  
- `-oN` : Guarda la salida en formato normal

```bash
cat targeted
```

<img width="1897" height="567" alt="Captura de pantalla 2025-11-17 205126" src="https://github.com/user-attachments/assets/01dfa4e5-75a8-42dd-b86c-aa863b1261ef" />


---

## 3. Enumeración del servicio web (HTTP - 80)

Ingresamos al sitio web y revisamos el código fuente.

<img width="1903" height="835" alt="Captura de pantalla 2025-11-17 205236" src="https://github.com/user-attachments/assets/cf9e2045-e12f-405e-b33b-b5f652e470b8" />



En un comentario HTML se revela información importante:

- Usuario identificado: **R1ckRul3s**

Al no encontrar más datos relevantes inicialmente, procedemos a un ataque de fuzzing.

---

## 4. Descubrimiento de rutas con Gobuster

```bash
gobuster dir -u 10.201.25.37 -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php,html
```
**Parámetros:**
- `dir` : Modo de descubrimiento de directorios  
- `-u` : URL objetivo 
- `-w` : Wordlist utilizada  
- `-x php,html` : Extensiones a probar

<img width="1900" height="868" alt="Captura de pantalla 2025-11-17 210503" src="https://github.com/user-attachments/assets/b2910bd8-d053-438a-8292-67a3a2869d63" />


Lo más interesante encontrado:

- `/login.php`
- `/portal.php`
- `/robots.txt`

---

## 5. Revisión de robots.txt

Dentro del archivo encontramos información útil.
Un texto que al parecer era la contraseña.
Con esto obtenemos las credenciales:

- Usuario: **R1ckRul3s**
- Contraseña: **Wubbalubbadubdub**

---

## 6. Acceso al panel y ejecución de comandos

Ingresamos al panel y probamos algunos comandos:

```bash
whoami
```

<img width="1169" height="223" alt="Captura de pantalla 2025-11-17 211001" src="https://github.com/user-attachments/assets/4ea3dee1-628f-4e17-9b10-a263851bf3bd" />

```bash
sudo -l
```

<img width="1164" height="305" alt="Captura de pantalla 2025-11-17 211020" src="https://github.com/user-attachments/assets/edb7cb7c-1f09-47ca-83c2-0c76c914f9c5" />


Resultado:

- Usuario actual: **www-data**
- Privilegios: **sudo sin restricciones** (acceso total al sistema)

---

## 7. Obtención de Flags

### Flag 1
Exploramos el directorio actual:

```bash
ls -la
```

<img width="1151" height="402" alt="Captura de pantalla 2025-11-17 211256" src="https://github.com/user-attachments/assets/ed71f9d2-afc0-4ab6-a537-cbc9b2481f32" />


Debido a restricciones de `cat`, usamos `less`:

```bash
less Sup3rS3cretPickl3Ingred.txt
```

<img width="1158" height="220" alt="Captura de pantalla 2025-11-17 211357" src="https://github.com/user-attachments/assets/1e60cd9e-53d2-4a24-a6e9-39c6ab717446" />


**Flag 1:** `mr. meeseek hair`

---

### Flag 2
Buscamos usuarios dentro de `/home`:

```bash
ls -al /home
```

Encontramos el usuario `rick`, entramos y localizamos la segunda flag:

<img width="1154" height="272" alt="Captura de pantalla 2025-11-17 211635" src="https://github.com/user-attachments/assets/7f2d8f47-5276-4e1f-9cc9-c8b4331747ec" />


```bash
less /home/rick/"second ingredients"
```

<img width="1158" height="215" alt="Captura de pantalla 2025-11-17 211600" src="https://github.com/user-attachments/assets/1d98dd71-620c-44fb-aaa6-57a5a5144691" />


**Flag 2:** `1 jerry tear`

---

### Flag 3
Gracias a los privilegios máximos podemos acceder al directorio root:

```bash
sudo ls -la /root/
```

<img width="1160" height="382" alt="Captura de pantalla 2025-11-17 211954" src="https://github.com/user-attachments/assets/606fde56-1da3-47a5-beaf-62a2585b9ea5" />


Leemos la tercera flag:

```bash
sudo less /root/3rd.txt
```

<img width="1148" height="211" alt="Captura de pantalla 2025-11-17 212106" src="https://github.com/user-attachments/assets/95e5c628-5797-4700-90bb-f2bd28b98afd" />


**Flag 3:** `fleeb juice`

---

## Lecciones aprendidas

- La enumeración es fundamental: comentarios HTML y robots.txt pueden contener credenciales.
- Incluso un panel web aparentemente básico puede permitir ejecución remota de comandos.
- Verificar `sudo -l` puede brindar acceso completo sin necesidad de escalar privilegios.
- La exploración sistemática de directorios (`/`, `/home`, `/root`) asegura no perder flags.
- Gobuster sigue siendo una herramienta esencial para descubrir rutas ocultas.
