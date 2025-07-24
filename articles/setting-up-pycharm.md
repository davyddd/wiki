# Setting up PyCharm

## Table of Contents

- [Getting started](#getting-started)
- [Configuring the Python Interpreter](#configuring-the-python-interpreter)
- [Setting up the Debugger](#setting-up-the-debugger)
- [Integrating with Git](#integrating-with-git)

## Getting started

**PyCharm** is an integrated development environment (IDE) for the Python programming language.

Before you start working, you need to install the IDE. Download PyCharm from [here](https://www.jetbrains.com/pycharm/) — 
the Professional version includes a 30-day free trial — and install it on your system. To start using PyCharm, 
you’ll need to create a [JetBrains account](https://account.jetbrains.com/signup). Then open the IDE, 
sign in with your credentials, and open your project.

**Note**: The free version of PyCharm has limited functionality — for example, it doesn’t support Docker- or 
Compose-based Python interpreters — which makes it unsuitable for commercial development.

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

The **Python interpreter** is the environment that PyCharm relies on to analyze and execute code.
Its configuration affects project navigation, access to dependency libraries (including their source code), 
and the highlighting of syntax errors and type mismatches. Without proper interpreter setup, 
development efficiency decreases and the likelihood of bugs increases.

Next, the focus will be on configuring how PyCharm interacts with the development environment inside Docker containers.
Therefore, if you’re not familiar with containerizing applications using Docker, 
or if Docker and Compose are not installed on your host machine, it is recommended to read [this article](https://github.com/davyddd/wiki/blob/main/articles/docker-and-compose.md).

On macOS, the first step is to enable Docker support in PyCharm:

- Open PyCharm settings (`Ctrl + Alt + S` or `Command + ,`).

- Navigate to `Build, Execution, Deployment` → `Docker`.

- Docker is not listed in the opened window, click the `+` icon and configure it as shown in Figure 1.

![Docker configuration](https://github.com/davyddd/wiki/blob/main/media/setting-up-pycharm/docker-configuration.png)

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

<img src="https://github.com/davyddd/wiki/blob/main/media/setting-up-pycharm/compose-interpreter-setup.png" width="400" title="Compose interpreter setup">

Figure 2 — Compose interpreter setup.

## Setting up the Debugger

## Integrating with Git