# Wayland + NVIDIA-Kompatibilität: Umfassende Lösungen und Fehlerbehebungen

---

## Überblick
Wayland ist der moderne Nachfolger von Xorg, bietet jedoch eingeschränkte native Unterstützung für NVIDIA-GPUs. Die Probleme entstehen durch NVIDIAs proprietäre Treiber und deren abweichende Implementierung von Buffer-Management-Systemen (EGLStreams vs. GBM). Dieser Leitfaden sammelt alle bekannten Lösungen und Troubleshooting-Schritte für stabile NVIDIA-Nutzung unter Wayland.

---

## Voraussetzungen
- **NVIDIA-Treiber ab Version 515+** (für Wayland-Unterstützung).
- **Kernel ab 6.2+** (empfohlen für bessere DRM/KMS-Integration).
- **Wayland-fähige Desktop-Umgebung** (GNOME, KDE Plasma, Sway).

---

## 1. Grundlegende Konfiguration

### **1.1 Neueste Treiber installieren**
```bash
# Ubuntu/Debian
sudo apt install nvidia-driver-550-open

# Fedora
sudo dnf install akmod-nvidia

# Arch/Manjaro
sudo pacman -S nvidia-dkms nvidia-utils
```

### **1.2 NVIDIA-DRM-Modeset aktivieren**  
*Erzwingt die Kernel-Mode-Setting (KMS) für bessere Wayland-Integration.*  
- Bearbeiten Sie die GRUB-Konfiguration:
  ```bash
  sudo nano /etc/default/grub
  ```
- Fügen Sie `nvidia-drm.modeset=1` zu `GRUB_CMDLINE_LINUX_DEFAULT` hinzu:
  ```bash
  GRUB_CMDLINE_LINUX_DEFAULT="nvidia-drm.modeset=1 quiet splash"
  ```
- Aktualisieren Sie GRUB:
  ```bash
  sudo update-grub
  ```

### **1.3 NVIDIA-Module laden**
Stellen Sie sicher, dass die Kernel-Module geladen sind:
```bash
sudo tee /etc/modprobe.d/nvidia.conf <<< "options nvidia NVreg_UsePageAttributeTable=1"
sudo modprobe nvidia-drm
```

---

## 2. Wayland-Sitzung erzwingen

### **2.1 GDM (GNOME)**
- Bearbeiten Sie die GDM-Konfiguration:
  ```bash
  sudo nano /etc/gdm3/custom.conf
  ```
- Entfernen Sie das `#` vor `WaylandEnable=true`:
  ```ini
  [daemon]
  WaylandEnable=true
  ```

### **2.2 SDDM (KDE Plasma)**
- Erstellen Sie eine neue Konfigurationsdatei:
  ```bash
  sudo nano /etc/sddm.conf.d/10-wayland.conf
  ```
- Fügen Sie Folgendes hinzu:
  ```ini
  [General]
  DisplayServer=wayland
  ```

---

## 3. Fehlerbehebungen für häufige Probleme

### **3.1 "Failed to initialize NVIDIA GPU"**  
*Symptome*: Schwarzer Bildschirm, Absturz bei Wayland-Start.  
**Lösung**:  
1. Deaktivieren Sie Secure Boot im BIOS/UEFI.  
2. Signieren Sie die NVIDIA-Module:
   ```bash
   sudo mokutil --disable-validation
   ```

### **3.2 Grafikartefakte oder Flackern**  
*Symptome*: Fehlerhafte Darstellung in Anwendungen.  
**Lösung**:  
- Erzwingen Sie Vollschirm-Reparatur in der `~/.config/environment.d/`:
  ```bash
  echo "__GL_ALLOW_FULL_OPENGL=1" | sudo tee -a /etc/environment
  ```
- Deaktivieren Sie Hardware-Cursor:
  ```bash
  echo "WLR_NO_HARDWARE_CURSORS=1" | sudo tee -a /etc/environment
  ```

### **3.3 Keine Hardwarebeschleunigung**  
*Symptome*: Hohe CPU-Last, langsame Performance.  
**Lösung**:  
- Prüfen Sie die VA-API-Integration:
  ```bash
  sudo apt install libva-nvidia-driver
  ```
- Setzen Sie Umgebungsvariablen in `~/.profile`:
  ```bash
  export LIBVA_DRIVER_NAME=nvidia
  export GBM_BACKEND=nvidia-drm
  export __GLX_VENDOR_LIBRARY_NAME=nvidia
  ```

---

## 4. Workarounds für spezielle Anwendungen

### **4.1 XWayland-Apps (z.B. Discord, Steam)**  
*Problem*: Apps laufen unter XWayland ohne Hardwarebeschleunigung.  
**Lösung**:  
- Erzwingen Sie Vulkan für XWayland:
  ```bash
  echo "GDK_BACKEND=wayland,x11" | sudo tee -a /etc/environment
  echo "SDL_VIDEODRIVER=wayland,x11" | sudo tee -a /etc/environment
  ```

### **4.2 Wine/Proton-Spiele**  
*Problem*: FPS-Einbrüche oder Abstürze.  
**Lösung**:  
- Nutzen Sie Gamescope als Compositor-Wrapper:
  ```bash
  gamescope -W 1920 -H 1080 -e -f -- %command%
  ```

---

## 5. Multi-GPU-Setups (Hybrid NVIDIA + Intel/AMD)

### **5.1 PRIME Offloading**  
*Für Laptops mit Hybrid-Grafik*:  
- Exportieren Sie Variablen vor dem Start der Anwendung:
  ```bash
  __NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia glxinfo | grep "OpenGL renderer"
  ```
- Automatisierung mit `envycontrol` (für Optimus):
  ```bash
  sudo pip install envycontrol
  sudo envycontrol --switch nvidia --wayland
  ```

---

## 6. Erweiterte Diagnose

### **6.1 Logging aktivieren**  
- Starten Sie die Wayland-Sitzung mit Debugging:
  ```bash
  XDG_SESSION_TYPE=wayland dbus-run-session gnome-shell --wayland --debug
  ```
- Überprüfen Sie NVIDIA-Status:
  ```bash
  nvidia-smi
  nvidia-settings --query=AllowNVIDIAGPUScreens
  ```

### **6.2 Überprüfen der Wayland-Integration**  
- Prüfen Sie, ob Wayland aktiv ist:
  ```bash
  echo $XDG_SESSION_TYPE
  ```
- Testen Sie EGL mit:
  ```bash
  glxinfo -B | grep "OpenGL renderer"
  ```

---

## 7. Fallback zu Xorg (Notfalllösung)

### **7.1 Xorg als Standard setzen**  
- Für GNOME:
  ```bash
  sudo nano /etc/gdm3/custom.conf
  # Entfernen Sie das # vor `WaylandEnable=false`
  ```
- Für KDE Plasma:
  ```bash
  sudo nano /etc/sddm.conf.d/10-xorg.conf
  [General]
  DisplayServer=xorg
  ```

---

## Fazit
Wayland mit NVIDIA erfordert sorgfältige Konfiguration, ist aber seit Treiberversion 515+ stabil nutzbar. Wichtige Schritte:  
1. **Aktuelle Treiber und Kernel** verwenden.  
2. **nvidia-drm.modeset=1** immer aktivieren.  
3. **Umgebungsvariablen** für Hardwarebeschleunigung setzen.  
4. Bei Problemen **XWayland-Apps gezielt debuggen**.

---

## Weiterführende Ressourcen  
- [NVIDIA Wayland Documentation](https://download.nvidia.com/XFree86/Linux-x86_64/550.54.14/README/wayland.html)  
- [KDE Plasma Wiki: NVIDIA](https://community.kde.org/Plasma/Wayland/NVIDIA)  
- [Arch Linux NVIDIA Troubleshooting](https://wiki.archlinux.org/title/NVIDIA/Troubleshooting)
