# Setup


## Configuration

The application needs a configuration file, placed in `src/config.yml`, mainly for local development, dependend on the setup environment. The configuration configures the access to a `Graph Database` (triple store). A sample configuration file with dummy values is placed in `src/example.config.yml`. Values can be empty for the local startup of the service.

To configure the app localy, move `src/example.config.yml` to `src/config.yml`.


## Installation

To install all the python dependencies, use `conda` __or__ `pip` or any other setup environment which you like. I shorty explain *two* options, to set up a local dev environment to have a clean base where the dependencies can be installed.

### Conda Environment

To set up a local environment with conda, first install anaconda, if not done yet using the installer from the website: https://www.anaconda.com/.

Then, to setup a dev environment use:

````bash
conda create --name similarity python=3.7
````

Then you need to activate the conda environment with

````bash
conda activate similarity
````

### Python PIP

To set up a local environment with the normal pip python package manager, we use the `venv` package from python. To do so, follow this three steps:

````bash
# 1. Create an environment called 'similarity'.
python3 -m venv similarity

# 2. Activate the environment.
source similarity/bin/activate

# 3. You can validate if the environmental setup works: The path should direct to the local environment.
which python3
````

### Install Dependencies

Now, we can install all dependencies in the clean environment. If the environment was **conda** or **venv** or any other manager, we can use the _same_ command, using `pip` and `python`.

First, switch the folder, by running following in the terminal:

````bash
cd src
````

Then, please run in the terminal:

````bash
# install all dependencies
pip -r requirements.txt

# load the nltk tokenizer
python -c "import nltk; nltk.download(\"punkt\")"
````


## Start Application

Next, after we set up everything, make sure to be in the `src` folder and run: 

````bash
python setup_datalake.py
````

This script takes about 2-3 minutes and set up required dataframes and trains a small fastText model which is required by the service.

Finally, we can start the application with:

````bash
python AppEntry.py
````


A documentation about the api is available as a swagger documentation: http://localhost:51840/swagger.


