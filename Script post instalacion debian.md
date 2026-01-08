cat > /tmp/postinstalacion-debian13.sh <<'__SCRIPT__'
#!/usr/bin/env bash
set -Eeuo pipefail
export DEBIAN_FRONTEND=noninteractive

# =========================
# Helpers sudo / run-as-user
# =========================
if [ "${EUID:-$(id -u)}" -eq 0 ]; then
  SUDO=""
else
  command -v sudo >/dev/null 2>&1 || { echo "Falta sudo"; exit 1; }
  SUDO="sudo"
fi

LOG="/tmp/postinstalacion-debian13.log"
# Log robusto: si hay sudo, tee escribe con privilegios
exec > >($SUDO tee -a "$LOG") 2>&1

trap 'ec=$?; echo; echo "ERROR (código $ec) en línea $LINENO"; echo "Log: $LOG"; exit $ec' ERR
log(){ printf "\n[%s] %s\n" "$(date +'%F %T')" "$*"; }

# =========================
# Parámetros (v1.2, sin sources.list.d)
# =========================
: "${INSTALL_RSTUDIO:=1}"
: "${RSTUDIO_URL:=https://download1.rstudio.org/electron/jammy/amd64/rstudio-2026.01.0-392-amd64.deb}"
: "${RSTUDIO_SHA256_PINNED:=a85734b70843d6157a423829ffbcff9eadc739403ab1c7425c17236f733f95d8}"

: "${CRAN_KEY_FPR:=95C0FAF38DB3CCAD0C080A7BDC78B2DDEABC47B7}"

# GRUB
: "${GRUB_TIMEOUT:=10}"
: "${GRUB_GFXMODE:=2560x1080}"
: "${GRUB_KERNEL_PARAMS:=quiet splash}"
: "${GRUB_ENABLE_OS_PROBER:=true}"   # true/false

# sudo
: "${SUDO_ENABLE_PWFEEDBACK:=true}"  # true/false

# Dash to Dock
: "${DASH_ICON_SIZE:=32}"
: "${DASH_USE_BUILTIN_THEME:=true}" # true/false

# Blur my Shell
: "${BLUR_PANEL_SIGMA:=20}"
: "${BLUR_PANEL_BRIGHTNESS:=0.85}"

# DING
: "${DING_ICON_SIZE:=small}"         # small/medium/large

# Usuario objetivo
TARGET_USER="${SUDO_USER:-$(id -un)}"
TARGET_HOME="$(getent passwd "$TARGET_USER" | cut -d: -f6)"

# Comando para ejecutar como usuario objetivo
if [ "${EUID:-$(id -u)}" -eq 0 ]; then
  command -v sudo >/dev/null 2>&1 || { apt-get update; apt-get install -y sudo; }
  RUN_AS_USER=(sudo -u "$TARGET_USER" -H)
else
  if [ "$(id -un)" = "$TARGET_USER" ]; then
    RUN_AS_USER=()
  else
    RUN_AS_USER=(sudo -u "$TARGET_USER" -H)
  fi
fi

log "Usuario objetivo: $TARGET_USER"
log "Home objetivo: $TARGET_HOME"
log "Log: $LOG"

# =========================
# 1) Repositorios Debian trixie (solo sources.list)
# =========================
log "Configurando repositorios Debian trixie en /etc/apt/sources.list"
$SUDO tee /etc/apt/sources.list >/dev/null <<'EOF'
deb http://deb.debian.org/debian trixie main contrib non-free non-free-firmware
deb http://deb.debian.org/debian trixie-updates main contrib non-free non-free-firmware
deb http://security.debian.org/debian-security trixie-security main contrib non-free non-free-firmware
EOF

$SUDO apt update
$SUDO apt upgrade -y

# =========================
# 2) Paquetes base
# =========================
log "Instalando paquetes base"
$SUDO apt install -y \
  ca-certificates curl wget gnupg dirmngr coreutils \
  exfat-fuse hfsplus ntfs-3g \
  fastfetch htop \
  gdebi gdebi-core synaptic \
  fonts-ubuntu fonts-ubuntu-console \
  fonts-freefont-ttf fonts-freefont-otf fonts-liberation fonts-liberation2 fonts-croscore fonts-dejavu fonts-noto \
  p7zip-full unrar-free \
  ffmpeg libavcodec-extra \
  gstreamer1.0-libav gstreamer1.0-plugins-ugly gstreamer1.0-plugins-bad gstreamer1.0-pulseaudio vorbis-tools \
  flatpak gnome-software-plugin-flatpak \
  vlc clementine \
  pipx \
  gir1.2-gnomedesktop-3.0 \
  thunderbird


# =========================
# 3) Flatpak + Extension Manager
# =========================
log "Configurando Flathub + Extension Manager"
$SUDO flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
$SUDO flatpak install -y flathub com.mattjakeman.ExtensionManager || true

# =========================
# 4) sudoers: env_reset + pwfeedback (editando /etc/sudoers)
# =========================
if [ "$SUDO_ENABLE_PWFEEDBACK" = "true" ]; then
  log "Configurando sudoers: env_reset,pwfeedback (edición directa de /etc/sudoers)"

  # Copia de seguridad
  $SUDO cp -a /etc/sudoers "/etc/sudoers.bak.$(date +%F_%H%M%S)"

  # Editar en fichero temporal y validar antes de aplicar
  tmp_sudoers="$(mktemp)"
  $SUDO cat /etc/sudoers > "$tmp_sudoers"

  # Si existe línea Defaults env_reset, añadir pwfeedback; si no, añadir una Defaults nueva.
  if grep -Eq '^[[:space:]]*Defaults[[:space:]]+env_reset' "$tmp_sudoers"; then
    # Añadir pwfeedback solo si no está ya
    if ! grep -Eq '^[[:space:]]*Defaults[[:space:]]+.*\bpwfeedback\b' "$tmp_sudoers"; then
      $SUDO sed -i 's/^\([[:space:]]*Defaults[[:space:]]\+.*\benv_reset\b[^#]*\)$/\1,pwfeedback/' "$tmp_sudoers"
    fi
  else
    printf '\nDefaults env_reset,pwfeedback\n' | $SUDO tee -a "$tmp_sudoers" >/dev/null
  fi

  # Validar sintaxis
  $SUDO visudo -cf "$tmp_sudoers"

  # Aplicar si valida
  $SUDO cp "$tmp_sudoers" /etc/sudoers
  $SUDO chmod 0440 /etc/sudoers
  rm -f "$tmp_sudoers"
else
  log "sudoers: pwfeedback omitido"
fi

# =========================
# 5) GRUB
# =========================
log "Configurando GRUB"
$SUDO apt install -y plymouth plymouth-themes

if [ "$GRUB_ENABLE_OS_PROBER" = "true" ]; then
  $SUDO apt install -y os-prober
  $SUDO sed -i 's/^#\?GRUB_DISABLE_OS_PROBER=.*/GRUB_DISABLE_OS_PROBER=false/' /etc/default/grub || true
  grep -q '^GRUB_DISABLE_OS_PROBER=' /etc/default/grub || echo 'GRUB_DISABLE_OS_PROBER=false' | $SUDO tee -a /etc/default/grub >/dev/null
else
  $SUDO sed -i 's/^#\?GRUB_DISABLE_OS_PROBER=.*/GRUB_DISABLE_OS_PROBER=true/' /etc/default/grub || true
  grep -q '^GRUB_DISABLE_OS_PROBER=' /etc/default/grub || echo 'GRUB_DISABLE_OS_PROBER=true' | $SUDO tee -a /etc/default/grub >/dev/null
fi

$SUDO sed -i "s/^GRUB_TIMEOUT=.*/GRUB_TIMEOUT=${GRUB_TIMEOUT}/" /etc/default/grub || true
grep -q '^GRUB_TIMEOUT=' /etc/default/grub || echo "GRUB_TIMEOUT=${GRUB_TIMEOUT}" | $SUDO tee -a /etc/default/grub >/dev/null

$SUDO sed -i "s/^#\?GRUB_GFXMODE=.*/GRUB_GFXMODE=${GRUB_GFXMODE}/" /etc/default/grub || true
grep -q '^GRUB_GFXMODE=' /etc/default/grub || echo "GRUB_GFXMODE=${GRUB_GFXMODE}" | $SUDO tee -a /etc/default/grub >/dev/null

$SUDO sed -i 's/^#\?GRUB_GFXPAYLOAD_LINUX=.*/GRUB_GFXPAYLOAD_LINUX=keep/' /etc/default/grub || true
grep -q '^GRUB_GFXPAYLOAD_LINUX=' /etc/default/grub || echo 'GRUB_GFXPAYLOAD_LINUX=keep' | $SUDO tee -a /etc/default/grub >/dev/null

$SUDO sed -i "s/^GRUB_CMDLINE_LINUX_DEFAULT=.*/GRUB_CMDLINE_LINUX_DEFAULT=\"${GRUB_KERNEL_PARAMS}\"/" /etc/default/grub || true
grep -q '^GRUB_CMDLINE_LINUX_DEFAULT=' /etc/default/grub || echo "GRUB_CMDLINE_LINUX_DEFAULT=\"${GRUB_KERNEL_PARAMS}\"" | $SUDO tee -a /etc/default/grub >/dev/null

$SUDO update-grub

# =========================
# 6) CRAN en /etc/apt/sources.list (sin sources.list.d)
# =========================
log "Configurando CRAN en /etc/apt/sources.list"
$SUDO sed -i '/cloud\.r-project\.org\/bin\/linux\/debian/d' /etc/apt/sources.list

$SUDO gpg --batch --keyserver keyserver.ubuntu.com --recv-key "$CRAN_KEY_FPR" || true
$SUDO gpg --batch --yes --armor --export "$CRAN_KEY_FPR" > /tmp/cran.asc
$SUDO mv /tmp/cran.asc /etc/apt/trusted.gpg.d/cran.asc
$SUDO chmod 0644 /etc/apt/trusted.gpg.d/cran.asc

echo "deb http://cloud.r-project.org/bin/linux/debian trixie-cran40/" | \
  $SUDO tee -a /etc/apt/sources.list >/dev/null

$SUDO apt update
$SUDO apt install -y r-base r-base-dev

# =========================
# 7) RStudio (opcional, SHA256 estricto, robusto)
# =========================
if [ "$INSTALL_RSTUDIO" = "1" ]; then
  log "RStudio: descarga + verificación SHA256 estricta"
  RSTUDIO_DEB="$(mktemp -p /tmp rstudio-XXXXXX.deb)"
  wget -O "$RSTUDIO_DEB" "$RSTUDIO_URL"

  got="$(sha256sum "$RSTUDIO_DEB" | awk '{print $1}')"
  if [ "$got" != "$RSTUDIO_SHA256_PINNED" ]; then
    echo
    echo "Checksum SHA256 no coincide."
    echo "Esperado: $RSTUDIO_SHA256_PINNED"
    echo "Obtenido:  $got"
    echo "Archivo:   $RSTUDIO_DEB"
    exit 1
  fi

  $SUDO apt install -y "$RSTUDIO_DEB" || $SUDO gdebi -n "$RSTUDIO_DEB"
  $SUDO rm -f "$RSTUDIO_DEB" || true
else
  log "RStudio omitido (INSTALL_RSTUDIO=0)"
fi

# =========================
# 8) gnome-extensions-cli (usuario)
# =========================
log "Instalando gnome-extensions-cli (usuario)"
"${RUN_AS_USER[@]}" pipx ensurepath >/dev/null 2>&1 || true
"${RUN_AS_USER[@]}" pipx install gnome-extensions-cli >/dev/null 2>&1 || true

# =========================
# 9) Postlogin GNOME (una sola vez)
# =========================
log "Creando postlogin GNOME"

cat > "$TARGET_HOME/postlogin-gnome.sh" <<EOF
#!/usr/bin/env bash
set -Eeuo pipefail

LOG="\$HOME/postlogin-gnome.log"
exec >>"\$LOG" 2>&1

echo
echo "=== postlogin-gnome: \$(date +'%F %T') ==="

if [ -z "\${DBUS_SESSION_BUS_ADDRESS:-}" ]; then
  echo "Sin DBUS_SESSION_BUS_ADDRESS. Salgo."
  exit 0
fi

compile_local_schemas() {
  for d in "\$HOME/.local/share/gnome-shell/extensions"/*/schemas; do
    [ -d "\$d" ] || continue
    [ -f "\$d/gschemas.compiled" ] || glib-compile-schemas "\$d" >/dev/null 2>&1 || true
  done
}

install_enable_ext() {
  local uuid="\$1"
  if command -v gext >/dev/null 2>&1; then
    if ! gnome-extensions info "\$uuid" >/dev/null 2>&1; then
      gext install "\$uuid" || true
    fi
    compile_local_schemas
    gnome-extensions enable "\$uuid" >/dev/null 2>&1 || true
  fi
}

# GNOME puro: no tocar comportamiento de Super
# Solo fijamos Super+A como vista de aplicaciones (estándar)
gsettings set org.gnome.shell.keybindings toggle-application-view "['<Super>a']" || true

# Botones de ventana
gsettings set org.gnome.desktop.wm.preferences button-layout 'appmenu:minimize,maximize,close' || true

# Dash to Dock
gsettings set org.gnome.shell.extensions.dash-to-dock dash-max-icon-size ${DASH_ICON_SIZE} || true
gsettings set org.gnome.shell.extensions.dash-to-dock apply-custom-theme ${DASH_USE_BUILTIN_THEME} || true
gsettings set org.gnome.shell.extensions.dash-to-dock click-action 'focus-minimize-or-previews' || true
gsettings set org.gnome.shell.extensions.dash-to-dock show-windows-preview true || true
gsettings set org.gnome.shell.extensions.dash-to-dock dock-fixed true || true
gsettings set org.gnome.shell.extensions.dash-to-dock autohide false || true
gsettings set org.gnome.shell.extensions.dash-to-dock dock-position 'BOTTOM' || true

# Blur my Shell (panel)
dconf write /org/gnome/shell/extensions/blur-my-shell/appfolder/blur false || true
dconf write /org/gnome/shell/extensions/blur-my-shell/window-list/blur false || true
dconf write /org/gnome/shell/extensions/blur-my-shell/panel/sigma ${BLUR_PANEL_SIGMA} || true
dconf write /org/gnome/shell/extensions/blur-my-shell/panel/brightness ${BLUR_PANEL_BRIGHTNESS} || true

# Vitals
dconf write /org/gnome/shell/extensions/vitals/use-higher-precision true || true

# Caffeine
dconf write /org/gnome/shell/extensions/caffeine/indicator-position-max 1 || true

# Desktop Icons NG (DING)
install_enable_ext "ding@rastersoft.com"
if gsettings list-schemas | grep -qx "org.gnome.shell.extensions.ding"; then
  gsettings set org.gnome.shell.extensions.ding icon-size '${DING_ICON_SIZE}' || true
fi

echo "postlogin-gnome terminado."
EOF

chmod +x "$TARGET_HOME/postlogin-gnome.sh"
chown "$TARGET_USER:$TARGET_USER" "$TARGET_HOME/postlogin-gnome.sh"

"${RUN_AS_USER[@]}" mkdir -p "$TARGET_HOME/.config/autostart"

cat > "$TARGET_HOME/.config/autostart/postlogin-gnome-once.desktop" <<EOF
[Desktop Entry]
Type=Application
Name=Postlogin GNOME (una vez)
Exec=sh -lc '$TARGET_HOME/postlogin-gnome.sh; rm -f "$TARGET_HOME/.config/autostart/postlogin-gnome-once.desktop"'
X-GNOME-Autostart-enabled=true
NoDisplay=true
EOF

chown "$TARGET_USER:$TARGET_USER" "$TARGET_HOME/.config/autostart/postlogin-gnome-once.desktop"

# =========================
# 10) Limpieza
# =========================
log "Limpieza"
$SUDO apt autoremove -y
$SUDO apt clean

log "Listo ✅"
echo
echo "Paso siguiente: cierra sesión y vuelve a entrar en GNOME."
echo "Se ejecutará una vez: $TARGET_HOME/postlogin-gnome.sh"
echo "Log postlogin: $TARGET_HOME/postlogin-gnome.log"
echo "Log instalación: $LOG"
__SCRIPT__

chmod +x /tmp/postinstalacion-debian13.sh
sudo bash /tmp/postinstalacion-debian13.sh

