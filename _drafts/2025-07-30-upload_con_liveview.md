---
title: Upload archivos en LiveView 
tags: 
   - desarrollo
   - LiveView
---

## Introducción
Crearemos un formulario LiveView que nos permita subir un archivo JSON. Dicho archivo contiene algunos datos que se usarán para crear un objeto CfgDominio

## Creación del proyecto

Primero, levantamos un contenedor con una base de datos para nuestro proyecto. Vamos a llamarlo `DemoUpload`, así que la base de datos en Phoenix, por defecto, se llamara `demo_upload_dev`. Aprovecharemos eso para facilitarnos la configuración en general:
```bash
podman run --name demo_upload_dev -e POSTGRES_DB=demo_upload_dev -e POSTGRES_PASSWORD=postgres -p 5432:5432 -d postgres:15-bookworm
```

Creamos el proyecto de la siguiente forma
```bash
mix phx.new demo_upload --install --no-gettext --no-mailer --binary-id
```

Entramos al proyecto, creamos la base de datos y echamos a correr el proyecto:
```bash
cd demo_upload
mix ecto.create
mix phx.server
```

Usaremos el comando `phx.gen.live` de la siguiente forma:
```bash
mix phx.gen.live Dominios CfgDominio cfg_dominios nombre:string cliente_email:string cliente_email_admin:string key_pem:string 
```

Actualizamos el repositorio corriendo las migraciones
```bash
mix ecto.migrate
```

Agregamos las rutas en `lib/demo_upload_web/router.ex`
```elixir

  #...
  scope "/", DemoUploadWeb do
    pipe_through :browser

    get "/", PageController, :home
    live "/dominios", DominioLive.Index, :index
    live "/dominios/new", DominioLive.Index, :new
    live "/dominios/:id/edit", DominioLive.Index, :edit

    live "/dominios/:id", DominioLive.Show, :show
    live "/dominios/:id/show/edit", DominioLive.Show, :edit
  end
  # ...

```

En este punto, podemos acceder a `http://localhost:4000/cfg_dominios/` y ver como esta funcionando la aplicación en este momento.

## Habilitando la subida de archivos en el componente dado

Habilitamos la subida de archivos agregando un mount a `lib/demo_upload_web/live/cfg_dominio_live/form_component.ex` y cambiando el render:

```elixir
  
  # ...
  @impl true
  def render(assigns) do
    ~H"""
    <div>
      <.header>
        {@title}
      </.header>

      <.simple_form for={@form} id="cfg_dominio-form" phx-target={@myself} phx-change="validate" phx-submit="save" >
        <.input field={@form[:nombre]} type="text" label="Nombre" />
        <.input field={@form[:cliente_email]} type="text" label="Cliente email" />
        <.input field={@form[:cliente_email_admin]} type="text" label="Cliente email admin" />
        <.input field={@form[:key_pem]} type="text" label="Key pem" />
        <!-- Zona de drag & drop -->
        <div>
          <.label>Subir archivo JSON</.label>
          <div class="mt-1 flex justify-center px-6 pt-5 pb-6 border-2 border-gray-300 border-dashed rounded-md hover:border-gray-400 transition-colors"
               phx-drop-target={@uploads.pem_file.ref}
               id="drop-zone">
            
            <div class="space-y-1 text-center">
              <svg class="mx-auto h-12 w-12 text-gray-400" stroke="currentColor" fill="none" viewBox="0 0 48 48">
                <path d="M28 8H12a4 4 0 00-4 4v20m32-12v8m0 0v8a4 4 0 01-4 4H12a4 4 0 01-4-4v-4m32-4l-3.172-3.172a4 4 0 00-5.656 0L28 28M8 32l9.172-9.172a4 4 0 015.656 0L28 28m0 0l4 4m4-24h8m-4-4v8m-12 4h.02" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
              
              <div class="text-sm text-gray-600">
                <.live_file_input upload={@uploads.pem_file} class="sr-only" />
                <label for={@uploads.pem_file.ref} class="cursor-pointer bg-white rounded-md font-medium text-indigo-600 hover:text-indigo-500">
                  <span>Selecciona un archivo</span>
                </label>
                <span class="pl-1">o arrastra y suelta aquí</span>
              </div>
              
              <p class="text-xs text-gray-500">JSON, PEM, TXT hasta 10MB</p>
            </div>
          </div>
          
          <!-- Archivos seleccionados -->
          <div :for={entry <- @uploads.pem_file.entries}>
            <div class="flex items-center justify-between p-2 bg-gray-100 rounded mt-2">
              <span class="text-sm">{entry.client_name}</span>
              <button type="button" phx-click="cancel-upload" phx-value-ref={entry.ref} phx-target={@myself} class="text-red-500 hover:text-red-700">
                ✕
              </button>
            </div>
          </div>
        </div>
        <:actions>
          <.button phx-disable-with="Saving...">Save Cfg dominio</.button>
        </:actions>
      </.simple_form>
    </div>
    """
  end

  @impl true
  def mount(socket) do
    {:ok,
      socket
      |> allow_upload(:pem_file, accept: ~w(.json), max_entries: 1, max_file_size: 10_000_000)}
  end
  # ...

```
