## Submitting annotated genomes to NCBI

This repo contains several miscellaneous Python scripts for preparing GenBank files of annotated genomes for submission to NCBI.

### Preparing your annotation
These instructions assume you have a GenBank (`.gb`, `.gbk`, or `.gbf`) file containing your genome sequence and annotated features, such as might be generated by [Prokka](https://github.com/tseemann/prokka/releases).

Next, go [here](https://submit.ncbi.nlm.nih.gov/genbank/template/submission/) to generate a submission template `.sbt` file (REQUIRED). Enter your BioProject and BioSample accession numbers - you should generate these in advance [here](https://submit.ncbi.nlm.nih.gov).

Before submission, you must first convert the curated GenBank file into `.tbl` format with this Perl script: ftp://ftp.ncbi.nlm.nih.gov//toolbox/ncbi_tools/converters/scripts/gbf2tbl.pl

Then, run [tbl2asn](http://www.ncbi.nlm.nih.gov/genbank/tbl2asn2/) to convert the `.tbl` file to `.sqn` for submission

    gbf2tbl.pl genome.gbk && tbl2asn -a s -V v -c b -Z discrep -i genome.fsa -f genome.tbl -t genome.sbt -X C

**Explanation of command line arguments**

*   `-a s` is required for any genome with >1 contig.

*   `-V v` runs NCBI's output verification to check for errors

*   `-c b` adds a comment to the output `.sqn` file for adjacent genes with the same product description. In the `discrep` report, these adjacent genes will still be listed as warnings, but the `discrep` report will also say e.g. 

~~~
DiscRep_ALL:OVERLAPPING_CDS::60 coding regions overlap another coding region with a similar or identical name.
DiscRep_SUB:OVERLAPPING_CDS::60 coding regions overlap another coding region with a similar or identical name but have the appropriate note text
~~~

*   `-Z discrep` runs the [discrepancy report](http://www.ncbi.nlm.nih.gov/genbank/asndisc) (recommended)

*   `-i` gives the file path to the genome sequence in FASTA format. **Note that tbl2asn WILL NOT WORK unless the genome has a `.fsa` extension,** although it will fool you into thinking it did work. Do not use `.fa`, `.fna`, or `.fasta` extensions unless you also pass the `-x` argument.

*   `-f` specifies the `.tbl` feature table (i.e., the annotation) generated by the `gbf2tbl.pl` script.

*   `-t` gives the file path to the [submission template](https://submit.ncbi.nlm.nih.gov/genbank/template/submission/) (REQUIRED)

*   `-X C` tells `tbl2asn` to include genome assembly structured comments from a `.cmt` file. If you don't include this, NCBI can still accept your genome, but will ask you for these details after the fact. An example `.cmt` file is included in this repo and can be generated [here](https://submit.ncbi.nlm.nih.gov/structcomment/genomes/). **NOTE:** The `.cmt` file MUST have the same prefix as your original GenBank file. The `-X C` argument does not specify a path to the `.cmt` file; rather, it looks for it in the current directory and assumes it has the same prefix as all other files.

When `tbl2asn` completes, check ALL of the following files for errors and correct them:

    errorsummary.val # total statistics; look in the next file for detailed explanations
    genome.val # where genome is the name of your curated GenBank file
    discrep

I like to keep the GenBank file open in a plain text editor (I use [TextWrangler](http://www.barebones.com/products/textwrangler/)) as well as in [Artemis](http://www.sanger.ac.uk/science/tools/artemis) for viewing. **Whenever you make changes to the GenBank file, re-open it in Artemis to make sure it can still be parsed properly.**

## One last thing

If you prefer not to deal with the frustration of NCBI sending you a rapid-response "your genome cannot be accepted because {insert esoteric excuse here}", upload your shiny, ready-to-submit genome in `.sqn` format to NCBI's online [Microbial Genome Submission Check Tool](http://www.ncbi.nlm.nih.gov/genomes/frameshifts/frameshifts.cgi). Be patient; it can take several hours depending on the job queue. Inspect the report for errors and be prepared to justify any errors you cannot or did not fix. Otherwise, go back to the GenBank file and/or Artemis to manually inspect any flagged frameshifts, long overlaps, or RNA overlaps. If you make changes, simply re-run the command above and check the error reports to see if the problems were fixed.

## Submit your genome

After all that work, the actual submission is rather anti-climactic. Go to [GenomesMacroSend](http://www.ncbi.nlm.nih.gov/projects/GenomeSubmit/genome_submit.cgi) and upload your `.sqn` file. Enter any comments, such as justifications for not fixing any known errors. You should also include a note saying when you want your genome to be publicly released (either immediately upon acceptance, or on a specific date†). Adding detailed comments will improve the chances, but not necessarily guarantee, that NCBI will not just kick the submission right back to you under the assumption that you are like most NCBI submitters and have probably screwed up something you didn't even know could be screwed up. In fact, the reason I wrote this README is because NCBI's instructions, if they exist, are either 1) far from clear 2) scattered across a dozen pages that may not have been updated since 2001 or 3) both. Don't believe me? Ignore what I told you about the `-f` flag for `tbl2asn` and try to find out what it does. Hint: `tbl2asn --help` doesn't really clear things up, and the online [documentation](http://www.ncbi.nlm.nih.gov/genbank/tbl2asn2/) doesn't even list a `-f` option.

†The genome must be publicly accessible *before* submitting to a journal, and it can take several business days after you request public release for your genome to appear in NCBI searches.
