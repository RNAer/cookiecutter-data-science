.PHONY: clean data lint pip_requirement conda_requirement dependencies sync_data_to_s3 sync_data_from_s3

#################################################################################
# GLOBALS                                                                       #
#################################################################################

PROJECT_DIR := $(shell dirname $(realpath $(lastword $(MAKEFILE_LIST))))
BUCKET = {{ cookiecutter.s3_bucket }}
PROJECT_NAME = {{ cookiecutter.repo_name }}
PYTHON_INTERPRETER = {{ cookiecutter.python_interpreter }}

ifeq (,$(shell which conda))
HAS_CONDA = False
else
HAS_CONDA = True
endif

#################################################################################
# COMMANDS                                                                      #
#################################################################################

## Install Python Dependencies
pip_requirement:
	pip install -r pip_requirements.txt
## Install conda Dependencies
conda_requirement:
	conda install -c conda-forge -c bioconda -c biocore --file conda_requirements.txt
## Install Dependencies
dependencies: conda_requirement pip_requirement
	@echo ">>> All dependencies installed"
## Preprocess Dataset
data:
	$(PYTHON_INTERPRETER) src/data/make_dataset.py

## Delete all compiled Python files
clean:
	find . -name "*.pyc" -exec rm {} \;

## Lint using flake8
lint:
	flake8 --exclude=lib/,bin/,docs/conf.py .

## Upload Data to S3
sync_data_to_s3:
	aws s3 sync data/ s3://$(BUCKET)/data/

## Download Data from S3
sync_data_from_s3:
	aws s3 sync s3://$(BUCKET)/data/ data/

## Set up python interpreter environment
make_env:
ifeq (True,$(HAS_CONDA))
		@echo ">>> Detected conda, creating conda environment."
ifeq (3,$(findstring 3,$(PYTHON_INTERPRETER)))
	conda create --name $(PROJECT_NAME) python=3.5
else
	conda create --name $(PROJECT_NAME) python=2.7
endif
		@echo ">>> New conda env created. Activate with:\nsource activate $(PROJECT_NAME)"
else
	@pip install -q virtualenv virtualenvwrapper
	@echo ">>> Installing virtualenvwrapper if not already intalled.\nMake sure the following lines are in shell startup file\n\
	export WORKON_HOME=$$HOME/.virtualenvs\nexport PROJECT_HOME=$$HOME/Devel\nsource /usr/local/bin/virtualenvwrapper.sh\n"
	@bash -c "source `which virtualenvwrapper.sh`;mkvirtualenv $(PROJECT_NAME) --python=$(PYTHON_INTERPRETER)"
	@echo ">>> New virtualenv created. Activate with:\nworkon $(PROJECT_NAME)"
endif

## Test python environment is setup correctly
test_env:
	$(PYTHON_INTERPRETER) test_environment.py

#################################################################################
# SYNC WITH REMOTE RSYNC_SERVER                                                       #
#################################################################################
RSYNC_SERVER = {{ cookiecutter.rsync_server }}
RSYNC_DIR = {{ cookiecutter.rsync_dir }}
# define the max file size your wanna sync.
# you can leave this as empty.
RSYNC_SIZE =
# define the file name containing the include/exclude patterns.
# you can leave this as empty
RSYNC_FILE = rsync.ie


# size filter
ifeq ($(RSYNC_SIZE),)
    RSYNC_SIZE_FILTER =
else
    RSYNC_SIZE_FILTER = --max-size=$(RSYNC_SIZE)
endif

# check if you have the exclude file
ifeq ($(wildcard $(RSYNC_FILE)),)
    RSYNC_FILTER =
else
    RSYNC_FILTER = --filter '. $(RSYNC_FILE)'
endif

RSYNC_FLAGS = -avuhz --prune-empty-dirs $(RSYNC_FILTER) $(RSYNC_SIZE_FILTER) -e ssh

pull:
	rsync $(RSYNC_FLAGS) -n $(RSYNC_SERVER):$(RSYNC_DIR)/ .
push:
	rsync $(RSYNC_FLAGS) -n . $(RSYNC_SERVER):$(RSYNC_DIR)/

pull_real:
	rsync $(RSYNC_FLAGS) $(RSYNC_SERVER):$(RSYNC_DIR)/ .
push_real:
	rsync $(RSYNC_FLAGS) . $(RSYNC_SERVER):$(RSYNC_DIR)/

#################################################################################
# PROJECT RULES                                                                 #
#################################################################################



#################################################################################
# Self Documenting Commands                                                     #
#################################################################################

.DEFAULT_GOAL := help

# Inspired by <http://marmelab.com/blog/2016/02/29/auto-documented-makefile.html>
# sed script explained:
# /^##/:
# 	* save line in hold space
# 	* purge line
# 	* Loop:
# 		* append newline + line to hold space
# 		* go to next line
# 		* if line starts with doc comment, strip comment character off and loop
# 	* remove target prerequisites
# 	* append hold space (+ newline) to line
# 	* replace newline plus comments by `---`
# 	* print line
# Separate expressions are necessary because labels cannot be delimited by
# semicolon; see <http://stackoverflow.com/a/11799865/1968>
.PHONY: help
help:
	@echo "$$(tput bold)Available rules:$$(tput sgr0)"
	@echo
	@sed -n -e "/^## / { \
		h; \
		s/.*//; \
		:doc" \
		-e "H; \
		n; \
		s/^## //; \
		t doc" \
		-e "s/:.*//; \
		G; \
		s/\\n## /---/; \
		s/\\n/ /g; \
		p; \
	}" ${MAKEFILE_LIST} \
	| awk -F '---' \
		-v ncol=$$(tput cols) \
		-v indent=19 \
		-v col_on="$$(tput setaf 6)" \
		-v col_off="$$(tput sgr0)" \
	'{ \
		printf "%s%*s%s ", col_on, -indent, $$1, col_off; \
		n = split($$2, words, " "); \
		line_length = ncol - indent; \
		for (i = 1; i <= n; i++) { \
			line_length -= length(words[i]) + 1; \
			if (line_length <= 0) { \
				line_length = ncol - indent - length(words[i]) - 1; \
				printf "\n%*s ", -indent, " "; \
			} \
			printf "%s ", words[i]; \
		} \
		printf "\n"; \
	}' \
	| more $(shell test $(shell uname) = Darwin && echo '--no-init --raw-control-chars')
