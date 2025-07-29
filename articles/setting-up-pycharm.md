# Setting up PyCharm

## Table of Contents

- [Getting started](#getting-started)
- [Configuring the Python Interpreter](#configuring-the-python-interpreter)
- [Setting up the Debugger](#setting-up-the-debugger)
- [Integrating with Git](#integrating-with-git)

## Getting started

**PyCharm** is an integrated development environment (IDE) for the Python programming language.

Before you start working, you need to install the IDE. Download PyCharm from <a href="https://www.jetbrains.com/pycharm/" target="_blank">here</a> — 
the Professional version includes a 30-day free trial — and install it on your system. To start using PyCharm, 
you’ll need to create a <a href="https://account.jetbrains.com/signup" target="_blank">JetBrains account</a>. Then open the IDE, 
sign in with your credentials, and open your project.

**Note**: The free version of PyCharm has limited functionality — for example, it doesn’t support Docker- or 
Compose-based Python Interpreters — which makes it unsuitable for commercial development.

Based Keyboard **Shortcuts**:

- `Ctrl + D` or `Command + D` – duplicate the current line below the cursor.

- `Ctrl + Shift + K` or `Command + Shift + K` – push to a remote Git repository.

- `Double Shift` – search for files by name.

- `Ctrl + Shift + F` or `Command + Shift + F` – search across all files by content.

- `Ctrl + /` or `Command + /` – comment or uncomment the selected line(s).

- `Tab / Shift + Tab` – indent or unindent the selected block of code.

- `Ctrl + Click` or `Command + Click` – navigate to a class, function, file, or variable (including viewing source code of installed libraries).

- `Alt + Enter` or `Option + Enter` – add an import statement for the selected element at the top of the file.

## Configuring the Python Interpreter

The **Python Interpreter** is the environment that PyCharm relies on to analyze and execute code.
Its configuration affects project navigation, access to dependency libraries (including their source code), 
and the highlighting of syntax errors and type mismatches. Without proper interpreter setup, 
development efficiency decreases and the likelihood of bugs increases.

Next, the focus will be on configuring how PyCharm interacts with the development environment inside Docker containers.
Therefore, if you’re not familiar with containerizing applications using Docker, 
or if Docker and Compose are not installed on your host machine, it is recommended to read <a href="https://github.com/davyddd/wiki/blob/main/articles/docker-and-compose.md" target="_blank">this article</a>.

On macOS, the first step is to enable Docker support in PyCharm:

- Open PyCharm settings (`Ctrl + Alt + S` or `Command + ,`).

- Navigate to `Build, Execution, Deployment` → `Docker`.

- Docker is not listed in the opened window, click the `+` icon and configure it as shown in Figure 1.

<img src="https://raw.githubusercontent.com/davyddd/wiki/refs/heads/main/media/setting-up-pycharm/docker-configuration.png" width="900" alt="Docker configuration in PyCharm settings" title="Docker configuration in PyCharm settings">

Figure 1 — Docker configuration in PyCharm settings.

By default, PyCharm looks for all project dependencies in the local environment. However, in this case, 
the dependencies are installed inside a Docker container. To let the IDE know where to find them, 
you need to configure the interpreter by following these steps:

- Open PyCharm settings.

- Expand the `Project: <your project name>` section.

- Go to the `Project Interpreter` tab.

- In the top-right corner, click the `Add Interpreter` and choose `On Docker Compose…`.

- In the window that opens, under the `Service` section, select your service defined in the `docker-compose.yml` file, as shown in Figure 2.

**Note**: The `docker-compose.yml` file must be located in the root of the project. 
Otherwise, PyCharm won’t be able to load the available services for selection.

<img src="https://raw.githubusercontent.com/davyddd/wiki/refs/heads/main/media/setting-up-pycharm/compose-interpreter-setup.png" width="400" alt="Compose interpreter setup" title="Compose interpreter setup">

Figure 2 — Compose interpreter setup.

**Note**: Even after configuring the Python Interpreter, PyCharm may not recognize local imports.
To resolve, need to mark the application directory as the source root:
- Right-click the directory in the project tree.
- Go to `Mark Directory as` → `Sources Root`.

## Setting up the Debugger

### Django

The first step is to enable Django support in PyCharm. This allows you to use the built-in `Django server` for running 
and debugging your application, and unlocks many other features as well (learn more <a href="https://www.jetbrains.com/help/pycharm/django-support7.html" target="_blank">here</a>).
To enable it, follow these steps (see Figure 3 to locate the Django settings section):

- Open PyCharm settings.

- Expand the `Languages & Frameworks section`.

- Navigate to the `Django` tab.

- Check the `Enable Django Support` box, then fill in the following fields:

  - `Django project root` – the directory that typically contains the `manage.py` file;

  - `Settings` – the path to the `settings.py` file;

  - `Manage script` – the path to the `manage.py` file itself.

<img src="https://raw.githubusercontent.com/davyddd/wiki/refs/heads/main/media/setting-up-pycharm/pycharm-django-settings.png" width="900" alt="Django support settings in PyCharm" title="Django support settings in PyCharm">

Figure 3 — Django support settings in PyCharm.

To configure the debugger in PyCharm, follow these steps:

- In the top-right corner of the IDE, locate the `Play` and `Bug` icons. To the left of them, you’ll see a dropdown — 
by default, it usually says `Current File`. Click it and select `Edit Configurations…`.

- In the window that opens, click the `+` icon, choose `Django server`, and configure the settings as shown in Figure 4.

**Note**: In the `Django server` configuration, it’s enough to specify the `Python Interpreter` created in the previous step 
and set the `host` and `port`. All other settings can be left at their default values.

<img src="https://raw.githubusercontent.com/davyddd/wiki/refs/heads/main/media/setting-up-pycharm/pycharm-django-server-config.png" width="900" alt="Django server configuration in PyCharm" title="Django server configuration in PyCharm">

Figure 4 — Django server configuration in PyCharm.

### FastAPI

To configure the debugger in PyCharm, follow these steps:

- In the top-right corner of the IDE, locate the `Play` and `Bug` icons. To the left of them, you’ll see a dropdown — 
by default, it usually says `Current File`. Click it and select `Edit Configurations…`.

- In the window that opens, click the `+` icon, choose `FastAPI`;

- Fill in the following fields (see Figure 5 as a reference):

  - `Application file` – the path to the file containing the application, typically named `main.py`;

  - `Application name` – the name of the application, typically `app`;

  - `Uvicorn options` – optional parameters for the Uvicorn web server;

  - `Python Interpreter` - the interpreter configured in the previous step.

<img src="https://raw.githubusercontent.com/davyddd/wiki/refs/heads/main/media/setting-up-pycharm/pycharm-fastapi-native-config.png" width="900" alt="FastAPI configuration using the built-in template in PyCharm" title="FastAPI configuration using the built-in template in PyCharm">

Figure 5 — FastAPI configuration using the built-in template in PyCharm.

In case the application needs to be launched based on environment variables, 
you can use <a href="https://github.com/fastapi/typer" target="_blank">Typer</a> to define a custom CLI 
and configure the startup behavior in detail (see the code snippet down).

```python
# manage.py

import typer
import uvicorn

from config.logging.config import LOG_CONFIG
from config.settings import settings

cli = typer.Typer()


@cli.command()
def runserver(host: str = '0.0.0.0', port: int = 8000) -> None:
    config = {'app': 'config.main:app', 'host': host, 'port': port, 'reload': settings.DEBUG, 'proxy_headers': True}
    if not settings.DEBUG:
        config['log_config'] = LOG_CONFIG

    uvicorn.run(**config)


if __name__ == '__main__':
    cli()
```

In this case, to configure the debugger in PyCharm, follow these steps:

- Open the `Run/Debug Configurations` window;

- Click the `+` icon and choose `Python`;

- Fill in the following fields (see Figure 6 for an example):

  - `Python Interpreter` – select the interpreter configured in the previous step;

  - `Script` – path to the `manage.py` file, usually located at the root of the project;

  - `Parameters` – specify the command to run, in this case: `runserver`.

<img src="https://raw.githubusercontent.com/davyddd/wiki/refs/heads/main/media/setting-up-pycharm/pycharm-fastapi-script-config.png" width="900" alt="FastAPI configuration using a CLI script in PyCharm" title="FastAPI configuration using a CLI script in PyCharm">

Figure 6 — FastAPI configuration using a CLI script in PyCharm.

## Integrating with Git