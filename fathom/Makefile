TREE_FILE = results/gisaid/gisaidAndPublic.2022-09-14.masked\ \(1\).pb.gz
METADATA_FILE = results/gisaid/metadata_2022_09_19.tsv.gz

# MAKECMD = $(MAKE) --makefile $(THIS_MAKEFILE)
MAKECMD = $(MAKE)


# AWS
.PHONY: setup-aws
setup-aws:
	sudo snap install aws-cli
	sudo ln -s /usr/bin/python3.8 /usr/bin/python
	sudo apt install python3.8-venv


# Assumes:
# - latest python 3.9.x
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


RUN_DT = $(date +%Y-%m-%d)
.PHONY: setup-results-dir
setup-results-dir:
	cd ..; \
	mkdir -p results.$(RUN_DT)/gisaid; \
	ln -s results.$(RUN_DT) results; \
	echo copy metadata_YYYY_MM_DD.tsv.gz to gisaid dir; \
	echo copy gisaidAndPublic.YYYY-MM-DD.masked\ \(1\).pb.gz to gisaid dir


.PHONY: run-data-gen
run-data-gen:
	cd ..; \
	fathom/venv/bin/ipython generate_epiToPublicAndDate.py


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
