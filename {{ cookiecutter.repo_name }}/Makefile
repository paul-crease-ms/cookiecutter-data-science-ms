.PHONY: login clean data lint requirements create_environment sync_data_to_s3 sync_data_from_s3

#################################################################################
# GLOBALS                                                                       #
#################################################################################

PROJECT_DIR := $(shell dirname $(realpath $(lastword $(MAKEFILE_LIST))))
MAKEFILE_PATH        := $(abspath $(lastword $(MAKEFILE_LIST)))
AWS_ACCOUNT_ID       := 050665733583
AWS_REGION           := eu-west-1
IMAGE_NAME           := ${PROJECT_DIR}
CONTAINER_REPOSITORY := ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_NAME}
COMMIT_HASH          := $$(git rev-parse HEAD)
DATETIME             := $$(date +%Y%m%d_%H%M)
GIT_BRANCH           := $(shell git rev-parse --abbrev-ref HEAD)
GIT_TAG              := $(shell git name-rev --tags --name-only --refs 'v*' ${COMMIT_HASH} | sed 's/\^0$$//g')
PYTHON_BASE          := $(shell cat docker/airflow-base/Dockerfile | grep FROM | awk -F: '{print $$2}')
TAGS                 := ${COMMIT_HASH} ${DATETIME} ${PYTHON_BASE} ${GIT_BRANCH}
BUCKET = {{ cookiecutter.s3_bucket }}
PROFILE = {{ cookiecutter.aws_profile }}
PROJECT_NAME = {{ cookiecutter.repo_name }}
PYTHON_INTERPRETER = {{ cookiecutter.python_interpreter }}


#################################################################################
# COMMANDS                                                                      #
#################################################################################
login:
	@aws ecr get-login-password --region eu-west-1 | docker login --username AWS --password-stdin ${CONTAINER_REPOSITORY}

setup_hedwig:
	docker-compose -f docker-compose.local.yml build
	docker-compose -f docker-compose.local.yml up -d metadata
	docker-compose -f docker-compose.local.yml run webserver airflow users create --username admin --firstname admin --lastname admin --role Admin --email admin@admin.com --password admin
	docker-compose -f docker-compose.local.yml up -d webserver

## Install Python Dependencies
requirements: test_environment
	$(PYTHON_INTERPRETER) -m pip install -U pip setuptools wheel
	$(PYTHON_INTERPRETER) -m pip install -r requirements.txt

## Make Dataset
data: requirements
	$(PYTHON_INTERPRETER) src/data/make_dataset.py data/raw data/processed

## Delete all compiled Python files
clean:
	find . -type f -name "*.py[co]" -delete
	find . -type d -name "__pycache__" -delete

## Lint using flake8
lint:
	black src

## Upload Data to S3
sync_data_to_s3:
ifeq (default,$(PROFILE))
	aws s3 sync data/ s3://$(BUCKET)/data/
else
	aws s3 sync data/ s3://$(BUCKET)/data/ --profile $(PROFILE)
endif

## Download Data from S3
sync_data_from_s3:
ifeq (default,$(PROFILE))
	aws s3 sync s3://$(BUCKET)/data/ data/
else
	aws s3 sync s3://$(BUCKET)/data/ data/ --profile $(PROFILE)
endif

## Set up python interpreter environment
create_environment:
	@echo ">>> Installing virtual environment if not already installed.":
	python3 -m venv venv
	@bash -c "mkdir -p hedwig/dags/test"
	@bash -c "touch hedwig/dags/test/test_dag.py"
	@echo ">>> Installing virtual environment if not already installed."
	@echo ">>> New virtualenv created. Activate with:venv/bin/activate"


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
	| LC_ALL='C' sort --ignore-case \
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
