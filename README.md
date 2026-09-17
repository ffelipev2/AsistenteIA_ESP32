# Asistente IA para ESP32-S3

Asistente de voz en español para una **ESP32-S3 SuperMini**. Combina firmware XiaoZhi adaptado para un micrófono PDM LMD2718 y un amplificador I2S NS4168 con un servidor XiaoZhi local para reconocimiento de voz, modelo de lenguaje y síntesis de voz.

La interacción prevista es:

```text
Alexa → ESP32-S3 → WebSocket → Vosk / Ollama / Edge TTS → parlante
```

## Características

- Activación local con la palabra `Alexa` mediante WakeNet.
- Reconocimiento de voz en español con Vosk.
- Respuestas generadas localmente con Ollama.
- Voz en español mediante Edge TTS.
- Comunicación por Wi-Fi y WebSocket con el servidor local.
- Aviso sonoro cuando el dispositivo ya está listo para escuchar la consulta.
- Sin pantalla, botones, LED, cámara ni codec I2C: la placa usa las abstracciones `NoDisplay` y `NoLed` de XiaoZhi.

## Hardware

La variante validada es `esp32-s3-supermini-pdm-f4r2`: ESP32-S3 QFN56, 4 MB de flash y 2 MB de PSRAM.

| Componente | Señal | GPIO |
| --- | --- | ---: |
| Micrófono LMD2718 | I2S0 PDM CLK | 7 |
| Micrófono LMD2718 | I2S0 PDM DATA | 8 |
| Amplificador NS4168 | I2S1 BCLK | 4 |
| Amplificador NS4168 | I2S1 LRCLK / WS | 5 |
| Amplificador NS4168 | I2S1 DATA / DOUT | 6 |

El micrófono funciona a 16 kHz, mono y 16 bits. La respuesta de audio sale a 24 kHz, 16 bits, I2S estéreo; el firmware duplica la señal mono en ambos canales para el NS4168.

## Estructura

| Directorio | Contenido |
| --- | --- |
| `xiaozhi-esp32/` | Firmware ESP-IDF/C++ basado en XiaoZhi, incluida la board `esp32-s3-supermini-pdm`. |
| `xiaozhi-esp32-server/` | Servidor XiaoZhi basado en Python, Java y Vue. |

Ambos directorios son repositorios Git independientes. El repositorio raíz guarda referencias a ellos, no una copia completa de sus archivos. Para reproducir el entorno desde cero, obtén también los dos proyectos y sitúalos en esas rutas:

```powershell
git clone https://github.com/ffelipev2/AsistenteIA_ESP32.git
Set-Location AsistenteIA_ESP32
git clone https://github.com/78/xiaozhi-esp32.git xiaozhi-esp32
git clone https://github.com/xinnan-tech/xiaozhi-esp32-server.git xiaozhi-esp32-server
```

Después aplica o publica las personalizaciones de este proyecto en los repositorios correspondientes antes de compilar. La documentación detallada del hardware está en [xiaozhi-esp32/README_SUPERMINI_PDM.md](xiaozhi-esp32/README_SUPERMINI_PDM.md).

## Requisitos

- ESP32-S3 SuperMini, LMD2718, NS4168, parlante y cable USB de datos.
- Red Wi-Fi de 2,4 GHz para la ESP32 y el equipo del servidor.
- ESP-IDF 6.1 (mínimo 6.0.1; ESP-IDF 5.x no es compatible).
- Python 3.10 para el servidor.
- [Ollama](https://ollama.com/) con un modelo compatible, por ejemplo `qwen2.5:7b`.
- Internet si se utiliza Edge TTS.

## Compilar el firmware

Desde `xiaozhi-esp32/`, inicializa el entorno de ESP-IDF y compila la variante de la placa:

```powershell
python scripts/build.py esp32-s3-supermini-pdm `
  --name esp32-s3-supermini-pdm-f4r2 `
  --language es-ES
```

El resultado se genera en `xiaozhi-esp32/build/xiaozhi.bin`. Para módulos de memoria distintos están disponibles las variantes `esp32-s3-supermini-pdm-n8` y `esp32-s3-supermini-pdm-n16r8`; confirma primero la memoria real del módulo.

La guía de la board contiene los comandos de flasheo, monitor serial, configuración de ESP-IDF y diagnóstico de audio.

## Configurar e iniciar el servidor

En `xiaozhi-esp32-server/main/xiaozhi-server/`:

1. Crea un entorno con Python 3.10 e instala las dependencias:

   ```powershell
   python -m venv .venv
   .\.venv\Scripts\Activate.ps1
   pip install -r requirements.txt
   ```

2. Descarga o instala el modelo Vosk en español configurado por el servidor.
3. Instala y ejecuta el modelo de Ollama:

   ```powershell
   ollama run qwen2.5:7b
   ```

4. Ajusta la configuración activa para usar Vosk, Ollama, Edge TTS en español y las direcciones de tu red local. La URL OTA configurada para la board debe apuntar al equipo que ejecuta el servidor.
5. Inicia el servicio:

   ```powershell
   python app.py
   ```

Consulta el README del servidor y sus documentos de despliegue para las opciones Docker y de instalación completa.

## Uso y comprobación

1. Flashea el firmware y configura las credenciales Wi-Fi.
2. Inicia el servidor y verifica que la ESP32 y el equipo estén en la misma red.
3. Enciende la placa y espera a que se conecte.
4. Di `Alexa`, espera el sonido de confirmación y formula una pregunta en español.

En el monitor serie, el flujo esperado es similar a:

```text
Wake word detected: Alexa
State: idle -> connecting
State: connecting -> listening
State: listening -> speaking
```

Si se detecta la palabra de activación pero no se llega a `listening`, revisa Wi-Fi, la URL OTA y WebSocket. Si se llega a `listening` pero no se oye el aviso, revisa la alimentación, masa común y el cableado del NS4168. Si hay aviso pero no hay respuesta, revisa los logs de Vosk, Ollama y TTS en el servidor.

## Créditos y licencias

Este proyecto se apoya en [xiaozhi-esp32](https://github.com/78/xiaozhi-esp32) y [xiaozhi-esp32-server](https://github.com/xinnan-tech/xiaozhi-esp32-server). Revisa las licencias y la documentación de cada componente antes de distribuir o desplegar la solución.
