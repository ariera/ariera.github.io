---
layout: post
title:  "PubMed Central Dataset"
date:   2019-08-08 15:02:09 +0200
---

# Downloading
Europe PMC provides bulk download mechanisims for the raw datasets: http://europepmc.org/downloads
For our purposes we are interested in the _supplementary files_, available at: http://europepmc.org/ftp/suppl/

However it will be much easier for you to download them via ftp

```
ftp  ftp.ebi.ac.uk
usr: anonymous
cd /pub/databases/pmc/suppl/OA/
mget **/*.zip
```

Alternatively you may prefer to use `wget`

```
wget -c -r ftp://anonymous@ftp.ebi.ac.uk/pub/databases/pmc/suppl/OA/
```

Mind you this operation can take days to complete and you will need over a terabyte of space to store the more than 2.5 million zip files


