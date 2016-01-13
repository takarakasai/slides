# Makefile for *.dot --> *.png

SRCS= $(wildcard *.dot)
#TGTS= $(basename $(SRCS)).png
TGTS= $(SRCS:.dot=.png)

CMD=dot

all: ${TGTS}

.SUFFIXES: .dot .png

.dot.png:
	@dot -Tpng -o$@ $<

.PHONY: list
list:
	@echo sources : ${SRCS}
	@echo targets : ${TGTS}

.PHONY: show
show: ${TGTS}
	@for TGT in ${TGTS} ; do \
		display $$TGT ; done

.PHONY: clean
clean:
	@rm ${TGTS}

