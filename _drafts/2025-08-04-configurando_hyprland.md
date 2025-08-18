---
title: Configurando Hyprland
tags:
    - comentarios
toc: false
---
## Introducción
Si bien XFCE ha sido mi eterna compañía por muchos años, es el momento de cambiar

Escogí `Hyprland` porque sí. Supongo que es el que mejor ha funcionado a la primera.

## Vista previa
Esta configuración busca dar la experiencia de usuarios más básica posible:

 * `cliphist`: Nuestra opción para tener un sistema de portapapeles coherente.
 * `xdg-desktop-portal-hyprland`: Es totalmente necesario para una experiencia de usuario muy cerca de lo normal
 * `hyprlock`: Porque sí, necesitamos poder hacer `CTRL + ALT + l`
 * `hyprshot`: Para tomar capturas de pantalla

Esta configuración no tiene `idle`, aunque esta comentado en el archivo y solo bastaría instalar `hypridle`.

Además, se crean cuatro `workspaces` fijos, y se crea uno `special` llamado `util`, que al iniciar el sistema arranca dos aplicaciones. 

Es posible acceder a el en cualquier momento con `CMD + U`. Vale la pena darle una oportunidad

## Procedimiento

Instalamos este repositorio, que no solo tiene las útimas versiones, sino que también tiene todos los paquetes de `Hyprland`:

```bash
sudo dnf copr enable solopasha/hyprland
```

Instalamos

```bash 
sudo dnf install cliphist xdg-desktop-portal-hyprland hyprlock hyprshot
```

Y configuramos según dos archivos:

```conf
# Configuración de Hyprland 
# Compatible con Hyprland 0.50.1
# Wiki: https://wiki.hyprland.org/Configuring/

################
### MONITORS ###
################
# Definir monitores PRIMERO antes que cualquier workspace
monitor=eDP-1, 1920x1200@60, 0x0, 1
monitor=HDMI-A-5, 1920x1080@60, 0x-1080, 1
monitor=,preferred,auto,1  # Fallback para monitores desconocidos

#############################
### ENVIRONMENT VARIABLES ###
#############################
# Variables de entorno ANTES de cualquier exec
env = XCURSOR_SIZE,24
env = HYPRCURSOR_SIZE,24

# Variables de PATH con ruta absoluta para evitar problemas
env = PATH,/home/alortiz/.asdf/shims:/home/alortiz/.local/bin:/home/alortiz/.cargo/bin:/home/alortiz/go/bin:/usr/lib64/ccache:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# Variables adicionales para Wayland (descomenta si necesitas)
# env = XDG_CURRENT_DESKTOP,Hyprland
# env = XDG_SESSION_TYPE,wayland
# env = XDG_SESSION_DESKTOP,Hyprland
# env = QT_QPA_PLATFORM,wayland;xcb
# env = QT_WAYLAND_DISABLE_WINDOWDECORATION,1
# env = MOZ_ENABLE_WAYLAND,1
# env = GDK_BACKEND,wayland,x11

# Variables para Nvidia (descomenta si usas Nvidia)
# env = LIBVA_DRIVER_NAME,nvidia
# env = GBM_BACKEND,nvidia-drm
# env = __GLX_VENDOR_LIBRARY_NAME,nvidia

################
### THEMES ###
################
source=~/.config/hypr/themes/onenord.conf

###################
### MY PROGRAMS ###
###################
$menu = rofi -show drun
$locker = hyprlock
$terminal = alacritty
$fileManager = alacritty -e yazi
$navegador = firefox

##############################
### WINDOWS AND WORKSPACES ###
##############################

# IMPORTANTE: Definir reglas de workspace ANTES de exec-once
# Workspaces persistentes - sintaxis corregida con persistent:true
workspace = 1, monitor:eDP-1, persistent:true
workspace = 2, monitor:HDMI-A-5, persistent:true
workspace = 3, monitor:eDP-1, persistent:true
workspace = 4, monitor:HDMI-A-5, persistent:true

# Workspace especial 'utils' - solo en laptop
workspace = special:utils, monitor:eDP-1

# Reglas de ventanas - DEBEN estar ANTES de exec-once
# Para que las aplicaciones se asignen correctamente al workspace especial
windowrulev2 = workspace special:utils silent, class:^(Spotify)$
windowrulev2 = workspace special:utils silent, class:^(Ferdium)$
windowrulev2 = workspace special:utils silent, title:^(Spotify.*)$
windowrulev2 = workspace special:utils silent, title:^(Ferdium.*)$

# Reglas adicionales para aplicaciones flotantes
windowrule = float, class:^(Alacritty)$, title:^(bluetui)$
windowrule = size 900 600, class:^(Alacritty)$, title:^(bluetui)$
windowrule = center, class:^(Alacritty)$, title:^(bluetui)$

windowrule = float, class:^(Alacritty)$, title:^(impala)$
windowrule = size 900 600, class:^(Alacritty)$, title:^(impala)$
windowrule = center, class:^(Alacritty)$, title:^(impala)$

windowrule = float, class:^(Alacritty)$, title:^(wiremix)$
windowrule = size 900 600, class:^(Alacritty)$, title:^(wiremix)$
windowrule = center, class:^(Alacritty)$, title:^(wiremix)$

# Ignore maximize requests
windowrulev2 = suppressevent maximize, class:.*

# Fix dragging issues with XWayland
windowrulev2 = nofocus,class:^$,title:^$,xwayland:1,floating:1,fullscreen:0,pinned:0

#################
### AUTOSTART ###
#################

# Servicios esenciales primero
# Descomentar si se requiere mako para notificaciones de escritorio
#exec-once = mako
exec-once = hyprpaper
# Descomentar si se necesita waybar
#exec-once = waybar --config ~/.config/waybar/config --style ~/.config/waybar/style.css --log-level info

# Gestión del portapapeles
exec-once = wl-paste --type text --watch cliphist store
exec-once = wl-paste --type image --watch cliphist store

# CORRECCIÓN: Aplicaciones en workspace especial
# Usar hyprctl dispatch para mejor control con flatpaks
exec-once = sleep 2 && hyprctl dispatch exec "[workspace special:utils silent]" "flatpak run com.spotify.Client"
exec-once = sleep 3 && hyprctl dispatch exec "[workspace special:utils silent]" "flatpak run org.ferdium.Ferdium"

# Servicios opcionales (descomenta si necesitas)
# exec-once = hypridle
# exec-once = /usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1

#####################
### LOOK AND FEEL ###
#####################

# General
general {
    gaps_in = 2
    gaps_out = 4
    border_size = 1
    col.active_border = rgba(81a1c1ee) rgba(8fbcbbee) 45deg
    col.inactive_border = rgba(4c566aaa)
    resize_on_border = false
    allow_tearing = false
    layout = dwindle
}

# Decoration
decoration {
    rounding = 10
    active_opacity = 1.0
    inactive_opacity = 0.95

    shadow {
        enabled = true
        range = 4
        render_power = 3
        color = rgba(1a1a1aee)
    }

    blur {
        enabled = true
        size = 3
        passes = 1
        vibrancy = 0.1696
    }
}

# Animations
animations {
    enabled = yes

    bezier = easeOutQuint,0.23,1,0.32,1
    bezier = easeInOutCubic,0.65,0.05,0.36,1
    bezier = linear,0,0,1,1
    bezier = almostLinear,0.5,0.5,0.75,1.0
    bezier = quick,0.15,0,0.1,1

    animation = global, 1, 10, default
    animation = border, 1, 5.39, easeOutQuint
    animation = windows, 1, 4.79, easeOutQuint
    animation = windowsIn, 1, 4.1, easeOutQuint, popin 87%
    animation = windowsOut, 1, 1.49, linear, popin 87%
    animation = fadeIn, 1, 1.73, almostLinear
    animation = fadeOut, 1, 1.46, almostLinear
    animation = fade, 1, 3.03, quick
    animation = layers, 1, 3.81, easeOutQuint
    animation = layersIn, 1, 4, easeOutQuint, fade
    animation = layersOut, 1, 1.5, linear, fade
    animation = fadeLayersIn, 1, 1.79, almostLinear
    animation = fadeLayersOut, 1, 1.39, almostLinear
    animation = workspaces, 1, 1.94, almostLinear, fade
    animation = workspacesIn, 1, 1.21, almostLinear, fade
    animation = workspacesOut, 1, 1.94, almostLinear, fade
}

# Layouts
dwindle {
    pseudotile = true
    preserve_split = true
}

master {
    new_status = master
}

# Misc
misc {
    force_default_wallpaper = 0
    disable_hyprland_logo = true
    background_color = 0x2e3440
    vfr = 0
}

# Cursor (para problemas de hardware)
cursor {
    no_hardware_cursors = true
}

# OpenGL settings
opengl {
    nvidia_anti_flicker = 0
}

# Debug
debug {
    damage_tracking = 0
    disable_logs = false
}

#############
### INPUT ###
#############

input {
    kb_layout = latam
    kb_variant =
    kb_model =
    kb_options =
    kb_rules =

    follow_mouse = 1
    sensitivity = 0

    touchpad {
        natural_scroll = false
        scroll_factor = 1.2
    }
}

gestures {
    workspace_swipe = true
    workspace_swipe_fingers = 3
}

###################
### KEYBINDINGS ###
###################

$mainMod = SUPER

# Aplicaciones básicas
bind = $mainMod, Q, exec, $terminal
bind = $mainMod, C, killactive,
bind = $mainMod, M, exit,
bind = $mainMod, E, exec, $fileManager
bind = $mainMod, V, togglefloating,
bind = $mainMod, R, exec, $menu
bind = $mainMod, P, pseudo,
bind = $mainMod, J, togglesplit,
bind = $mainMod, B, exec, $navegador

# Bloqueo de pantalla
bind = Control + Alt, L, exec, $locker

# Toggle workspace especial 'utils'
bind = $mainMod, U, togglespecialworkspace, utils

# Move focus
bind = $mainMod, up, movefocus, u
bind = $mainMod, left, movefocus, l
bind = $mainMod, down, movefocus, d
bind = $mainMod, right, movefocus, r

# Switch workspaces
bind = $mainMod, 1, workspace, 1
bind = $mainMod, 2, workspace, 2
bind = $mainMod, 3, workspace, 3
bind = $mainMod, 4, workspace, 4
bind = $mainMod, 5, workspace, 5
bind = $mainMod, 6, workspace, 6
bind = $mainMod, 7, workspace, 7
bind = $mainMod, 8, workspace, 8
bind = $mainMod, 9, workspace, 9
bind = $mainMod, 0, workspace, 10

# Move active window to workspace
bind = $mainMod SHIFT, 1, movetoworkspace, 1
bind = $mainMod SHIFT, 2, movetoworkspace, 2
bind = $mainMod SHIFT, 3, movetoworkspace, 3
bind = $mainMod SHIFT, 4, movetoworkspace, 4
bind = $mainMod SHIFT, 5, movetoworkspace, 5
bind = $mainMod SHIFT, 6, movetoworkspace, 6
bind = $mainMod SHIFT, 7, movetoworkspace, 7
bind = $mainMod SHIFT, 8, movetoworkspace, 8
bind = $mainMod SHIFT, 9, movetoworkspace, 9
bind = $mainMod SHIFT, 0, movetoworkspace, 10

# Workspace navigation
bind = $mainMod, E, movetoworkspace, empty
bind = CTRL ALT, right, workspace, e+1
bind = CTRL ALT, left, workspace, e-1

# Mouse bindings
bindm = $mainMod, mouse:272, movewindow
bindm = $mainMod, mouse:273, resizewindow

# Multimedia keys
bindel = ,XF86AudioRaiseVolume, exec, wpctl set-volume @DEFAULT_AUDIO_SINK@ 5%+
bindel = ,XF86AudioLowerVolume, exec, wpctl set-volume @DEFAULT_AUDIO_SINK@ 5%-
bindel = ,XF86AudioMute, exec, wpctl set-mute @DEFAULT_AUDIO_SINK@ toggle
bindel = ,XF86AudioMicMute, exec, wpctl set-mute @DEFAULT_AUDIO_SOURCE@ toggle
bindel = ,XF86MonBrightnessUp, exec, brightnessctl s 5%+
bindel = ,XF86MonBrightnessDown, exec, brightnessctl s 5%-

# Media control
bindl = CTRL ALT, f, exec, playerctl next
bindl = CTRL ALT, p, exec, playerctl play-pause
bindl = CTRL ALT, b, exec, playerctl previous

# Screenshots con hyprshot
bind = , Print, exec, hyprshot -m output
bind = SHIFT, Print, exec, hyprshot -m region
bind = CTRL, Print, exec, hyprshot -m window
bind = ALT, Print, exec, hyprshot -m region -r
bind = SUPER, Print, exec, hyprshot -m output -c
bind = SUPER SHIFT, Print, exec, hyprshot -m region -c
bind = SUPER CTRL, Print, exec, hyprshot -m window -c

# Notificaciones con Mako
bind = $mainMod, N, exec, makoctl dismiss
bind = $mainMod SHIFT, N, exec, makoctl dismiss -a
bind = $mainMod CTRL, N, exec, makoctl restore
```
