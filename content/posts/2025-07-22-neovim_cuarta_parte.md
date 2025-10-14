---
date: "2025-07-22T00:00:00Z"
tags:
- configuraciones
- desarrollo
- neovim
title: 'Configurando Neovim: UI'
---

## Introducción
Gracias a los post anteriores, `nvim` ahora tiene mayores funcionalidades presentes. 

Esta parte va sobre algunas herramientas que, como `snacks`, pueden ser una mejora significativa a cosas que ya se podían hacer, o que como `noise`, practicamente tranforman a `nvim` en otra cosa.

## Herramientas

### snacks
He llegado a `snacks` después de haber usado `fzf-lua` y `aerial` por un tiempo, estando complacido de sus capacidades. Pero decidí probar `snacks`, con quién no solo tengo las funcionalidades que usaba de estos plugins, sino que incluso algunas más.

De hecho, recomiendo que se consulte la [lista oficial](https://github.com/folke/snacks.nvim?tab=readme-ov-file#-features) en busca de otras caracterísiticas que puedan resultar interesantes.

```lua
-- ~/.config/nvim/lua/config/snacks.lua
return {
  -- Performance: deshabilita features en archivos grandes
  bigfile = {
    enabled = true,
    size = 1024 * 1024, -- 1MB
  },

  -- Renderiza archivos rápido antes de cargar plugins
  quickfile = { enabled = true },

  -- Esta es la funcionalidad que más uso: Es capaz de darnos muchas de las caracterísiticas de fzf-lua
  picker = {
    enabled = true,
    layout = {
      preset = "telescope", -- "select", "ivy", "telescope"
    },
    sources = {
      explorer = {
        hidden = true,      -- Cambiar a true para ver archivos ocultos
        ignored = false,    -- Cambiar a true para ver .gitignore files
      },
    },
  },

  -- File explorer (reemplaza netrw)
  explorer = {
    enabled = true,
    replace_netrw = true,
  },

  -- vim.ui.input mejorado
  input = {
    enabled = true,
    relative = "editor",
    position = "float",
    border = "rounded",
  },

  -- Statuscolumn con git signs, folds, etc
  statuscolumn = {
    enabled = true,
    left = { "mark", "sign" },
    right = { "fold", "git" },
    folds = {
      open = true,
      git_hl = true,
    },
  },

  -- Terminal flotante
  terminal = {
    enabled = true,
    win = {
      position = "float",
      border = "rounded",
      width = 0.8,
      height = 0.8,
    },
  },

  -- Extras útiles 
  zen = { enabled = true },
  rename = { enabled = true },
  scratch = { enabled = true },
  bufdelete = { enabled = true },
  gitbrowse = { enabled = true },

  -- Animaciones (requerido por otros módulos)
  animate = {
    enabled = true,
    fps = 60,
  },

  -- Estilos globales
  styles = {
    notification = {
      wo = { wrap = true },
    },
  },
}
```

```lua
-- ~/.config/nvim/lua/plugins/snacks.lua

return {
  'folke/snacks.nvim',
  lazy = false,
  priority = 1000,
  config = function()
    require("snacks").setup(require("config.snacks"))
  end,
}
```

## UI
Las de acá son más bien visuales, y no por eso dejan de ser útiles. 

Sin embargo, cada una añade un extra de "disrupción", con lo que me refiero es que cada vez más nos alejamos de la comodida de `Vim` 

### lualine
Sé que esta es la razón por la cuál muchos buscan guías sobre como configurar NeoVim, porque ese fue mi caso.
```lua
-- ~/.config/nvim/lua/config/lualine.lua

-- Lo único que necesitamos es agregar la señal de grabación de macro
--   parece ser lo único que se pierde en medio de toda esta configuración
--   Necesita noice
return {
  options = {
    theme = "catppuccin"
  },
  sections = {
    lualine_x = {
      {
        require("noice").api.statusline.mode.get,
        cond = function()
          local mode = require("noice").api.statusline.mode.get()
          return mode and mode:find("recording") ~= nil
        end,
        color = "WarningMsg",
      },
      'encoding',
      'fileformat',
      'filetype'
    },
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
}
```

### Bufferline 
Esta es sin duda la segunda razón por la que cualquiera empieza con esto de configurar `nvim`. 
Aunque visual, lo cierto es que este plugin necesita un cambio para muchos: El uso de buffers en lugar de tabs

```lua
-- ~/.config/nvim/lua/config/bufferline.lua

return {
    options = {
        diagnostics = "nvim_lsp",
        separator_style = "slope"
    }
}
```

```lua
-- ~/.config/nvim/lua/plugins/bufferline.lua

return {
    'akinsho/bufferline.nvim',
    version = "*",
    dependencies = 'nvim-tree/nvim-web-devicons',
    config = function()
        require('bufferline').setup(require('config.bufferline'))
    end
}
```

### noice.lua
De lejos, la más innecesariamente necesaria de todas. Pero llegando a este punto, sería más problemático no ponerla 

```lua
-- ~/.config/nvim/lua/config/noice.lua
return {
  -- Solo override de markdown para LSP - el resto usa defaults
  lsp = {
    override = {
      ["vim.lsp.util.convert_input_to_markdown_lines"] = true,
      ["vim.lsp.util.stylize_markdown"] = true,
      ["cmp.entry.get_documentation"] = true,
    },
  },
  
  presets = {
    bottom_search = true,         -- Búsqueda clásica en la parte inferior
    command_palette = true,       -- Cmdline y popupmenu juntos
    long_message_to_split = true, -- Mensajes largos en split
    lsp_doc_border = true,        -- Bordes en documentación LSP
  },
}
```

```lua
-- ~/.config/nvim/lua/plugins/noice.lua

return {
  "folke/noice.nvim",
  event = "VeryLazy",
  dependencies = {
    "MunifTanjim/nui.nvim",
    "rcarriga/nvim-notify",
  },
  config = function()
    require("noice").setup(require("config.noice"))
  end,
}
```
