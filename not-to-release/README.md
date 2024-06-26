# Experiments with multiword expressions

The Greek Parseme corpus is available in the CoNLL-U Plus format (\*.cupt) from
[http://hdl.handle.net/11372/LRT-5124](http://hdl.handle.net/11372/LRT-5124). The CoNLL-U Plus
format uses eleven columns, unlike standard CoNLL-U, which uses ten columns. If we want to use
[Udapi](https://udapi.github.io/) to process the corpus, we must get rid of the eleventh column
and pack the multiword expression annotations as the Mwe= attribute in the tenth column (MISC).
The following command will do it (on a system that has Bash, Perl, Python, and Udapi installed):

```bash
cat train.cupt | perl -pe 'if(m/\t/) { chomp; @f=split(/\t/); unless($f[10] eq "*") { @misc=(); unless($f[9] eq "_") { @misc=split(/\|/, @f[9]) } push(@misc, "Mwe=".$f[10]); $f[9]=join("|", @misc) } pop(@f); $_=join("\t", @f)."\n" }' > train.conllu

cat ../../parseme/el/test.conllu | perl -CDS -e '$sid = 1; while(<>) { chomp; if(m/^\#\s*sent_id/) { $_="\# sent_id = $sid"; $sid++ } $_ .= "\n"; print }' | less

cp mwe_examples.conllu backup.conllu ; cat backup.conllu | udapy -s mwe.Normalize > mwe_examples.conllu ; rm backup.conllu
```


# Forcing parser to predict multiword expressions

A regular trainable UD parser can be taught to predict multiword expressions. The trick lies in
projecting the MWE annotation to dependency relation subtypes because prediction of these is
part of the parsing model. The following Udapi command will copy the MWE annotation from MISC
to the DEPREL column:

```bash
cat mwe_train.conllu | udapy -s mwe.ToSubDeprels > mwe_train_subdeprels.conllu
```
