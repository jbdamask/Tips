###Pull oracle path in a larger path
```$ echo ${LD_LIBRARY_PATH} | perl -pe "s/:/\n/g;" | grep ora
/usr/prog/oracle/11.2.0/oracle/product/11.2.0/client_1/lib```

###See installed packages on Unix system
```$ aptitude search '~i!~M'```

## Vertica
###This is more like a two liner. I wanted to find all columns in a vertica database that were in more than one table

$ for line in `vqb -Atc "\d" | grep trapd  | cut -d "|" -f 1,2  | sed 's/|/./' `; do vqb -Atc "\d ${line}"; done > out.txt
$ perl -pe '%cols = (); while(<>) {@t = split(/\|/, $_); $cols{$t[2]}++;} while(($k, $v) = each (%cols)){ if ($v>1) {print "$k => $v\n";}}' <  out.txt

-- Guess column datatype (number or text)
$ head -n15 msms.txt | tail -n14 |cut -f5| perl -ne 'use Scalar::Util qw(looks_like_number); print if looks_like_number($_)' | awk '{s+=1}END{print s}' | perl -ne 'if ($_ == "14") {print "number\n";} else {print "text\n";}'

-- Count columns in csv
$ awk -F'\t' '{print NF; exit}' msms.txt


— Similar to Vertica (same root: Mike Stonebreaker)
— Write table record counts to eponymously named files
— Note the ‘&’ makes this multithreaded
$ for tbl in `psql hithub -Atc "\dt go.*" | cut -d\| -f2`; do psql -c "select count(*) from go.${tbl}" hithub  > go_${tbl}_length.csv & done;

— Dump all tables under “go” schema to csv files
$ for tbl in `psql hithub -Atc "\dt go.*" | cut -d\| -f2`; do psql -c "COPY go.${tbl} TO STDOUT CSV HEADER" hithub > go_${tbl}.csv & done;

-- Print matched pattern
Parsing a log file for table names used in SQL UPDATE statements. Grep gets the line and awk naturally tokenizes on whitespace so I print the 2nd element in awk's array. Then sort unique
$ grep  'UPDATE' log.txt | awk '{print $2}' | sort -u
PDSSD.ADM_LOADED_EXP_ANALYSIS

PDSSD.EXPERIMENT_DIM
PDSSD.PURE_TO_TREAT_BRIDGE

-- print line if regular expression match on specific column
$ awk -F'\t' '$11~/NONE|NO VALUE|NOT PROVIDED|NOTHING/ {print $0}' clean.txt | wc

— Prepend text to file
$ echo -e "to be prepended\n$(cat text.txt)" > text.txt

-- Sudo cp with wildcard doesn't work! But this trick does
$ sudo bash -c "cp /admhome/adm_damasjo1/*.txt /usr/share/tomcat6/solr/mrm_assay/conf"

-- Grep a file for the line that contains my column definition (the first column is known to be "keep_in_assay", then print each element
$ grep -i keep_in_assay /usr/share/tomcat6/solr/mrm.csv |sort -u | tr , "\n" | sort

-- Count number of gz files in subdirectories, exclude the internal_analysis dir
$ find -name *.gz -not -path "./internal_analysis*" | wc -l

-- Direct stdout to grep
Let's say I have a multicolumn file and I want to cut column 2 and make the values a list I pass to grep via a pipe
In unix:
$ cut -f2 db_wrong_seq_counts.txt | grep -f - potentially_effected_samples_2014.txt
This doesn't work on Mac OSX but this does:
$ cut -f2 db_wrong_seq_counts.txt | grep -f /dev/fd/0 potentially_effected_samples_2014.txt
     "in osx, stdin can be represented by /dev/fd/0, allowing the following workflow:
     foo1 has 3 columns. I want to find any instances of patterns from the first column in foo2. So I:
     cut -f1 foo1 | grep -f /dev/fd/0 foo2
     where /dev/fd/0 represents the output of cut."

-- Be careful when using -m switch with grep
You cannot use this switch when searching multiple files. If you do, only the first file will be searched.
Example:
$ cat > f1.txt
this
$ cat > f2.txt
this

$ grep -m 1 this *.txt
f1.txt:this               <== WRONG

$ grep this *.txt
f1.txt:this
f2.txt:this               <== RIGHT

For my specific use case, I wanted to cat a file that contained directories, then go through each gripping all dat files for certain terms:
$ for line in `cat ~/tmp/mascot/2014_paths.txt`; do grep -m 1 sequences= /Volumes/d$/Inetpub/wwwroot/ISB/data/SAN/${line}/*.dat >> ~/tmp/mascot/seq_count_2014.txt;

grep -m 1 fastafile  /Volumes/d$/Inetpub/wwwroot/ISB/data/SAN/${line}/*.dat >> ~/tmp/mascot/fastafile_2014_with_db.txt;
done;

This simply grepped the first file in each directory! Gregg showed me that a nested loop is the right way (note I want the "-m 1" switch, otherwise the command takes too long)
$ for line in `cat ~/tmp/mascot/2014_paths.txt`;

    do
        for file in `ls /Volumes/d$/Inetpub/wwwroot/ISB/data/SAN/${line}/*.dat`;
            do
                grep -H -m 1 sequences= ${file} >> dat_fasta_seq_count2.txt;
                grep -H -m 1 fastafile ${file} >> dat_fasta_seq_count2.txt;
            done;
    done;

Note: If the lines in your file contain whitespace the “for” loop will break on all! To ensure it breaks on carriage returns, only, you have to specify what the character is like so:
$ IFS=$’\n’; for line in `cat ~/tmp/mascot/2014_paths.txt`; do grep -m 1 sequences= /Volumes/d$/Inetpub/wwwroot/ISB/data/SAN/${line}/*.dat >> ~/tmp/mascot/seq_count_2014.txt;

grep -m 1 fastafile  /Volumes/d$/Inetpub/wwwroot/ISB/data/SAN/${line}/*.dat >> ~/tmp/mascot/fastafile_2014_with_db.txt;
done;

Print only lines for which a particular field isn’t blank
# In this case my file is tab delimited and I want to make sure column 30 has a value
$ cat clean.txt | awk '  BEGIN {FS="\t"} $30!=""  {print} ‘

Print index and name of column in delimited file
$ cat foo.txt
PROJECT_NAME PROJECT_CODE EXPERIMENT_NAME
AWK:
$ awk 'NR == 1 { while (i++ < NF) print i-1, $i }' foo.txt0 PROJECT_NAME
PERL:
head -n1 foo.txt | perl -ne '$i=0; @w = split(" ", $_); for $i (0..$#w) { print $i . " " . $w[$i] . "\n"; }’

Both produce:
1 PROJECT_CODE
2 EXPERIMENT_NAME

Note quite a one liner...
Goal: Determine if instrument attributes differ between pep.xml files
$ cat > ~/tmp/scratch/keys.txt
msManufacturer
msModel
msIonization
msMassAnalyzer
msDetector
$ grep msms_run_summary XAVtacs*.pep.xml > ~/tmp/scratch/proteomics/mixed_instrument.txt
$ cat ~/tmp/scratch/proteomics/mixed_instrument.txt  |perl -ne '@a = $_ =~ /.+?(?="\s)/g; print "$_\n" for @a;' | grep -f ~/tmp/scratch/keys.txt | sort |uniq

" msDetector="inductive detector
" msIonization="nanoelectrospray
" msManufacturer="Thermo Scientific
" msMassAnalyzer="quadrupole
" msModel="Q Exactive
Note the perl syntax for matching everything up to the first double quote followed by a space. Explanation of regex’s un-greedy match

Scrape HTML page for all csv file links and automatically download
I used this to download all supporting resource files from MIT edX MOOC, Analytics Edge (csv, R and pdf)
Note there are lots of files so each curl gets its own thread. “set -m” enables Job Control, which means I can spawn threads.
$ set -m; for path in `curl -s "https://courses.edx.org/courses/MITx/15.071x_2/1T2015/a36e4c3815534ee5965d96974a0ec06a/" | grep '\.csv\|\.pdf\|\.R' | grep -o 'asset[^"]*'`; do `curl -s "https://courses.edx.org/c4x/MITx/15.071x_2/${path}" > ${path} &`; done;  while [ 1 ]; do fg 2> /dev/null; [ $? == 1 ] && break; done;

Bash Line Breaks
When looping through a file from bash, the default behavior is to tokenize by whitespace. To set the environment to tokenize by line breaks (for, say, iterating through a tab delimited file) do:
$ IFS=$(echo -en "\n\0”)

Join two files
Coming from the database world, Unix join can be a mystery. This is because it assumes your files are pre-ordered. The proper way to join two column-separated files by their second column is:
$ join -t"," -1 2 -2 2 <(sort -t"," -k2 results_name.csv) <(sort -t"," -k2 results_cleanname.csv)

Auth with cURL
If I need an auth token I open the URL I want in Chrome then use the "cookie.txt export" Chrome plugin to print all cookie info. I can save this as cookies.txt in the directly I’m cURLing from and call it like so:
$  curl -b cookies.txt http://whatever

Note that I only need the ObSSO part so I can do this:
$ grep 'ObSSO' cookies.txt > ObSSO.txt
$  curl -b ObSSO.txt http://whatever

Add column names to first line of delimited file:
$ echo -e "SPECTRUM,PEPTIDE,ION_SCORE,IDENTITY_SCORE,HOMOLOGY_SCORE,EXPECT,STAR,PEPTIDE_PROBABILITY\n$(cat $out)" > $out

Oracle
Ok, this is a couple-liner. Still valuable when I want to do stuff in Oracle from the command line


$ cat > get_atppicl_cell_lines.sql
SET PAGESIZE 50000
select distinct CELL_LINE_TISSUE from tppxml.met_reagent;




$ sqlplus -s <schema>/<pass>@<host> @get_atppicl_cell_lines.sql > output.txt



Find and zip files of type pep.xml:
find . -name “*.pep.xml" -exec gzip {} \;

Perl
Regex replace and assign to new variable
Need to use parens
$inFile = CHEMG_20151013_G1240_NAPTH_PAL_QE2.sql
($outFile = $inFile) =~ s/.sql/_FILTERED.sql/;
print $outFile;
>> CHEMG_20151013_G1240_NAPTH_PAL_QE2_FILTERED.sql

Find space taken up by particular file type
$ for d in `ld`; do echo ${d}; du -ch ${d}/*.mzXML | tail -n1; done;

Remove all files of type .RAW
find . -type f -name '*.RAW' -delete

R
Pipe stdout to R using Rscript and scan()
Helpful links: scan, matrix
$ cat file_that_has_list_of_numbers.txt | Rscript -e 'a<-scan(file="stdin")' -e 'summary(a)'

Pipe stdout to R as matrix, generate scatterplot from data
$ tail -n+2 data.csv | cut -d, -f4,7 | Rscript -e 'a<-matrix(scan(file="stdin", sep=","), ncol=2, byrow=TRUE)' -e 'plot(a, main="Peptide counts", xlab="iProphet", ylab="ProteinProphet")'

Here’s how GROUP BY in R
> aggregate(Sepal.Length ~ Species, summary, data=iris)

Apply heat density to scatterplot
$ tail -n+2 all_scores.csv | cut -d, -f6,10 | Rscript -e 'a<-matrix(scan(file="stdin", sep=","), ncol=2, byrow=TRUE)' -e 'LSD::heatscatter(a[,1],a[,2], main="ProteinProphet Scores by Peptide", xlab="ProteinProphet Probability", ylab="Median Peptide Score")'

sqlite3
NOTE: This idiom is sometimes useful, however sqlite3 doesn’t have very good math capabilities out of the box (it doesn’t even have a median function). It would probably be more efficient to do this with R dplyr.
Load tab-delimited file (note this works on my mac [sqlite3 3.8.5] but not on older unix redhead [sqlite 3.3.6]

$  echo -e ".mode tabs\n.import report_err_rate_0_01.tsv report_err_rate_0_01
\n" | sqlite3 test.db

Cast text columns to read and divide

$ sqlite3 test.db "select MACHINE_TYPE, (CAST(a.INTERSECTION_ as real) / CAST(a.UNION_ as real)) as jaccard from report_err_rate_0_01 a group by MACHINE_TYPE"

Print formatted columns to stdout

$ echo -e '.mode column\n.header on\nselect * from G1283_LYR_1uM_CCDR_REsearch where star = 1 and peptide_probability > 0.9;' | sqlite3 G1283.db | more

xmllint
On my work Mac (v 20900)
This is a database connection file exported from Oracle SQL Developer. I want to print the connection name (parent) when the service name (child) matches my search. Specifically, the connection name is an attribute in the parent element.

$ xmllint --xpath "//Reference[./RefAddresses/StringRefAddr/Contents = 'CADEXP']/@name" connections.xml

xmlstarlet
Good page with examples
Select element whose attribute matches my desired text (this uses pep.xml file)

$ xml sel -t -m "//_:spectrum_query[@spectrum='G1283_LYR_1uM_CCDR_01_RE.05024.05024.5']" -c . -n  G1283_LYR_1uM_CCDR_01_RE.pep.xml

Print the unique elements in a file

$ xml el -u G1283_LYR_1uM_CCDR_01_RE.pep.xml

XML to CSV
Note that XPath doesn’t always return what I expect. This can be due to a namespace in the file described here)
See this note for the file used in this example. Note that I can run this over the entire XML file by removing the highlighted piece. The program is super fast and I can parse a 250MB XML file in seconds.

$ xml sel -N x="http://regis-web.systemsbiology.net/pepXML" -T -t -m "//x:spectrum_query" -v "concat(@spectrum,',',x:search_result/x:search_hit/@peptide,',',x:search_result/x:search_hit/x:search_score[@name='ionscore']/@value,',',x:search_result/x:search_hit/x:search_score[@name='identityscore']/@value,',',x:search_result/x:search_hit/x:search_score[@name='homologyscore']/@value,',',x:search_result/x:search_hit/x:search_score[@name='expect']/@value,',',x:search_result/x:search_hit/x:search_score[@name='star']/@value,',',x:search_result/x:search_hit/x:analysis_result/x:peptideprophet_result/@probability)" -n interact-G1040_LYR_ChyTp_RE.pep.xml
G1040_LYR_8402_ChyTp_qE_09_RE.28602.28602.2,AIGGF,18.96,28.65,23.89,0.47,1,0.8430

Compute median of Mascot identity scores
$ xml sel -N x="http://regis-web.systemsbiology.net/pepXML" -T -t -m "//x:search_score[@name='identityscore']" -v @value -c . -n  interact-G1283_LYR_1uM_CCDR_REsearch.pep.xml | Rscript -e 'a<-scan(file="stdin")' -e 'median(a)'
Read 169032 items
[1] 34.84

Text window function:
Save as bash script
$ more ~/bin/window.sh
for e in `grep -n $1 $2 | cut -f1 -d:`; do lower=$((e - $3)); upper=$((e + $3)); awk -v low="$lower" -v high="$upper" 'NR>low&&NR<high' $2; echo ""; done;

$ window.sh QATAPL G1283_LYR_1uM_CCDR_01_RE.pep.xml 2
    <search_result>
      <search_hit hit_rank="1" peptide="QATAPL" peptide_prev_aa="L" peptide_next_aa="L" protein="Q07687" num_tot_proteins="1" num_matched_ions="3" tot_num_ions="10" calc_neutral_pep_mass="599.3279" massdiff="-0.0024" num_tol_term="2" num_missed_cleavages="0" is_rejected="0">
        <search_score name="ionscore" value="3.48"/>

    <search_result>
      <search_hit hit_rank="1" peptide="QATAPL" peptide_prev_aa="L" peptide_next_aa="L" protein="Q07687" num_tot_proteins="1" num_matched_ions="3" tot_num_ions="10" calc_neutral_pep_mass="599.3279" massdiff="-0.0003" num_tol_term="2" num_missed_cleavages="0" is_rejected="0">
        <search_score name="ionscore" value="4.06"/>
