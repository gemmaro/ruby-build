all: ja/share/man/man1/ruby-build.1
.PHONY: all

ja/share/man/man1/ruby-build.1: ja/share/man/man1/ruby-build.1.adoc ja/Makefile ja/bin/ruby-build
	${MAKE} --directory ja share/man/man1/ruby-build.1

ja/bin/ruby-build: ../bin/ruby-build
	mkdir -p $(dir $@)
	ln $< $@

ja/Makefile: ../Makefile
	ln $< $@

ja/share/man/man1/ruby-build.1.adoc: ../share/man/man1/ruby-build.1.adoc po/ja.po po4a.cfg po/en.pot credit.adoc credit.md po/ja.po
	po4a po4a.cfg

po/en.pot:
	touch $@
