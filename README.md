# Environment creator for Conda and Python

## Overview
This script automates the process of creating Python and Conda environments, it checks if the required directories exists and if Conda or Python is installed.
As user you only need to specify the environment you want to create (Conda or Python), the path for the symbolic link and the environment name.

## Requirements
- Conda
- Python
- Bash (Linux shell)

## How to use
1. Create a dir in /home/$USER called bin
``` bash
mkdir /home/$USER/bin
```

2. Clone this repository, copy the script into the bin directory, make it executable and remove the cloned dir
``` bash
git clone https://github.com/Rogudev/tool_env_conda-python_creator.git

cp tool_env_conda-python_creator/env_creator /home/$USER/bin/

chmod +x /home/$USER/bin/env_creator

rm tool_env_conda-python_creator
```

3. Export the path 
``` bash
export PATH="$PATH:/home/$USER/bin"
```

4. Use the script from shell by typing 
```bash
env_creator
```

### Note
You can change the script's name if you want to use a different command to activate it.

## How it works
1. Paths definitions
```bash
conda_env_dir="/home/$USER/Environments/conda"
python_env_dir="/home/$USER/Environments/python"
```

2. Check the existence of these paths
```bash
if [ ! -e "$conda_env_dir" ]; then
    echo -e "\nCreating the requred dir for Conda envs."
    mkdir -p $conda_env_dir
    echo created $conda_env_dir path
fi
if [ ! -e "$python_env_dir" ]; then
    echo -e "\nCreating the requred dir for Python envs."
    mkdir -p $python_env_dir
    echo created $python_env_dir path
fi
```

3. Environment selection and check

- Ask the environment type in a ```bash while true; do``` loop to handle an invalid input, then the choice is handled to check if Conda or python is installed:
```bash
while true; do
    echo -e "\nConda or Python env? \n\t1-Conda \n\t2-Python"
    read -p ">> " environment
```
- Conda check, if exists break the loop
```bash
    # handle choice
    if [ "$environment" -eq 1 ]; then
        echo "You selected Conda environment."
        # check if conda exists with 'which conda'
        exists_conda=$(which conda)
        if [[ -z "$exists_conda" ]]; then
            echo "Conda not found, install it and run again"
            echo "Press enter to exit"
            read
            exit
        else 
            echo "Existing envs:"
            conda env list
            break 
        fi
```
- To check Python, there are 2 commands: _python_ and _python3_, if the command works break the loop
```bash
    elif [ "$environment" -eq 2 ]; then
        echo "You selected Python environment."
        # check if conda exists with 'which python3'
        exists_python=$(which python3)

        if [[ -z "$exists_python" ]]; then
            exists_python=$(which python)

            if [[ -z "exists_python" ]]; then
                echo "Python not found, install it and run again"
                echo "Press enter to exit"
                read
                exit
            else
                echo "Python found at $exists_python"
                python_command="python"
                echo -e "\nExisting envs:"
                echo "----------------------------"
                ls $python_env_dir
                echo "----------------------------"
                break 
            fi

        else 
            echo "Python3 found at $exists_python"
            python_command="python3"
            echo -e "\nExisting envs:"
            echo "----------------------------"
            ls $python_env_dir
            echo "----------------------------"
            break 
        fi
```
- If the selection is not valid, notify it, then the loop starts again
```bash
        
    else
        echo -e "\nInvalid choice. Please select 1 or 2."
    fi
done
```

4. Ask for the path to create the access link and remove quotes
```bash
echo -e "\nInsert the path where to create the access link"
read -p ">> " link_dir
# remove quotes
link_dir=${link_dir//\'/}
link_dir=${link_dir//\"/}
```
5. Get environment name
```bash
# get env name
echo -e "\nInsert the env name"
read -p ">> " envname
echo
```

6. Environment creation process

- **Conda env**, the python version has a default value, 3.9 in this case
```bash
if [ "$environment" -eq 1 ]; then
    # for Conda ask python version
    read -p "Input the Python version (3.9 by default): " python_version
    python_version=${python_version:-3.9}

    # create env
    echo "Creating Conda environment..."
    conda create --name "$envname" python="$python_version"

    # link env: target and link_path
    ln -s $conda_env_dir/$envname $link_dir
```

- **Python env** with the system python version
### **NOTE**
To create a python venv there are 2 differents commands: _python_ and _python3_, the script checks which one works and creates the venv using the correct command 
```bash
elif [ "$environment" -eq 2 ]; then
    echo "Creating Python environment..."
    cd $python_env_dir

    # check if exists
    if ls "$envname" > /dev/null 2>&1; then
        echo -e "Python env exists replace? \n\t1 - yes \n\t2 - no"
        read -p ">> " choice

        if [ "$choice" -eq 1 ]; then
            $python_command -m venv $envname
            echo "Python env created successfully"

        elif [ "$choice" -eq 2 ]; then
            echo "$python_command venv not created"
        fi
    else
        $python_command -m venv $envname
        echo "Python env created successfully"
    fi
```

7. **Access link** creation
**NOTE**
The quotes were removed in **step 4** to avoid issues I got in test phase, here, to avoid issues with name's spaces, I used the quotes again
```bash
 echo "Creating the link..."
    
    if [ -e "$link_dir/$envname" ]; then
        echo -e "\nLink already exists at $link_dir. Removing the existing link."
        rm -rf "$link_dir/$envname" 
    fi

    ln -s "$python_env_dir/$envname" "$link_dir"
fi
```