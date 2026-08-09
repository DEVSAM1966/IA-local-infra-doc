# Instalación del modelo de Inteligencia artificial

Se procede a instalar el modelo ``DeepSeek-R1-7B``que cuenta con las siguientes caracteristicas:

- Razonamiento matemático y resolución de problemas paso a paso.
- Procesamiento de lenguaje natural (PNL).
- Asistentes de código y QA.
- Pipelines RAG y contextos largos.
- Aplicaciones empresariales y productividad.
- Eficiente en el uso del Hardware.
- Compatibilidad multilingüe.


## I - Instalación del modelo.

Se instala con:
```bash
ollama pull deepseek-r1:7b
```

![Instalación de DeepSeek-R1-7B](/assets/7-instalacion-DeepSeek-R1-7B.jpg)


Descarga el modelo de un tamaño entre 4 a 8 GB.


## II - Probar el modelo

Lanzamos el modelo, ejecutando:
```bash
ollama run deepseek-r1:7b
```

Y luego escribiremos algo así:
```bash
Hola, ¿puedes generar un ejemplo de API REST en Node.js con Express?
```

![Contestación a la peticion del usuario](/assets/8-respuesta-DeepSeek.jpg)

Contesta relativamente rápido a pesar de correr en un equipo tan ajustado.


## III - Configurar el acceso a OpenCode

En mi caso tengo instalado OpenCode en el equipo local y aprovechare que la VM es visible desde el equipo (VM tiene red NAT con ``virbr0``) para conectar la IA local que esta en la VM vía HTTP a OpenCode.


**1. Ver la IP de la VM**

En la VM ejecutar:
```bash
ip a
```

Deberemos buscar algo como ``192.168.122.X``.   Para mi caso es ``192.168.122.218``.


**2. En OpenCode (equipo local)**

Configura la IA local con:

   - URL: http://192.168.122.218:11434

   - Modelo: deepseek-r1:7b

Y ya puedes usar la IA local desde el equipo local.


## IV - Optimización final

Para que Ollama use roda la RAM disponible. editaremos:
```bash
sudo nano /etc/systemd/system/ollama.service
```

Añadir dentro de ``[Service]`` las siguientes lineas:
```xml
Environment="OLLAMA_HOST=0.0.0.0"
Environment="OLLAMA_NUM_THREADS=2"
Environment="OLLAMA_MAX_LOADED_MODELS=1"
```

- ``Environment="OLLAMA_HOST=0.0.0.0"`` Esto hace visible el ollama y su modelo de inteligencia artificial a traves de HTTP en el equipo local.
- ``Environment="OLLAMA_NUM_THREADS=2" y ``Environment="OLLAMA_MAX_LOADED_MODELS=1"`` evita que cargue varios modelos a la vez y saturar la RAM.

Luego recargar:
```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama
```



