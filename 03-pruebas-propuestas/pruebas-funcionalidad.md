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



## IV - Resumen técnico del problema de latencia Equipo local ↔ VM con Ollama

La diferencia entre el tiempo de respuesta dentro de la VM (~1:37) y desde el equipo local (~17 minutos) no se debe al modelo DeepSeek‑R1‑7B, sino al canal de comunicación entre el equipo local y la máquina virtual. 

Cuando el equipo local consulta la API de Ollama en la VM, la respuesta se transmite mediante streaming de tokens, un flujo continuo de fragmentos pequeños. 

En este entorno, **la red en modo bridge introduce latencia adicional y OpenCode no procesa el streaming en tiempo real**, sino que espera a recibir la respuesta completa antes de mostrarla. 

Esto provoca que el buffer de la conexión se llene y que el equipo local permanezca en espera mucho más tiempo del que tarda realmente el modelo en generar la respuesta. En consecuencia, el procesamiento del modelo es rápido, pero la entrega de la respuesta al equipo es lenta debido a la combinación de latencia de red, ausencia de streaming efectivo y el comportamiento del cliente HTTP de OpenCode.

**Diagrama técnico del problema de latencia.**

```bash
                ┌──────────────────────────────┐
                │     Equipo local (OpenCode)  │
                └───────────────┬──────────────┘
                                │
                                │ 1. Solicitud HTTP a la VM
                                ▼
                ┌──────────────────────────────┐
                │   Red Virtual (Bridge Mode)  │
                └───────────────┬──────────────┘
                                │
                                │ 2. Latencia añadida por la red
                                ▼
                ┌──────────────────────────────┐
                │        VM con Ollama         │
                └───────────────┬──────────────┘
                                │
                                │ 3. DeepSeek‑R1‑7B procesa la petición
                                │    (≈ 1 min 37 s)
                                ▼
                ┌──────────────────────────────┐
                │  Ollama genera tokens (stream)│
                └───────────────┬──────────────┘
                                │
                                │ 4. Streaming → muchos fragmentos pequeños
                                ▼
                ┌──────────────────────────────┐
                │   Red Virtual (Bridge Mode)  │
                └───────────────┬──────────────┘
                                │
                                │ 5. El buffer se llena por latencia + streaming
                                ▼
                ┌──────────────────────────────┐
                │     Equipo local (OpenCode)  │
                └───────────────┬──────────────┘
                                │
                                │ 6. OpenCode NO muestra tokens en streaming
                                │    → espera la respuesta completa
                                ▼
                ┌──────────────────────────────┐
                │   Tiempo total: ~17 minutos  │
                └──────────────────────────────┘
```



## V - Solución técnica al problema de latencia

### 1. Cambiar la tipologia de la red de la VM

El modo Bridge introduce latencia, congestión y buffering cuando se transmiten cientos de tokens en streaming.

La solución es usar:


- **Host‑Only** → comunicación directa y rápida entre Equipo local ↔ VM

- **NAT** → acceso a Internet desde la VM sin afectar la comunicación interna

Esto reduce la latencia de red de milisegundos a microsegundos.

Los pasos a seguir usando **virt-manager (libvirt/KVM)** son:


####🧩 1.1. Estado actual (VM apagada)

- virbr0 (NAT) → ya te da acceso a Internet desde la VM.
- virtio → correcto, es el modelo más eficiente.
- IP desconocida → normal, porque la VM aún no está arrancada o el DHCP no ha asignado dirección.

👉 No elimines esta interfaz; la mantendremos como **Adaptador 1**.


#### 🧩 1.2. Crear la red Host‑Only (si no aparece)

En el Equipo local:
```bash
sudo virsh net-list --all
```

Si no ves ``hostonly``, créala:
```bash
sudo nano /tmp/hostonly.xml

```

con este contenido:
```xml
<network>
  <name>hostonly</name>
  <bridge name='virbr1' stp='on' delay='0'/>
  <ip address='192.168.56.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.56.100' end='192.168.56.200'/>
    </dhcp>
  </ip>
</network>
```

Luego activarla y autoinicia:
```bash
sudo virsh net-define /tmp/hostonly.xml
sudo virsh net-start hostonly
sudo virsh net-autostart hostonly
```

![Pantalla con los comando creacion de red hostonly para la VM](/assets/1-1-definir-activar-red-VM.jpg)

Comprobar que aparece:
```bash
sudo virsh net-list --all
```

![Verificación de la existencia de hostonly](/assets/1-2-hostonly-activa-VM.jpg)


#### 🧩 1.3. Añadir la red Host‑Only

1. Apaga la VM (es importante hacerlo con la VM apagada).

2. En virt‑manager, abre Detalles → Añadir hardware → Red.

3. En “Fuente de red”, selecciona hostonly **(si no existe, la creamos en el  - punto 1.2 y luego volver aqui)**.

4. Modelo de dispositivo: virtio.

![Configurar nuevo adaptador networking en VM](/assets/1-3-Añadir-red-hostonly-VM.jpg)

5. Guardar

6. Guarda los cambios.

Tendremos dos tarjetas de red.

- Adaptador 1 → NAT (virbr0)
- Adaptador 2 → Host‑Only (virbr1)


#### 🧩 1.4 — Arrancar la VM y obtener la IP Host‑Only

Dentro de la VM:
```bash
ip a
```

Verás dos interfaces:

- enp1s0 → activa, con IP 192.168.122.218 (red NAT virbr0)
- enp7s0 → inactiva, sin IP **(la Host‑Only que acabamos de añadir)**

![Verificación que tiene IP la host-only de la VM](/assets/1-4-Verificar-hostonly-activa-VM.jpg)


Eso significa que la red Host‑Only está creada, pero la VM aún no la está usando porque el DHCP de virbr1 no le ha asignado dirección.

En la VM se ejecuta la instalación del cliente dhcp:
```bash
sudo apt update
sudo apt install isc-dhcp-client -y
```

Ahora activamos la enp7s0:
```bash
sudo ip link set enp7s0 up
sudo dhclient enp7s0
```

Volvemos a ejecutar:
```bash
ip a
```

![Verificación por segunda vez de la IP enp7s0](/assets/1-5-Verificar-hostonly-activa-VM.jpg)

Observamos que el DHCP de host-only asigno la IP 192.168.56.126/24 y esta dentro del rango  192.168.56.100 – 192.168.200.

Pero cada vez que arranquemos la VM nos asignara una nueva IP de manera dinamica y esto afecterá a la conectividad desde OpenCode.  Para solucionarlo deberemos fijar una IP dentro de la VM por ejemplo la 192.168.56.101 mediante.
```bash
sudo nano /etc/netplan/01-network-manager-all.yaml
```

Añadir este bloque (o modificar el existente):
```bash
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enp7s0:
      addresses:
        - 192.168.56.101/24
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```
Atención con los espacio, es un fichero yaml.

Deberemos instalar también NetworkManager mediante:
```bash
sudo apt update
sudo apt install network-manager -y
sudo systemctl enable NetworkManager
sudo systemctl start NetworkManager
```

Procedemos apagar y encender la VM y se consigue fijar la IP 192.168.56.101.

Desde el equipo local en un navegador de internet tendremos acceso al link: http://192.168.56.101:11434/api/tags

![Verificación de acceso a Ollama desde la IP 192.168.56.101](/assets/1-6-Verificar-hostonly-activa-host.jpg)


### 2. Activar streaming real en Ollama

Ollama ya soporta streaming, pero OpenCode no lo usa por defecto.
Se debe activar el endpoint correcto:

En tu ``opencode.jsonc``:
```bash
nano ~/.config/opencode/opencode.jsonc
```
Ahora editar esta parte y lo dejamos asi (usamos la IP fija):
```xml
"options": {
  "baseURL": "http://192.168.56.101:11434"
}
```

⚠️ **Sin /v1**, porque ese endpoint bloquea el streaming.  Debe quedar asi:
```xml
{
  "$schema": "https://opencode.ai/config.json",
  "model": "ollama/deepseek-r1:7b",
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (local)",
      "options": {
        "baseURL": "http://192.168.56.101:11434"
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

Esto permite que OpenCode reciba tokens en tiempo real.  Verificamos que esta bien accediendo al link:  http://192.168.56.101:11434/api/tags

Para ver como genera y envia tokens desde el equipo local lanzar en la terminal:
```bash
curl -N -X POST http://192.168.56.101:11434/api/generate \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-r1:7b",
    "prompt": "Cuenta hasta diez, una palabra por línea."
  }'
```

Se vera una repuesta fragmentada line a linea y devuelve un JSON inmenso pero rapido en respuesta, teniendo en cuenta el hardware del equipo.


### 3. Reducir el tamaño del contexto del modelo

En Ollama:
```bash
ollama run deepseek-r1:7b --num_ctx 2048
```

Esto reduce:

- tokens generados
- tamaño del streaming
- presión sobre el buffer de red

**Pero no funciona en el modelo DeepSeek, en cambio en modelo como LLaMA, Mistral, Phi si.  DeepSeek tiene otro sistema de configuración que no esta disponible.**  Dejamos esta parte.



### 4. Asignar más CPU a la VM (si es posible)

Tu Equipo local tiene:

- Intel Core i3 4ª gen
- 16 GB RAM
- SSD

La VM tiene:

- 2 vCPUs
- 8 GB RAM

Si puedes subir a 3 vCPUs con la VM parada y dejando 1 hilo, el tiempo de generación baja un 20–30%.

![Aumento de la CPU de la VM](/assets/1-7-Aumento-CPU-VM.jpg)


### 🧠 Resumen técnico de la solución

La latencia extrema se debe a que OpenCode recibe la respuesta completa en un único bloque debido al uso del endpoint /v1 y la red en modo bridge.

Al cambiar la red a Host‑Only + NAT y usar el endpoint nativo de Ollama sin /v1, OpenCode recibe tokens en streaming real, eliminando el buffering y reduciendo el tiempo de entrega de 17 minutos a pocos segundos.

**En la practica no funciono ya que el propio Opencode no envia de manera correcta tokens en streaming para Ollama.  A vesar de usar la ultima versión de OpenCode v1.18.16 no funciono.**

**Por lo que en la practica es mejor juntos en la misma máquina Ollama y OpenCode, hasta que los chicos de OpenCode saquen una versión que solucione este problema**

Recordemos que funciona si en la VM esta configurada la tarjeta de red virtual como Bridge, pero siendo conscientes de la latencia de respuesta y aqui la importania de tener una maquina potente.


---



