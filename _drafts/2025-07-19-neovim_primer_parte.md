---
title: "Configurando Neovim: Introducción e inicialización"
tags:
    - configuraciones
    - desarrollo
    - neovim
---

## Introducción
Esta guía va sobre registrar todas las configuraciones en las que he podido ir trabajando de manera orgánica por algún tiempo.

Esta guía pretende:
  * Ser sencilla
  * Ser mínima
  * Si es posible, configuraciones por defecto

Creamos nuestro esquema de directorios y nos ubicamos en `~/.config/nvim/`:
```bash
mkdir -p ~/.config/nvim/lua/{keymap,config,plugins}
cd ~/.config/nvim/
```

## config/folding.lua
Pues sí, haremos que esta vez funcione
```lua
-- ~/.config/nvim/lua/config/folding.lua
-- Configuración simple de folding con treesitter

-- Configuración mínima
vim.opt.foldmethod = 'expr'
vim.opt.foldexpr = 'nvim_treesitter#foldexpr()'
vim.opt.foldlevel = 99      -- Empezar con todo abierto

-- Atajo simple: Space Space para abrir/cerrar
vim.keymap.set('n', '<Space><Space>', 'za', { desc = 'Toggle fold' })
```

## config/diagnostics
Configura lo que pasará cuando un LSP detecte errores

```lua
-- ~/.config/nvim/lua/config/diagnostics.lua

-- Configuración básica de diagnósticos (En mi caso, noice manejará la presentación)
vim.diagnostic.config({
  virtual_text = {
    format = function(diagnostic)
      local icons = { "󰅚", "󰀪", "󰋽", "󰌶" }
      return string.format("%s %s", icons[diagnostic.severity], diagnostic.message)
    end,
  },
  signs = {
    text = { "󰅚", "󰀪", "󰋽", "󰌶" },
  },
  severity_sort = true,
})

-- Keymaps esenciales para diagnósticos
local map = vim.keymap.set
map("n", "<leader>d", vim.diagnostic.open_float, { desc = "Mostrar diagnóstico" })
map("n", "]d", vim.diagnostic.goto_next, { desc = "Siguiente diagnóstico" })
map("n", "[d", vim.diagnostic.goto_prev, { desc = "Diagnóstico anterior" })

-- -- Toggle simple de virtual text (Opcional)
-- local function toggle_virtual_text()
--   local current = vim.diagnostic.config().virtual_text
--   vim.diagnostic.config({ virtual_text = not current })
--   vim.notify("Virtual text: " .. (current and "OFF" or "ON"))
-- end

-- map("n", "<leader>dt", toggle_virtual_text, { desc = "Toggle diagnósticos" })
--
```

## config/options
Vale mucho la pena revisar cada opción, pero el principio que sigue es configurar el comportamiento básico que se espera de `vim`

```lua
-- ~/.config/nvim/lua/config/options.lua

-- Opciones generales de Neovim
vim.opt.number = true             -- Muestra números de línea
vim.opt.relativenumber = true     -- Muestra números de línea relativos
vim.opt.mouse = "a"               -- Habilita el soporte del ratón en todos los modos
vim.opt.clipboard = "unnamedplus" -- Permite copiar/pegar desde/hacia el portapapeles del sistema
vim.opt.splitright = true         -- Abre nuevas ventanas divididas a la derecha
vim.opt.splitbelow = true         -- Abre nuevas ventanas divididas abajo
vim.opt.wrap = false              -- Deshabilita el ajuste de línea
vim.opt.scrolloff = 8             -- Mantiene 8 líneas de contexto al desplazarse
vim.opt.sidescrolloff = 8         -- Mantiene 8 columnas de contexto al desplazarse horizontalmente
vim.opt.hlsearch = true           -- Resalta todas las coincidencias de búsqueda
vim.opt.incsearch = true          -- Muestra las coincidencias de búsqueda a medida que se escribe
vim.opt.ignorecase = true         -- Ignora mayúsculas/minúsculas en la búsqueda
vim.opt.smartcase = true          -- No ignora mayúsculas/minúsculas si la búsqueda contiene mayúsculas
vim.opt.updatetime = 300          -- Tiempo en ms para escribir el archivo swap y disparar eventos CursorHold
vim.opt.signcolumn = "yes"        -- Muestra siempre la columna de signos (para diagnósticos, git, etc.)
vim.opt.undofile = true           -- Habilita el historial de deshacer persistente
vim.opt.undodir = vim.fn.stdpath("data").. "/undodir" -- Directorio para archivos de deshacer

-- Configuración de indentación: 4 espacios por defecto
vim.opt.tabstop = 4         -- Un tabulador se muestra como 4 espacios
vim.opt.shiftwidth = 4      -- Las operaciones de indentación usan 4 espacios
vim.opt.expandtab = true    -- Inserta espacios en lugar de tabuladores al presionar Tab
vim.opt.softtabstop = 4     -- Las teclas Tab/Backspace se comportan como si fueran 4 espacios

-- -- Parece ser la mejor forma para el autocompletado de comandos
vim.opt.wildmenu = true
vim.opt.wildmode = { "longest:full", "full" }
vim.opt.wildoptions = "pum"  -- popup menu + fuzzy matching
vim.opt.completeopt = 'menu,menuone,noselect'
vim.opt.cmdheight= 0
vim.opt.cmdwinheight= 1

---- Para que recuerde donde estaba abierto tal archivo
vim.api.nvim_create_autocmd("BufReadPost", {
  callback = function()
    local mark = vim.api.nvim_buf_get_mark(0, '"')
    local lcount = vim.api.nvim_buf_line_count(0)
    if mark[1] > 0 and mark[1] <= lcount then
      pcall(vim.api.nvim_win_set_cursor, 0, mark)
    end
  end,
})

vim.lsp.set_log_level("warn")
vim.opt.showtabline = 2         -- Siempre muestra una tab
```

## lua.init
Ahora unimos todo en nuestro lua.init
```lua
-- ~/.config/nvim/init.lua

-- Establece el líder de mapeo de teclas (<leader>) (Usualmente la barra espaciadora)
vim.g.mapleader = " "
vim.g.maplocalleader = "\\"

-- Habilita los colores verdaderos en el terminal, esencial para los temas modernos
vim.opt.termguicolors = true

require("config.options")       -- Carga las opciones generales del editor desde lua/config/options.lua
require("config.diagnostics")   -- Carga las opciones de diagnóstico del LSP desde lua/config/diagnostics.lua
require("config.folding")       --Carga la configuración del folding gracias a treesitter

-- Bootstrap lazy.nvim
-- Este bloque de código asegura que lazy.nvim esté instalado.
-- Si no lo está, lo clona desde GitHub.
local lazypath = vim.fn.stdpath("data").. "/lazy/lazy.nvim"
if not (vim.uv or vim.loop).fs_stat(lazypath) then
  vim.fn.system({
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable", -- última versión estable
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

-- Configura lazy.nvim para cargar tus plugins desde `lua/plugins/`
require("lazy").setup({
  spec = {
    { import = "plugins" },
  },
  -- Opciones adicionales para lazy.nvim
  checker = {
    enabled = true,     -- Habilita la verificación automática de actualizaciones de plugins
    frequency = 86400,  -- Frecuencia de verificación en segundos (cada 24 horas)
  },
})

```

Algo que también vale mucho la pena reiniciar `nvim` y ver los primeros cambios, aunque veremos en consola un mensaje como el siguiente:

```
Se ha detectado un error al procesar /home/alortiz/.config/nvim/init.lua:
No specs found for module "plugins"
```

Causado porque aún no hay plugins que cargar, pero que lo arreglaremos en la siguiente entrada
