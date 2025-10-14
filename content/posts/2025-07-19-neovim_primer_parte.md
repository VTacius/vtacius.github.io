---
date: "2025-07-19T00:00:00Z"
tags:
- configuraciones
- desarrollo
- neovim
title: 'Configurando Neovim: Introducción e inicialización'
---

## Introducción
Esta guía va sobre registrar todas las configuraciones en las que he podido ir trabajando de manera orgánica por algún tiempo.

Esta guía pretende:
  * Ser sencilla
  * Ser mínima
  * Si es posible, configuraciones por defecto

Creamos nuestro esquema de directorios y nos ubicamos en `~/.config/nvim/`:
```bash
mkdir -p ~/.config/nvim/lua/{config,plugins}
cd ~/.config/nvim/
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
```

## config/folding.lua
Algunos lo llaman por su traducción al español, “pliegues”
```lua
-- ~/.config/nvim/lua/config/folding.lua
-- Configuración simple de folding con treesitter

-- Configuración mínima
vim.opt.foldmethod = 'expr'
vim.opt.foldexpr = 'nvim_treesitter#foldexpr()'
vim.opt.foldlevel = 99      -- Empezar con todo abierto
```

## config/options
La mayoría son bastante descriptivas de lo que hacen, y es recomendable tomarse el tiempo de revisarlas para asegurar que son de nuestro gusto. Mi idea fue tomar de base el comportamiento que se suele esperar de vim.

```lua
-- ~/.config/nvim/lua/config/options.lua

-- Opciones generales de Neovim (Que incluso podrían estar en vim)
vim.opt.title = true                -- Busca cambiar el titulo de la ventana en que la que se ejecuta nvim
vim.opt.titlestring = 'nvim - %{expand("%:t")}' 
vim.opt.mouse = "a"                 -- Habilita el soporte del ratón en todos los modos
vim.opt.number = true               -- Muestra números de línea
vim.opt.cursorline = true           -- Resalta la línea en la que nos encontramos
vim.opt.relativenumber = true       -- Muestra números de línea relativos

-- Identación (Otra configuración básica, de toda la vida)
vim.opt.tabstop = 4
vim.opt.shiftwidth = 4
vim.opt.expandtab = true
vim.opt.smartindent = true
vim.opt.autoindent = true

-- Busqueda
vim.opt.hlsearch = true             -- Resalta todas las coincidencias de búsqueda
vim.opt.incsearch = true            -- Muestra las coincidencias de búsqueda a medida que se escribe
vim.opt.ignorecase = true           -- Ignora mayúsculas/minúsculas en la búsqueda
vim.opt.smartcase = true            -- No ignora mayúsculas/minúsculas si la búsqueda contiene mayúsculas

-- Configuración visual
vim.opt.termguicolors = true        -- Habilita true color, esencial para temas modernos
vim.opt.signcolumn = "yes"          -- Siempre muestra un espacio en la columna izquierda para indicadores visuales, lo cual es preferible para consistencia visual 
vim.opt.showtabline = 2             -- Siempre muestra una tab, lo cual es preferible para consistencia visual
--vim.opt.lazyredraw = true         -- No redibuja durante macros. No activar si se usará noice
--  NOTA: Las siguientes configuraciones implican el uso de algo como snacks / noice + lualine, u otros plugins similares 
vim.opt.cmdheight = 0               -- Eliminamos el área de cmdline. Implica el uso de lualine y de algo como noice, para manejar mensajes
vim.opt.cmdwinheight = 1            -- Para usar en conjunto con cmdheight

-- Configuración de comportamiento
vim.opt.clipboard = "unnamedplus"   -- Permite copiar/pegar desde/hacia el portapapeles del sistema
vim.opt.splitright = true           -- Abre nuevas ventanas divididas a la derecha
vim.opt.splitbelow = true           -- Abre nuevas ventanas divididas abajo
vim.opt.wrap = false                -- Deshabilita el ajuste de línea
vim.opt.scrolloff = 5               -- Mantiene 8 líneas de contexto al desplazarse
vim.opt.sidescrolloff = 5           -- Mantiene 8 columnas de contexto al desplazarse horizontalmente
vim.opt.updatetime = 300            -- Tiempo en ms para escribir el archivo swap y disparar eventos CursorHold
vim.opt.undofile = true             -- Habilita el historial de deshacer persistente
vim.opt.undodir = vim.fn.stdpath("data").. "/undodir" -- Directorio para guardar archivos de undo

-- Sobre el autocompletado en cmd 
vim.opt.wildmenu = true
vim.opt.wildoptions = "pum"         -- popup menu + fuzzy matching
vim.opt.wildmode = { "longest:full", "full" }
vim.opt.completeopt = 'menu,menuone,noselect'

-- Para que recuerde donde estaba abierto tal archivo
vim.api.nvim_create_autocmd("BufReadPost", {
  callback = function()
    local mark = vim.api.nvim_buf_get_mark(0, '"')
    local lcount = vim.api.nvim_buf_line_count(0)
    if mark[1] > 0 and mark[1] <= lcount then
      pcall(vim.api.nvim_win_set_cursor, 0, mark)
    end
  end,
})

-- Los siguientes podrían usarse si no se quiere usar noice o lualine, 
-- vim.api.nvim_create_autocmd("RecordingEnter", {
--   callback = function()
--     vim.notify("Recording @" .. vim.fn.reg_recording())
--   end,
-- })
-- 
-- vim.api.nvim_create_autocmd("RecordingLeave", {
--   callback = function()
--     vim.notify("Recording stopped")
--   end,
-- })

```

## config/keymaps
```lua
-- ~/.config/nvim/lua/config/keymaps.lua

local map = vim.keymap.set

-- Para usar con diágnostico
map("n", "<leader>dd", vim.diagnostic.open_float, { desc = "Mostrar diagnóstico" })
map("n", "{d", vim.diagnostic.goto_prev, { desc = "Diagnóstico anterior" })
map("n", "}d", vim.diagnostic.goto_next, { desc = "Siguiente diagnóstico" })

-- Mantener cursor centrado al hacer scroll
map("n", "<C-d>", "<C-d>zz", { desc = "Scroll down and center" })
map("n", "<C-u>", "<C-u>zz", { desc = "Scroll up and center" })

-- Mantener cursor centrado en búsquedas
map("n", "n", "nzzzv", { desc = "Next search result (centered)" })
map("n", "N", "Nzzzv", { desc = "Prev search result (centered)" })

-- Permite mover bloques de contenido, tanto en modo visual como normal, de forma más visual
map("n", "<A-k>", ":m .-2<CR>==", { desc = "Move line up" })
map("n", "<A-j>", ":m .+1<CR>==", { desc = "Move line down" })
map("v", "<A-j>", ":m '>+1<CR>gv=gv", { desc = "Move selection down" })
map("v", "<A-k>", ":m '<-2<CR>gv=gv", { desc = "Move selection up" })

-- Mejora el identado en modo visual
map("v", "<", "<gv", { desc = "Indent left and reselect" })
map("v", ">", ">gv", { desc = "Indent right and reselect" })

-- Guardado más rápido
map({ "i", "x", "n", "s" }, "<C-s>", "<cmd>w<CR><Esc>", { desc = "Save File" })

-- Mejora el comportamiento de J
map("n", "J", "mzJ`z", { desc = "Join lines and keep cursor position" })

--- Borra las highlights
map("n", "<leader>uh", ":nohlsearch<CR>", { desc = "Clear search highlights" })

-- Quick access
map("n", "<leader><space>", function() Snacks.picker.smart() end, { desc = "Smart Find Files" })
map("n", "<leader>,", function() Snacks.picker.buffers() end, { desc = "Buffers" })

-- Explorer
map("n", "<leader>e", function() Snacks.explorer() end, { desc = "Explorer" })

-- Find
map("n", "<leader>fb", function() Snacks.picker.buffers() end, { desc = "Buffers" })
map("n", "<leader>fr", function() Snacks.picker.recent() end, { desc = "Recent Files" })
map("n", "<leader>ff", function() Snacks.picker.files() end, { desc = "Find Files (all)" })
map("n", "<leader>fg", function() Snacks.picker.git_files() end, { desc = "Find Git Files" })

-- Search
map("n", "<leader>sg", function() Snacks.picker.grep() end, { desc = "Grep (live)" })
map({ "n", "x" }, "<leader>sw", function() Snacks.picker.grep_word() end, { desc = "Grep Word/Selection" })
map("n", "<leader>sW", function() Snacks.picker.grep_word({ word = vim.fn.expand("<cword>") }) end, { desc = "Grep Word Under Cursor" })
map("n", "<leader>sh", function() Snacks.picker.help() end, { desc = "Help Pages" })
map("n", "<leader>sk", function() Snacks.picker.keymaps() end, { desc = "Keymaps" })
map("n", "<leader>sc", function() Snacks.picker.command_history() end, { desc = "Command History" })
map("n", "<leader>sd", function() Snacks.picker.diagnostics() end, { desc = "Diagnostics" })
map("n", "<leader>sy", function() Snacks.picker.cliphist() end, { desc = "Cliphist" })

-- Code Action
map("n", "<leader>ca", vim.lsp.buf.code_action, { desc = "Code Action" })

-- LSP
map("n", "<leader>o", function() Snacks.picker.lsp_symbols() end, { desc = "Outline/Symbols" })
map("n", "gd", function() Snacks.picker.lsp_definitions() end, { desc = "Goto Definition" })
map("n", "gr", function() Snacks.picker.lsp_references() end, { desc = "Find References" })
map("n", "gI", function() Snacks.picker.lsp_implementations() end, { desc = "Goto Implementation" })
map("n", "gy", function() Snacks.picker.lsp_type_definitions() end, { desc = "Goto Type Definition" })

-- Notifications
map("n", "<leader>nh", ":NoiceHistory<CR>", { desc = "Message History" })
map("n", "<leader>nd", ":NoiceDismiss<CR>", { desc = "Dismiss Messages" })
map("n", "<leader>nl", ":NoiceLast<CR>", { desc = "Show last message" })

-- Terminal 
map("n", "<leader>tt", function() Snacks.terminal() end, { desc = "Terminal" })

-- Buffer
map("n", "<leader>bd", function() Snacks.bufdelete() end, { desc = "Delete Buffer" })
map("n", "}b", '<cmd>bnext<CR>', { desc = "Next Buffer" })
map("n", "{b", '<cmd>bprevious<CR>', { desc = "Prev Buffer" })
map("n", "<S-l>", '<cmd>bnext<CR>', { desc = "Next Buffer" })
map("n", "<S-h>", '<cmd>bprevious<CR>', { desc = "Prev Buffer" })

-- Words navigation
map("n", "}}", function() Snacks.words.jump(vim.v.count1) end, { desc = "Next Reference" })
map("n", "{{", function() Snacks.words.jump(-vim.v.count1) end, { desc = "Prev Reference" })

-- Extras
map("n", "<leader>z", function() Snacks.zen() end, { desc = "Zen Mode" })
map("n", "<leader>.", function() Snacks.scratch() end, { desc = "Scratch Buffer" })
map("n", "<leader>S", function() Snacks.scratch.select() end, { desc = "Select Scratch" })
map("n", "<leader>cR", function() Snacks.rename.rename_file() end, { desc = "Rename File" })

-- GIT
map("n", "<leader>gb", function() Snacks.gitbrowse() end, { desc = "Git Browse" })
map("n", "<leader>gl", function() Snacks.picker.git_log() end, { desc = "Git Log" })
map("n", "<leader>gs", function() Snacks.picker.git_status() end, { desc = "Git Status" })
map("n", "<leader>gL", function() Snacks.picker.git_log_file() end, { desc = "Git Log (file)" })

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

require("config.options")       -- Carga las opciones generales del editor
require("config.diagnostics")   -- Carga las opciones de diagnóstico del LSP
require("config.folding")       -- Carga la configuración del folding 
require("config.keymaps")       -- Carga nuestros keymaps

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
