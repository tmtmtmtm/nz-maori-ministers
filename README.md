Note: This repo is largely a snapshop record of bring Wikidata
information in line with Wikipedia, rather than code specifically
deisgned to be reused.

The code and queries etc here are unlikely to be updated as my process
evolves. Later repos will likely have progressively different approaches
and more elaborate tooling, as my habit is to try to improve at least
one part of the process each time around.

---------

Step 1: Check the Position Item
===============================

The Wikidata item for the 
[Minister for Māori Development](https://www.wikidata.org/wiki/Q991876)
looks mostly good, except it was still using the old name of Minister of
Māori Affairs, and wasn't connected (in either direction) to the
[Ministry of Māori Development](https://www.wikidata.org/wiki/Q1768409),
so I fixed those.  (I did the former solely as a label change, not as
strutured data — I'll leave it for someone else to do that!)

Step 2: Tracking page
=====================

Initial PositionHolderHistory list set up at https://www.wikidata.org/w/index.php?title=Talk:Q991876&oldid=1232904681

Current status: knows of 6 historic officeholders, with 9 warnings.

Step 3: Set up the metadata
===========================

The first step now in a new repo is always to edit [add_P39.js script](add_P39.js)
to set up the Item ID and source URL.

Step 4: Scrape
==============

Comparison/source = [Minister for Māori Development](https://en.wikipedia.org/wiki/Minister_for_M%C4%81ori_Development)

      bundle exec ruby scraper.rb https://en.wikipedia.org/wiki/Minister_for_M%C4%81ori_Development | tee wikipedia.csv

Scraped cleanly on first pass.

(NB I couldn't find a way of handling the encoding issues around turning
the URL containing Māori into the one with M%C4%81ori. That looks like a
bit of a rabbit hole, so I'm just provided the URL explicitly for now,
and will return to this question later.

Step 5: Get local copy of Wikidata information
==============================================

We now get the argument to this from the JSON, so call it as:

    wd ee --dry add_P39.js | jq -r '.claims.P39.value' |
      xargs wd sparql office-holders.js | tee wikidata.json


Step 6: Create missing P39s
===========================

    bundle exec ruby new-P39s.rb wikipedia.csv wikidata.json |
      wd ee --batch --summary "Add missing P39s, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

43 new additions as officeholders -> https://tools.wmflabs.org/editgroups/b/wikibase-cli/02787aa611063/

Step 7: Add missing qualifiers
==============================

    bundle exec ruby new-qualifiers.rb wikipedia.csv wikidata.json |
      wd aq --batch --summary "Add missing qualifiers, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

8 additions -> https://tools.wmflabs.org/editgroups/b/wikibase-cli/18374feaf533b/

I updated the `new-qualifiers` script to produce more human-readable
output, so it's now easier to see what modifications it's also
suggesting. Here, it's giving 4:

1. Winston Peters ordinal: 35 -> 36
2. Winston Peters start: 1990 -> 1990-11-02
3. Winston Peters end: 1991 → 1991-10-02
4. Nanaia Mahuta start: 2017-10-25 → 2017-10-26

Those all look fine, so I'll accept them en masse using

    bundle exec ruby new-qualifiers.rb wikipedia.csv wikidata.json 2>&1 >/dev/null |
      egrep "\t" | wd uq --batch --summary \
        "Update qualifiers, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

-> https://tools.wmflabs.org/editgroups/b/wikibase-cli/7f12d6ba02404/

Step 8: Refresh the Tracking Page
=================================


