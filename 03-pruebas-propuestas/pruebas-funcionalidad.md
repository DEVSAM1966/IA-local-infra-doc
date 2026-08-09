# Pruebas de funcionalidad de la IA local

Ahora vamos a comprobar que tal va el modelo instalado en la VM desde el equipo local que tiene instalado OpenCode. Todo estos pasos se reaizan en el equipo local.



## I - Visibilidad de Ollama en el equipo local

Si configuramos en el anterior capitulo el parámetro  ``Environment="OLLAMA_HOST=0.0.0.0"`` dentro del fichero ``/etc/systemd/system/ollama.service``, será visible desde el equipo local via HTTP.

Para verificarlo abrir un navegador de internet y lanzamos la URL: http://192.168.122.218:11434/api/tags y debera de devolver algo parecido a la imagen.

![Respuesta de Ollama desde la VM](/assets/9-respuesta-via-HTTP.jpg)

Esto significa que el Ollama y el modelo de inteligencia son accesibles.



## II - Conectar OpenCode con Ollama/DeepSeek

Ejecutamos desde una terminal (recordemos que mi equipo tiene Lubuntu).
```bash
opencode
```

Una vez abierto OpenCode vía terminal, en su chat lanzamos este comando:
```opencode
opencode ai set-provider ollama
```

Se puso a pensar y después en mi caso sugerio que modificase manualmente el fichero ``~/.config/opencode/opencode.jsonc``.  Desde otra terminal, se lanza:
```bash
nano ~/.config/opencode/opencode.jsonc
```

Nos mostrará que tiene las siguientes entrada:
```bash
{
  "$schema": "https://opencode.ai/config.json",
}
```

Deberemos dejar el fichero de esta manera:
```bash
{
  "$schema": "https://opencode.ai/config.json",
  "model": "ollama/deepseek-r1:7b",
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (local)",
      "options": {
        "baseURL": "http://192.168.122.218:11434/v1"
      },
      "models": {
        "deepseek-r1:7b": {
          "name": "DeepSeek R1 7B"
        }
      }
    }
  }
}
```
Gravamos y salimos.

Nos salimos de OpenCode y volvemos acceder.  Veremos que el modelo de la VM esta cargado.

![Modelo cargado en OpenCode](/assets/10-opencode.jpg)



## III - Pruebas

Se le pregunto: ``Que modelo de IA eres.``

Tardo en contestar unos 17 minutos, ver el siguiente gráfico.

![Respuesta de DeepSeek a traves de OpenCode](/assets/11-Prueba-funcionalidad-IA.jpg)

Como podemos observar, penaliza trabajar con OpenCode con una IA local aislada en una VM, donde pone en evidencia las limitaciones de CPU del equipo.


Trabajar directamente con Ollama dentro de la VM es más rápido. La misma pregunta tardo 1 minuto en responderla.  Ver el siguiente gráfico.

![Respuesta de DeepSeek a traves de Ollama](/assets/12-Prueba-funcionalidad-Ollama.jpg)


---



