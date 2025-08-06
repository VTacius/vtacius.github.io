---
title: "Configurando Neovim: Estilos"
tags:
    - configuraciones
    - desarrollo
    - neovim
---

## Introducción
Puede parecer poca cosa, pero el estilo adecuado puede mejorar por mucho nuestras condiciones de trabajo. 

Para escoger los temas, me he basado en el hecho que estos tengan el más amplio soporte en otras aplicaciones. De esta forma, podría usar `catppuccin` tanto en Neovim como en Alacritty

## web-devicons
```lua
-- ~/.config/nvim/lua/plugins/nvim-web-devicons.lua

return {
  'nvim-tree/nvim-web-devicons',
  lazy = true, 
  config = function()
    require('nvim-web-devicons').setup { }
  end,
}

```

Si guardamos sin cerrar, podremos ver en nuestra barra este mensaje:
```
# Config Change Detected. Reloading...

- **changed**: `~/.config/nvim/lua/plugins/nvim-web-devicons.lua`
```

Que será una constante en los siguientes plugins

Además, si reiniciamos `nvim`, veremos que ya no muestra el mensaje de error del final de la guía anterior

## colorscheme
Este será un pequeño truco para facilitar, sin complicar en demasía, la configuración de más temas

```lua
-- ~/.config/nvim/lua/config/colorscheme.lua

-- return "catppuccin"
-- return "nordic"
return "onenord"
```

## Temas
Por ahora solo he probado tres: `catppuccin`, `nord`, `onenord`, los dos últimos basados en la paleta de colores [Nord](https://www.nordtheme.com/)

### Catppuccin
```lua
-- ~/.config/nvim/lua/config/catppuccin.lua

return {
  flavour = "latte",      -- Opciones: "latte", "frappe", "macchiato", "mocha"
  background = {          -- Configuración de fondo para modos claro/oscuro
    light = "latte",
    dark = "mocha",
  },
  transparent_background = false, -- Establecer en `true` si se desea un fondo transparente
  term_colors = true,             -- Habilita la configuración de colores del terminal
  compile = {
    enabled = true,               -- Habilita la compilación del tema para un inicio más rápido
    path = vim.fn.stdpath("cache").. "/catppuccin", -- Ruta de caché para el tema compilado
  },
  ntegrations = {        -- Integraciones con otros plugins para una tematización cohesiva
    cmp = true,           -- Integración con blink.cmp
    aerial = true,
    treesitter = true,    -- Integración con nvim-treesitter
    -- Añadir otras integraciones según los plugins instalados
  },
}
```

```lua
-- ~/.config/nvim/lua/plugins/catppuccin.lua

return {
  "catppuccin/nvim",
  name = "catppuccin",
  priority = 1000,
  lazy = false,
  config = function()
    local colorscheme = require("config.colorscheme")
    if colorscheme == "catppuccin" then
      require(colorscheme).setup(require("config.catppuccin"))
      vim.cmd.colorscheme(colorscheme)
    end
  end
}

```

### Nordic
```lua
-- ~/.config/nvim/lua/config/nordic.lua

return {
  theme = "nordic",         -- Al parece, no tiene más opciones 
  bold_keywords = true,
  italic_comments = true,
  transparent = {
    bg = false,             -- Enable transparent background.
    float = true,           -- Enable transparent background for floating windows.
  },
  bright_border = false,    -- Enable brighter float border.
  reduced_blue = true,
  noice = {
    style = 'flat',         -- Available styles: `classic`, `flat`.
  },
  cursorline = {            -- Modifica como se ve la selección con mouse
    theme = "dark",         -- o "dark"
  },
}
```

```lua
-- ~/.config/nvim/lua/plugins/nordic.lua

return {
  "AlexvZyl/nordic.nvim",
  lazy = false,
  priority = 1000,
  lazy = false,
  config = function()
    local colorscheme = require("config.colorscheme")
    if colorscheme == "nordic" then
      require(colorscheme).load(require("config.nordic"))
      vim.cmd.colorscheme(colorscheme)
    end
  end,
}
```

### onenord
```lua
-- ~/.config/nvim/lua/config/onenord.lua

return {
  theme = light, -- "dark" or "light". Alternatively, remove the option and set vim.o.background instead
  -- borders = false, -- Split window borders
  -- fade_nc = false, -- Fade non-current windows, making them more distinguishable
  -- -- Style that is applied to various groups: see `highlight-args` for options
  -- styles = {
  --   comments = "NONE",
  --   strings = "NONE",
  --   keywords = "NONE",
  --   functions = "NONE",
  --   variables = "NONE",
  --   diagnostics = "underline",
  -- },
  -- disable = {
  --   background = false, -- Disable setting the background color
  --   float_background = false, -- Disable setting the background color for floating windows
  --   cursorline = false, -- Disable the cursorline
  --   eob_lines = true, -- Hide the end-of-buffer lines
  -- },
  -- -- Inverse highlight for different groups
  -- inverse = {
  --   match_paren = false,
  -- },
  -- custom_highlights = {}, -- Overwrite default highlight groups
  -- custom_colors = {}, -- Overwrite default colors
}

```

```lua
-- ~/.config/nvim/lua/plugins/onenord.lua

return {
  "rmehri01/onenord.nvim",
  --name = "onenord",
  priority = 1000,
  lazy = false,
  config = function()
    local colorscheme = require("config.colorscheme")
    if colorscheme == "onenord" then
      require(colorscheme).setup(require("config.onenord"))
      vim.cmd.colorscheme(colorscheme)
    end
  end
}
```

En este punto se puede reiniciar. y puede verse como el tema `onenord` toma el control de nvim una vez se termina de instalar. Creo que `onenord` es un muy buen tema, para que negarlo. 

Cambiando en `~/.config/nvim/lua/config/catppuccin.lua` podemos ver como es la versión light de `catppuccin`, o `nordic`, que puede ser lo que se quiera en algunos casos

De igual forma, se pueden seguir agregando temas al gusto.
