---
date: "2025-07-28T00:00:00Z"
tags:
- desarrollo
- LiveView
title: Introducción innecesariamente básica a Phoenix LiveView
---
## Introducción
Este minúsculo proyecto esta hecho para ir copiando manualmente, y entender así cada detalle en los comentarios. Son dos archivos, así que se me hace justo

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

Empezamos por la lógica web: Crearemos el directorio `lib/demo_web/live/`: Si bien el módulo LiveView ya se encuentra, por defecto, en el proyecto, el directorio asignado a guardar sus componentes no se crea por defecto

```bash
mkdir lib/demo_web/live
```

En él, creamos el fichero `lib/demo_web/live/dominio_live.ex` con el siguiente contenido:
```elixir
defmodule DemoWeb.DominioLive do
  use DemoWeb, :live_view 
  
  @doc"""
    Se encarga de inicializar el componente cuando se accede a él por primera vez 
  """
  def mount(_params, _session, socket) do
    {:ok, socket} 
  end

  @doc"""
    Se encarga de cambiar el componente cada vez que los paramétros de la URL cambian
    Usamos apply_action/3 para manejar efectivamente cada ruta, según el comportamiento por defecto de Phoenix
  """
  def handle_params(params, _url, socket) do
    {:noreply, apply_action(socket, socket.assigns.live_action, params)}
  end
  
  # Configuramos los assigns que el socket mantiene según se necesite. En esta caso, solo cambiamos el title html
  defp apply_action(socket, :index, _params) do
    socket |> assign(:page_title, "Lista de dominios")
  end

  # Obtenemos 'id' desestructurando 'params' de la url, lo configuramos mediante 'assign' ne el socket
  defp apply_action(socket, :detail, %{"id" => id}) do
    socket |> assign(:page_title, "Dominio #{id}") |> assign(:id, id)
  end
    
  @doc"""
    Otra desestructuración especifica para asignar este render a :detail
    Esta forma de acceder a un valor en assings mantiene la "reactividad" del sistema 
  """
  def render(%{live_action: :detail} = assigns) do
    ~H"<h2>Detalle de dominio #{@id}</h2>"
  end
  
  def render(%{live_action: :index} = assigns) do
    ~H"<h2>Lista de dominios</h2>"
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
    # url - controlador - assign = %{live_action: :index}
    live "/dominios", DominioLive, :index
    # url - controlador - assign = %{live_action: :detail}
    live "/dominios/:id", DominioLive, :detail
  end

  # Other scopes may use custom stacks.
  # ...
```

Ahora, es posible acceder a `http://localhost:4000/dominios/` y a `http://localhost:4000/dominios/230` para ver nuestras dos sencillas rutas 

### Segunda fase: Agregamos el contexto
Para entender el [contexto](https://hexdocs.pm/phoenix/contexts.html), existen muchas lecturas adecuadas, pero yo recomiendo: 

* [Kill your Phoenix Context](https://peterullrich.com/phoenix-contexts)
* [Resurrect your Phoenix Context](https://peterullrich.com/resurrect-your-phoenix-context)

Por nuestra parte, seguiremos de modelo el código que se crearía con el siguiente comando: 

```bash
mix phx.gen.live Dominios Dominio dominios correo:string correo_administrador:string
```

Creamos el fichero `lib/demo/dominios.ex` con el siguiente contenido:

```elixir
defmodule Demo.Dominios do
  @moduledoc """
  Contexto para Dominios
  """

  @doc """
  Retorna la lista de dominios 
  """
  def listar() do
    [
      %{id: 1, nombre: "dominio.com", correo: "cpena@dominio.com", correo_administrador: "admin@dominio.com"},
      %{id: 1, nombre: "ejemplo.com", correo: "kpena@ejemplo.com", correo_administrador: "admin@ejemplo.com"},
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
![Elixir REPL](/images/screenshot_2025-07-29_14-05-35.png)

(Sé que no es el primer lenguaje con una REPL, pero si uno de los que más conveniente puede llegar a ser)

### Tercera fase: Mejores templates
Vamos a unir nuestra breve lógica de negocios con nuestras sencillas rutas. Pero antes, vamos a mejorar el template (Que por ahora esta 'inline' en el código) y de una vez lo colocamos en su propio archivo.

Empezamos creando un directorio en nuestro dominio web (`DemoWeb`) para `DominioLive`

```bash
mkdir lib/demo_web/live/dominio_live/ 
```
Y creamos el template `lib/demo_web/live/dominio_live/index.html.heex`:

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
        <%= @dominio |> Map.get(:nombre, "") %>
      </h1>
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
        <%= @dominio |> Map.get(:nombre, "") %>
      </h1>
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

Modificamos nuestro controlador (Esa esa la función de `lib/demo_web/live/dominio_live.ex`) en tres partes:
  * Incluimos los templates como funciones gracias a `embed_templates`: (1)
  * Cambiamos los apply:
    * Para `:index`, usamos `assign` para agregar un valor a `dominios`: (2)
    * Para `:detail`, usamos `assign` para agregar un valor a `dominio`: (3)
  * Cambiamos los render:
    * Para `%{live_action: :index}`, usamos `index`: El template en `lib/demo_web/live/dominio_live/index.html.heex` convertido a función en el paso 1: (4)
    * Para `%{live_action: :detail}`, usamos `detail`: El template en `lib/demo_web/live/dominio_live/detail.html.heex` convertido a función en el paso 1: (5)

```elixir
defmodule DemoWeb.DominioLive do
  use DemoWeb, :live_view

  embed_templates "dominio_live/*"  # (1)

  @doc"""
  Se encarga de inicializar el componente cuando accedemos a eĺ por primera vez
  """ 
  def mount(_params, _session, socket) do
    {:ok, socket}
  end
  
  @doc"""
  Se encarga de cambiar el componente cada vez que los parametros de la URL cambian
  Usamos apply_action/3 para manejar efectivamente cada ruta, según el comportamiento por defecto de Phoenix
  """
  def handle_params(params, _url, socket) do
    {:noreply, apply_action(socket, socket.assigns.live_action, params)}
  end
 
  # Obtenemos 'id' desestructurando 'params' de la url, lo configuramos mediante 'assign' en el socket
  defp apply_action(socket, :detail, %{"id" => id}) do
    dominio = %{}   # (3)
    socket |> assign(:page_title, "Dominio #{id})") |> assign(:dominio, dominio)
  end

  # Configuramos los assigns que el socket manitne según se necesite. Es este caso, solo cambiamos el title HTML
  defp apply_action(socket, :index, _params) do
    dominios = []   # (2)
    socket |> assign(:page_title, "Lista de dominios") |> assign(:dominios, dominios)
  end
  
  @doc"""
    Otra desestructuración especifica para asignar este render a :detail
    Esta forma de acceder a un valor en assings mantiene la "reactividad" del sistema 
  """
  def render(%{live_action: :detail} = assigns) do
    detail(assigns)   # (4)
  end
  
  def render(%{live_action: :index} = assigns) do
    index(assigns)    # (5)
  end

end
```

Accedemos de nuevo a `http://localhost:4000/archivos/` y a `http://localhost:4000/archivos/230`, y vemos como el maquetado ha mejorado

### Cuarta fase: Unimos el dominio web con el dominio de nuesta lógica de negocios
Para esto, solo hay dos cosas que hacer:
  * Agregamos un alias a `Demo.Dominios`, el punto de partida para nuestra lógica de negocios. Esto es opcional (Podríamos ir escribiendo la ruta completa en donde corresponda), pero es una buena práctica: (1)
  * Agregamos nuestra lógica de negocios en los `apply_action`:
    * Para `:index`, usamos `Dominios.listar/0` para asignarle valor a `dominios`: (2)
    * Para `:detail`, usamos `Dominios.obtener/1` para asignarle valor a `dominio`: (3)

```elixir
defmodule DemoWeb.DominioLive do
  alias Demo.Dominios  # (1)
  use DemoWeb, :live_view

  embed_templates "dominio_live/*"

  @doc"""
  Se encarga de inicializar el componente cuando accedemos a eĺ por primera vez
  """ 
  def mount(_params, _session, socket) do
    {:ok, socket}
  end
  
  @doc"""
  Se encarga de cambiar el componente cada vez que los parametros de la URL cambian
  Usamos apply_action/3 para manejar efectivamente cada ruta, según el comportamiento por defecto de Phoenix
  """
  def handle_params(params, _url, socket) do
    {:noreply, apply_action(socket, socket.assigns.live_action, params)}
  end

  # Configuramos los assigns que el socket manitne según se necesite. Es este caso, solo cambiamos el title HTML
  defp apply_action(socket, :index, _params) do
    dominios = Dominios.listar()   # (2)
    socket |> assign(:page_title, "Lista de dominios") |> assign(:dominios, dominios)
  end
 
  # Obtenemos 'id' desestructurando 'params' de la url, lo configuramos mediante 'assign' en el socket
  defp apply_action(socket, :detail, %{"id" => id}) do
    dominio = Dominios.obtener(id)  # (3)
    socket |> assign(:page_title, "Dominio #{id})") |> assign(:dominio, dominio)
  end
  
  @doc"""
  Renders
  """
  def render(%{live_action: :detail} = assigns) do
    detail(assigns)
  end
  
  def render(%{live_action: :index} = assigns) do
    index(assigns)
  end

end
```

### Posibles conclusiones
Esto es más o menos Phoenix LiveView. Es posible que sea un ejemplo demasiado básico para mostrar todas sus caracteristicas, sin embargo, lo cierto es que con lo que esta acá se puede empezar a experimentar bastante sobre el tema
