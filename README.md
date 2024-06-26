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

¡Y no olvides activarlo!

```
$ source .venv/bin/activate
(.venv) $ 
```

A partir de aquí ya puedes instalar las dependencias que necesites.
Por ejemplo, para instalar [IPython](https://ipython.readthedocs.io/):

```
(.venv) $ uv pip install ipython
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

## 2. Creación de paquetes con `flit`

Un paquete Python está definidi por un archivo `pyproject.toml` con ciertos metadatos
y una estructura de directorios concreta.

Puedes crear el archivo `pyproject.toml` y los directorios a mano,
o puedes utilizar alguna de las múltiples herramientas que existen para ello.
Algunas muy conocidas son Pipenv, flit, Poetry, PDM, Hatch, rye...
Sin embargo, algunas como Pipenv y Poetry no son compatibles con [los últimos estándares de Python]((https://peps.python.org/pep-0621/).

La más sencilla de todas es [`flit`](https://github.com/pypa/flit/).
`flit` es una herramienta para crear y publicar paquetes Python.

Para empezar, con tu entorno virtual activado, instala `flit` con `uv`:

```
(.venv) $ uv pip install flit
...
```

Y a continuación:

```
(.venv) $ flit init
Module name: nuevo_proyecto
Author: (tu nombre)
Author email: (tu email)
Home page: 
Choose a license (see http://choosealicense.com/ for more info)
1. MIT - simple and permissive
2. Apache - explicitly grants patent rights
3. GPL - ensures that code based on this is shared with the same terms
4. Skip - choose a license later
Enter 1-4 [4]: 4

Written pyproject.toml; edit that file to add optional extra info.
```

Solo necesitas nombre del módulo (debe ser un nombre válido en Python, por tanto nada de espacios ni guiones),
tu nombre, tu email, y nada más.

Ya tienes tu archivo `pyproject.toml` creado:

```
(.venv) $ tree
.
└── pyproject.toml

1 directory, 1 file
(.venv) $ cat pyproject.toml
[build-system]
requires = ["flit_core >=3.2,<4"]
build-backend = "flit_core.buildapi"

[project]
name = "nuevo_proyecto"
authors = [{name = "Juan Luis Cano Rodríguez", email = "hello@juanlu.space"}]
dynamic = ["version", "description"]
```

### Primer archivo de código

Todavía quedan cosas por hacer, eso sí.
Para comprobarlo, trata de instalar tu propio paquete con `uv` en modo editable:

```
(.venv) $ uv pip install --editable .
error: Failed to download and build: `nuevo-proyecto @ file:///private/tmp/nuevo-proyecto`
  Caused by: Failed to build: `nuevo-proyecto @ file:///private/tmp/nuevo-proyecto`
  Caused by: Build backend failed to determine extra requires with `build_editable()` with exit status: 1
--- stdout:

--- stderr:
Traceback (most recent call last):
  File "<string>", line 14, in <module>
  File "/Users/juan_cano/Library/Caches/uv/environments-v0/.tmpnvIDGY/lib/python3.12/site-packages/flit_core/buildapi.py", line 31, in get_requires_for_build_wheel
    module = Module(info.module, Path.cwd())
             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Users/juan_cano/Library/Caches/uv/environments-v0/.tmpnvIDGY/lib/python3.12/site-packages/flit_core/common.py", line 59, in __init__
    raise ValueError("No file/folder found for module {}".format(name))
ValueError: No file/folder found for module nuevo_proyecto
---
```

Claro, ¡tu paquete aún no tiene archivos!
En primer lugar, crea un directorio con el nombre de tu módulo dentro de otro directorio `src`:

```
(.venv) $ mkdir -p src/nuevo_proyecto/
```

A continuación, crea un archivo `__init__.py` en dicho directorio:

```
(.venv) $ touch src/nuevo_proyecto/__init__.py
```

¿Ya hemos terminado? Todavía no:

```
(.venv) $ uv pip install -e .
error: Failed to download and build: `nuevo-proyecto @ file:///private/tmp/nuevo-proyecto`
  Caused by: Failed to build: `nuevo-proyecto @ file:///private/tmp/nuevo-proyecto`
  Caused by: Build backend failed to determine metadata through `prepare_metadata_for_build_editable` with exit status: 1
--- stdout:

--- stderr:
Traceback (most recent call last):
  File "<string>", line 14, in <module>
  File "/Users/juan_cano/Library/Caches/uv/environments-v0/.tmpnJNK4S/lib/python3.12/site-packages/flit_core/buildapi.py", line 49, in prepare_metadata_for_build_wheel
    metadata = make_metadata(module, ini_info)
               ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Users/juan_cano/Library/Caches/uv/environments-v0/.tmpnJNK4S/lib/python3.12/site-packages/flit_core/common.py", line 425, in make_metadata
    md_dict.update(get_info_from_module(module, ini_info.dynamic_metadata))
                   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Users/juan_cano/Library/Caches/uv/environments-v0/.tmpnJNK4S/lib/python3.12/site-packages/flit_core/common.py", line 228, in get_info_from_module
    raise NoDocstringError(
flit_core.common.NoDocstringError: Flit cannot package module without docstring, or empty docstring. Please add a docstring to your module (/private/tmp/nuevo-proyecto/src/nuevo_proyecto/__init__.py).
---
```

Falta la [cadena de documentación](https://docs.python.org/es/3.12/glossary.html#term-docstring) ("docstring" en inglés).
Modifica tu archivo `__init__.py` para que tenga este aspecto:

```
(.venv) $ cat src/nuevo_proyecto/__init__.py
"""
Nuevo proyecto.
"""

__version__ = "0.1.dev0"

```

> [!NOTE]  
> La versión `__version__ = "0.1.dev0"` es obligatoria también

Y ahora sí:

```
(.venv) $ uv pip install -e .
Resolved 1 package in 298ms
   Built nuevo-proyecto @ file:///private/tmp/nuevo-proyecto
Prepared 1 package in 215ms
Installed 1 package in 4ms
 + nuevo-proyecto==0.1.dev0 (from file:///private/tmp/nuevo-proyecto)
```

Comprueba que está todo bien:

```
(.venv) $ python
Python 3.12.3 (main, Apr  9 2024, 08:09:14) [Clang 15.0.0 (clang-1500.3.9.4)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import nuevo_proyecto
>>> nuevo_proyecto.__version__
'0.1.dev0'
```

¡Listo! 🎉

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
