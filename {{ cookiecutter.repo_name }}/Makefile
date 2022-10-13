.PHONY: clean data lint format requirements sync_data_down sync_data_up help

#################################################################################
# GLOBALS                                                                       #
#################################################################################

PROJECT_NAME = {{ cookiecutter.repo_name }}
PYTHON_VERSION = {{ cookiecutter.python_version_number }}
PYTHON_INTERPRETER = python


#################################################################################
# COMMANDS                                                                      #
#################################################################################

{% if cookiecutter.dependency_file != 'none' %}
requirements:  ## Install Python Dependencies
	{% if "requirements.txt" == cookiecutter.dependency_file -%}
	$(PYTHON_INTERPRETER) -m pip install -U pip setuptools wheel
	$(PYTHON_INTERPRETER) -m pip install -r requirements.txt
	{% elif "environment.yml" == cookiecutter.dependency_file -%}
	conda env update --name $(PROJECT_NAME) --file environment.yml --prune
	{% elif "Pipfile" == cookiecutter.dependency_file -%}
	pipenv install
	{% endif %}
{% endif %}

clean:  ## Delete all compiled Python files
	find . -type f -name "*.py[co]" -delete
	find . -type d -name "__pycache__" -delete

lint:  ## Lint using flake8 and black (use `make format` to do formatting)
	flake8 {{ cookiecutter.module_name }}
	black --check --config pyproject.toml {{ cookiecutter.module_name }}
	isort --check {{ cookiecutter.module_name }}

format:  ## Format source code with black and isort
	isort {{ cookiecutter.module_name }}
	black --config pyproject.toml {{ cookiecutter.module_name }}

{% if not cookiecutter.dataset_storage.none %}
sync_data_down:  ## Download data from storage system
	{% if cookiecutter.dataset_storage.s3 -%}
	aws s3 sync s3://{{ cookiecutter.dataset_storage.s3.bucket }}/data/\
		data/ {% if cookiecutter.dataset_storage.s3.aws_profile != 'default' %} --profile {{ cookiecutter.dataset_storage.s3.aws_profile }}{% endif %}
	{% elif cookiecutter.dataset_storage.azure -%}
	az storage blob download-batch -s {{ cookiecutter.dataset_storage.azure.container }}/data/ \
		-d data/
	{% elif cookiecutter.dataset_storage.gcs -%}
	gsutil rsync gs://{{ cookiecutter.dataset_storage.gcs.bucket }}/data/ data/
	{% endif %}

sync_data_up:  ## Upload data to storage system
	{% if cookiecutter.dataset_storage.s3 -%}
	aws s3 sync s3://{{ cookiecutter.dataset_storage.s3.bucket }}/data/ data/\
		{% if cookiecutter.dataset_storage.s3.aws_profile %} --profile $(PROFILE){% endif %}
	{% elif cookiecutter.dataset_storage.azure -%}
	az storage blob upload-batch -d {{ cookiecutter.dataset_storage.azure.container }}/data/ \
		-s data/
	{% elif cookiecutter.dataset_storage.gcs -%}
	gsutil rsync data/ gs://{{ cookiecutter.dataset_storage.gcs.bucket }}/data/
	{% endif %}
{% endif %}

{% if cookiecutter.environment_manager != 'none' %}
create_environment:  ## Set up python interpreter environment
	{% if cookiecutter.environment_manager == 'conda' -%}
	{% if cookiecutter.dependency_file != 'environment.yml' %}
	conda create --name $(PROJECT_NAME) python=$(PYTHON_VERSION) -y
	{% else -%}
	conda env create --name $(PROJECT_NAME) python=$(PYTHON_VERSION) -f environment.yml
	{% endif %}
	@echo ">>> conda env created. Activate with:\nconda activate $(PROJECT_NAME)"
	{% elif cookiecutter.environment_manager == 'virtualenv' -%}
	@bash -c "if [ ! -z `which virtualenvwrapper.sh` ]; then source `which virtualenvwrapper.sh`; mkvirtualenv $(PROJECT_NAME) --python=$(PYTHON_INTERPRETER); else mkvirtualenv.bat $(PROJECT_NAME) --python=$(PYTHON_INTERPRETER); fi"
	@echo ">>> New virtualenv created. Activate with:\nworkon $(PROJECT_NAME)"
	{% elif cookiecutter.environment_manager == 'pipenv' -%}
	pipenv --python $(PYTHON_VERSION)
	@echo ">>> New pipenv created. Activate with:\npipenv shell"
	{% endif %}
{% endif %}


#################################################################################
# PROJECT RULES                                                                 #
#################################################################################

{% if cookiecutter.include_code_scaffold != 'Yes' %}
data: requirements  ## Create data/processed from data/raw
	$(PYTHON_INTERPRETER) {{ cookiecutter.module_name }}/data/process.py
{% endif %}


#################################################################################
# Self Documenting Commands                                                     #
#################################################################################

.DEFAULT_GOAL := help

define PRINT_HELP_PYSCRIPT
import re, sys

for line in sys.stdin:
	match = re.match(r'^([a-zA-Z_-]+):.*?## (.*)$$', line)
	if match:
		target, help = match.groups()
		print("%-20s %s" % (target, help))
endef
export PRINT_HELP_PYSCRIPT

help:
	@python -c "$$PRINT_HELP_PYSCRIPT" < $(MAKEFILE_LIST)