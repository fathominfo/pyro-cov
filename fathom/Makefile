TREE_FILE = results/gisaid/gisaidAndPublic.2022-09-14.masked.pb.gz
METADATA_FILE = results/gisaid/metadata_2022_09_19.tsv.gz

# MAKECMD = $(MAKE) --makefile $(THIS_MAKEFILE)
MAKECMD = $(MAKE)


# AWS setup task, is probably not going to be run, but is here as a reference
# Notes:
#   - 16GB memory is not enough to run generate_epiToPublicAndDate.ipynb
#   - 64GB is more than enough, not sure how much is needed exactly
#   - g5.4xlarge is probably the sweet spot (only available in select regions, I used Oregon)
#   - Use the "Deep Learning AMI GPU PyTorch 1.12.1 (Ubuntu 20.04) 20221019" AMI, PyTorch 1.12.1 support CUDA 11.6 which is installed (check with nvidia-smi)
#   - There is a pyro-dev ssh key-pair available in each region (stored in the 1Password CSO Vault, which can be moved)
#   - Needs 300 GB root disk (200GB was not enough the github repos are huge)
#   - Add Pyro-Dev-S3 role to security for S3 access
#   - Allow Custom TCP inbound on port 5000 for Jupyter access (if needed)
# Required manual prerequisites:
# mkdir ~/work && cd ~/work; \
# git clone https://github.com/fathominfo/pyro-cov.git; \
# cd ~/work/pyro-cov; \
# git checkout <fathom development branch>; \
# cd fathom; \
# make setup-aws
.PHONY: setup-aws
setup-aws: aws-installs setup
	@echo >>>>>>>>>> setup-aws done.


.PHONY: aws-installs
aws-installs:
	sudo snap install aws-cli --classic
	sudo ln -s /usr/bin/python3.8 /usr/bin/python
	sudo apt install -y python3.8-venv
	@echo >>>>>>>>>> aws-installs done.


.PHONY: setup
setup: update-all-github-repos setup-results-dir populate-results-dir setup-python-env
	@echo >>>>>>>>>> setup done.


# Assumes:
# - latest python 3.9.x (on MacOS), 3.8.x (on EC2, Ubuntu)
# - source venv/bin/activate (or fish equivalent)
# - using notes from https://pytorch.org/get-started/locally/ for PyTorch 1.12.1, Linux, pip, and CUDA 11.6
.PHONY: setup-python-env
setup-python-env:
	python -m venv venv; \
	cd ..; \
	fathom/venv/bin/pip install --upgrade pip; \
	fathom/venv/bin/pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu116; \
	fathom/venv/bin/pip install -e .; \
	fathom/venv/bin/pip install -r fathom/requirements.txt
	@echo >>>>>>>>>> done


.PHONY: update-github-repo
update-github-repo:
	@if [ -d "$${HOME}/github/$${ORG_NAME}/$${REPO_NAME}" ]; then \
		echo "updating $${HOME}/github/$${ORG_NAME}/$${REPO_NAME}"; \
		cd $${HOME}/github/$${ORG_NAME}/$${REPO_NAME}; \
		git pull; \
	else \
		echo "creating $${HOME}/github/$${ORG_NAME}/$${REPO_NAME}"; \
		mkdir -p $${HOME}/github/$${ORG_NAME}; \
		cd $${HOME}/github/$${ORG_NAME}; \
		# cd $${HOME}/github; \
		# aws s3 cp s3://pyr0/github/$${ORG_NAME}--$${REPO_NAME}-221019.tgz .; \
		# tar xvfz $${ORG_NAME}--$${REPO_NAME}-221019.tgz; \
		# cd $${ORG_NAME}/$${REPO_NAME} && git pull; \
		git clone --depth 1 https://github.com/$${ORG_NAME}/$${REPO_NAME}.git; \
	fi


.PHONY: update-all-github-repos
update-all-github-repos:
	ORG_NAME=cov-lineages   REPO_NAME=pango-designation $(MAKECMD) update-github-repo
	ORG_NAME=nextstrain     REPO_NAME=nextclade         $(MAKECMD) update-github-repo
	ORG_NAME=CSSEGISandData REPO_NAME=COVID-19          $(MAKECMD) update-github-repo
	#ORG_NAME=owid           REPO_NAME=covid-19-data     $(MAKECMD) update-github-repo


RUN_DT = $(shell date +%Y-%m-%d)
RESULTS_DIR = ~/work/results/results.$(RUN_DT)
.PHONY: setup-results-dir
setup-results-dir:
	@cd ..; \
	mkdir -p $(RESULTS_DIR)/gisaid; \
	ln -s $(RESULTS_DIR) results


.PHONY: populate-results-dir
populate-results-dir:
	@echo ">>>>>>>>>> results/gisaid dir needs data files copied from S3"; \
	echo ">>>>>>>>>> note: timestamps will change when data is updated"; \
	echo ">>>>>>>>>> note: aws s3 access requires aws-cli and a special policy/role"
	aws s3 cp s3://pyr0/gisaid/gisaidAndPublic.2022-09-14.masked.pb.gz ../results/gisaid/
	aws s3 cp s3://pyr0/gisaid/metadata_2022_09_19.tsv.gz ../results/gisaid/


.PHONY: run-data-gen
run-data-gen:
	cd ..; \
	fathom/venv/bin/ipython generate_epiToPublicAndDate.ipynb


.PHONY: run-usher
run-usher:
	cd ..; \
	date; \
	PYTHONPATH=. fathom/venv/bin/python scripts/preprocess_usher.py --tree-file-in $(TREE_FILE) --gisaid-metadata-file-in $(METADATA_FILE); \
	date


.PHONY: run-pyro-model
run-pyro-model:
	cd ..; \
	date; \
	PYTHONPATH=. fathom/venv/bin/python scripts/mutrans.py; \
	date


.PHONY: run-pyro-model-vary-holdout
run-pyro-model-vary-holdout:
	cd ..; \
	date; \
	PYTHONPATH=. fathom/venv/bin/python scripts/mutrans.py --vary-holdout; \
	date


.PHONY: run-notebook
run-notebook:
	cd ..; \
	PYTHONPATH=. fathom/venv/bin/jupyter-notebook --no-browser --ip=0.0.0.0 --port=5000
