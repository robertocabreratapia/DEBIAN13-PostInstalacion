# Debian 13 – Script de postinstalación automática (GNOME)

**IMPORTANTE:**

-   **Este script no instala Debian. Se ejecuta después de haber instalado Debian 13 con entorno gráfico GNOME.**

-   **Se recomienda utilizarlo solo en instalaciones limpias, inmediatamente después de instalar Debian 13**

## Por qué este repositorio

Cada vez que instalaba Debian acababa perdiendo mucho tiempo repitiendo los mismos pasos. Normalmente seguía algún vídeo de YouTube (como [este,](https://www.youtube.com/watch?v=KQTB-ycBeB8)de Soplos Linux, que es uno de los mejores, aunque el de Debian 12, [era mucho más claro](https://www.youtube.com/watch?v=oPf_0h8mG2Q)) y, aun así, el resultado final nunca quedaba exactamente igual. Con el tiempo me di cuenta de que el problema no era Debian, sino la falta de un proceso reproducible.

En respuesta a este problema, este repositorio nace con dos objetivos muy concretos:

-   Reducir al mínimo el tiempo dedicado a la postinstalación de Debian.
-   Asegurar que la base del sistema sea siempre la misma en todas mis instalaciones.

Así, este repositorio contiene un script prepara Debian 13 automáticamente tras una instalación limpia, sin necesidad de conocimientos previos de Linux. Está pensado para:

-   Personas que vienen de Windows o macOS.
-   Usuarios que reinstalan con frecuencia.
-   Uso académico, profesional o personal.
-   Quienes quieren un sistema GNOME listo para usar con un solo comando.

## Qué es Debian

Debian es un sistema operativo libre, estable y muy fiable. Es similar a Ubuntu, pero más conservador y predecible, lo que lo hace habitual en entornos académicos y profesionales.

## Qué hace este script

Tras una instalación limpia de Debian 13 con GNOME, el script realiza las siguientes tareas.

### Sistema

-   Actualiza el sistema completo.
-   Configura el menú de arranque (GRUB):
    -   Tiempo de espera en pantalla de inicio: 10 segundos.
    -   Resolución de pantalla: 2560 × 1080.
    -   Detección de otros sistemas operativos (por ejemplo Windows).
    -   Arranque limpio y silencioso (`quiet splash`).

### Usuario y permisos

-   Ajusta el sistema para que, al escribir la contraseña con permisos de administrador, exista feedback visual.
-   No modifica el comportamiento normal de GNOME.

### Programas instalados

-   Herramientas básicas del sistema.
-   Soporte para discos USB, NTFS, exFAT y HFS+.
-   Reproductores multimedia (VLC, Clementine).
-   Soporte completo de audio y vídeo.
-   Flatpak y Flathub.
-   Thunderbird como cliente de correo electrónico.
-   R (lenguaje estadístico) desde el repositorio oficial CRAN.
-   RStudio Desktop (opcional, con verificación de integridad mediante SHA256).

### Escritorio GNOME (configuración automática)

Al iniciar sesión por primera vez, el sistema ajusta automáticamente GNOME:

-   Barra inferior con programas más usados, o dock inferior (Dash to Dock):
    -   Iconos pequeños.
    -   Apariencia integrada con el tema del sistema.
-   Menús legibles y sin transparencias molestas.
-   Monitorización del sistema (CPU, memoria, red).
-   Iconos pequeños en el escritorio.
-   Botones de ventana: minimizar, maximizar y cerrar.
-   Tipografías Ubuntu (incluida Ubuntu Mono).

Estos ajustes se aplican una sola vez y luego el sistema vuelve a su funcionamiento normal.

## Qué es sudo

sudo es un comando que permite realizar cambios importantes en el sistema introduciendo la contraseña del usuario.

-   Es normal.\
-   Es seguro.\
-   Debian lo utiliza para evitar errores accidentales.

Cuando el script lo solicite, escribe tu contraseña y pulsa Enter.\
No se mostrarán letras ni símbolos en pantalla. Esto es el comportamiento esperado.

## Requisitos

-   Debian 13 recién instalado.
-   Entorno gráfico GNOME.
-   Conexión a Internet.
-   Usuario con permisos de administrador (sudo).

El script funciona tanto en Wayland como en X11.

## Cómo usarlo

### Descargar el script

En la terminal de linux, copia y pega:

``` bash
wget https://raw.githubusercontent.com/robertocabreratapia/DEBIAN13_Postinstalacion/main/postinstalacion-debian13.sh
```

### Dar permisos de ejecución

En la terminal de linux, copia y pega:

``` bash
sudo bash postinstalacion-debian13.sh
```

En la terminal, introduce tu contraseña cuando se te pida.

### Paso final

Cuando el script termine: 1. Cierra sesión. 2. Vuelve a iniciar sesión en GNOME.

El escritorio se ajustará automáticamente en ese primer inicio.

## Opciones

Si no deseas instalar RStudio, ejecuta:

``` bash
INSTALL_RSTUDIO=0 sudo bash postinstalacion-debian13.sh
```

El resto del script se ejecutará con normalidad.

## Qué modifica el script

Para total transparencia, el script modifica o configura:

-   /etc/apt/sources.list

-   /etc/default/grub

-   /etc/sudoers

-   Instalación de software y extensiones GNOME oficiales

## Es seguro

-   Usa repositorios oficiales.\
-   Verifica RStudio con comprobación criptográfica.\
-   No elimina datos personales.\
-   No altera la lógica ni el comportamiento base de GNOME.

## Reconocimiento

Este proyecto fue desarrollado con el apoyo de ChatGPT (OpenAI) como herramienta de asistencia para la programación y la documentación.
