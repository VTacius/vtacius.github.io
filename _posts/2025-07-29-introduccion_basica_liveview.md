---
title: Introducción innecesariamente básica a Phoenix LiveView
tags: 
   - desarrollo
   - LiveView
---

## Creación del proyecto
Para facilitar las cosas, vamos a iniciar un proyecto sin `ecto`, por el momento una base de datos no es realmente necesaria

```bash
mix phx.new demo --no-ecto
```

Ya puestos a remover del proyecto elementos que por ahora no son necesarios, dejemosle sin internacionalización o envío de correo:

```bash
mix phx.new demo --no-ecto --no-gettext --no-mailer
```

Y para evitar el `Y` que nos preguntará luego para efectivamente instalar todo, se lo pasamos de una vez:

```bash
mix phx.new demo --no-ecto --no-gettext --no-mailer --install
```

Solo nos falta ingresar al proyecto ingresando al directorio creado, `demo` e iniciar el proyecto:
```bash
cd demo
mix phx.server
```

### Primera fase: Rutas apenas funcionales
A ver: Phoenix separa nuestro modulo `Demo` en dos dominios: `Demo` propiamente, donde debe desarrollarse toda la lógica de negocios, y `DemoWeb`, dónde vamos a desarrollar todas las cuestiones sobre la presentación web.

Empezamos por la lógica web: Crearemos el directorio `lib/demo_web/live/`: Si bien el módulo LiveView ya se encuentra por defecto en el proyecto, el directorio designado para guardar sus componentes no se crea por defecto

```bash
mkdir lib/demo_web/live
```

En él, creamos el fichero `lib/demo_web/live/dominio_live.ex` con el siguiente contenido:
```elixir
defmodule DemoWeb.DominioLive do
  use DemoWeb, :live_view 
  
  @doc """
    Se encarga de inicializar el componente cuando se accede a él por primera vez 
  """
  def mount(_params, _session, socket) do
    {:ok, socket} 
  end

  @doc """
    Se encarga de cambiar el componente cada vez que los paramétros de la URL cambian
    Usamos apply_action/3 para manejar efectivamente cada ruta, según el comportamiento por defecto de Phoenix
  """
  def handle_params(params, _url, socket) do
    {:noreply, apply_action(socket, socket.assigns.live_action, params)}
  end
  
  # Configuramos los assigns que el socket mantiene según se necesite. En esta caso, solo cambiamos el title html
  defp  apply_action(socket, :index, _params) do
    socket |> assign(:page_title, "Lista de dominios")
  end

  # Hemos obtenido el id mediante params de la url, lo configuramos ahora en los assing del socket
  defp apply_action(socket, :detail, %{"id" => id}) do
    socket |> assign(:page_title, "Dominio #{id}") |> assign(:id, id)
  end

  def render(%{live_action: :index} = assigns) do
    ~H"<h2>Lista de dominios</h2>"
  end
 
  # Esta forma de acceder a los assings que se encuentra en el socket es la manera recomendada para mantener la "reactividad" del sistema 
  def render(%{live_action: :detail} = assigns) do
    ~H"<h2>Detalle de dominio #{@id}</h2>"
  end
end
```

Ahora registramos el controlador con su ruta correspondiente en `lib/demo_web/router.ex`:
```elixir
  # ...
  pipeline :api do
    plug :accepts, ["json"]
  end

  scope "/", DemoWeb do
    pipe_through :browser

    get "/", PageController, :home
    live "/dominios", DominioLive, :index
    live "/dominios/*id", DominioLive, :detail
  end

  # Other scopes may use custom stacks.
  # ...
```

Ahora, es posible acceder a `http://localhost:4000/archivos/` y a `http://localhost:4000/archivos/230` para ver nuestras dos sencillas rutas 

## Segunda fase: Agregamos el contexto
Para entender el [contexto](https://hexdocs.pm/phoenix/contexts.html), existen muchas lecturas adecuadas, pero yo recomiendo: 

* [Kill your Phoenix Context](https://peterullrich.com/phoenix-contexts)
* [Resurrect your Phoenix Context](https://peterullrich.com/resurrect-your-phoenix-context)


Por nuestra parte, seguiremos modelo que se crearía con: 

```bash
mix phx.gen.live Dominios Dominio dominios correo:string correo_administrador:string
```

Creamos el fichero `lib/demo/dominios.ex` con el siguiente contenido:

```elixir
defmodule Demo.Dominios do
  @moduledoc """
  Context para Dominios
  """

  @doc """
  Retorna la lista de dominios 
  """
  def listar() do
    [
      %{id: 1, correo: "cpena@dominio.com", correo_administrador: "admin@dominio.com"},
      %{id: 2, correo: "kpena@ejemplo.com", correo_administrador: "admin@ejemplo.com"}
    ]
  end
  
  @doc """
  Retorna el dominio si existe, `nil` en caso contrario 
  """
  def obtener(id) when is_integer(id) do
    listar() |> Enum.find(fn dominio -> dominio.id == id end)
   end

  def obtener(id) when is_binary(id) do
    case Integer.parse(id) do
      {id, ""} -> obtener(id)
      _ -> nil
    end
  end

  @doc """
  Retorna el dominio si existe, emite un error en caso contrario 
  """
  def obtener!(id) do
   case obtener(id) do
      nil -> raise "Dominio con ID #{id} no encontrado"
      dominio -> dominio
    end
  end
end
```

Una vez guardado, podemos probarlo en la REPL de Elixir de la siguiente forma:
![Elixir REPL]({{ site.url }}{{ site.baseurl }}/assets/images/Captura de pantalla_2025-07-29_14-05-35.png)

(Sé que no es el primer lenguaje con una REPL, pero si uno de los que más conveniente puede llegar a ser)

## Tercera fase: Mejores templates
Vamos a unir nuestra breve lógica de negocios con nuestras sencillas rutas, pero antes, vamos a mejorar el template y lo aprovecharemos a poner en sus propios archivos.

Empezamos creando un directorio en nuestro dominio web (`DemoWeb`) para `DominioLive`

```bash
mkdir lib/demo_web/live/dominio_live/ 
```
Y creamos el template `lib/demo_web/live/dominio_live/index.html.heex`:

```elixir
<div class="mx-auto max-w-4xl px-4 py-8">
  <h1 class="text-3xl font-bold text-gray-900 mb-8">Lista de dominios</h1>

  <div class="bg-white shadow overflow-hidden sm:rounded-md">
    <ul class="divide-y divide-gray-200">
      <%= for dominio <- @dominios do %>
        <li>
          <.link navigate={~p"/dominios/#{dominio.id}"} class="block hover:bg-gray-50 px-4 py-4 sm:px-6" >
            <div class="flex items-center justify-between">
              <div class="flex-1 min-w-0">
                <p class="text-sm font-medium text-indigo-600 truncate"> dominio #<%= dominio.id %> </p>
                <p class="text-sm text-gray-500"> Correo: <%= dominio.correo %> </p>
                <p class="text-sm text-gray-500"> Admin: <%= dominio.correo_administrador %> </p> </div>
              <div class="flex-shrink-0">
                <svg class="h-5 w-5 text-gray-400" fill="currentColor" viewBox="0 0 20 20">
                  <path fill-rule="evenodd" d="M7.293 14.707a1 1 0 010-1.414L10.586 10 7.293 6.707a1 1 0 011.414-1.414l4 4a1 1 0 010 1.414l-4 4a1 1 0 01-1.414 0z" clip-rule="evenodd" />
                </svg>
              </div>
            </div>
          </.link>
        </li>
      <% end %>
    </ul>
  </div>
</div>
```

Y también a `lib/demo_web/live/dominio_live/detail.html.heex`:

```elixir
<div class="mx-auto max-w-4xl px-4 py-8">
  <div class="mb-6">
    <.link navigate={~p"/dominios"} class="text-indigo-600 hover:text-indigo-500 text-sm font-medium">
      ← Volver a la lista
    </.link>
  </div>

  <div class="bg-white shadow overflow-hidden sm:rounded-lg">
    <div class="px-4 py-5 sm:px-6">
      <h1 class="text-3xl font-bold text-gray-900">
        Detalle del dominio #<%= @dominio |> Map.get(:id, "") %>
      </h1>
      <p class="mt-1 max-w-2xl text-sm text-gray-500">
        Información completa del dominio seleccionado
      </p>
    </div>

    <div class="border-t border-gray-200">
      <dl>
        <div class="bg-gray-50 px-4 py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
          <dt class="text-sm font-medium text-gray-500">ID</dt>
          <dd class="mt-1 text-sm text-gray-900 sm:col-span-2 sm:mt-0">
            <%= @dominio |> Map.get(:id, "") %>
          </dd>
        </div>

        <div class="bg-white px-4 py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
          <dt class="text-sm font-medium text-gray-500">Correo</dt>
          <dd class="mt-1 text-sm text-gray-900 sm:col-span-2 sm:mt-0">
            <%= @dominio |> Map.get(:correo, "") %>
          </dd>
        </div>

        <div class="bg-gray-50 px-4 py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
          <dt class="text-sm font-medium text-gray-500">Correo Administrador</dt>
          <dd class="mt-1 text-sm text-gray-900 sm:col-span-2 sm:mt-0">
            <%= @dominio |> Map.get(:correo_administrador, "") %>
          </dd> </div> </dl>
    </div>
  </div>
</div>
```

Modificamos nuestro controlador (Esa esa la función de `archivo_live.ex`) en tres cosas:
  * Incluimos los templates como funciones gracias a `embed_templates`
  * Creamos un render `render/1` por cada live_action dado, y usamos los templates, ya convertidos a función
  * Por el momento, usamos valores por defecto para que carguen sin problemas 

```elixir
defmodule DemoWeb.DominioLive do
  use DemoWeb, :live_view 
  
  embed_templates "dominio_live/*"
  
  @doc """
    Se encarga de inicializar el componente cuando se accede a él por primera vez 
  """
  def mount(_params, _session, socket) do
    {:ok, socket} 
  end

  @doc """
    Se encarga de cambiar el componente cada vez que los paramétros de la URL cambian
    Usamos apply_action/3 para manejar efectivamente cada ruta, según el comportamiento por defecto de Phoenix
  """
  def handle_params(params, _url, socket) do
    {:noreply, apply_action(socket, socket.assigns.live_action, params)}
  end
  
  # Configuramos los assigns que el socket mantiene según se necesite. En esta caso, solo cambiamos el title html
  defp  apply_action(socket, :index, _params) do
    dominios = []
    socket |> assign(:page_title, "Lista de dominios") |> assign(:dominios, dominios)
  end

  # Hemos obtenido el id mediante params de la url, lo configuramos ahora en los assing del socket
  defp apply_action(socket, :detail, %{"id" => id}) do
    dominio = %{} 
    socket |> assign(:page_title, "Dominio #{id}") |> assign(:dominio, dominio)
  end

  def render(%{live_action: :index} = assigns) do
    index(assigns) 
  end
 
  # Esta forma de acceder a los assings que se encuentra en el socket es la manera recomendada para mantener la "reactividad" del sistema 
  def render(%{live_action: :detail} = assigns) do
    detail(assigns) 
  end
end
```

Accedemos de nuevo a `http://localhost:4000/archivos/` y a `http://localhost:4000/archivos/230`, y vemos como el maquetado ha mejorado

## Cuarta fase: Unimos el dominio web con el dominio de nuestra aplicación
Aunque le hace falta un par de cosas, por ahora, terminanos con esto:

```elixir
defmodule DemoWeb.DominioLive do
  alias Demo.Dominios

  use DemoWeb, :live_view 
  
  embed_templates "dominio_live/*"
  
  @doc """
    Se encarga de inicializar el componente cuando se accede a él por primera vez 
  """
  def mount(_params, _session, socket) do
    {:ok, socket} 
  end

  @doc """
    Se encarga de cambiar el componente cada vez que los paramétros de la URL cambian
    Usamos apply_action/3 para manejar efectivamente cada ruta, según el comportamiento por defecto de Phoenix
  """
  def handle_params(params, _url, socket) do
    {:noreply, apply_action(socket, socket.assigns.live_action, params)}
  end
  
  # Configuramos los assigns que el socket mantiene según se necesite. En esta caso, solo cambiamos el title html
  defp  apply_action(socket, :index, _params) do
    dominios = Dominios.listar()
    socket |> assign(:page_title, "Lista de dominios") |> assign(:dominios, dominios)
  end

  # Hemos obtenido el id mediante params de la url, lo configuramos ahora en los assing del socket
  defp apply_action(socket, :detail, %{"id" => [id]}) do
    dominio = Dominios.obtener(id) 
    socket |> assign(:page_title, "Dominio #{id}") |> assign(:dominio, dominio)
  end

  def render(%{live_action: :index} = assigns) do
    index(assigns) 
  end
 
  # Esta forma de acceder a los assings que se encuentra en el socket es la manera recomendada para mantener la "reactividad" del sistema 
  def render(%{live_action: :detail} = assigns) do
    detail(assigns) 
  end
end
```
