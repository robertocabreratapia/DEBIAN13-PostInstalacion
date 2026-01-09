# Debian 13 (GNOME) – Postinstalación automatizada

**IMPORTANTE**

- Este repositorio **no instala Debian**.
- Los scripts se ejecutan **después** de una instalación limpia de **Debian 13 con entorno gráfico GNOME**.
- Se recomienda utilizarlos **inmediatamente tras instalar Debian 13**, antes de personalizar el sistema manualmente.
- El contenido del repositorio está en formato **Markdown**, pensado para **copiar, pegar y ejecutar directamente en la terminal**.

---

## Por qué existe este repositorio

Cada vez que instalaba Debian acababa perdiendo mucho tiempo repitiendo siempre los mismos pasos. Normalmente seguía algún vídeo de YouTube y, aun así, el resultado final nunca quedaba exactamente igual.

Con el tiempo me di cuenta de que el problema no era Debian, sino la falta de un proceso reproducible.

Este repositorio nace con dos objetivos muy concretos:

- Reducir al mínimo el tiempo dedicado a la postinstalación.
- Asegurar que la base del sistema sea siempre la misma en todas mis instalaciones.

La idea es sencilla: tras una instalación limpia de Debian, ejecutar siempre los mismos pasos, en el mismo orden, y obtener siempre el mismo punto de partida.

---

## Estructura del repositorio

El repositorio contiene **dos scripts**, separados por intención y alcance:

### 1. `Script post instalacion Debian 13.md`

Script principal de **postinstalación del sistema**.  
Prepara Debian 13 para un uso general y deja GNOME configurado de forma coherente y funcional.

Este script es el **único realmente imprescindible**.

### 2. `Script post instalacion Debian 13, parte 2, personalizacion.md`

Script complementario y **opcional**, orientado a:

- Trabajo académico y técnico con R
- Compilación de paquetes
- LaTeX y OCR
- Personalización avanzada del ratón en Wayland (control de volumen)

Está pensado para ejecutarse **después** del primero, solo si ese tipo de configuración es necesaria.

---

## Qué hace el script principal

Tras una instalación limpia de Debian 13 con GNOME, el primer script realiza las siguientes tareas.

### Sistema

- Configura los repositorios oficiales de Debian en `/etc/apt/sources.list`.
- Actualiza completamente el sistema.
- Instala herramientas básicas y utilidades habituales.
- Añade soporte para discos NTFS, exFAT y HFS+.
- Instala reproductores multimedia y soporte completo de audio y vídeo.
- Instala Flatpak y configura Flathub.
- Instala Thunderbird como cliente de correo.

### GRUB (arranque)

Configura el gestor de arranque en `/etc/default/grub`:

- Tiempo de espera antes del inicio de sesión: 10 segundos.
- Resolución: 2560 × 1080.
- Detección de otros sistemas operativos (por ejemplo Windows).
- Arranque limpio y silencioso (`quiet splash`).

### Usuario y permisos

- Modifica directamente `/etc/sudoers` (con copia de seguridad y validación).
- Activa feedback visual al escribir la contraseña con `sudo`.
- **No altera** el modelo de permisos ni la lógica de seguridad de Debian.

### R y RStudio

- Añade el repositorio oficial CRAN usando un keyring seguro.
- Instala R (`r-base`, `r-base-dev`).
- Instala RStudio Desktop de forma opcional.
- Verifica criptográficamente el instalador de RStudio mediante SHA256.

### GNOME (configuración automática)

Al **primer inicio de sesión**, el sistema aplica automáticamente una serie de ajustes en GNOME.  
Estos ajustes se ejecutan **una sola vez** y luego el sistema vuelve a su funcionamiento normal.

Entre otros:

- Dock inferior (Dash to Dock):
  - Iconos pequeños.
  - Apariencia integrada con el tema del sistema.
- Menús más legibles, sin transparencias innecesarias.
- Monitorización del sistema (CPU, memoria, red).
- Iconos pequeños en el escritorio (Desktop Icons NG).
- Botones de ventana: minimizar, maximizar y cerrar.
- Tipografías Ubuntu (incluida Ubuntu Mono).
- Atajo estándar `Super + A` para la vista de aplicaciones.

El comportamiento normal de GNOME **no se modifica**.

---

## Qué hace la segunda parte (opcional, es lo más personal dado mi uso)

El segundo script amplía el sistema con:

- Dependencias de compilación habituales para R y paquetes científicos.
- LaTeX completo (XeTeX y extras).
- Pandoc.
- OCR con Tesseract (inglés y español).
- Configuración del ratón en Wayland para controlar el volumen mediante botones laterales, usando `input-remapper` sin interfaz gráfica.

Esta parte está pensada para entornos académicos o técnicos y **no es necesaria** para un uso general del sistema.

---

## Qué es `sudo`

`sudo` es un comando que permite realizar cambios importantes en el sistema introduciendo la contraseña del usuario.

- Es normal.
- Es seguro.
- Debian lo utiliza para evitar errores accidentales.

Cuando el script lo solicite, escribe tu contraseña y pulsa Enter.  
No se mostrarán letras ni símbolos en pantalla. Esto es el comportamiento esperado.

---

## Requisitos

- Debian 13 recién instalado.
- Entorno gráfico GNOME.
- Conexión a Internet.
- Usuario con permisos de administrador (`sudo`).

El sistema funciona tanto en **Wayland como en X11**.

---

## Cómo usar los scripts

Abre una terminal y **copia y pega** el contenido del archivo correspondiente:

1. Ejecuta primero el contenido de  
   **`Script post instalacion Debian 13.md`**
2. Cierra sesión cuando el script lo indique.
3. Vuelve a iniciar sesión en GNOME.
4. De forma opcional, ejecuta después  
   **`Script post instalacion Debian 13, parte 2, personalizacion.md`**

---

## Qué modifica el sistema

Por transparencia, los scripts modifican o configuran:

- `/etc/apt/sources.list`
- `/etc/default/grub`
- `/etc/sudoers`
- Instalación de software y extensiones GNOME oficiales

Se recomienda usarlos únicamente en instalaciones limpias.

---

## Seguridad

- Usa repositorios oficiales.
- Verifica RStudio mediante comprobación criptográfica.
- No elimina datos personales.
- No altera la lógica ni el comportamiento base de GNOME.

---

## Reconocimiento

Este proyecto fue desarrollado con el apoyo de **ChatGPT (OpenAI)** como herramienta de asistencia para la programación, depuración y documentación.
