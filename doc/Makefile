# Makefile for LAMMPS documentation

SHELL 	      = /bin/bash
SHA1          = $(shell echo $USER-$PWD | python utils/sha1sum.py)
BUILDDIR      = /tmp/lammps-docs-$(SHA1)
RSTDIR        = $(BUILDDIR)/rst
VENV          = $(BUILDDIR)/docenv
TXT2RST       = $(VENV)/bin/txt2rst
ANCHORCHECK   = $(VENV)/bin/doc_anchor_check

PYTHON        = $(shell which python3)
VIRTUALENV     = virtualenv
HAS_PYTHON3    = NO
HAS_VIRTUALENV = NO

ifeq ($(shell which python3 >/dev/null 2>&1; echo $$?), 0)
HAS_PYTHON3 = YES
endif

ifeq ($(shell which virtualenv-3 >/dev/null 2>&1; echo $$?), 0)
VIRTUALENV     = virtualenv-3
HAS_VIRTUALENV = YES
endif

ifeq ($(shell which virtualenv >/dev/null 2>&1; echo $$?), 0)
VIRTUALENV     = virtualenv
HAS_VIRTUALENV = YES
endif

SPHINXEXTRA = -j $(shell $(PYTHON) -c 'import multiprocessing;print(multiprocessing.cpu_count())')
SOURCES=$(filter-out $(wildcard src/lammps_commands*.txt) src/lammps_support.txt src/lammps_tutorials.txt,$(wildcard src/*.txt))
OBJECTS=$(SOURCES:src/%.txt=$(RSTDIR)/%.rst)

.PHONY: help clean-all clean epub mobi html pdf old venv spelling anchor_check

# ------------------------------------------

help:
	@echo "Please use \`make <target>' where <target> is one of"
	@echo "  html       create HTML doc pages in html dir"
	@echo "  pdf        create Manual.pdf and Developer.pdf in this dir"
	@echo "  old        create old-style HTML doc pages in old dir"
	@echo "  fetch      fetch HTML and PDF files from LAMMPS web site"
	@echo "  epub       create ePUB format manual for e-book readers"
	@echo "  mobi       convert ePUB to MOBI format manual for e-book readers (e.g. Kindle)"
	@echo "                      (requires ebook-convert tool from calibre)"
	@echo "  clean      remove all intermediate RST files"
	@echo "  clean-all  reset the entire build environment"
	@echo "  txt2html   build txt2html tool"
	@echo "  anchor_check  scan for duplicate anchor labels"

# ------------------------------------------

clean-all: clean
	rm -rf $(BUILDDIR)/* utils/txt2html/txt2html.exe

clean:
	rm -rf $(RSTDIR) html old epub
	rm -rf spelling

clean-spelling:
	rm -rf spelling

html: $(OBJECTS) $(ANCHORCHECK)
	@(\
		. $(VENV)/bin/activate ;\
		cp -r src/* $(RSTDIR)/ ;\
		sphinx-build $(SPHINXEXTRA) -b html -c utils/sphinx-config -d $(BUILDDIR)/doctrees $(RSTDIR) html ;\
		echo "############################################" ;\
		doc_anchor_check src/*.txt ;\
		echo "############################################" ;\
		deactivate ;\
	)
	-rm html/searchindex.js
	@rm -rf html/_sources
	@rm -rf html/PDF
	@rm -rf html/USER
	@cp -r src/PDF html/PDF
	@cp -r src/USER html/USER
	@rm -rf html/PDF/.[sg]*
	@rm -rf html/USER/.[sg]*
	@rm -rf html/USER/*/.[sg]*
	@rm -rf html/USER/*/*.[sg]*
	@echo "Build finished. The HTML pages are in doc/html."

spelling: $(OBJECTS) utils/sphinx-config/false_positives.txt
	@(\
		. $(VENV)/bin/activate ;\
		pip install sphinxcontrib-spelling ;\
		cp -r src/* $(RSTDIR)/ ;\
        cp utils/sphinx-config/false_positives.txt $(RSTDIR)/ ;\
		sphinx-build -b spelling -c utils/sphinx-config -d $(BUILDDIR)/doctrees $(RSTDIR) spelling ;\
		deactivate ;\
	)
	@echo "Spell check finished."

epub: $(OBJECTS)
	@mkdir -p epub
	@rm -f LAMMPS.epub
	@cp src/JPG/lammps-logo.png epub/
	@(\
		. $(VENV)/bin/activate ;\
		cp -r src/* $(RSTDIR)/ ;\
		sphinx-build $(SPHINXEXTRA) -b epub -c utils/sphinx-config -d $(BUILDDIR)/doctrees $(RSTDIR) epub ;\
		deactivate ;\
	)
	@mv  epub/LAMMPS.epub .
	@rm -rf epub
	@echo "Build finished. The ePUB manual file is created."

mobi: epub
	@rm -f LAMMPS.mobi
	@ebook-convert LAMMPS.epub LAMMPS.mobi
	@echo "Conversion finished. The MOBI manual file is created."

pdf: utils/txt2html/txt2html.exe
	@(\
		set -e; \
		cd src; \
		../utils/txt2html/txt2html.exe -b *.txt; \
		htmldoc --batch lammps.book;          \
		for s in `echo *.txt | sed -e 's,\.txt,\.html,g'` ; \
			do grep -q $$s lammps.book || \
			echo doc file $$s missing in src/lammps.book; done; \
		rm *.html; \
		cd Developer; \
		pdflatex developer; \
		pdflatex developer; \
		mv developer.pdf ../../Developer.pdf; \
	)

old: utils/txt2html/txt2html.exe
	@rm -rf old
	@mkdir old; mkdir old/Eqs; mkdir old/JPG; mkdir old/PDF
	@cd src; ../utils/txt2html/txt2html.exe -b *.txt; \
	  mv *.html ../old; \
	  cp Eqs/*.jpg ../old/Eqs; \
	  cp JPG/* ../old/JPG; \
	  cp PDF/* ../old/PDF;

fetch:
	@rm -rf html_www Manual_www.pdf Developer_www.pdf
	@curl -s -o Manual_www.pdf http://lammps.sandia.gov/doc/Manual.pdf
	@curl -s -o Developer_www.pdf http://lammps.sandia.gov/doc/Developer.pdf
	@curl -s -o lammps-doc.tar.gz http://lammps.sandia.gov/tars/lammps-doc.tar.gz
	@tar xzf lammps-doc.tar.gz
	@rm -f lammps-doc.tar.gz

txt2html: utils/txt2html/txt2html.exe

anchor_check : $(ANCHORCHECK)
	@(\
		. $(VENV)/bin/activate ;\
		doc_anchor_check src/*.txt ;\
		deactivate ;\
	)

# ------------------------------------------

utils/txt2html/txt2html.exe: utils/txt2html/txt2html.cpp
	g++ -O -Wall -o $@ $<

$(RSTDIR)/%.rst : src/%.txt $(TXT2RST)
	@(\
		mkdir -p $(RSTDIR) ; \
		. $(VENV)/bin/activate ;\
		txt2rst -v $< > $@ ;\
		deactivate ;\
	)

$(VENV):
	@if [ "$(HAS_PYTHON3)" == "NO" ] ; then echo "Python3 was not found! Please check README.md for further instructions" 1>&2; exit 1; fi
	@if [ "$(HAS_VIRTUALENV)" == "NO" ] ; then echo "virtualenv was not found! Please check README.md for further instructions" 1>&2; exit 1; fi
	@( \
		$(VIRTUALENV) -p $(PYTHON) $(VENV); \
		. $(VENV)/bin/activate; \
		pip install Sphinx; \
		pip install sphinxcontrib-images; \
		deactivate;\
	)

$(TXT2RST) $(ANCHORCHECK): $(VENV)
	@( \
		. $(VENV)/bin/activate; \
		(cd utils/converters;\
		python setup.py develop);\
		deactivate;\
	)
