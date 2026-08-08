# ⚙️ Estado inicial del sistema.

A nivel de equipo partimos con lo siguiente:

1. CPU (i3‑4xxx) soporta AVX2, lo cual es oro para IA local.
2. Disco SDD de 500 Gb.
3. 16 GB de RAM, el límite razonable es:

    - Modelos 2B–4B → perfectos
    - Modelos 7B → posibles con optimización y swap
    - Modelos 13B → no recomendables en VM

Los mejores modelos que nos propone cualquier IA es:

    - Qwen 2.5 3B → rápido, muy bueno en razonamiento
    - Phi‑3 Mini → extremadamente eficiente
    - Mistral 7B Instruct → si quieres algo más grande
    - Gemma 2 2B → muy ligero y sorprendentemente bueno
    - DeepSeek‑R1‑7B → ligero, ideal para programación y contextos largos


También hay que mencionar que para separar la IA local por tema de seguridad se opta por crear previamente una máquina virtual en el equipo local (S. O. Lubuntu) con esta configuración inicial de VM bajo QEMU/KVM:

    - **2 vCPUS (host‑passthrough activado)** - Perfecto: aprovecha las instrucciones AVX2 del i3‑4ª gen.
    - **6 GB de RAM** - Correcta para modelos pequeños (2B–4B).
    - **70 GB(VirtIO) de Disco virtual** - Excelente rendimiento; VirtIO es el bus más rápido.
    - **Firmware BIOS (chipset Q35)** - Compatible con Ubuntu 24.04 y Ollama.
    - S.O. Ubuntu 24.04

---

