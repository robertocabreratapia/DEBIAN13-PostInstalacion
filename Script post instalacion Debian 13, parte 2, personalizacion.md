# =========================
# 2.1) Paquetes base (RStudio, etc.)
# =========================
sudo apt update
sudo apt install -y \
  build-essential \
  cmake \
  pkg-config \
  libcurl4-openssl-dev \
  libcairo2-dev \
  libpng-dev \
  libjpeg-dev \
  libtiff-dev \
  libxml2-dev \
  libssl-dev \
  gdal-bin \
  libgdal-dev \
  libglpk-dev \
  libpoppler-cpp-dev \
  libudunits2-dev \
  pandoc \
  texlive-xetex \
  texlive-fonts-recommended \
  texlive-latex-recommended \
  texlive-latex-extra \
  libharfbuzz-dev \
  libfribidi-dev \
  libfreetype6-dev \
  libmagick++-dev \
  default-jdk \
  libtesseract-dev \
  libleptonica-dev \
  tesseract-ocr-eng \
  tesseract-ocr-spa \
  gsfonts \
  libnode-dev

# =========================
# 2.2) Mouse: volumen en Wayland (input-remapper, sin GUI)
# =========================
sudo apt install -y input-remapper input-remapper-daemon python3-evdev

TARGET_HOME="$HOME"
TARGET_USER="$USER"

MOUSE_EVENT="/dev/input/by-id/usb-CX_Trust_BAYO_Multi-device-event-mouse"
DEVICE_NAME="CX Trust BAYO Multi-device Mouse"
PRESET_NAME="mouse-volume"

# Orientación corregida para tu ratón:
# BTN_EXTRA (276) = subir volumen
# BTN_SIDE  (275) = bajar volumen
BTN_UP_CODE=276
BTN_DOWN_CODE=275

# 1) Forzar que el demonio lea tu HOME (y no /root)
sudo mkdir -p /etc/systemd/system/input-remapper-daemon.service.d
sudo tee /etc/systemd/system/input-remapper-daemon.service.d/override.conf >/dev/null <<EOF
[Service]
Environment=HOME=$TARGET_HOME
Environment=XDG_CONFIG_HOME=$TARGET_HOME/.config
EOF
sudo systemctl daemon-reload
sudo systemctl enable --now input-remapper-daemon.service

# 2) Limpiar preset inválido previo (si existiera) y reescribir mouse-volume.json + config.json correctos
sudo env \
  TARGET_HOME="$TARGET_HOME" \
  MOUSE_EVENT="$MOUSE_EVENT" \
  PRESET_NAME="$PRESET_NAME" \
  BTN_UP_CODE="$BTN_UP_CODE" \
  BTN_DOWN_CODE="$BTN_DOWN_CODE" \
python3 - <<'PY'
import json, os, re, pathlib
from hashlib import md5
from evdev import InputDevice, ecodes

home = os.environ["TARGET_HOME"]
mouse_event = os.environ["MOUSE_EVENT"]
preset_name = os.environ["PRESET_NAME"]
code_up = int(os.environ["BTN_UP_CODE"])
code_down = int(os.environ["BTN_DOWN_CODE"])

cfg_dir = pathlib.Path(home) / ".config" / "input-remapper-2"
presets_dir = cfg_dir / "presets"
config_path = cfg_dir / "config.json"

def sanitize(s: str) -> str:
    return re.sub(r'[\/\\\?\%\*\:\|"<>\']', "_", s)

dev = InputDevice(mouse_event)
cap = dev.capabilities(absinfo=False)
origin_hash = md5((str(cap) + dev.name).encode()).hexdigest().lower()

group_dir = presets_dir / sanitize(dev.name)
group_dir.mkdir(parents=True, exist_ok=True)

# Eliminar preset inválido típico si existiera
bad = group_dir / "new preset.json"
if bad.exists():
    bad.unlink()

preset_path = group_dir / f"{preset_name}.json"

preset = [
    {
        "input_combination": [{"type": ecodes.EV_KEY, "code": code_up, "origin_hash": origin_hash}],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_VOLUMEUP",
        "name": "volume-up",
        "mapping_type": "key_macro",
    },
    {
        "input_combination": [{"type": ecodes.EV_KEY, "code": code_down, "origin_hash": origin_hash}],
        "target_uinput": "keyboard",
        "output_symbol": "KEY_VOLUMEDOWN",
        "name": "volume-down",
        "mapping_type": "key_macro",
    },
]

cfg_dir.mkdir(parents=True, exist_ok=True)
preset_path.write_text(json.dumps(preset, indent=4) + "\n", encoding="utf-8")

cfg = {}
if config_path.exists():
    try:
        cfg = json.loads(config_path.read_text(encoding="utf-8")) or {}
    except json.JSONDecodeError:
        cfg = {}

# version debe ser cadena para input-remapper-control
cfg["version"] = str(cfg.get("version", "2"))
cfg.setdefault("autoload", {})
cfg["autoload"][dev.name] = preset_name
cfg["autoload"][origin_hash] = preset_name

config_path.write_text(json.dumps(cfg, indent=4) + "\n", encoding="utf-8")

print("OK")
print("Device:", dev.name)
print("Hash:", origin_hash)
print("Preset:", str(preset_path))
print("Config:", str(config_path))
PY

# 3) Dejar la config como tuya (no de root)
sudo chown -R "$TARGET_USER:$TARGET_USER" "$TARGET_HOME/.config/input-remapper-2" 2>/dev/null || true

# 4) Reiniciar y aplicar autoload por nombre de dispositivo
sudo systemctl restart input-remapper-daemon.service
sudo input-remapper-control --command autoload --device "$DEVICE_NAME"
