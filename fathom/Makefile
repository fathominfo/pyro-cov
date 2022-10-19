TREE_FILE = results/gisaid/gisaidAndPublic.2022-09-14.masked.pb.gz
METADATA_FILE = results/gisaid/metadata_2022_09_19.tsv.gz

# MAKECMD = $(MAKE) --makefile $(THIS_MAKEFILE)
MAKECMD = $(MAKE)


# AWS setup task, is probably not going to be run, but is here as a reference
# Required manual prerequisites:
# mkdir ~/work && cd ~/work; \
# git clone https://github.com/fathominfo/pyro-cov.git; \
# cd ~/work/pyro-cov/fathom; \
# make setup-aws
.PHONY: setup-aws
setup-aws:
	sudo snap install aws-cli --classic
	sudo ln -s /usr/bin/python3.8 /usr/bin/python
	sudo apt install python3.8-venv


# Assumes:
# - latest python 3.9.x (on MacOS)
# - source venv/bin/activate (or fish equivalent)
.PHONY: setup
setup: update-all-github-repos
	python -m venv venv; \
	venv/bin/pip install --upgrade pip; \
	venv/bin/pip install -r requirements.txt
	echo "done"


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
		git clone https://github.com/$${ORG_NAME}/$${REPO_NAME}.git; \
	fi


.PHONY: update-all-github-repos
update-all-github-repos:
	ORG_NAME=cov-lineages   REPO_NAME=pango-designation $(MAKECMD) update-github-repo
	ORG_NAME=nextstrain     REPO_NAME=nextclade         $(MAKECMD) update-github-repo
	ORG_NAME=CSSEGISandData REPO_NAME=COVID-19          $(MAKECMD) update-github-repo
	ORG_NAME=owid           REPO_NAME=covid-19-data     $(MAKECMD) update-github-repo


RUN_DT = $(shell date +%Y-%m-%d)
.PHONY: setup-results-dir
setup-results-dir:
	cd ..; \
	mkdir -p results.$(RUN_DT)/gisaid; \
	ln -s results.$(RUN_DT) results; \
	@echo in results/gisaid dir, run s3 copy with updated file name (timestamp); \
	@echo run: aws s3 cp s3://pyr0/gisaid/gisaidAndPublic.2022-09-14.masked.pb.gz .;\
	@echo copy gisaidAndPublic.YYYY-MM-DD.masked\ \(1\).pb.gz to gisaid dir


.PHONY: run-data-gen
run-data-gen:
	cd ..; \
	fathom/venv/bin/ipython generate_epiToPublicAndDate.ipynb


.PHONY: run-usher
run-usher:
	cd ..; \
	date; \
	PYTHONPATH=. fathom/venv/bin/python scripts/preprocess_usher.py --tree-file-in $(TREE_FILE) --gisaid-metadata-file-in $(METADATA_FILE)
	date

.PHONY: run-pyro-model
run-pyro-model:
	cd ..; \
	date; \
	PYTHONPATH=. fathom/venv/bin/python scripts/mutrans.py
	date
