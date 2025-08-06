---
title: "Configurando Neovim: IDE"
tags:
    - configuraciones
    - desarrollo
    - neovim
---

## Introducción
Este es lo realmente indispensable en toda configuración de este tipo. Cuesta creer lo importante que es el resaltado de sintaxis hasta que se trabaja sin ella. Y aprovechando, ya que estamos leyendo AST, usando un LSP por cada lenguaje, así además tenemos una revisión temprana del código. `NeoVim` casi como un IDE

## Luasnip
Es un dependencia de `blink-cmp` que definiremos en su propio archivo:

```lua
-- ~/.config/nvim/lua/plugins/luasnip.lua

return {
	"L3MON4D3/LuaSnip",
	-- follow latest release.
	version = "v2.*", -- Replace <CurrentMajor> by the latest released major (first number of latest release)
	-- install jsregexp (optional!).
	build = "make install_jsregexp"
}
```

## Treesitter
```lua
-- ~/.config/nvim/lua/config/treesitter.lua

return {
    highlight = {
      enable = true,        -- Habilita el resaltado de sintaxis basado en Tree-sitter
      --injections = true,    -- Se supone que activa custom injections
      additional_vim_regex_highlighting = false, -- Deshabilitar el resaltado adicional de expresiones regulares de Vim para prevenir resaltados duplicados y posibles ralentizaciones del rendimiento
    },
    indent = {
      enable = true         -- Habilita la indentación inteligente, puede seguir patrones específicos del language
    },
    fold = {
        enable = true
    },
    ensure_installed = {    -- Una lista de analizadores de lenguaje para asegurar que estén instalados
      "eex",                -- Para plantillas Elixir EEx
      "elixir",             -- Para código Elixir
      "heex",               -- Para plantillas Phoenix HEEx
      "rust",               -- Para código Rust
      "html",               -- Esencial para el desarrollo web
      "surface",            -- Habilita plantillas HEEX como sigil ~H, aunque aún no reconoce el sigil en sí
      "javascript",         -- Esencial para el desarrollo web
      "embedded_template",
      "typescript",         -- Esencial para el desarrollo web
      "css",                -- Esencial para el desarrollo web
      "json",               -- Para archivos de configuración y formatos de datos
      "lua",                -- Para archivos de configuración de Neovim
      "markdown",
      "markdown_inline",    -- Importante para bloques de código
      "regex",              -- No sabía que lo necesitara hasta que me hizo demasiada falta
      "bash",               -- Pues si aún como de eso 
    },

}
```

```lua
-- ~/.config/nvim/lua/plugins/treesitter.lua

return {
  "nvim-treesitter/nvim-treesitter",
  build = ":TSUpdate", -- Instala y actualiza automáticamente los analizadores al instalar/actualizar el plugin
  lazy = false,        -- Este plugin debe iniciar con el sistema 
  config = function()
    require("nvim-treesitter.configs").setup(require("config.treesitter"))
  end,
}
```

Es posible que el siguiente error aparezca:
```
[] [12/16] Treesitter parser for rust has been installed
nvim-treesitter[typescript]: Error during compilation
cc1: error fatal: src/scanner.c: No existe el fichero o el directorio
compilación terminada.
```

No pude hallar más referencias que usar `:TSUpdate`, que dicho sea de paso, es la forma de actualizar todos sus parser y la posible solución a la mayoría de problemas (Que realmente son pocos) que puede presentar la herramienta

Si la idea de seguir una guía de configuración lo más sencilla posible, debo admitir que este facilmente podría ser el punto final. Al final, sería como usar un `vim` con un par de superpoderes, pero vim al fin y al cabo.

Pero, es altamente recomendable seguir con al menos el siguiente componente

## blink-cmp
Se configura antes que `LSP` porque este le necesita en su configuración

```lua
-- ~/.config/nvim/lua/config/blink-cmp.lua

return {
  snippets = {
    preset = "luasnip",             -- Usar LuaSnip como su fuente de snippets
    expand = function(snippet, _)
      return require("luasnip").lsp_expand(snippet)
    end,
  },
  appearance = {
    nerd_font_variant = "mono",
    use_nvim_cmp_as_default = true, -- Mantiene familiaridad visual
  },
  fuzzy = {
    implementation = "prefer_rust"
  },
  completion = {
    accept = {
      auto_brackets = {
        enabled = true              -- Soporte experimental para auto-paréntesis
      },
    },
    menu = {
      enabled = true,  -- Cambiar de función a booleano
      draw = {
        treesitter = { "lsp" },     -- Permite que Treesitter dibuje el menú de autocompletado LSP
        columns = { { "label", "label_description", gap = 1 }, { "kind_icon", "kind" } }
      },
    },
    documentation = {
      auto_show = true,             -- Muestra automáticamente la documentación
      auto_show_delay_ms = 200,
    },
    ghost_text = {
      enabled = true                -- Muestra la finalización como texto fantasma
    },
  },
  signature = {
    enabled = true                  -- Habilita el soporte de ayuda de firma (experimental)
  },
  sources = {
    default = { 'lsp', 'path', 'snippets', 'buffer' },
    providers = {
      lsp = { score_offset = 0 },
      path = { score_offset = 3 },
      buffer = { score_offset = -3 },
      snippets = { score_offset = -1 },
      cmdline = { score_offset = 2 },
    },
  },
  keymap = {
    preset = "default", -- Usa los atajos de teclado predeterminados de blink.cmp
  },
  cmdline = {
    enabled = true,
    sources = { 'cmdline' },  -- Aquí va ahora
    completion = {
      menu = { 
        auto_show = true  -- Simplificado
      }
    },
    keymap = {
      preset = 'inherit'  -- Hereda los mapeos del modo normal
    }
  },
}

```

```lua
-- ~/.config/nvim/lua/plugins/blink-cmp.lua

return {
  {
    "saghen/blink.cmp",
    version = "*",
    dependencies = {
      "L3MON4D3/LuaSnip",
      "rafamadriz/friendly-snippets",
    },
    opts = require("config.blink-cmp"),
    config = function(_, opts)
      require("luasnip.loaders.from_vscode").lazy_load()
      require("luasnip.loaders.from_vscode").lazy_load({ paths = { vim.fn.stdpath("config") .. "/snippets" } })
      require("blink.cmp").setup(opts)
    end,
  },
}

```

## LSP
La configuración de LSP es el siguiente mejor salto cualitativo que se puede hacer. En este caso, esta compuesto por `mason` y `mason-lspconfig`. Para el caso de `Elixir`, uso `ElixirLS` por medio de `elixir-tools`

```lua
-- ~/.config/nvim/lua/config/lsp.lua

return {
  -- Como por ahora solo estoy usando NVim para desarrollo de Elixir, esto es lo que se necesita además de elixir-tools
  ensure_installed = {"html", "rust_analyzer", "tailwindcss", "lua_ls", "bashls"},
  
  servers = {
    rust_analyzer = {
      settings = {
        ["rust-analyzer"] = {
          checkOnSave = { command = "clippy" },
          procMacro = { enable = true },
        },
      },
    },
    html = {
      filetypes = { "html", "heex" },
      init_options = {
        configurationSection = { "html", "css", "javascript" },
        embeddedLanguages = { css = true, javascript = true }
      }
    },
  },
  
}

```

```lua
-- ~/.config/nvim/lua/plugins/lsp.lua

return {
  "neovim/nvim-lspconfig",
  dependencies = {
    "williamboman/mason.nvim",
    "williamboman/mason-lspconfig.nvim",
  },
  config = function()
    local lsp_config = require("config.lsp")
    
    require("mason").setup()
    require("mason-lspconfig").setup({
      ensure_installed = lsp_config.ensure_installed,
      handlers = {
        function(server_name)
          local server_opts = lsp_config.servers[server_name] or {}
          server_opts.capabilities = require("blink.cmp").get_lsp_capabilities()
          
          require("lspconfig")[server_name].setup(server_opts)
        end,
      },
    })
  end,
}
```

Una lista de lenguajes disponibles pueden encontrarse en el [enlace](https://github.com/neovim/nvim-lspconfig/blob/master/doc/configs.md)

Es podría ser otro final de la guía de configuración para muchos. Por mi parte, aún necesito del siguiente para mejorar sus capacidades en Elixir 

## elixir-tools
```lua
-- ~/.config/nvim/lua/config/elixir-tools.lua

return {
  nextls = {
    enable = false -- Nos aseguramos que este desactivo por el momento 
  },
  elixirls = {
    enable = true,
    settings =  {
      autoBuild = true,         -- Se encarga de compilar el proyecto si se guarda desde Neovim 
      suggestSpecs = true,      -- Sugiere anotaciones de tipo @specs
      fetchDeps = true,         -- Obtiene dependencias del proyecto cuando compila 
      dialyzerEnabled = true,   -- Esto podría desactivarse en algunos equipos legacy 
      enableTestLenses = true,  -- Habilita las lentes de código para ejecutar pruebas directamente desde el editor
  },
  projectionist = {
    enable = false
  },
  on_attach = function(_, bufnr)
    -- Atajos de teclado personalizados para operaciones de tubería específicas de Elixir
    vim.keymap.set("n", "<space>fp", ":ElixirFromPipe<cr>", { buffer = bufnr, noremap = true, desc = "Elixir: From Pipe" })
    vim.keymap.set("n", "<space>tp", ":ElixirToPipe<cr>", { buffer = bufnr, noremap = true, desc = "Elixir: To Pipe" })
    vim.keymap.set("v", "<space>em", ":ElixirExpandMacro<cr>", { buffer = bufnr, noremap = true, desc = "Elixir: Expand Macro" })
  end,
  },
}
```

```lua
-- ~/.config/nvim/lua/plugins/elixir-tools.lua

return {
  "elixir-tools/elixir-tools.nvim",
  version = "*",
  event = { "BufReadPre", "BufNewFile" },
  config = function(_, opts)
    require("elixir").setup(opts)
  end,
  dependencies = {
    "nvim-lua/plenary.nvim",
  },
}
```

Y con eso, `NeoVim sigue siendo un editor de texto, pero con características de IDE. 
