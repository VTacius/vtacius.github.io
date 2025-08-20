---
title: Upload archivos en LiveView 
tags: 
   - desarrollo
   - LiveView
---

## Introducción
Crearemos un formulario LiveView que nos permita subir un archivo JSON. 

Dicho archivo contiene algunos datos que se usarán para rellenar un formulario con valores por defecto.

## Creación del proyecto

Creamos el proyecto de la siguiente forma
```bash
mix phx.new demo_upload --install --no-ecto --no-gettext --no-mailer
```

Entramos al proyecto y lo echamos a correr:
```bash
cd demo_upload
mix phx.server
```

Antes de nada, creamos el directorio `live`, que no se crea automáticamente:
```bash
mkdir lib/demo_upload_web/live
```

Y ahora creamos el archivo `lib/demo_upload_web/live/upload_live.ex`:
```elixir
defmodule DemoUploadWeb.UploadLive do
  use DemoUploadWeb, :live_view

  def mount(_params, _session, socket) do
    form_data = %{
      "domain_name" => "",
      "nombre_institucion" => "",
      "client_email" => "",
      "private_key_id" => ""
    }

    socket =
      socket
      |> assign(:form, to_form(form_data))
      |> assign(:uploaded_cfg, nil)         # Guardamos la referencia al archivo guardado 
      |> allow_upload(:cfg_json,            # Nombre con el que nos vamos a referir al archivo
        accept: ~w(.json),                  # Especificamos la extensión del archivo (No se hacen comprobaciones profundad de tipo)
        auto_upload: true,                  # En cuanto sea seleccionado, empezará a subirse
        progress: &handle_progress/3,       # Para realmente lograr lo de auto_upload: true, se requiere una función que maneje su progreso
        max_entries: 1
      )

    {:ok, socket}
  end

  defp consume_file(path) do
    case File.read(path) do
      {:ok, content} ->
        case Jason.decode(content) do
          {:ok, json_data} -> {:ok, json_data}
          error -> error
        end
      error -> error
    end
  end

  def handle_progress(:cfg_json, entry, socket) do
    if entry.done? do
      current_data = socket.assigns.form.params
      # consume_uploaded_entry consume UNA entrada, justo nuestro caso. La función pasada al último debe devolver
      # {:ok, resultado} | {:pending, resultado}
      # En nuestro caso, basta con el primer caso, que hará que consume_uploaded_entry devuelva 'resultado'
      json_path = consume_uploaded_entry(socket, entry, fn %{path: path} -> {:ok, path} end)

      socket =
        case json_path && consume_file(json_path) do
          {:ok, json_data} ->
            updated_data = Map.merge(current_data, json_data)

            socket
            |> assign(:form, to_form(updated_data))
            |> assign(:uploaded_cfg, %{ref: entry.ref, name: entry.client_name})

          {:error, reason} ->
            put_flash(socket, :error, inspect(reason))

          _ ->
            put_flash(socket, :error, "No se pudo procesar el archivo")
        end

      {:noreply, socket}
    else
      {:noreply, socket}
    end
  end

  def handle_event("validate", %{"form" => form_params}, socket) do
    {:noreply, assign(socket, :form, to_form(form_params))}
  end

  def handle_event("save", %{"form" => _form_params}, socket) do
    {:noreply, put_flash(socket, :info, "Formulario procesado correctamente")}
  end

  def handle_event("cancel-upload", _params, socket) do
    # Limpiamos el estado local
    {:noreply, assign(socket, :uploaded_cfg, nil)}
  end

  defp error_to_string(:too_large), do: "Archivo demasiado grande"
  defp error_to_string(:too_many_files), do: "Demasiados archivos seleccionados"
  defp error_to_string(:not_accepted), do: "Tipo de archivo no permitido"
end
````

Y creamos su render en el archivo por defecto `lib/demo_upload_web/live/upload_live.ex`:
```heex
<Layouts.app flash={@flash}>
  <form id="upload-form" phx-submit="save" phx-change="validate">
    <!-- Área de drop target -->

<section
  phx-drop-target={@uploads.cfg_json.ref}
  class="border-2 border-dashed border-gray-300 rounded-lg p-6 text-center hover:border-blue-500 transition-colors duration-200">

  <.live_file_input upload={@uploads.cfg_json} class="hidden" />

  <!-- Mostrar archivo cargado desde assign -->
  <%= if @uploaded_cfg do %>
    <div class="space-y-3">
      <p class="text-sm text-green-600 font-medium">
        ✓ Configuración por defecto desde  <%= @uploaded_cfg.name %> 
      </p>

      <button
        type="button"
        phx-click="cancel-upload"
        class="inline-flex items-center px-3 py-1 border border-red-300 text-sm font-medium rounded text-red-700 bg-red-50 hover:bg-red-100 transition-colors">
        Remover archivo
      </button>
    </div>
  <% else %>
    <!-- Estado inicial -->
    <div class="space-y-3">
      <svg class="mx-auto h-12 w-12 text-gray-400" stroke="currentColor" fill="none" viewBox="0 0 48 48">
        <path d="M28 8H12a4 4 0 00-4 4v20m32-12v8m0 0v8a4 4 0 01-4 4H12a4 4 0 01-4-4v-4m32-4l-3.172-3.172a4 4 0 00-5.656 0L28 28M8 32l9.172-9.172a4 4 0 015.656 0L28 28m0 0l4 4m4-24h8m-4-4v8m-12 4h.02"
              stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
      </svg>

      <div>
        <button type="button"
          onclick="this.closest('section').querySelector('input[type=file]').click()"
          class="inline-flex items-center px-4 py-2 border border-blue-300 text-sm font-medium rounded-md text-blue-700 bg-blue-50 hover:bg-blue-100 transition-colors">
          Seleccionar archivo
        </button>
        <p class="text-sm text-gray-500 mt-2">o arrástralo acá</p>
      </div>

      <p class="text-xs text-gray-400">Archivo de configuración JSON creado en Google Workspace</p>
    </div>
  <% end %>
</section>
    <!-- Campos del formulario -->
    <.input field={@form[:domain_name]} type="text" label="Nombre de dominio" name="form[domain_name]" />
    <.input field={@form[:client_email]} type="text" label="Client Email" name="form[client_email]" />
    <.input field={@form[:private_key_id]} type="text" label="Private key ID" name="form[private_key_id]" />
    <.input field={@form[:nombre_institucion]} type="text" label="Nombre Institución" name="form[nombre_institucion]" />

    <button type="submit"
      class="mt-4 px-4 py-2 rounded-md bg-blue-600 text-white hover:bg-blue-700 transition-colors">
      Crear Dominio
    </button>
  </form>

</Layouts.app>
```

Agregamos las rutas en `lib/demo_upload_web/router.ex`
```elixir

  #...
  scope "/", DemoUploadWeb do
    pipe_through :browser

    get "/", PageController, :home
    live "/upload", UploadLive
  end
  # ...

```

En este punto, podemos acceder a `http://localhost:4000/upload/` y revisar el comportamiento de la aplicación. 

