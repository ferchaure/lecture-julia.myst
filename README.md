# lecture-julia.myst

Source for julia.quantecon.org and notebooks in https://github.com/QuantEcon/lecture-julia.notebooks

To contribute, you can use GitHub's online editor for small changes, and do a full local installation for large ones.

See https://github.com/QuantEcon/lecture-julia.myst/blob/main/style.md for some basic coding standards.

## Online Editor

On this website hit `.` to enter into the web editor.  From this, you can submit suggested patches and fix typos.  This will help you create a [pull request](https://quantecon.github.io/lecture-julia.myst/software_engineering/version_control.html#collaboration-via-pull-request) for maintainers here to examine.

## Local Development

It is straightforward to install the Jupyter Book and Julia software necessary for more significant changes.  For Windows support, it is best to use WSL.  See `[WSL Setup Instructions](./wsl.md)` for more details.

### Setup

1. [Install Julia, Conda, and VS Code](https://quantecon.github.io/lecture-julia.myst/getting_started_julia/getting_started.html) following the documentation for using these notes.
2. Modify [VS Code settings](https://quantecon.github.io/lecture-julia.myst/software_engineering/tools_editors.html#optional-extensions-and-settings) and consider [additional extensions](https://quantecon.github.io/lecture-julia.myst/software_engineering/tools_editors.html#optional-extensions).  Some others to consider are the [MyST-Markdown](https://github.com/executablebooks/myst-vs-code) and [Spell Checking](https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker) extensions.  You will also likely want the Python and Jupyter extensions installed
3. Ensure that [Git](https://quantecon.github.io/lecture-julia.myst/software_engineering/version_control.html#setup) is set up correctly.  In particular, this ensures that Windows users use the Linux end-of-line characters.
4. Clone this repository (in VS Code, you can use `<Ctrl+Shift+P>` then `Clone` then `Clone from GitHub` then choose the repo as `https://github.com/QuantEcon/lecture-julia.myst`).

6. Open this repository in VS Code.  If you cloned in a separate terminal, navigate to the directory and type `code .`

7. Start a VS Code terminal with ``<Ctrl+`>`` or through any other method.  Create a conda environment.

    ```bash
    conda create -n lecture-julia.myst python=3.11
    conda activate lecture-julia.myst
    pip install -r requirements.txt
    ```

    This will install all the Jupyter Book packages required to edit and build the lectures.

8.  Set the default interpreter for VS Code's Python extension to be the conda environment
    - Press `<Ctrl-Shift-P>` then `Python: Select Interpreter`.
    - Choose the interpreter with `lecture-julia.myst` which should now be automatically activated in the terminal.
    - If the interpreter does not show up in the drop-down, close and reopen VS Code, then try again. Alternatively, you can run this step at the end of the setup process.
        - Whenever reopening VS Code, re-run `conda activate lecture-julia.myst` to ensure the environment remains active.
9.  Ensure that IJulia is installed for the main Julia environment
    - Ensure you have the conda environment activated with `conda activate lecture-julia.myst`
    - To do this, run `julia` in the terminal and then `] add IJulia` in the Julia REPL.  You will want to do this in the main environment and not the `lectures` environment, as used below.
10.  Install the Julia packages required for the lecture notes.

     ```bash
     julia --project=lectures --threads auto -e 'using Pkg; Pkg.instantiate();'
     ```


**(Optional) REPL Integration**
With [MyST-Markdown](https://github.com/executablebooks/myst-vs-code) and [Julia](https://marketplace.visualstudio.com/items?itemName=julialang.language-julia) installed, you can ensure that pressing `<Ctrl-Enter>` on lines of code are sent to a Julia REPL.
1.  Open Key Bindings with `<Ctrl-K Ctrl-S>`.
2.  Search for the `Julia: Send Current Line or Selection to REPL` binding.
3.  Right Click on the key binding with `juliamarkdown` on it, and choose `Change When Expression`, and change `juliamarkdown` to just `markdown`.


## Executing Code in Markdown Files
If you installed the REPL Integration above, then in a `.md` file,

1. Start a Julia REPL with `> Julia: Start REPL`.
2. Activate the project file in the REPL with `] activate lectures`.
3. Then, assuming that you set up the keybindings above, you can send a line of code in the markdown to the REPL with `<Ctrl-Enter>`.

Code can be executed line by line, or you can select a chunk of code and execute it.

## Executing Intermediate Code in Jupyter Notebooks
From a built jupyterbook, you can navigate to the `.ipynb` used for the generation.  The files are located in `lectures/_build/jupyter_execute/dynamic_programming/mccall_model.ipynb` etc.

There are two options:

1. With the Julia vscode extension installed, open the file and run code cells.
  - The formatting will be incorrect, but the code will be.
  - The first time you run this you may need to choose the kernel.  Assuming you installed with juliaup, you should be able to choose `Julia` as the kernel and the juliaup installed julia version
  - This will automatically activate the appropriate project and manifest file.
2. With your conda environment activated, go `cd lectures` then `jupyter lab`
  - Navigate to the files in jupyter itself.  Because you ran this in the environment with the `Project.toml` and `Manifest.toml` files, the environment should be activated correctly.
  - If you have any issues, it is likely that you need to activate the environment in the terminal, or that you have note done the `] add IJulia` step from a julia REPL (whihc must be done in the default environment, i.e. not activated in `lectures`).

## Example Operations
### Building the lectures
To do a full build of the lectures:

```bash
jupyter-book build lectures
```

or

```bash
jb build lectures
```

This will take a while. But it will populate your cache, so future iteration is faster.
<!--
It is suggested to use WSL on Windows On Windows, if you get the following error:

```
ImportError: DLL load failed while importing win32api: The specified procedure could not be found.
```

then run `conda install pywin32` and build the lectures again.

If you have [Live Preview](https://marketplace.visualstudio.com/items?itemName=ms-vscode.live-server) installed, then go to `_build/html/index.html` in the explorer, and right-click to choose `Live Preview: Show Preview`.
-->

### Cleaning Lectures
To clean up (i.e., delete the build)

```bash
jupyter-book clean lectures
```

or

```bash
jb clean lectures
```

and to clean the `execution` cache you can use

```bash
jb clean lectures --all
```
### Debugging Generated Content

After execution, you can find the generated `.ipynb` and `.jl` files in `_build/jupyter_execute` for each lecture.
- To see errors, you can open these in JupyterLab, the Jupyter support within VS Code, etc.
- If using the Julia REPL in VS Code, make sure to do `] activate lectures` prior to testing to ensure the packages are activated.  This is not necessary when opening in Jupyter.

## Formatting code
Julia code blocks in the myst `.md` files can be formatted using a script in this folder.  To manually do so, insure you have the `] add JuliaFormatter` within your default julia environment, then call on the commandline like
    
```bash 
julia format_myst.jl lectures/getting_started_julia/getting_started.md
```

As a helper, you can call a shell script to do it for an entire folder

```bash
bash format_all_directory.sh lectures/dynamic_programming
```

or to also do the unicode substitutions

```bash
bash format_all_directory.sh lectures/dynamic_programming true
```