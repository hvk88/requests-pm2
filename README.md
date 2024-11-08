Para automatizar este proceso en Python, puedes usar un bucle que haga las peticiones a intervalos regulares. La biblioteca **`time`** permite configurar pausas entre cada chequeo, y al ejecutarlo con **pm2** podrás mantener el proceso activo sin intervención. A continuación, te muestro cómo podrías estructurarlo:

### Ejemplo de Código

```python
import requests
import os
import time

# URLs y configuraciones
version_url = "https://example.com/api/latest_version"
download_url = "https://example.com/files/program_v{}.pdf"
MINIMUM_VERSION = 30
DOWNLOAD_DIR = "descargas"  # Directorio donde se guardarán los archivos

# Crea el directorio de descargas si no existe
os.makedirs(DOWNLOAD_DIR, exist_ok=True)

def check_and_download():
    try:
        # Realiza la solicitud para obtener la versión actual en el servidor
        response = requests.get(version_url)
        response.raise_for_status()
        latest_version = int(response.json().get("version"))

        # Verifica si la nueva versión es mayor a la mínima requerida
        if latest_version > MINIMUM_VERSION:
            print(f"Nueva versión encontrada: {latest_version}. Descargando...")

            # Construye la URL de descarga y realiza la solicitud
            download_link = download_url.format(latest_version)
            file_response = requests.get(download_link, stream=True)
            file_response.raise_for_status()

            # Guarda el archivo descargado en el directorio de descargas
            file_name = f"program_v{latest_version}.pdf"
            file_path = os.path.join(DOWNLOAD_DIR, file_name)
            with open(file_path, "wb") as file:
                for chunk in file_response.iter_content(chunk_size=8192):
                    file.write(chunk)

            print(f"Descarga completa. Archivo guardado en {file_path}")
        else:
            print("No hay una nueva versión disponible.")
    except requests.RequestException as e:
        print(f"Error en la solicitud: {e}")
    except ValueError:
        print("Error al interpretar la versión obtenida.")

# Ejecución periódica cada 6 horas
INTERVALO = 6 * 60 * 60  # En segundos (6 horas)

while True:
    check_and_download()
    print(f"Esperando {INTERVALO / 3600} horas para el próximo chequeo...")
    time.sleep(INTERVALO)
```

### Explicación:

1. **Función `check_and_download`**: Hace la petición para verificar la versión actual y descarga el archivo si es superior a la mínima.
2. **Directorio de Descargas**: Crea el directorio `"descargas"` si no existe para guardar los PDFs.
3. **Bucle Infinito con Intervalo**: El `while True` ejecuta `check_and_download()` y espera 6 horas (configurable) antes de la siguiente verificación.
4. **Ejecución con pm2**: Para iniciar este script con **pm2**, ejecuta:
   ```bash
   pm2 start script.py --name "version-checker"
   ```
   Esto mantendrá el script en ejecución continua en el VPS, haciendo las verificaciones periódicas.
