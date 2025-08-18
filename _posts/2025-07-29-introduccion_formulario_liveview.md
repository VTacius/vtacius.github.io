---
title: Breve introducción al manejo de formularios en Phoenix LiveView
tags: 
   - desarrollo
   - LiveView
---

## Introducción
La presente aplicación tiene dos objetos: `Dominio` y `CfgDominio`. A cada `dominio` le corresponde un `cfg_dominio`. Veamos como se trabaja algo así en Phoenix

Esta vez, usaremos más características de las que vimos en [nuestra introducción](https://vtacius.github.io/blog/introduccion_basica_liveview/), por lo que esta vez si usaremos una base de datos.

Por cierto: Esta guía ya usa [Phoenix 1.8](https://www.phoenixframework.org/blog/phoenix-1-8-released)

## Creación del proyecto

Primero, levantamos un contenedor con una base de datos para nuestro proyecto. El proyecto se llamará `DemoForm`, así que la base de datos en Phoenix, por defecto, se llamara `demo_form_dev`. Aprovecharemos eso para facilitarnos la configuración en general:

```bash
podman run --name demo_form_dev -e POSTGRES_DB=demo_form_dev -e POSTGRES_PASSWORD=postgres -p 5432:5432 -d postgres:15-bookworm
```

Creamos el proyecto de la siguiente forma: Esta vez si usaremos `ecto`, la opción `--binary-id` configura la aplicación para que cree las tablas con un ID de tipo `UUID`:
```bash
mix phx.new demo_form --install --no-gettext --no-mailer --binary-id
```

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

La salida del comando muestra todo lo que se ha creado: Su dominio web, su contexto, tests, y por supuesto, su módulo ecto y sus correspondientes migraciones. 

De crear, crea hasta las rutas, aunque debamos agregarlas por nuestra cuenta en `lib/demo_form_web/router.ex`, pero antes haremos las migraciones, de este modo no veremos mensajes de error en `mix phx.server`.

La cantidad y calidad del código creado podría ser suficiente, o al menos, un buen punto de partida para nuestro proyecto.

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

Ahora sí, agregamos las rutas en `lib/demo_form_web/router.ex`:

```elixir
  #...
  scope "/", DemoFormWeb do
    pipe_through :browser

    get "/", PageController, :home
    
    live "/dominios", DominioLive.Index, :index
    live "/dominios/new", DominioLive.Form, :new
    live "/dominios/:id", DominioLive.Show, :show
    live "/dominios/:id/edit", DominioLive.Form, :edit
  end
  # ...
```

En este punto, podemos acceder a `http://localhost:4000/dominios/` y ver como esta funcionando la aplicación en este momento.

### Agregando el objeto CfgDominio

Ahora necesitamos el segundo objeto, la configuración de cada dominio. Debido a que su presentación siempre será asociada a `Dominio`, y a que lo pondremos bajo el mismo contexto de `Dominios`, es decir`Dominios`, solo crearemos su schema ecto y sus migraciones

```bash
mix phx.gen.schema Dominios.CfgDominio cfg_dominios cliente_email:string cliente_email_admin:string key_pem:text dominio_id:references:dominios
```

Podemos ver que como ha creado menos elementos

Antes de otra cosa, actualizamos el repositorio corriendo las migraciones para `CfgDominio`:
```bash
mix ecto.migrate
```

### Cambios en los esquemas 

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

    has_one :cfg_dominio, DemoForm.Dominios.CfgDominio, foreign_key: :dominio_id # Agregando
    
    timestamps(type: :utc_datetime)
  end

  @doc false
  def changeset(dominio, attrs) do
    dominio
    |> cast(attrs, [:nombre, :institucion, :is_active])
    |> validate_required([:nombre, :institucion, :is_active])
    |> cast_assoc(:cfg_dominio, required: true) # Agregando
  end
end
```

`CfgDominio` no necesita cambios. El ya tiene la relación con `Dominio` desde que definimos la tabla

### Cambios en el contexto
Modificamos el contexto de `Dominios` (`lib/demo_form/dominios.ex`) para que, junto con `Dominio`, también cargue `cfg_dominios` al cargar el detalle de dominio:

```elixir
  # ...
  def get_dominio!(id) do
    Repo.get!(Dominio, id)
    |> Repo.preload(:cfg_dominio)
  end
  # ...
```

### Cambios en los controladores
Modificamos `lib/demo_form_web/live/dominio_live/form.ex` de las siguiente formas:

  * Agregamos un alias de entidad CfgDominio 
```elixir
  
  # ...
  use DemoFormWeb, :live_view

  alias DemoForm.Dominios
  alias DemoForm.Dominios.Dominio
  alias DemoForm.Dominios.CfgDominio # Agregamos
  # ...
```

  * Agregamos un `CfgDominio` vacío como atributo `cfg_dominio` en el `apply_action/3` para `:index`:
```elixir
  # ...
  defp apply_action(socket, :new, _params) do
    dominio = %Dominio{cfg_dominio: %CfgDominio{}} # Modificamos

    socket
    |> assign(:page_title, "New Dominio")
    |> assign(:dominio, dominio)
    |> assign(:form, to_form(Dominios.change_dominio(dominio)))
  end
  # ...
```

  * Cambiamos `render/1` para agregar los campos de `CfgDominio`:

```elixir
  # ...
    @impl true
    def render(assigns) do
    ~H"""
    <Layouts.app flash={@flash}>
      <.header>
        {@page_title}
        <:subtitle>Use this form to manage dominio records in your database.</:subtitle>
      </.header>

      <.form for={@form} id="dominio-form" phx-change="validate" phx-submit="save">
        <.input field={@form[:nombre]} type="text" label="Nombre" />
        <.input field={@form[:institucion]} type="text" label="Institucion" />
        <.inputs_for :let={cf} field={@form[:cfg_dominio]}>
          <.input field={cf[:cliente_email]} type="text" label="Cliente email" />
          <.input field={cf[:cliente_email_admin]} type="text" label="Admin email" />
          <.input field={cf[:key_pem]} type="textarea" label="Key PEM" />
        </.inputs_for>
        <.input field={@form[:is_active]} type="checkbox" label="Is active" />
        <footer>
          <.button phx-disable-with="Saving..." variant="primary">Save Dominio</.button>
          <.button navigate={return_path(@return_to, @dominio)}>Cancel</.button>
        </footer>
      </.form>
    </Layouts.app>
    """
  end
  # ...
```

Todo lo que se necesita es `inputs_for` para relacionar ambos objetos en un mismo formulario. 

Podemos ir a `http://localhost:4000/dominios/new` y crear un dominio, con su correspodiente configuración.

Una vez creado, podemos editarlo. Igual funciona

Para terminar, modificamos la plantilla que muestra el detalle del dominio en `lib/demo_form_web/live/dominio_live/show.ex`:

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

## Posibles conclusiones
`Phoenix 1.8` parece tener grandes cambios para simplificar el trabajo de controladores a los desarrolladores, a cambio de aumentar [la complejidad ](https://daisyui.com/)  del propio framework.
