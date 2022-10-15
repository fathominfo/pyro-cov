TREE_FILE = results/gisaid/gisaidAndPublic.2022-09-14.masked\ \(1\).pb.gz
METADATA_FILE = results/gisaid/metadata_2022_09_19.tsv.gz

# Assumes:
# - latest python 3.9.x
# - source venv/bin/activate (or fish equivalent)
.PHONY: setup
setup:
	python -m venv venv; \
	venv/bin/pip install --upgrad pip; \
	venv/bin/pip install -r fathom/requirements.txt; \
	mkdir -p ~/github/cov-lineages; \
	cd ~/github/cov-lineages; \
	git clone https://github.com/cov-lineages/pango-designation.git; \
	mkdir -p ~/github/netstrain; \
	cd ~/github/nextstrain; \
	git clone https://github.com/nextstrain/nextclade.git; \
	echo "done"


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
