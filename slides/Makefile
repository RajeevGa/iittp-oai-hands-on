# OAI 5G NR Workshop - IIT Tirupati, March 2026

PANDOC := pandoc
SLIDY_OPTS := -t slidy -s --slide-level=2
CSS_FILE := css/custom.css

# Mermaid support: pandoc header include
HEADER_FILE := css/header.html

# Source and output
AFTERNOON_SRC := day1-afternoon-5gc.md
AFTERNOON_HTML := day1-afternoon-5gc.html

DAY2_SRC := day2-ran.md
DAY2_HTML := day2-ran.html

.PHONY: all slides afternoon day2 clean package help

all: slides

slides: afternoon day2

afternoon: $(AFTERNOON_HTML)

day2: $(DAY2_HTML)

$(AFTERNOON_HTML): $(AFTERNOON_SRC) $(CSS_FILE) $(HEADER_FILE)
	$(PANDOC) $(SLIDY_OPTS) \
		--css=$(CSS_FILE) \
		-H $(HEADER_FILE) \
		-o $@ $<
	@echo "Built: $@"

$(DAY2_HTML): $(DAY2_SRC) $(CSS_FILE) $(HEADER_FILE)
	$(PANDOC) $(SLIDY_OPTS) \
		--css=$(CSS_FILE) \
		-H $(HEADER_FILE) \
		-o $@ $<
	@echo "Built: $@"

clean:
	rm -f $(AFTERNOON_HTML) $(DAY2_HTML)
	@echo "Cleaned"

package: slides
	@mkdir -p dist
	zip -r dist/oai_workshop_2026.zip \
		README.md  css/ images/ configs/ pcap_traces/ utils/ \
		-x "*.git*"
	@echo "Package created"

help:
	@grep -E '^[a-zA-Z_-]+:.*?##' $(MAKEFILE_LIST) | \
		awk 'BEGIN {FS = ":.*?## "}; {printf "  %-15s %s\n", $$1, $$2}'
