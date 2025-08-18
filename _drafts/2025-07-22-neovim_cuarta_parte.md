---
title: "Configurando Neovim: UI"
tags:
    - configuraciones
    - desarrollo
    - neovim
---

## Introducción
Gracias a los post anteriores, `NeoVim` ahora tiene mayores funcionalidades presentes. Esta última parte va sobre algunas herramientas que, como arial, pueden ser una mejora significativa a cosas que ya se podían hacer, o como `noise` que casi tranforman a `NeoVim` en otra cosa.

## Herramientas

### Aerial
Aerial ha sido un complemento bastante sorpresivo. No sabía que lo necesitaba hasta que lo instalé, y espero que con esta configuración también sea el caso para el lector
```lua
-- ~/.config/nvim/lua/config/aerial.lua
return {
  backends = { "treesitter", "lsp", "markdown" },
  layout = {
    max_width = { 40, 0.25 },
    min_width = 30,
    default_direction = "prefer_right",
  },
  
  attach_mode = "window",
  close_on_select = false,
  filter_kind = {
    "Class",
    "Constructor", 
    "Enum",
    "Function",
    "Interface",
    "Method",
    "Module",
    "Struct",
  },

  highlight_closest = true,
  highlight_on_jump = 300,
  
  -- Abrir automáticamente para archivos con muchos símbolos
  open_automatic = function(bufnr)
    local count = require("aerial").num_symbols(bufnr)
    return count > 5 and vim.api.nvim_buf_line_count(bufnr) > 80
  end,
}
```

```lua
-- ~/.config/nvim/lua/keymap/aerial.lua

local aerial = require("aerial")

local function map(mode, lhs, rhs, opts)
  opts = opts or {}
  opts.silent = opts.silent ~= false
  vim.keymap.set(mode, lhs, rhs, opts)
end

-- Toggle aerial (abrir/cerrar) - El principal
-- No hay equivalente nativo para vista de símbolos
map("n", "<leader>a", "<cmd>AerialToggle<CR>", { desc = "Toggle Aerial" })

-- Navegar símbolos (siguiente/anterior)
-- Reemplaza: ]] y [[ pero más inteligente
-- Atajos alternativos más cómodos para teclado latino
map("n", "<leader>j", "<cmd>AerialNext<CR>", { desc = "Next symbol" })
map("n", "<leader>k", "<cmd>AerialPrev<CR>", { desc = "Previous symbol" })

-- Estos son los originales 
-- map("n", "]s", "<cmd>AerialNext<CR>", { desc = "Next symbol" })
-- map("n", "[s", "<cmd>AerialPrev<CR>", { desc = "Previous symbol" })

vim.api.nvim_create_autocmd("FileType", {
  pattern = "aerial",
  callback = function()
    local buf = vim.api.nvim_get_current_buf()
    local aerial = require("aerial")
    
    -- Enter: saltar al símbolo
    map("n", "<CR>", function() aerial.select() end, { buffer = buf })
    
    -- p: preview sin saltar (navegar pero manteniendo foco en aerial)
    map("n", "p", function() aerial.select({ jump = false }) end, { buffer = buf })
    
    -- q: cerrar aerial
    map("n", "q", "<cmd>AerialClose<CR>", { buffer = buf })
    
    -- Folding básico
    map("n", "za", "<cmd>AerialToggle<CR>", { buffer = buf })
    map("n", "zM", "<cmd>AerialCloseAll<CR>", { buffer = buf })
    map("n", "zR", "<cmd>AerialOpenAll<CR>", { buffer = buf })
  end,
})
```

```lua
-- ~/.config/nvim/lua/plugins/aerial.lua

return {
  "stevearc/aerial.nvim",
  dependencies = {
    "nvim-treesitter/nvim-treesitter",
    "nvim-tree/nvim-web-devicons",
  },
  config = function()
    local opts = require("config.aerial")
    require("aerial").setup(opts)
    require("keymap.aerial")
  end,
  -- Carga bajo demanda para mejor rendimiento
  cmd = { "AerialToggle", "AerialOpen", "AerialClose", "AerialNext", "AerialPrev" },
  keys = {
    { "<leader>a", desc = "Aerial Toggle" },
    { "<leader>j", desc = "Next symbol" },
    { "<leader>k", desc = "Previous symbol" },
  },
}
```

### fzf-lua
Para este, incluso necesitaremos un par de dependencias en el sistema. Nada del otro mundo, y valdrá mucho la pena:

```bash
dnf install fd rg bat delta
```
Además de un par de dependencias de plugins. Como dije, valdrá la pena:

#### render-markdown
Nos permite una previsualización de markdown. Es una caracterísitica que vale mucho la pena, sobre todo si se trabaja con blogs estáticos, como es mi caso.
```lua
-- ~/.config/nvim/lua/plugins/render-markdown.lua

return {
    'MeanderingProgrammer/render-markdown.nvim',
    dependencies = { 'nvim-treesitter/nvim-treesitter', 'nvim-tree/nvim-web-devicons' }, -- if you prefer nvim-web-devicons
    ---@module 'render-markdown'
    ---@type render.md.UserConfig
    opts = {},
}
```

#### nvim-treesitter-context
La barrita esa del contexto, jajaja, que nombre tan misterioso

```lua
-- ~/.config/nvim/lua/plugins/treesitter-context.lua

return {
    'nvim-treesitter/nvim-treesitter-context'
}
```

#### nvim-dap
Estas son palabras mayores.
```lua
-- ~/.config/nvim/lua/plugins/nvim-dap.lua

return {
    'mfussenegger/nvim-dap'
}
```

Y terminamos, con al fin, lo que queríamos instalar.

Extrañaba hacer eso de instalar dependencies manualmente. Me recuerdan a mis primeros días con OpenSuSE
```lua

-- ~/.config/nvim/lua/config/fzf-lua.lua
{
  { "fzf-native", "borderless" }
}

```

```lua
-- ~/.config/nvim/lua/keymap/fzf-lua.lua

local function map(mode, lhs, rhs, opts)
  opts = opts or {}
  opts.silent = opts.silent ~= false
  vim.keymap.set(mode, lhs, rhs, opts)
end

map("n", "<leader>ff", "<cmd>FzfLua files<CR>", { desc = "Find files" })        -- Archivos - El más usado
map("n", "<leader>fg", "<cmd>FzfLua live_grep<CR>", { desc = "Live grep" })     -- Grep en todo el proyecto - Segundo más usado  
map("n", "<leader>fb", "<cmd>FzfLua buffers<CR>", { desc = "Buffers" })         -- Buffers abiertos
map("n", "<leader>fs", "<cmd>FzfLua lsp_document_symbols<CR>", { desc = "Document symbols" })   -- Símbolos del documento (funciones, clases, etc)

map("n", "<leader>/", "<cmd>FzfLua lgrep_curbuf<CR>", { desc = "Search in buffer" })    -- Buscar en buffer actual (como / pero con fuzzy)
map("n", "<leader>fr", "<cmd>FzfLua oldfiles<CR>", { desc = "Recent files" })           -- Archivos recientes
map("n", "<leader>f.", "<cmd>FzfLua resume<CR>", { desc = "Resume last search" })       -- Resume última búsqueda (muy útil!)

map("n", "<leader>f?", "<cmd>FzfLua builtin<CR>", { desc = "FzfLua builtins" })         -- Ver todos los comandos disponibles de FzfLua
```

```lua
-- ~/.config/nvim/lua/plugins/fzf-lua.lua

return {
  "ibhagwan/fzf-lua",
  -- optional for icon support
  dependencies = { "nvim-tree/nvim-web-devicons" },
  -- or if using mini.icons/mini.nvim
  -- dependencies = { "echasnovski/mini.icons" },
  config = function()
    require("fzf-lua").setup(require("config.fzf-lua"))
    require("keymap.fzf-lua")
  end
}
```

## UI
Las de acá son más bien visuales, y no por eso dejan de ser útiles. 

Sin embargo, cada una añade un extra de "disrupcción", con lo que me refiero es que cada vez más nos alejamos de la comodida de `Vim` 

### lualine
Sé que esta es la razón por la cuál muchos buscan guías sobre como configurar NeoVim, porque ese fue mi caso.
```lua
-- ~/.config/nvim/lua/config/lualine.lua

return {
  {
    globalstatus = true,            -- Muestra la línea de estado incluso cuando solo hay una ventana abierta
    icons_enabled = true,
    always_divide_middle = true,    -- Asegura que las secciones centrales no ocupen toda la línea de estado
    theme = require("config.colorscheme"),              -- Aplica el tema configurado globalmente
    section_separators = { left = '', right = ''},    -- Separadores estéticos para secciones
    component_separators = { left = '', right = ''},  -- Separadores estéticos para componentes
  },
  sections = {
    lualine_a = {'mode'},                           -- Muestra el modo Vim actual
    lualine_b = {'branch', 'diff', 'diagnostics'},  -- Muestra la rama de Git, el estado del diff del archivo y los diagnósticos LSP
    lualine_c = {'filename'},                       -- Muestra el nombre del archivo actual
    lualine_x = {
      {
        "aerial",       -- Muestra aerial, que es mejor
        sep = "  ",    -- The separator to be used to separate symbols in status line.
      },
    },
    -- lualine_y = {'filetype', 'encoding', 'fileformat'}, -- Muestra el tipo de archivo, la codificación y el formato del archivo
    lualine_y = {'progress'},                       -- Muestra el progreso dentro del archivo
    lualine_z = {'location'},                       -- Muestra la línea y columna del cursor
  },
}
```

```lua
-- ~/.config/nvim/lua/plugins/lualine.lua

return {
  'nvim-lualine/lualine.nvim',
  dependencies = { 'nvim-tree/nvim-web-devicons' }, -- Para iconos en la barra de estado
  config = function()
    require('lualine').setup(require("config.lualine"))
  end,
  after = require("config.colorscheme"),
}
```

### Tabby.nvim
Siendo sincero, `bufferline` con en modo `tabs` podría servir, pero no voy a desperdiciar la noche que me pasé trabajando en esta configuración
```lua
-- ~/.config/nvim/lua/plugins/tabby.lua
local theme = {
  fill        = 'TabLineFill',
  head        = 'TabLine',
  current_tab = 'lualine_a_normal',    -- Tab activa (debería ser colorida)
  tab         = 'TabLine',              -- Tab inactiva (gris/oscura)
  win         = 'lualine_a_insert',
  tail        = 'TabLine',
}

local opts = {
  tab_name = {
    mode = 'unique',
  },
  buf_name = {
    mode = 'unique',
  },
}

local line = function(line)
  local webdevicons = require('nvim-web-devicons')
  local tab_name = require('tabby.feature.tab_name')
  return {
    line.tabs().foreach(function(tab)
      local is_current = tab.is_current()
      local hl_group = is_current and theme.current_tab or theme.tab  -- Cambio aquí: theme.tab en lugar de theme.fill
      local tab_bg = vim.fn.synIDattr(vim.fn.hlID(hl_group), 'bg')
      local tab_fg = vim.fn.synIDattr(vim.fn.hlID(hl_group), 'fg')
      -- Obtener información del buffer
      local win = tab.current_win()
      local buf = win.buf().id
      -- IMPORTANTE: Pasar la configuración al obtener el nombre
      local name = tab_name.get(tab.id, opts.tab_name)
      -- Para el icono, necesitamos el nombre completo del archivo
      local full_name = vim.api.nvim_buf_get_name(buf)
      local file_name = vim.fn.fnamemodify(full_name, ':t')
      -- Obtener icono y color
      local icon, icon_color = webdevicons.get_icon_color(file_name, vim.bo[buf].filetype, { default = true })
      icon = icon or ''
      -- Color del icono: solo con color real si es la tab activa
      local icon_hl_fg = is_current and (icon_color or tab_fg) or tab_fg
      return {
        line.sep('', hl_group, theme.fill),
        { ' ' .. tab.number() .. ' ', hl = hl_group },
        { icon .. ' ', hl = { fg = icon_hl_fg, bg = tab_bg } },
        { name .. ' ', hl = hl_group },
        tab.close_btn(''),
        line.sep('', hl_group, theme.fill),
        hl = hl_group,
        margin = ' ',
      }
    end),
    -- Espaciador
    line.spacer(),
    -- Windows de la tab actual
    line.wins_in_tab(line.api.get_current_tab()).foreach(function(win)
      local win_hl = theme.win
      return {
        line.sep('', win_hl, theme.fill),
        win.is_current() and '' or '',
        ' ' .. win.buf_name() .. ' ',
        line.sep('', win_hl, theme.fill),
        hl = win_hl,
        margin = ' ',
      }
    end),
    -- Pie
    {
      line.sep('', theme.tail, theme.fill),
      { '  ', hl = theme.tail },
    },
    hl = theme.fill,
  }
end

return {
  'nanozuki/tabby.nvim',
  dependencies = {
    'nvim-tree/nvim-web-devicons',
  },
  config = function()
    require('tabby').setup({
      options = opts,
      line = line,
    })
  end,
}
```

### noice.lua
De lejos, la más innecesariamente necesaria de todas. Sin embargo, algo en neovim y todos los plugins y configuraciones hacen que muchos mensajes puedan perderse. A cambio, y esto debo admitirlo, se pierde aquella esencia de `VIM` para trabajar sin distracciones.

```lua
-- lua/config/noice.lua
return {
  -- Solo override de markdown para LSP - el resto usa defaults
  lsp = {
    override = {
      ["vim.lsp.util.convert_input_to_markdown_lines"] = true,
      ["vim.lsp.util.stylize_markdown"] = true,
      ["cmp.entry.get_documentation"] = true,
    },
  },
  
  -- Presets profesionales - estos mejoran la experiencia por defecto
  presets = {
    bottom_search = true,         -- Búsqueda clásica en la parte inferior
    command_palette = true,       -- Cmdline y popupmenu juntos
    long_message_to_split = true, -- Mensajes largos en split
    lsp_doc_border = true,        -- Bordes en documentación LSP
  },
}
```

```lua
-- lua/keymaps/noice.lua
local map = vim.keymap.set

-- Navegación en el historial de mensajes
map("n", "<leader>nl", "<cmd>NoiceHistory<CR>", { desc = "Ver historial de mensajes" })

-- Toggle de noice (por si necesitas deshabilitarlo temporalmente)
map("n", "<leader>nd", "<cmd>NoiceDisable<CR>", { desc = "Deshabilitar Noice" })
map("n", "<leader>ne", "<cmd>NoiceEnable<CR>", { desc = "Habilitar Noice" })

-- Ver último mensaje en detalle
map("n", "<leader>nm", "<cmd>NoiceLast<CR>", { desc = "Último mensaje" })
```

```lua

-- lua/plugins/noice.lua
return {
  "folke/noice.nvim",
  event = "VeryLazy",
  dependencies = {
    "MunifTanjim/nui.nvim",
    "rcarriga/nvim-notify",
  },
  config = function()
    require("noice").setup(require("config.noice"))
    require("keymap.noice")
  end,
}
```
