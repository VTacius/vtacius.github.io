---
title: Configuración básica de NeoVim 
tags:
    - configuraciones
    - desarrollo
    - neovim
---

## Introducción
Ningún blog de este tipo estaría completo sin una guía de configuración de NeoVim. Y sí, sé que todas dicen que son "Guías básicas", pero trataré en gran manera que esta si lo sea, al limitarse al primer acercamiento que yo mismo he hecho a la herramienta

La configuración parte de la idea de darle más capacidades a NeoVim, pero que siga siendo un editor de texto, no un IDE. 

Además, actualmente estoy empezando a trabajar con Phoenix LiveView, así que le añadiré soporte para ello. Y para Rust, porque sí.

Sin embargo, voy a usar como referencia a [LazyVim](https://www.lazyvim.org/), que no es lo mismo que [lazy.vim](https://lazy.folke.io/), aunque pareciera que sí.

## Pasos previos
* Instalar NeoVim
* Instalar NerdFont
* Instalar Alacritty
* Configurar Alacritty
* Asegurarse de tener instalador tanto elixir como erlang.

Creamos el esquema de directorio con 
```bash
mkdir -p  ~/.config/nvim/lua/{plugins,config}
```

## Configuración

De acá en adelante, todo será copiar y pegar archivos. La idea es copiar un archivo, cerrar nvim, y volverlo a abrir para aplicar cambios, así se trabaja una configuración progresiva

### init.lua

Este archivo marca la entrada para nuestra configuración. Establece un par de configuraciones al inicio, pero en un momento, con require("config.options"), incluye todas nuestras configuraciones desde el archivo `lua/config/options.lua`
Verifica que `Lazy` este instalado, y después, carga los plugins y su configuración. Ese es uno de tantos enfoques.
```bash
cat <<MAFI> ~/.config/nvim/lua/init.lua
-- ~/.config/nvim/init.lua

-- Establece el líder de mapeo de teclas (usualmente la barra espaciadora)
vim.g.mapleader = " "
vim.g.maplocalleader = "\\"

-- Habilita los colores verdaderos en el terminal, esencial para los temas modernos
vim.opt.termguicolors = true

-- Carga las opciones generales del editor desde lua/config/options.lua
-- Esto asegura que las opciones como la indentación se apliquen temprano.
require("config.options")

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

-- Configura lazy.nvim para cargar tus plugins
-- La forma más robusta de cargar todos los plugins desde el directorio `lua/plugins/`
require("lazy").setup({
  spec = {
    -- Importa todas las especificaciones de plugins desde el directorio `lua/plugins/`
    { import = "plugins" },
  },
  -- Opciones adicionales para lazy.nvim
  checker = {
    enabled = true, -- Habilita la verificación automática de actualizaciones de plugins
    frequency = 86400, -- Frecuencia de verificación en segundos (cada 24 horas)
  },
})

MAFI
```

### lua/config/options.lua
Acá guardamos la configuración más primaria. Algunas cosas, en mi caso `relativenumber` no las había probado y la verdad es que me sorprendió bastante una vez entendí su utilidad
Al abrir este archivo veremos un ligero mensaje de error. Se resolverá en cuanto empecemos a configurar los plugins
```bash
cat <<MAFI> ~/.config/nvim/lua/config/options.lua
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
MAFI
```

### lua/plugins/catppuccin.lua

Este tema me encanta. Es simple, y aún así sus cuatros versiones son bastante buenas, además de estar disponible para más software, como Alacritty 

De igual forma, puede cambiarse a conveniencia.

```bash
cat <<MAFI> ~/.config/nvim/lua/plugins/catppuccin.lua
-- ~/.config/nvim/lua/plugins/catppuccin.lua

return {
  "catppuccin/nvim",
  name = "catppuccin", -- Nombre del plugin para lazy.nvim
  priority = 1000,     -- Asegura que el esquema de colores se cargue primero
  lazy = false,        -- Esencial: El esquema de colores no debe cargarse de forma perezosa
  opts = {
    flavour = "latte", -- Opciones: "latte", "frappe", "macchiato", "mocha"
    background = {     -- Configuración de fondo para modos claro/oscuro
      light = "latte",
      dark = "mocha",
    },
    transparent_background = false, -- Establecer en `true` si se desea un fondo transparente
    term_colors = true,             -- Habilita la configuración de colores del terminal
    compile = {
      enabled = true, -- Habilita la compilación del tema para un inicio más rápido
      path = vim.fn.stdpath("cache").. "/catppuccin", -- Ruta de caché para el tema compilado
    },
    integrations = { -- Integraciones con otros plugins para una tematización cohesiva
      cmp = true,       -- Integración con blink.cmp
      gitsigns = true,
      nvimtree = true,
      treesitter = true, -- Integración con nvim-treesitter
      bufferline = true, -- Integración con bufferline.nvim
      -- Añadir otras integraciones según los plugins instalados
    },
    -- custom_highlights = function(colors) -- Opcional: Sobrescribir grupos de resaltado específicos
    --   return {
    --     Comment = { fg = colors.flamingo, style = { "italic" } },
    --   }
    -- end,
  },
  config = function(_, opts)
    require("catppuccin").setup(opts)
    vim.cmd.colorscheme("catppuccin") -- Establece el esquema de colores
  end,
}
MAFI
```

### lua/plugins/web-devicons.lua
Proporciona los íconos para bufferline y lualine. Tengo que ser sincero: Estas son casi que las opciones por defecto. En lo visual, esta configuración es lo esencial.
Al menos me parece que si le pone íconos bonitos a casi todo.
```bash
cat <<MAFI> ~/.config/nvim/lua/plugins/web-devicons.lua
-- ~/.config/nvim/lua/plugins/web-devicons.lua

return {
  'nvim-tree/nvim-web-devicons',
  lazy = true, -- Se carga cuando lo necesiten otros plugins
  config = function()
    require('nvim-web-devicons').setup {
      -- Puedes personalizar los iconos aquí si lo deseas
      -- por ejemplo, para añadir iconos para tipos de archivo específicos
    }
  end,
}
MAFI
```

### lua/plugins/treesitter.lua
Activa el resaltado de sintaxis para los lenguajes que están en `ensure_installed`. Por el momento, todo lo necesario para desarrollo web con Phoenix LiveView. Y Rust, porque sí.

```bash
cat <<MAFI> ~/.config/nvim/lua/plugins/treesitter.lua
-- ~/.config/nvim/lua/plugins/treesitter.lua

return {
  "nvim-treesitter/nvim-treesitter",
  build = ":TSUpdate", -- Instala y actualiza automáticamente los analizadores al instalar/actualizar el plugin
  lazy = false,        -- Esencial: Este plugin no debe cargarse de forma perezosa
  opts = {
    highlight = { enable = true, injections = true }, -- Habilita el resaltado de sintaxis basado en Tree-sitter
    indent = { enable = true },    -- Habilita la indentación inteligente basada en Tree-sitter
    ensure_installed = { -- Una lista de analizadores de lenguaje para asegurar que estén instalados
      "eex",            -- Para plantillas Elixir EEx
      "elixir",         -- Para código Elixir
      "heex",           -- Para plantillas Phoenix HEEx
      "rust",           -- Para código Rust
      "html",           -- Esencial para el desarrollo web
      "surface",        -- Creo que esto arreglo el problema con las plantillas heex embebidas en .ex
      "javascript",     -- Esencial para el desarrollo web
      "embedded_template",
      "typescript",     -- Esencial para el desarrollo web
      "css",            -- Esencial para el desarrollo web
      "json",           -- Para archivos de configuración y formatos de datos
      "lua",            -- Para archivos de configuración de Neovim
      -- Añadir otros lenguajes según sea necesario para su pila de desarrollo específica
    },
    -- Deshabilitar el resaltado adicional de expresiones regulares de Vim para prevenir resaltados duplicados y posibles ralentizaciones del rendimiento
    additional_vim_regex_highlighting = false,
  },
  config = function(_, opts)
    require("nvim-treesitter.configs").setup(opts)
  end,
}
MAFI
```

### lua/plugins/lsp.lua
En este archivo hay dos cosas configuradas: `mason` y `mason-lspconfig`. Es acá donde nos encargamos de mantener un rastreo de los LSP que necesitamos. Y sí, aún son configurables

```bash
cat <<MAFI> ~/.config/nvim/lua/plugins/lsp.lua 
-- ~/.config/nvim/lua/plugins/lsp.lua

return {
  {
    "neovim/nvim-lspconfig",
    dependencies = {
      "williamboman/mason.nvim",        -- Gestor de paquetes para LSP, DAP, linters, etc.
      "williamboman/mason-lspconfig.nvim", -- Puente entre Mason y nvim-lspconfig
    },
    opts = {
      servers = {
        -- Configuración específica para rust_analyzer
        rust_analyzer = {
          settings = {
            ["rust-analyzer"] = {
              checkOnSave = {
                command = "clippy", -- Configura `rust-analyzer` para ejecutar Clippy para linting al guardar
              },
              procMacro = {
                enable = true, -- Habilita el soporte de macros procedimentales
              },
              -- Añadir otras configuraciones específicas de rust-analyzer aquí según sea necesario,
              -- como `cargo.allFeatures = true` para el conjunto completo de características
            },
          },
          -- Definir una función `on_attach` para atajos de teclado o configuraciones específicas de Rust
          on_attach = function(client, bufnr)
            -- Puedes añadir atajos de teclado específicos de Rust aquí
            -- Por ejemplo:
            -- vim.keymap.set("n", "<leader>ra", function() vim.cmd("RustLsp status") end, { buffer = bufnr, desc = "Rust Analyzer Status" })
            -- Asegúrate de que los atajos de teclado LSP estándar también se configuren
            -- Ejemplo de mapeos LSP comunes (puedes expandir esto)
            local opts = { noremap = true, silent = true, buffer = bufnr }
            vim.keymap.set('n', 'gD', vim.lsp.buf.declaration, opts)
            vim.keymap.set('n', 'gd', vim.lsp.buf.definition, opts)
            vim.keymap.set('n', 'K', vim.lsp.buf.hover, opts)
            vim.keymap.set('n', 'gi', vim.lsp.buf.implementation, opts)
            vim.keymap.set('n', '<C-k>', vim.lsp.buf.signature_help, opts)
            vim.keymap.set('n', '<space>wa', vim.lsp.buf.add_workspace_folder, opts)
            vim.keymap.set('n', '<space>wr', vim.lsp.buf.remove_workspace_folder, opts)
            vim.keymap.set('n', '<space>wl', function() print(vim.inspect(vim.lsp.buf.list_workspace_folders())) end, opts)
            vim.keymap.set('n', '<space>D', vim.lsp.buf.type_definition, opts)
            vim.keymap.set('n', '<space>rn', vim.lsp.buf.rename, opts)
            vim.keymap.set('n', '<space>ca', vim.lsp.buf.code_action, opts)
            vim.keymap.set('n', 'gr', vim.lsp.buf.references, opts)
            vim.keymap.set('n', '<space>f', function() vim.lsp.buf.format { async = true } end, opts)
          end,
        },
        -- elixir_ls no necesita una configuración directa aquí si elixir-tools.nvim lo gestiona
        -- pero se incluye en ensure_installed para que Mason lo instale.
      },
    },
    -- Esencial: Configurar mason-lspconfig para manejar la instalación y configuración de servidores LSP
    config = function(_, opts)
      require("mason").setup() -- Asegura que Mason esté configurado
      require("mason-lspconfig").setup({
        ensure_installed = {"html", "elixirls", "rust_analyzer", "tailwindcss" }, -- Usa "elixirls" para Mason
        handlers = {
          -- Manejador predeterminado para todos los servidores no especificados
          function(server_name)
            require("lspconfig")[server_name].setup(opts.servers[server_name] or {})
          end,
          -- Manejador personalizado para rust_analyzer
          rust_analyzer = function()
            require("lspconfig").rust_analyzer.setup(opts.servers.rust_analyzer)
          end,
          -- elixirls es gestionado por elixir-tools.nvim, por lo que no necesita un setup aquí
          elixirls = function() -- Asegúrate de que el nombre del manejador coincida con el de ensure_installed
            -- No hacer nada, elixir-tools.nvim se encargará de esto
          end,
        },
      })
      require('lspconfig').html.setup({
        filetypes = { "html", "heex", "ex", "elixir" },
        init_options = {
          configurationSection = { "html", "css", "javascript" },
          embeddedLanguages = {
            css = true,
            javascript = true
          }
        }
      })
      --require("elixir").setup({
      --  nextls = { enable = true },
      --  elixirls = { enable = false }, -- Desactiva este
      --})
      -- Configurar el cliente LSP globalmente para que blink.cmp obtenga las capacidades correctas
      vim.lsp.set_log_level("debug") -- Habilita el registro de depuración para LSP

      -- Configura las opciones de diagnóstico usando vim.diagnostic.config()
      vim.diagnostic.config({
        virtual_text = true,
        signs = true,
        underline = true,
        update_in_insert = false,
      })

      vim.lsp.config("*", {
        capabilities = require("blink.cmp").get_lsp_capabilities(),
        on_attach = function(client, bufnr)
          -- Aquí puedes añadir mapeos de teclas comunes para todos los LSP
          -- o llamar a una función de utilidad que los configure.
          -- Los mapeos específicos de Elixir y Rust se manejan en sus respectivos plugins.
          local opts = { noremap = true, silent = true, buffer = bufnr }
          vim.keymap.set('n', 'gD', vim.lsp.buf.declaration, opts)
          vim.keymap.set('n', 'gd', vim.lsp.buf.definition, opts)
          vim.keymap.set('n', 'K', vim.lsp.buf.hover, opts)
          vim.keymap.set('n', 'gi', vim.lsp.buf.implementation, opts)
          vim.keymap.set('n', '<C-k>', vim.lsp.buf.signature_help, opts)
          vim.keymap.set('n', '<space>wa', vim.lsp.buf.add_workspace_folder, opts)
          vim.keymap.set('n', '<space>wr', vim.lsp.buf.remove_workspace_folder, opts)
          vim.keymap.set('n', '<space>wl', function() print(vim.inspect(vim.lsp.buf.list_workspace_folders())) end, opts)
          vim.keymap.set('n', '<space>D', vim.lsp.buf.type_definition, opts)
          vim.keymap.set('n', '<space>rn', vim.lsp.buf.rename, opts)
          vim.keymap.set('n', '<space>ca', vim.lsp.buf.code_action, opts)
          vim.keymap.set('n', 'gr', vim.lsp.buf.references, opts)
          vim.keymap.set('n', '<space>f', function() vim.lsp.buf.format { async = true } end, opts)
        end,
      })
    end,
  },
}

MAFI
```

### lua/plugins/bufferline.lua
Este es ornamental. Pero el crear una pestaña aún cuando solo hay un buffer abierto, para mí es todo lo que necesitaba desde hace tiempo.

```bash
cat <<MAFI> ~/.config/nvim/lua/plugins/bufferline.lua
-- ~/.config/nvim/lua/plugins/bufferline.lua

return {
  'akinsho/bufferline.nvim',
  version = "*", -- Usa la última versión estable
  dependencies = 'nvim-tree/nvim-web-devicons', -- Para iconos de tipo de archivo
  config = function()
    require('bufferline').setup({
      options = {
        mode = "buffers", -- Muestra todos los buffers abiertos. Cambiar a "tabs" para páginas de pestañas
        style_preset = "flat_slant", -- Usando la cadena literal del estilo
        diagnostics = "nvim_lsp", -- Muestra diagnósticos LSP (errores, advertencias) en las pestañas
        offsets = {
            filetype = "NvimTree", text = "File Explorer", text_align = "left"
        }, -- Evita que la línea de buffers se dibuje sobre los paneles laterales
        -- Puedes añadir más opciones aquí si son necesarias
      },
      -- Crucial: Integra los resaltados del tema Catppuccin.
      -- `get()` se asegura de que los colores de Catppuccin se apliquen correctamente.
      highlights = require("catppuccin.groups.integrations.bufferline").get()
    })
  end,
  -- Crucial: Asegura que bufferline se cargue *después* de catppuccin para una aplicación de tema correcta
  after = "catppuccin",
}
MAFI

```

### lua/plugins/lualine.lua
Otro ornamental. Y sin embargo, creo que es algo que todos queremos una vez que lo vemos en algún podcast en youtube

```bash
cat <<MAFI>~/.config/nvim/lua/plugins/lualine.lua
-- ~/.config/nvim/lua/plugins/lualine.lua

return {
  'nvim-lualine/lualine.nvim',
  dependencies = { 'nvim-tree/nvim-web-devicons' }, -- Para iconos en la barra de estado
  config = function()
    require('lualine').setup({
      options = {
        icons_enabled = true,
        theme = 'catppuccin', -- Aplica el tema Catppuccin a Lualine
        component_separators = { left = '', right = ''}, -- Separadores estéticos para componentes
        section_separators = { left = '', right = ''}, -- Separadores estéticos para secciones
        always_divide_middle = true, -- Asegura que las secciones centrales no ocupen toda la línea de estado
        globalstatus = true, -- Muestra la línea de estado incluso cuando solo hay una ventana abierta
      },
      sections = {
        lualine_a = {'mode'}, -- Muestra el modo Vim actual
        lualine_b = {'branch', 'diff', 'diagnostics'}, -- Muestra la rama de Git, el estado del diff del archivo y los diagnósticos LSP
        lualine_c = {'filename'}, -- Muestra el nombre del archivo actual
        lualine_x = {'filetype', 'encoding', 'fileformat'}, -- Muestra el tipo de archivo, la codificación y el formato del archivo
        lualine_y = {'progress'}, -- Muestra el progreso dentro del archivo
        lualine_z = {'location'}, -- Muestra la línea y columna del cursor
      },
      -- Opcional: Configuración para la línea de pestañas, si prefiere que Lualine la gestione
      -- tabline = {
      --   lualine_a = {'buffers'},
      --   lualine_b = {},
      --   lualine_c = {},
      --   lualine_x = {},
      --   lualine_y = {},
      --   lualine_z = {'tabs'},
      -- }
    })
  end,
}
MAFI
```

### lua/plugins/elixir-tools.lua
No recuerdo porque tuve que ponerlo en su propio archivo, pero funciona y por ahora es lo importante

```bash
cat <<MAFI> ~/.config/nvim/lua/plugins/elixir-tools.lua
-- ~/.config/nvim/lua/plugins/elixir-tools.lua

return {
  "elixir-tools/elixir-tools.nvim",
  version = "*", -- Usa la última versión estable
  event = { "BufReadPre", "BufNewFile" }, -- Carga el plugin cuando se lee un buffer de Elixir o se crea uno nuevo
  dependencies = { "nvim-lua/plenary.nvim" }, -- Una dependencia de utilidad común para muchos plugins Lua
  config = function()
    local elixir = require("elixir")
    local elixirls = require("elixir.elixirls")
    elixir.setup {
      nextls = { enable = true }, -- Habilita Next LS, un servidor de lenguaje Elixir más nuevo
      elixirls = {
        enable = true,
        settings = elixirls.settings {
          dialyzerEnabled = false, -- Ejemplo: Deshabilita las verificaciones de Dialyzer para una retroalimentación más rápida
          enableTestLenses = true, -- Habilita las lentes de código para ejecutar pruebas directamente desde el editor
        },
        on_attach = function(client, bufnr)
          -- Atajos de teclado personalizados para operaciones de tubería específicas de Elixir
          vim.keymap.set("n", "<space>fp", ":ElixirFromPipe<cr>", { buffer = bufnr, noremap = true, desc = "Elixir: From Pipe" })
          vim.keymap.set("n", "<space>tp", ":ElixirToPipe<cr>", { buffer = bufnr, noremap = true, desc = "Elixir: To Pipe" })
          vim.keymap.set("v", "<space>em", ":ElixirExpandMacro<cr>", { buffer = bufnr, noremap = true, desc = "Elixir: Expand Macro" })

          -- Puedes añadir más mapeos o configuraciones aquí que dependan del LSP de Elixir
        end,
      },
      projectionist = { enable = true }, -- Habilita la integración con vim-projectionist para el scaffolding de archivos
    }
  end,
}
MAFI
```

### lua/plugins/blink-cmp.lua
Y este es el plugin que escogí para el autocompletado. Hasta ahora, ha funcionado bien

```bash
cat <<MAFI> ~/.config/nvim/lua/plugins/blink-cmp.lua
-- ~/.config/nvim/lua/plugins/blink-cmp.lua
return {
  {
    "saghen/blink.cmp",
    version = "*", -- Se recomienda usar una etiqueta de versión estable, por ejemplo, '1.*'
    dependencies = {
      "L3MON4D3/LuaSnip", -- Motor de snippets
      "rafamadriz/friendly-snippets", -- Colección de snippets para LuaSnip
      -- "saghen/blink.compat", -- Opcional: Capa de compatibilidad para fuentes de nvim-cmp si es necesario
    },
    opts = {
      snippets = {
        preset = "luasnip", -- Configura blink.cmp para usar LuaSnip como su fuente de snippets
        expand = function(snippet, _)
          return require("luasnip").lsp_expand(snippet) -- Función para expandir snippets LSP
        end,
      },
      appearance = {
        -- Configuración de apariencia, por ejemplo, para la alineación de iconos con Nerd Fonts
        nerd_font_variant = "mono", -- Ajusta el espaciado para asegurar que los iconos estén alineados
      },
      completion = {
        accept = {
          auto_brackets = { enabled = true }, -- Soporte experimental para auto-paréntesis
        },
        menu = {
          draw = {
            treesitter = { "lsp" }, -- Permite que Treesitter dibuje el menú de autocompletado LSP
          },
        },
        documentation = {
          auto_show = true, -- Muestra automáticamente la documentación
          auto_show_delay_ms = 200,
        },
        ghost_text = { enabled = true }, -- Muestra la finalización como texto fantasma
      },
      signature = { enabled = true }, -- Habilita el soporte de ayuda de firma (experimental)
      sources = {
        -- Configuración de proveedores de fuentes. LSP, buffer, path son incorporados
        providers = {
          lsp = { score_offset = 0 },      -- Prioridad para las sugerencias LSP
          snippets = { score_offset = -1 }, -- Prioridad para las sugerencias de snippets
          buffer = { score_offset = -3 },   -- Prioridad para las sugerencias del buffer
          path = { score_offset = 3 },      -- Prioridad para las sugerencias de ruta
          }
        },
      },
      keymap = {
        preset = "default", -- Usa los atajos de teclado predeterminados de blink.cmp
        -- Puedes personalizar los atajos de teclado aquí, por ejemplo:
        -- ["<C-y>"] = { "select_and_accept" }, -- Aceptar la sugerencia seleccionada
        -- = { "snippet_forward", "ai_accept", "fallback" }, -- Tab para avanzar en snippets o aceptar
        -- = { "snippet_backward", "fallback" }, -- Shift+Tab para retroceder en snippets
      },
    },
    config = function(_, opts)
      -- Carga LuaSnip y friendly-snippets
      require("luasnip.loaders.from_vscode").lazy_load()
      require("luasnip.loaders.from_vscode").lazy_load({ paths = { vim.fn.stdpath("config").. "/snippets" } })

      -- Configura blink.cmp
      require("blink.cmp").setup(opts)
    end,
  },
}
MAFI
```

### Bonus

* Es necesario instalar `xclip` para activar el portapapeles
* Ante el mensaje de error `LSP[elixirls][Warning] Formatter plugin Phoenix.LiveView.HTMLFormatter is not loaded and will be skipped.
`, es necesario compilar el proyecto por primera vez con `mix compile` 
