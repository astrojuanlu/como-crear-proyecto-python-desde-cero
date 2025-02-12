# Cómo crear un proyecto Python desde cero

## 1. Entornos virtuales e instalación de dependencias con `uv`

Lo primero de todo siempre es crear un [entorno virtual](https://docs.python.org/es/3.12/library/venv.html).
Los entornos virtuales ("virtual environments" en inglés) sirven para crear una instalación de Python aislada e independiente
para que puedas desarrollar tu nuevo proyecto con tranquilidad.

[`uv`](https://github.com/astral-sh/uv/) es un instalador de paquetes Python extremadamente rápido, diseñado para imitar a `pip` y `pip-tools`.

Hay varias formas de instalar `uv`. De su documentación:

```
# En macOS y Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# En Windows
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"

# Con pip
pip install uv

# Con pipx
pipx install uv

# Con Homebrew
brew install uv
```

(El resto de este documento asume una línea de comandos de Linux y un intérprete Bash)

Una vez instalado `uv`, crea un directorio para tu nuevo proyecto y un entorno virtual en él:

```
$ mkdir nuevo-proyecto && cd nuevo-proyecto
$ uv venv
Using Python 3.12.3 interpreter at: .../bin/python3.12
Creating virtualenv at: .venv
Activate with: source .venv/bin/activate.fish
```

> [!TIP]
> Tradicionalmente había que activar el entorno virtual, pero ¡con uv no hace falta!
> Aun así, si quieres hacerlo, en Linux y macOS se hace de esta forma:
> ```
> $ source .venv/bin/activate
> (.venv) $ 
> ```
>
> Si estás en Windows, mejor usar Windows Subsystem for Linux (WSL), Git Bash,
> o alguna otra terminal que emule un entorno Linux.

A partir de aquí ya puedes instalar las dependencias que necesites.
Por ejemplo, para instalar [IPython](https://ipython.readthedocs.io/):

```
$ uv pip install ipython
Resolved 16 packages in 243ms
Installed 16 packages in 241ms
 + asttokens==2.4.1
 + decorator==5.1.1
 + executing==2.0.1
 + ipython==8.25.0
 + jedi==0.19.1
 + matplotlib-inline==0.1.7
 + parso==0.8.4
 + pexpect==4.9.0
 + prompt-toolkit==3.0.47
 + ptyprocess==0.7.0
 + pure-eval==0.2.2
 + pygments==2.18.0
 + six==1.16.0
 + stack-data==0.6.3
 + traitlets==5.14.3
 + wcwidth==0.2.13
```

## 2. Metadatos del proyecto

Un paquete Python está definidi por un archivo `pyproject.toml` con ciertos metadatos
y una estructura de directorios concreta.

Puedes crear el archivo `pyproject.toml` y los directorios a mano,
o puedes utilizar alguna de las múltiples herramientas que existen para ello.

La más sencilla de todas es, de nuevo, `uv`:

```
$ uv init --bare
Initialized project `nuevo-proyecto`
```

Ya tienes tu archivo `pyproject.toml` creado:

```
$ tree
.
└── pyproject.toml

1 directory, 1 file
$ cat pyproject.toml
[project]
name = "nuevo-proyecto"
version = "0.1.0"
requires-python = ">=3.13"
dependencies = []
```

### Gestión de dependencias

`uv` también te ayuda a agregar dependencias a tu proyecto.

Por ejemplo, si quieres utilizar la biblioteca Pydantic:

```
$ uv add pydantic
 Resolved 5 packages in 7.28s
Prepared 1 package in 1.71s
Installed 4 packages in 33ms
 + annotated-types==0.7.0
 + pydantic==2.10.6
 + pydantic-core==2.27.2
 + typing-extensions==4.12.2
```

Esto tiene varios efectos:
- Se añade `pydantic>=2.10.6` a la lista de dependencias en `pyproject.toml`
- Se instala en el entorno virtual, así como sus dependencias transitivas
- Se crea un archivo `uv.lock`, que contiene la información "congelada" del entorno para que sea fácilmente reproducible.

Puedes comprobar que funciona borrando el entorno virtual y dejando que uv lo cree desde cero:

```
$ rm -rf .venv  # Oh no!
$ uv sync  # uv al rescate
Using CPython 3.13.0
Creating virtual environment at: .venv
Resolved 5 packages in 0.54ms
Installed 4 packages in 11ms
 + annotated-types==0.7.0
 + pydantic==2.10.6
 + pydantic-core==2.27.2
 + typing-extensions==4.12.2
```

### 3. Código reutilizable

Ahora podríamos crear archivos `.py`, pero vamos a darle una vuelta de tuerca más al proyecto
y utilizar la estructura de directorios estándar para que nuestro código sea reutilizable. 

Primero crea un directorio con el nombre de tu paquete dentro de otro directorio `src`:

```
$ mkdir -p src/nuevo_proyecto/
```

A continuación, crea un archivo `__init__.py` en dicho directorio:

```
$ echo 'print("Hello, world!")' > src/nuevo_proyecto/__init__.py
```

Por último, tenemos que modificar nuestro `pyproject.toml` para que nuestro paquete sea instalable:

```
$ head -n6 pyproject.toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "nuevo-proyecto"
```

Y ahora sí, corremos nuestro intérprete con uv:

```
$ uv run python -c "import nuevo_proyecto"
Installed 1 package in 24ms
Hello, world!
```

¡Y ya está! Ahora tu proyecto se importa como cualquier otro paquete Python 🎉

> [!TIP]
> ¡Buen momento para guardar tus cambios con `git`!
>
> ```
> (.venv) $ git init
> Initialized empty Git repository in .../nuevo-proyecto/.git/
> (.venv) $ curl -sL "https://gitignore.io/api/python,jupyternotebooks" > .gitignore
> (.venv) $ git add .
> (.venv) $ git commit -am 'First commit, package created with flit'
> [main (root-commit) 3e34eca] First commit, package created with flit
> 2 files changed, 195 insertions(+)
> create mode 100644 .gitignore
> create mode 100644 pyproject.toml
> ```
