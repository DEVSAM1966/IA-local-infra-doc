# Mejoras de la VM

Vamos a realizar mejoras en la VM en este capitulo.

## I - Subir la RAM de la VM a 8 GB

Con la **VM apagada** se accede al administrador de VM (en mi caso es virt-manager):

1. En virt-manager:  Clic derecho sobre la VM → **Open** (o **Detalles**).
2. En el panel izquierdo ir a **Memory** y cambiamos:
    - **Current allocation:**  ``8192 MB`` 
    - **Maximun allocation:**  ``8192 MB`` o más si hay margen.
    - Pulsar **Apply**

    Según la versión de virt-manager se debe pulsar al icono que es como una **bombilla** para acceder a las propiedades de la VM.

![Aumento RAM de la VM](assets/1-aumento-memoria-VM.jpg)


Con esto ya tenemos asignados 8 GB en la VM.


---


Si se prefiere vía comando desde la terminal del equipo local:

```bash
sudo virsh shutdown ubuntu24.04
sudo virsh setmem ubuntu24.04 8G --config
sudo virsh start ubuntu24.04
```
    Sustituye ubuntu24.04 por el nombre real de tu VM si es distinto.



## II - Activar Shared Memory de la VM

Este paso consiste en **activar la memoria compartida (Shared Memory)** en virt‑manager.
Esto mejora el rendimiento de la VM, especialmente cuando ejecutas IAs local (Ollama, LM Studio, etc.), porque reduce la latencia entre el equipo local y la VM.

Por el método gráfico **(virt-manager)** aplicamos los siguientes pasos:

1. Apagar la VM Ubuntu 24.04.
2. Abrir virt‑manager.
3. Selecciona tu VM → Open.
4. En el panel izquierdo, ve a Memory o bien pulsar → Icono **bombilla** y ve a Memory.
5. Activa la casilla:
    **✔ Enable shared memory**
6. Pulsa Apply.

![Activar memoria compartida en la VM](assets/2-activar-memoria-compartida.jpg)


---


Si se prefiere desde la terminal del equipo local lanzar:
```bash
sudo virsh edit ubuntu24.04
```

Buscar el bloque ``<memoryBacking>``y añadimos lo siguiente en formato XML:
```xml
<memoryBacking>
  <source type='memfd'/>
  <access mode='shared'/>
</memoryBacking>
```
**Guardar y cerrar.**

Las mejoras que obtenemos con este ajuste son:

- Menos latencia entre host ↔ VM
- Mejor rendimiento de modelos locales
- Menos overhead en operaciones de memoria
- Más estabilidad cuando cargas modelos grandes (4B–7B)



## III - Ajuste en las vCPUs y topología de CPU en la VM

VM ya usa ``host‑passthrough``, lo cual es excelente, pero vamos a asegurarnos de que la **topología de CPU** esté correctamente configurada para que QEMU/KVM entregue el máximo rendimiento.

- Asegurar que la VM use host‑passthrough (ya lo tenemos).
- Configurar 2 vCPUs con topología correcta:

    - 1 socket
    - 2 cores
    - 1 thread por core

- Activar **hypervisor=off** para reducir overhead.
- Activar cache **mode = passthrough**.

Esto mejora el rendimiento de modelos locales en Ollama.

Desde el modo gráfico con **virt-manager**:

1. Apagar la VM Ubuntu 24.04.
2. Abrir virt‑manager.
3. Selecciona tu VM → Open.
4. En el panel izquierdo o pulsando el icono **bombilla**, ve a CPU.

Ahora ajusta:

✔ CPU Model

- Selecciona: host‑passthrough  
    (Si ya está activado, perfecto.)

✔ Topología

Configura:

- Sockets: 1
- Cores: 2
- Threads: 1

✔ Opciones avanzadas

Haz clic en “Copy host CPU configuration” si aparece.

Luego activa:

- ✔ Enable host CPU features
- ✔ Disable hypervisor vendor id (esto es hypervisor=off)
- ✔ Cache mode: passthrough

5. Pulsa Apply.

![Ajuste de la CPU de la VM](assets/3-ajuste-CPU-VM.jpg)


--- 


Si se prefiere realizarlo via comandos desde la terminal del equipo local:
```bash
sudo virsh edit ubuntu24.04
```

Buscar el bloque ``<cpu>`` y asegurarse de que tenga:
```xml
<cpu mode='host-passthrough' check='none'>
  <topology sockets='1' cores='2' threads='1'/>
  <feature policy='require' name='avx2'/>
  <feature policy='require' name='fma'/>
</cpu>
```
**Guardar y cerrar**


## IV - Optimizar disco y caché de la VM

El objetivo de este cambio es:

- Usar VirtIO como controlador de disco (ya lo tienes).
- Activar cache = writeback para acelerar lectura/escritura.
- Activar discard (TRIM) para que el SSD del host mantenga rendimiento.
- Activar IO mode = native para reducir latencia.

Esto mejora muchísimo el rendimiento de Ollama y LM Studio.

Usando el método gráfico con **virt-manager**:

1. Apagar la VM Ubuntu 24.04.
2. Abrir virt‑manager.
3. Selecciona tu VM → Open.
4. En el panel izquierdo o pulsar el icono **bombilla**, ve a “SATA Disk” o “VirtIO Disk” (según tu configuración).
5. Ajusta los siguientes parámetros:

**✔ Bus / Device type**
Debe ser:

  - **VirtIO**  
    Si no lo es, cámbialo.

**✔ Cache mode**
Selecciona:

  - **writeback**
    Es el modo más rápido para IA local.

**✔ IO mode**
Selecciona:

  - **native**
    Reduce la latencia del disco.

**✔ unmap (TRIM)**
Activa:

  - **✔ Enable discard**
    Esto permite que el SSD del host mantenga velocidad y no se degrade.

**✔ Advanced options**
Si aparece:

  - **✔ Detect zeroes → activado**
  - **✔ Write zeroes → activado**

6. Pulsa Apply.

![Mejoras del disco de la VM](assets/4-mejoras-disco-VM.jpg)


---


Desde terminal en el equipo local si se prefiere hacerlo por aquí:
```bash
sudo virsh edit ubuntu24.04
```

Buscar el bloque ``<disk>`` y asegúrarse de que incluya:

```xml
<driver name='qemu' type='qcow2' cache='writeback' io='native'/>
<discard='unmap'/>
```
**Guardar y cerrar.**



## V - Optimizar red VirtIO de la VM

El objetivo de este cambio es conseguir:

- Usar el controlador VirtIO (ya lo tienes).
- Activar modo virtio-net con características avanzadas.
- Ajustar el modelo de dispositivo y el modo de caché de red.
- Habilitar multiqueue para aprovechar tus 2 vCPUs.

Empleando el método gráfico con **virt-manager**:

1. Apagar la VM Ubuntu 24.04.
2. Abrir virt‑manager.
3. Selecciona tu VM → Open.
4. En el panel izquierdo, haz clic en NIC :5b:7c:fe (tu interfaz de red).
5. Ajusta los siguientes parámetros:

**✔ Device model**
Selecciona:

  - **virtio**  
    (si ya está, perfecto)

**✔ Source mode**
Debe ser: **Bridge interface o NAT**

  - Si usas la VM para IA local y no necesitas acceso externo, NAT es suficiente.

  - Si quieres que la VM se comunique directamente con el equipo local (por ejemplo, para integrar Ollama con Cursor), usar **Bridge (es la elegida)**.  En este caso el nombre del dispositivo sera: **virbr0**

**✔ MAC address**
Déjala como está (no es necesario cambiarla).

**✔ Advanced options**  No aparece en la versión de virt-manager que tengo.

Despliega y activa:
  - **✔ Multiqueue** → selecciona **2 queues** (una por vCPU).
  - **✔ vhost acceleration** → mejora el rendimiento de red.
  - **✔ Checksum offload** → reduce carga de CPU.
  - **✔ Generic segmentation offload (GSO)** → mejora throughput.

Para aplicar esta parte debo entrar por via comandos desde la termina del equipo local:
```bash
sudo virsh edit ubuntu24.04
```

Buscar el bloque ``<interface>`` y añadir la linea ``<driver name='vhost' queues='2'/>``:
```xml
<interface type='bridge'>
  <mac address='52:54:00:5b:7c:fe'/>
  <source bridge='virbr0'/>
  <model type='virtio'/>
  <driver name='vhost' queues='2'/>
  <address type='pci' domain='0x0000' bus='0x01' slot='0x00' function='0x0'/>
</interface>
```
**Guardar y salir**

6. Pulsa Apply desde el virt-manager.

7. Arrancar la VM por ejemplo desde terminal del host:
```bash
sudo virsh start ubuntu24.04
```
Verificar que arranca bien.


---


Si se prefiere realizar todos los cambios desde la terminal del equipo local:
```bash
sudo virsh edit ubuntu24.04
```

Buscar el bloque ``<interface>`` y asegurarse de que tenga:
```xml
<interface type='bridge'>
  <mac address='52:54:00:5b:7c:fe'/>
  <source bridge='virbr0'/>
  <model type='virtio'/>
  <driver name='vhost' queues='2'/>
  <address type='pci' domain='0x0000' bus='0x01' slot='0x00' function='0x0'/>
</interface>

```


## VI Ajustes finales en red VirtIO de la VM

Arrancamos la VM y entramos en su consola, para realizar esta comprobación:

```bash
lsmod | grep vhost
```

No me aparecio ningún modulo vhost,  por lo que se tuvo que activarlo manualmente el modulo vhost_net:

```bash
sudo modprobe vhost_net
```

Podremos verificar que aparece modulos vhost_net activos al lanzar de nuevo la instrucción ``lsmod | grep vhost``.

Para que arranque el modulo vhost_net en cada inicio ejecutamos lo siguiente:

```bach
echo vhost_net | sudo tee -a /etc/modules
```

Con esto ya esta terminada el ajuste de la VM.


---

