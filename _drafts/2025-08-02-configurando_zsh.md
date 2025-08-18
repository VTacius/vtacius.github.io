---
title: Configuración básica de ZSH 
tags:
    - configuraciones
    - zsh
---


## Introducción
Siempre dejé para después la migración a ZSH. Me molestaba la idea de que trabajaría en un entorno que no estaría en los servidores porque sería muy engorroso de configurar. Por otra parte, adoro las cosas engorrosas de configurar, así que al fin me dispuse

## Vista previa
Se inicia con la configuración de `asdf`. Es la mejor forma de mantener un sistema limpio. 

Después de ello, se sigue `starship`, `fd`, `pistol`, `fzf` (Realmente la joya de la corona) y `yazi`

Todo se vertebra en nuestro archivo `~/.zshrc`. Entre otras cosas, se carga el gestor de plugins (Escogi `sheldom` por estar hecho en Rust, y porque se sencillo)

Hay muchas cosas que se cargan mediante configuraciones provistas por los mismos comandos. Lo normal es que se haga con `eval`, pero quise usar un plugin para ello, `smartcache`

### Configuración

```zsh
# ~/.zshrc - Configuración ZSH optimizada para Fedora 42

# === CONFIGURACIÓN DE PATH ===
typeset -U path
path=(
    "$HOME/.asdf/shims"(N)
    "$HOME/.local/bin"(N)
    "$HOME/.cargo/bin"(N)
    "$HOME/go/bin"(N)
    $path
)
export PATH

# === CONFIGURACIÓN DE FPATH ===
# Solo agregar si existen y no están ya presentes
typeset -U fpath
if [[ -d "$HOME/.asdf/completions" && ! " ${fpath[*]} " =~ " $HOME/.asdf/completions " ]]; then
    fpath=("$HOME/.asdf/completions" $fpath)
fi

# === CONFIGURACIÓN INICIAL ===
# Cargar funciones esenciales
autoload -Uz compinit promptinit add-zsh-hook colors
compinit
promptinit
colors

# === VARIABLES DE ENTORNO ===
export HISTSIZE=10000
export SAVEHIST=10000
export HISTFILE=~/.zsh_history
export ZSH_SMARTCACHE_DIR="${XDG_CACHE_HOME:-$HOME/.cache}/zsh-smartcache"

# Crear directorio de caché si no existe
[[ ! -d "$ZSH_SMARTCACHE_DIR" ]] && mkdir -p "$ZSH_SMARTCACHE_DIR"
# FZF configuración base - sin preview por defecto
# === CONFIGURACIÓN FZF ===
# Comando por defecto para FZF - usa fd para respetar .gitignore
export FZF_DEFAULT_COMMAND='fd --type f --hidden --follow --exclude .git'

# Opciones base de FZF (sin preview por defecto para evitar conflictos)
export FZF_DEFAULT_OPTS='--height 60% --layout=reverse --border'

# Cambiar el trigger de completado de ** a ++
export FZF_COMPLETION_TRIGGER='++'

# Opciones globales para completado
export FZF_COMPLETION_OPTS='--border --info=inline'

# FZF para Ctrl+R (historial) - tu configuración actual está perfecta
export FZF_CTRL_T_COMMAND='fd --type f --hidden --follow --exclude .git'
export FZF_CTRL_T_OPTS="
  --preview 'pistol {}'
  --preview-window right:60%:wrap
  --bind 'ctrl-/:toggle-preview'
  --header 'ENTER: seleccionar | Ctrl-/: toggle preview'"

# Configuración para historial (Ctrl+R)
# Cambiado pbcopy a wl-copy para Wayland
export FZF_CTRL_R_OPTS="
  --preview 'echo {}' --preview-window up:3:hidden:wrap
  --bind 'ctrl-/:toggle-preview'
  --bind 'ctrl-y:execute-silent(echo -n {2..} | wl-copy)+abort'
  --color header:italic
  --header 'Press CTRL-Y to copy command into clipboard'"

# Configuración para cambiar directorio (Alt+C)
export FZF_ALT_C_COMMAND='fd --type d --hidden --follow --exclude .git'
export FZF_ALT_C_OPTS="
  --preview 'eza --tree --level=2 --icons=always {}'
  --preview-window right:50%
  --header 'ENTER: cambiar al directorio'"

# === OPCIONES ZSH ===
setopt SHARE_HISTORY
setopt APPEND_HISTORY
setopt INC_APPEND_HISTORY
setopt HIST_IGNORE_DUPS
setopt HIST_IGNORE_ALL_DUPS
setopt HIST_IGNORE_SPACE
setopt HIST_REDUCE_BLANKS

# === PLUGINS con Sheldon ===
eval "$(sheldon source)"

# === HERRAMIENTAS EXTERNAS ===
smartcache eval zoxide init zsh
eval "$(starship init zsh)"
source <(fzf --zsh)

# === ALIASES ===
alias myip="curl -s http://ipecho.net/plain; echo"
alias internet="ping -c 1 8.8.8.8"
alias ls='eza --icons=always'
alias ll='eza --icons=always -1'
alias cat='bat --paging=never --style=rule'

# === FUNCIÓN YAZI ===
function y() {
    local tmp="$(mktemp -t "yazi-cwd.XXXXXX")" cwd
    yazi "$@" --cwd-file="$tmp"
    if [[ -r "$tmp" ]]; then
        IFS= read -r -d '' cwd < "$tmp"
        [[ -n "$cwd" && "$cwd" != "$PWD" ]] && builtin cd -- "$cwd"
    fi
    rm -f -- "$tmp"
}

# === CONFIGURACIÓN DE COMPLETIONS ===
# Mejorar autocompletado con sudo
zstyle ':completion::complete:*' gain-privileges 1
# Hacer completions case-insensitive
zstyle ':completion:*' matcher-list 'm:{a-z}={A-Za-z}'
# Colores en completions
zstyle ':completion:*' list-colors ${(s.:.)LS_COLORS}

```

