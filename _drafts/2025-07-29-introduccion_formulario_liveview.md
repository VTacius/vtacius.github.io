---
title: Breve introducción al manejo de formularios en Phoenix LiveView
tags: 
   - desarrollo
   - LiveView
---

## Introducción
La presente aplicación tiene dos objetos: `Dominio` y `CfgDominio`. A cada `dominio` le corresponde una `cfg_dominio`. Veamos como se trabaja algo así en Phoenix

## Creación del proyecto

Primero, levantamos un contenedor con una base de datos para nuestro proyecto. El proyecto se llamará `DemoForm`, así que la base de datos en Phoenix, por defecto, se llamara `demo_form_dev`. Aprovecharemos eso para facilitarnos la configuración en general:

```bash
podman run --name demo_form_dev -e POSTGRES_DB=demo_form_dev -e POSTGRES_PASSWORD=postgres -p 5432:5432 -d postgres:15-bookworm
```

Creamos el proyecto de la siguiente forma (Esta vez si usaremos `ecto`):

```bash
mix phx.new demo_form --install --no-gettext --no-mailer --binary-id
```

La opción `--binary-id` configura la aplicación para que cree los esquemas con un ID de tipo `UUID`

Entramos al proyecto, creamos la base de datos y lo echamos a correr:

```bash
cd demo_form
mix ecto.create
mix phx.server
```

Esta vez, haremos gran parte del trabajo con el comando `phx.gen.live`. Como dije, tenemos un objeto Dominio en torno al cual trabajaremos:

```bash
mix phx.gen.live Dominios Dominio dominios nombre:string institucion:string is_active:boolean 
```

La salida del comando muestra todo lo que se ha creado: Su dominio web, su contexto, test y por supuesto, su módulo ecto y sus correspondientes migraciones. La cantidad y calidad del código creado podría ser suficiente para muchos casos, y en caso de no serlo, es un muy buen punto de partida

Antes de ejecutar las migraciones, podría quererse modificar el campo `is_active` para que sea `true` por defecto en `lib/demo_form/dominios/dominio.ex`:

```elixir

  # ...
  @primary_key {:id, :binary_id, autogenerate: true}
  @foreign_key_type :binary_id
  schema "dominios" do
    field :nombre, :string
    field :institucion, :string
    field :is_active, :boolean, default: true

    timestamps(type: :utc_datetime)
  end
  # ...
```

Actualizamos el repositorio corriendo las migraciones
```bash
mix ecto.migrate
```

Además, nos muestra las rutas que debemos agregar en `lib/demo_form_web/router.ex`:

```elixir

  #...
  scope "/", DemoFormWeb do
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

En este punto, podemos acceder a `http://localhost:4000/dominios/` y ver como esta funcionando la aplicación en este momento.

## Agregando el objeto CfgDominio

Ahora necesitamos el segundo objeto, la configuración de cada dominio. Ya que su presentación siempre será asociada a `Dominio`, lo pondremos bajo el mismo contexto, es decir`Dominios`, y solo crearemos su schema ecto y sus migraciones

```bash
mix phx.gen.schema Dominios.CfgDominio cfg_dominios cliente_email:string cliente_email_admin:string key_pem:text dominio_id:references:dominios
```

Podemos ver que como ha creado menos elementos

Antes de otra cosa, actualizamos el repositorio corriendo las migraciones
```bash
mix ecto.migrate
```

## Cambios en los esquemas 

Modificamos el schema en `lib/demo_form/dominios/dominio.ex`
```elixir
defmodule DemoForm.Dominios.Dominio do
  use Ecto.Schema
  import Ecto.Changeset

  @primary_key {:id, :binary_id, autogenerate: true}
  @foreign_key_type :binary_id
  schema "dominios" do
    field :nombre, :string
    field :institucion, :string
    field :is_active, :boolean, default: true

    has_one :cfg_dominio, DemoForm.Dominios.CfgDominio, foreign_key: :dominio_id
    
    timestamps(type: :utc_datetime)
  end

  @doc false
  def changeset(dominio, attrs) do
    dominio
    |> cast(attrs, [:nombre, :institucion, :is_active])
    |> validate_required([:nombre, :institucion, :is_active])
    |> cast_assoc(:cfg_dominio, required: true)
  end
end
```

`CfgDominio` no necesita cambios. El ya tiene la relación con `Dominio` desde que definimos la tabla

## Cambios en el contexto

Modificamos el contexto de `Dominios` (`lib/demo_form/dominios.ex`) para que, junto con `Dominio`, también cargue `cfg_dominios` al cargar el detalle de dominio:

```elixir
  # ...
  def get_dominio!(id) do
    Repo.get!(Dominio, id)
    |> Repo.preload(:cfg_dominio)
  end
  # ...
```
## Cambios en los controladores

Modificamos `lib/demo_form_web/live/dominio_live/index.ex`, cambiando `apply_action/1` para `:new` de la siguiente forma:

```elixir
  
  # ...
  use DemoFormWeb, :live_view

  alias DemoForm.Dominios
  alias DemoForm.Dominios.Dominio
  alias DemoForm.Dominios.CfgDominio
  # ...

  # ...
  defp apply_action(socket, :new, _params) do
    socket
    |> assign(:page_title, "Crear Dominio")
    |> assign(:dominio, %Dominio{cfg_dominio: %CfgDominio{}})
  end
  # ...

```

Además, cambiamos `render/1` y `update/2` en `lib/demo_form_web/live/dominio_live/form_component.ex` para reflejar ese cambio:

```elixir
  
  # ...
    @impl true
    def render(assigns) do
    ~H"""
    <div>
      <.header>
        {@title}
      </.header>

      <.simple_form for={@form} id="dominio-form" phx-target={@myself} phx-change="validate" phx-submit="save" >
        <.input field={@form[:nombre]} type="text" label="Nombre" />
        <.input field={@form[:institucion]} type="text" label="Institucion" />
        <.input field={@form[:is_active]} type="checkbox" label="Is active" />
        <.inputs_for :let={cf} field={@form[:cfg_dominio]}>
          <.input field={cf[:cliente_email]} type="text" label="Cliente email" />
          <.input field={cf[:cliente_email_admin]} type="text" label="Admin email" />
          <.input field={cf[:key_pem]} type="textarea" label="Key PEM" />
        </.inputs_for>

        <:actions>
          <.button phx-disable-with="Saving...">Save Dominio</.button>
        </:actions>
      </.simple_form>
    </div>
    """
  end

  @impl true
  def update(%{dominio: dominio} = assigns, socket) do
    dominio = Map.put_new(dominio, :cfg_dominio, %Dominios.CfgDominio{})
    {:ok,
     socket
     |> assign(assigns)
     |> assign_new(:form, fn ->
        Dominios.change_dominio(dominio) |> to_form()
     end)}
  end
  # ...

```

Todo lo que se necesita es `inputs_for` para relacionar ambos objetos en un mismo formulario. Podemos ir a `http://localhost:4000/dominios/new` y crear un dominio, con su correspodiente configuración.

Una vez creado, podemos editarlo. Igual funciona

Para terminar, modificamos la plantilla que muestra el detalle del dominio en `lib/demo_form_web/live/dominio_live/show.html.heex`:

```elixir

# ...
<.list>
  <:item title="Nombre">{@dominio.nombre}</:item>
  <:item title="Institucion">{@dominio.institucion}</:item>
  <:item title="Is active">{@dominio.is_active}</:item>
  <:item title="Cliente Email">{@dominio.cfg_dominio && @dominio.cfg_dominio.cliente_email}</:item>
  <:item title="Cliente Email Admin">{@dominio.cfg_dominio && @dominio.cfg_dominio.cliente_email_admin}</:item>
  <:item title="Key PEM">
    <div class="text-center">
      {@dominio.cfg_dominio && @dominio.cfg_dominio.key_pem}
    </div>
  </:item>
</.list>
# ...

```
