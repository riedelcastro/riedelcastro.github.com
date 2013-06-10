---
layout: site-tweets
title: Universal Schema
---

Relation Extraction with Matrix Factorization and Universal Schemas NAACL 2013
==============================================================================

This file describes how to run our relation extraction model in the NAACL 2013 paper:

    @inproceedings{riedel13relation,
        author={Sebastian Riedel and Limin Yao and Benjamin M. Marlin and Andrew McCallum},
        title={Relation Extraction with Matrix Factorization and Universal Schemas},
        booktitle={Joint Human Language Technology Conference/Annual Meeting of the North American Chapter of the Association for Computational Linguistics (HLT-NAACL '13)},
        year={2013},
    }

We will also show how to evaluate our output, as well as any new output you may create using your own
models. The description should allow to reproduce the results from our NAACL 2013 paper,
up to a small delta resulting from minor changes we made since then. The original NAACL 2013 output files
are also contained in this package.

Note that all the following commands assume you are in the top level project directory. We also
assume that you have unpacked the NAACL archive into a `naacl2013` subdirectory.

Compilation
-----------
First you need to build the software. We use `sbt` as build platform:

    sbt compile

This compiles the code. Then run

    sbt start-script

to create a start-script you can use to run the built programs.

Training and Testing the Models
-------------------------------
Currently our code performs training and prediction on the test set in the same program. If you call

    target/start edu.umass.cs.iesl.spdb.PcaDSRun naacl13-model-N.conf

the software will train model N on the NAACL training set, apply it to the test set, and write out the results
for the subsample of test tuples we evaluate on in the NAACL paper.

Notice that `naacl13-model-N.conf` is configuration file in `src/main/resources` and specifies both the
parameters of the model, as well as the training/test file locations etc.

To run the other models in the paper (F, NF, NFE) simply replace the corresponding model name in the conf file name.

The results of each run will be stored in a subdirectory of `out` corresponding to the date of the run. The
directory of the last run will also be soft-linked by `out/latest`. This directory will contain log files,
the configuration file used to create the run, and, most importantly, the output files generated for
the test set. The output file relevant for evaluation is `nyt_pair.sub.rank.txt`.

Evaluation
----------

The evaluation program takes as input a configuration file, a gold annotation file, and the system outputs
(in the format of `nyt_pair.sub.rank.txt` mentioned above):

    target/start edu.umass.cs.iesl.spdb.EvaluationTool eval-ds.conf OUTFILE1:NAME1 OUTFILE2:NAME2 ...

For example, to evaluate the NAACL predictions and generate the results from the paper, you can call

    target/start edu.umass.cs.iesl.spdb.EvaluationTool eval-naacl13-structured.conf \
    naacl2013/structured/test-mintz09.txt:MI09 \
    naacl2013/structured/test-yao11.txt:YA11 \
    naacl2013/structured/test-surdeanu12.txt:SU12 \
    naacl2013/structured/test-riedel13-model-N.txt:N \
    naacl2013/structured/test-riedel13-model-F.txt:F \
    naacl2013/structured/test-riedel13-model-NF.txt:NF \
    naacl2013/structured/test-riedel13-model-NFE.txt:NFE

If you want to compare these results to your own predictions, simply append your results to the list above.
Notice that all the output files listed above use the sub-sampled tuples in
`naacl2013/nyt-freebase.test.subsample-10000.tuples.txt`.

The evaluation script generates as output tables corresponding to tables in the paper.
In fact, it also generates the latex table, and calculates pairwise sign test significance values.

The evaluation program also generates a series of precision recall curve in `eval`. This includes
individual curves for relations, and aggregate curves in `11pointPrecRecall.gpl`. The curves
 are in GnuPlot format, and can be turned into pdf graphs by using the `gnuplot` command.


The archive also contains the output of the evaluation tool for various runs (the corresponding files
 end with `.out.txt`).

Note that all our models are optimized to rank tuples *within the same relation*. They will not be able
to rank facts between relations. That is, true facts of relation A may have substantially lower scores than
false facts of relation B, even though facts within relation B are correctly ordered. This means that any
global ranking will perform poorly and should not be compared against.


Annotation
----------

The evaluation is based on pooling results from the systems we compared. Your own models and algorithms may
produce correct tuples not yet predicted by any of the previous models. To add these predictions to the pool,
you can use the AnnotationTool:

    target/start edu.umass.cs.iesl.spdb.AnnotationTool OUTFILE ANNOTATIONDIR MENTIONFILE RELPATTERN OLDANNOTATIONS

Here OUTFILE is the system output, ANNOTATIONDIR the directory into which the tool will write your new annotations
into, MENTIONFILE the file that contains the entity tuple mentions that will help you to annotate the tuples,
RELPATTERN a regular expression that determines which relations should be included in the predictions to be annotated,
 and OLDANNOTATIONS the file with previous annotations. The annotation tool will go through your ranked file
 as query the user for Yes/No decisions for all predictions that aren't yet annotated in OLDANNOTATIONS. An example
call could be:

    target/start edu.umass.cs.iesl.spdb.AnnotationTool \
    naacl2013/structured/test-riedel13-model-F.txt  \
    annotations  \
    naacl2013/nyt-freebase.test.mentions.txt \
    company$ \
    naacl2013/naacl2013.gold.tsv

This annotates all facts in `naacl2013/structured/test-riedel13-model-F.txt` with relations ending with `company`.

You will notice that the annotation tool provides the current number of annotations after each question. Once
this number goes over 100 (the pool depth), you can stop annotating, as only the top 100 predictions are considered
in the pool.

Notice that when annotating we generally check whether, based on the mentions, we can with probability assume
the fact is true---not whether the fact is actually true. While this makes the annotation more subjective,
it's the only way in cases where information on the true fact is not available (as is the case
for many not-so-salient entities).




