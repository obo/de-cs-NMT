all: tokenize

install_moses moses:
	wget https://github.com/moses-smt/mosesdecoder/archive/mmt-mvp-v0.12.1.zip -O moses.zip
	unzip moses.zip
	mv mosesdecoder-mmt-mvp-v0.12.1/ moses

DATA_CS=train.cs test.cs dev.cs
DATA_DE=train.de test.de dev.de
DATA=$(DATA_CS) $(DATA_DE)

data $(DATA):
	make -C europarl
	cp europarl/europarl.cs train.cs 
	cp europarl/europarl.de train.de
	make -C news13_test
	cp news13_test/news13.cs test.cs 
	cp news13_test/news13.de test.de
	make -C news11_dev
	cp news11_dev/news11.cs dev.cs 
	cp news11_dev/news11.de dev.de



MOSES=moses/scripts/tokenizer
preprocess: $(DATA) moses
	for d in $(DATA); do \
		echo $$d ; \
		cat $$d | \
		perl $(MOSES)/remove-non-printing-char.perl | \
		perl $(MOSES)/replace-unicode-punctuation.perl | \
		perl $(MOSES)/normalize-punctuation.perl > $$d"".prep ;\
	done

TOKENIZED=dev.cs.prep.tok dev.de.prep.tok train.cs.prep.tok test.cs.prep.tok train.de.prep.tok test.de.prep.tok	
TOK_CMD=perl $(MOSES)/tokenizer.perl -no-escape -threads `nproc`
tokenize $(TOKENIZED): preprocess moses $(DATA)
	for d in $(DATA_CS); do	\
		echo $$d"".prep ; \
		$(TOK_CMD) -l cs < $$d"".prep > $$d"".prep.tok; \
	done
	for d in $(DATA_DE); do	\
		echo $$d"".prep ; \
		$(TOK_CMD) -l de < $$d > $$d"".prep.tok; \
	done


# I skip truecasing...
#RECASE=$(MOSES)/scripts/recaser
#recase:
#	:

install_morfessor p3:
	virtualenv -p python3 p3
	p3/bin/pip install morfessor polyglot pyicu six pycld2 numpy
	p3/bin/python3 -c 'from polyglot.downloader import downloader; downloader.download("morph2.cs"); downloader.download("morph2.de")'

morfesize: morphesize.py $(TOKENIZED)
	for d in $(DATA_CS); do	\
		echo $$d"".prep.tok ; \
		p3/bin/python3 morphesize.py cs < $$d"".prep.tok > $$d"".prep.tok.mor & \
	done
	wait
	for d in $(DATA_DE); do	\
		echo $$d"".prep.tok ; \
		p3/bin/python3 morphesize.py de < $$d"".prep.tok > $$d"".prep.tok.mor & \
	done
	wait

# I skip training my own morfessor model, the polyglot one seems to be OK
#morf_train:
#	p3/bin/morfessor-train 

oov_analysis: $(TOKENIZED)
	perl $(MOSES)/../analysis/oov.pl dev.cs.prep.tok < train.cs.prep.tok
	perl $(MOSES)/../analysis/oov.pl dev.de.prep.tok < train.de.prep.tok
	perl $(MOSES)/../analysis/oov.pl dev.cs.prep.tok.mor < train.cs.prep.tok.mor
	perl $(MOSES)/../analysis/oov.pl dev.de.prep.tok.mor < train.de.prep.tok.mor

check_overlap: $(TOKENIZED)
	python3 check_overlap.py train.cs.prep.tok test.cs.prep.tok	
	python3 check_overlap.py train.de.prep.tok test.de.prep.tok
	python3 check_overlap.py train.cs.prep.tok dev.cs.prep.tok	
	python3 check_overlap.py train.de.prep.tok dev.de.prep.tok

clean_data:
	rm -rf dev* train* test*
