# Instalación de una IA local

Este proyecto recopila los pasos realizados para implementar la instalación de una IA local en un ordenador de configuración hardware limitado y es una guia/ejemplo de como llevarlo a cabo.

---

## 📑 Índice General

### 01 - Situación de partida.

- [Estado inicial del equipo local](01-situacion/estado-inicial.md)


### 02 - Instalación y ajustes.

- [Ajustes en la Virtual Machine](01-situacion/mejoras-iniciales.md)

- [Instalación Ollama](/02-instalacion/Ollama.md)

- [Instalación de DeepSeek](/02-instalacion/DeepSeek-R1-7B.md)


### 03 - Pruebas y propuestas de mejoras.

- [Pruebas de funcionalidad](/03-pruebas-propuestas/pruebas-funcionalidad.md)

---

## Conclusión

Con esta implantación de IA local (exigente para el equipo), se demuestra que podemos trabajar directamente desde Ollama en la VM de manera profesional con equipos modestos en hardware, donde no es necesario tener tarjetas gráficas dedicadas, la CPU más potente o una cantidad monstruosa de RAM. Su rendimiento fue medio aceptable.

Juega un papel importante los sistemas operativos **Lubuntu / Ubuntu** que fue capaz de gestionar el  hardware disponible y permitio una configuración fina de lo que tenemos.

Donde se encontro con problemas de rendimiento es al integra OpenCode con la IA local debido a la arquitectura empleada de tener aislada la IA en una VM.  Aqui nos falto una CPU con más nucleos para tener un rendimiento decente.


---

🖖 **“Cuando eliminas lo imposible, lo que queda, por improbable que parezca, debe ser la verdad.”**



