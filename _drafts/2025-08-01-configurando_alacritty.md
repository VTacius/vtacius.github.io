---
title: Configuración básica de Alacritty 
tags:
    - configuraciones
    - alacritty
---

## Introducción
Una consola es siempre la mejor amiga de quien trabaja en Linux. Así que no tiene nada de raro que pasemos tanto tiempo configurando.

Sin embargo, esta vez deje ese enfoque. Me he vuelto de la mejor terminal posible, `xfce4-terminal` a la mejor terminal con soporte TTY. Y soporte de GPU. Y escrita en Rust. Baśicamente una herramienta de gran calidad.

## Configuración previa

Instalas `Alacritty`. En mi caso, usaré la versión en los repos de mi distro
```bash
dnf install alacritty
```

Instalas una fuente de verdad. Es decir, que incluya grafemas unicode avanzados. Iconos, pues
Yo escogi [Hack](https://www.programmingfonts.org/#hack) de [Nerd Font](https://www.nerdfonts.com/font-downloads). (Suelen referirse a ella como "Hack Nerd Font")

Como todo la paleta de colores que escogí es bastante profesional, me gusta el contraste con una letra que solo es ligeramente festiva.

Para ello, escoges tu tipo de letras, lo descargas y lo descomprimes así:
```bash
unzip Hack.zip -c /usr/share/fonts/
```

Parece que no es la mejor forma del mundo para usarla, pero como no voy a instalar más creo que estará bien así. Lo importante es que aparezca cuando ejecutas `fc-list`:
```bash
❯ fc-list | rg Hack
/usr/share/fonts/HackNerdFontPropo-Italic.ttf: Hack Nerd Font Propo:style=Italic
/usr/share/fonts/HackNerdFontMono-Bold.ttf: Hack Nerd Font Mono:style=Bold
/usr/share/fonts/HackNerdFontPropo-BoldItalic.ttf: Hack Nerd Font Propo:style=Bold Italic
/usr/share/fonts/HackNerdFontPropo-Regular.ttf: Hack Nerd Font Propo:style=Regular
/usr/share/fonts/HackNerdFontMono-Italic.ttf: Hack Nerd Font Mono:style=Italic
/usr/share/fonts/HackNerdFontPropo-Bold.ttf: Hack Nerd Font Propo:style=Bold
/usr/share/fonts/HackNerdFont-Regular.ttf: Hack Nerd Font:style=Regular
/usr/share/fonts/HackNerdFont-Bold.ttf: Hack Nerd Font:style=Bold
/usr/share/fonts/HackNerdFont-Italic.ttf: Hack Nerd Font:style=Italic
/usr/share/fonts/HackNerdFont-BoldItalic.ttf: Hack Nerd Font:style=Bold Italic
/usr/share/fonts/HackNerdFontMono-BoldItalic.ttf: Hack Nerd Font Mono:style=Bold Italic
/usr/share/fonts/HackNerdFontMono-Regular.ttf: Hack Nerd Font Mono:style=Regular
```

## Configuración

La configuración es demasiado simple. Entre otras cosas, la consola tiene un [modo vi](https://alacritty.org/config-alacritty-bindings.html#key-bindings) si se quiere hacer cosas realmente complejas. Se entra y sale de él con  `Ctrl + Shift + Space`
```bash
mkdir -p ~/.config/alacritty/themes
git clone https://github.com/alacritty/alacritty-theme ~/.config/alacritty/themes
```

Y luego configurar `Alacritty` de la siguiente forma:
```toml
## ~/.config/alacritty/alacritty.toml

[env]
TERM = "xterm-256color"

[window]
opacity = 1
padding.x = 4
padding.y = 4
decorations = "buttonless"
dynamic_padding = true
dimensions = { columns = 140, lines = 35 }

[general]
import = [
    "~/.config/alacritty/themes/themes/nordfox.toml"
]

[terminal]
shell = { program = "/bin/zsh", args = ["-l"] }

[font]
size = 10.0

[font.bold]
family = "Hack Nerd Font Mono"
style = "Bold"

[font.bold_italic]
family = "Hack Nerd Font Mono"
style = "Bold Italic"

[font.italic]
family = "Hack Nerd Font Mono"
style = "Italic"

[font.normal]
family = "Hack Nerd Font Mono"
style = "Regular"

[keyboard]
bindings = [
   { key = "Return", mods = "Control|Shift", action = "SpawnNewInstance" }
]

```

Esta es una forma mínima viable. Se ve bien, trabaja increíble y es posible usarla como la base de toda una pila de herramientas
