# Instalación de Ollama

En este capitulo se procede a instalar Ollama que es una plataforma Open Source que permite ejecutar modelos de inteligencia artificial directamente en un ordenador, de forma local y privada, sin depender de la nube.

Los pasos a seguir son:


## I - Actualizar la VM

Arrancada la VM tenemos que actualizar el sistema mediante:

```bash
sudo apt update && sudo apt upgrade -y
```

Con esto nos aseguramos que Ollama se instale sin dependencias rotas.


## II - Instalar Ollama

Se realiza ejecutando el siguiente comando:
```bash
curl -fsSL https://ollama.com/install.sh | sh
```

![Lanzado instalación Ollama](/assets/5-descarga-ollama.jpg)

![Finalización instalación Ollama](/assets/6-finalizacion-instalacion-Ollama.jpg)


Esto te deja:

- Servicio ollama activo

- API local en http://localhost:11434

- Carpeta de modelos en ``/usr/share/ollama`` o ``/var/lib/ollama``


## III - Verificar que Ollama funciona

Ejecutamos:
```bash
ollama --version
```

Nos devuelve la version Ollama instalada, en este caso ``ollama version in 0.32.6``.

Ahora comprobamos que no tiene modelos cargados con:
```bash
ollama list
```

Mostrará una lista vacía, sin modelos de Inteligencia artificial.


En el siguiente capitulo se procederá a **instalar DeepSeek-R1-7B**.



---

