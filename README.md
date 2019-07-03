# SNP_selection_for_validation
Useful code for prepare the file you need to send for SNP validation at a company

ex:
```
scaffold29|size405917_22946	G	A	22946	>scaffold29|size405917	AAAAAACAAAACAAAACAAAACAAAACAAAAAATCCCAAAAAATCAAGGGACTCTCAACTACAGTAAAACTTTCATTTCGATTGTTCCTTTTATGCAAAACTCTTGTACTATAAGCCTGGAATACCAAGTGAGTAATAAATGGAAATAAAGAACTACCTATCCACGAAATTGGCCACCAATAATGCAGTTACTCCTTTTGTTGCTGACCAGAGAAGGCAAAGTGTGTCGTTTTTCCAATGTCTCCCTGACAATGGATCCGCAAATCCACCCCAAACATCGATAAGAAGTTCTCCTTTAT[G/A]ATAGAATGCCACACTGGTTCCACGTTCTTCTCCATTTTCTACATGTTTTCTACAAAATCAAATTTCAAGAGACGTACAATTAGCATGATCAAGGAACATAAAATGAAAGAAAGTTTGTTGCTTATTACCTTCAAATCTTTTGCCATTATGCATGTATGATTCATTCCTCGGAGACATTGTGTAGAGGTATAACTGACAAATGAGCGTGGACAAAAACAGAAAGGAGATACATATTGTAACGACATCGTCCCAGCTTGTGAAACTGGGAAGGTATTACTCGGAAAGGGGTAGGGGTGGATA
```
Zoom in the middel of the sequence:

```
TTTAT[G/A]ATAGA
```

Here there is the SNP name, the reference base, the alternative base, the position of the SNP, the orginal sequence name, the "special sequence".
The "special sequence" here is a portion of the original genomic sequence, with 300 bp before AND after the SNP (so 600 pb), and at the middle of the sequence, there is the SNP with this format: [G/A] ([ref/alt]). This format will be usefull for the primers design later.

## Create a database of your sequences 
Fasta format is boring to deal with. It's better to have one line per sequence with 2 columns: $1= name of the sequence; $2= sequence.

So the code below the examples permit to pass for that:

```
>scaffold29|size405917
CAAAAAATCAAGGGACTCTCAACTACAGTAAAACTTTCATTTCGATTGTTCCTTTTATGCAAAACTCTT
GTACTATAAGCCTGGAATACCAAGTGAGTAATAAATGGAAATAAAGAACTACCTATCCACGAAATTGGC
CACCAATAATGCAGTTACTCC....
>scaffold22|size4059
CTTTTATGCAAAACTCTTGTACTATAAGCCTGGAATACCAAGTGAGTAATAAATGGAAATAAAGAACTA
CCTATCCACGAAATTGGCCACCAATAATGCAGTTACTCCCAAAAAATCAAGGGACTCTCAACTACAGTA
AAACTTTCATTTCGATTGTTCCAAAAAATCAAGGGACTCTCAACTACAGTAAAACTTTCATTTCGATTG
TTC...
```

to 

```
>scaffold29|size405917  CAAAAAATCAAGGGACTCTCAACTACAGTAAAACTTTCATTTCGATTGTTCCTTTTATGCAAAACTCTTGTACTATAAGCCTGGAATACCAAGTGAGTAATAAATGGAAATAAAGAACTACCTATCCACGAAATTGGCCACCAATAATGCAGTTACTCC....
>scaffold22|size4059  CTTTTATGCAAAACTCTTGTACTATAAGCCTGGAATACCAAGTGAGTAATAAATGGAAATAAAGAACTACCTATCCACGAAATTGGCCACCAATAATGCAGTTACTCCCAAAAAATCAAGGGACTCTCAACTACAGTAAAACTTTCATTTCGATTGTTCCAAAAAATCAAGGGACTCTCAACTACAGTAAAACTTTCATTTCGATTGTTC...
```

The code:
```
awk 'ORS = $1 ~ /A/ ? " " : "\n"' sspace.final.scaffolds.fasta > sspace.final.scaffolds_database_awk.txt
awk 'ORS = $1 ~ /T/ ? " " : "\n"' sspace.final.scaffolds_database_awk.txt > sspace.final.scaffolds_database_awk_2.txt
rm sspace.final.scaffolds_database_awk.txt 
awk 'ORS = $1 ~ /C/ ? " " : "\n"' sspace.final.scaffolds_database_awk_2.txt > sspace.final.scaffolds_database_awk_3.txt
rm sspace.final.scaffolds_database_awk_2.txt 
awk 'ORS = $1 ~ /G/ ? " " : "\n"' sspace.final.scaffolds_database_awk_3.txt > sspace.final.scaffolds_database_awk_4.txt
rm sspace.final.scaffolds_database_awk_3.txt 
awk 'ORS = $1 ~ /N/ ? " " : "\n"' sspace.final.scaffolds_database_awk_4.txt > sspace.final.scaffolds_database_awk_5.txt
rm sspace.final.scaffolds_database_awk_4.txt 
sed 's/N /N/g' sspace.final.scaffolds_database_awk_5.txt > sspace.final.scaffolds_database_awk_6.txt
rm sspace.final.scaffolds_database_awk_5.txt 
sed 's/A /A/g' sspace.final.scaffolds_database_awk_6.txt > sspace.final.scaffolds_database_awk_7.txt
rm sspace.final.scaffolds_database_awk_6.txt 
sed 's/C /C/g' sspace.final.scaffolds_database_awk_7.txt > sspace.final.scaffolds_database_awk_8.txt
rm sspace.final.scaffolds_database_awk_7.txt 
sed 's/T /T/g' sspace.final.scaffolds_database_awk_8.txt > sspace.final.scaffolds_database_awk_9.txt
rm sspace.final.scaffolds_database_awk_8.txt 
sed 's/G /G/g' sspace.final.scaffolds_database_awk_9.txt > sspace.final.scaffolds_database_awk_10.txt
rm sspace.final.scaffolds_database_awk_9.txt 
tr n '\n' < space.final.scaffolds_database_awk_10.txt > space.final.scaffolds_database.txt
rm sspace.final.scaffolds_database_awk_10.txt 
```

## Combine the database file with you SNP data

Create a file with `paste` and/or `awk` to combine your SNP data (SNP name, ref base, alt base, position, transcrit name) with you sequence database file (transcrit (or sequence) name, sequence), in order to have a file with 6 columns in this order, with one line by SNP:

```
scaffold507|size519548_340	C	G	340	>scaffold507|size519548	AATCCTTGAGTGCACATGTTAGGAAGATTTGGGTATCAA...
scaffold745|size619512_342	A	G	342	>scaffold507|size519548	CCCTTGCCAATTTTGTGGGGTTTTTTTGTTTTTGATTTTTTTTTTGGCTGCGATTCCTCAACCCAGATGGCCGC...
...
```

## Extract necesssary info

The code below will work only for the first line of your file (e.g. your first SNP). This is for explanation, for all SNPs, see the code in the next section.

This will create files for all variables in order to extarct them and put them in a code (last line). The last line permit to extract from your whole genomic sequence 300pb before the SNP, put a `[`, the reference base, a slash (`/`), the alternative base, a `]`, and then the next 300 pb. The before last line print the name of SNP and a `  YYYYYYY`. Later, change `  YYYYYYY/n` by tabulation (`\t`) to clean your file.

```
# SNP_name
awk '{print $1}' test.txt > SNP_name_2.txt
sed '1q;d' SNP_name_2.txt > SNP_name_3.txt

# SNP_alternatif
awk '{print $3}' test.txt > SNP_alternatif_2.txt
sed '1q;d' SNP_alternatif_2.txt > SNP_alternatif_3.txt

# sequence
awk '{print $6}' test.txt > sequence_2.txt
sed '1q;d' sequence_2.txt > sequence_3.txt

# position
awk '{print $4}' test.txt > position_2.txt
sed '1q;d' position_2.txt > position_3.txt

#INPUT='someletters_12345_moreleters.ext'
#SUBSTRING=$(echo $INPUT| cut -d'_' -f 2)
#echo $SUBSTRING

SEQUENCE=$(head -n 1 sequence_3.txt)
POSITION=$(head -n 1 position_3.txt)
SNP_NAME=$(head -n 1 SNP_name_3.txt)
SNP_ALTERNIF=$(head -n 1 SNP_alternatif_3.txt)

# Print 300 pb before SNP and 300pb after SNP
# varibale : where I start : How many I print
# So, VAR : 300 - position SNP : 600
echo "$SNP_NAME YYYYYYY" >> result.txt
echo ${SEQUENCE:POSITION-300:299}[${SEQUENCE:POSITION-1:1}/$SNP_ALTERNIF]${SEQUENCE:POSITION:300} >> result.txt
```

## Extract info for all your SNP.

You will create many duplication of the code below, one per SNP line. The portion line `sed '1q;d'` of the code bellow precise "extract the first line of my file". So if you put `sed '2q;d'`, this will extract only the second line.
So the idea is to create a loop than will create N code for your N SNP by change changing the `sed 'Nq;d'` part.

```
#!/usr/bin/env bash

echo "#!/usr/bin/env bash" >> lunch_loop.sh

for i in `seq 1 128`; do 

echo "# SNP_name" >> lunch_loop.sh ;
echo "awk XXXXX{print DOLLAR1}XXXXX test.txt > SNP_name_2.txt" >> lunch_loop.sh ;
echo "sed XXXXX$i ZZZZZZq;dXXXXX SNP_name_2.txt > SNP_name_3.txt" >> lunch_loop.sh 

echo "# SNP_alternatif" >> lunch_loop.sh
echo "awk XXXXX{print DOLLAR3}XXXXX test.txt > SNP_alternatif_2.txt" >> lunch_loop.sh
echo "sed XXXXX$i ZZZZZZq;dXXXXX SNP_alternatif_2.txt > SNP_alternatif_3.txt" >> lunch_loop.sh

echo "# sequence" >> lunch_loop.sh
echo "awk XXXXX{print DOLLAR6}XXXXX test.txt > sequence_2.txt" >> lunch_loop.sh
echo "sed XXXXX$i ZZZZZZq;dXXXXX sequence_2.txt > sequence_3.txt" >> lunch_loop.sh

echo "# position" >> lunch_loop.sh
echo "awk XXXXX{print DOLLAR4}XXXXX test.txt > position_2.txt" >> lunch_loop.sh
echo "sed XXXXX$i ZZZZZZq;dXXXXX position_2.txt > position_3.txt" >> lunch_loop.sh

echo 'SEQUENCE=DOLLAR(head -n 1 sequence_3.txt)' >> lunch_loop.sh
echo 'POSITION=DOLLAR(head -n 1 position_3.txt)' >> lunch_loop.sh
echo 'SNP_NAME=DOLLAR(head -n 1 SNP_name_3.txt)' >> lunch_loop.sh
echo 'SNP_ALTERNIF=DOLLAR(head -n 1 SNP_alternatif_3.txt)' >> lunch_loop.sh

echo 'echo XXXXXDOLLARSNP_NAME YYYYYYYXXXXX >> result2.txt' >> lunch_loop.sh
echo "echo XXXXXDOLLAR{SEQUENCE:POSITION-300:299}[DOLLAR{SEQUENCE:POSITION-1:1}/DOLLARSNP_ALTERNIF]DOLLAR{SEQUENCE:POSITION:300}XXXXX >> result2.txt" >> lunch_loop.sh ; done
```

This script will create a `lunch_loop.sh` script for 128 SNPs (see `for i in ``seq 1 128``; do`  part).
After running this script, you need to clean it. Change:
- `XXXXX` by `'`
- `DOLLAR` by `$`
- ` ZZZZZZ` by nothing
 
 You need to let YYYYYYY pattern for after (see above section for explanations).

At the and, run the `lunch_loop.sh` script. Here, your input file (see section `Combine the database file with you SNP data`), need to be named `test.txt`, or change the name all over the scripts.

This will create a `result2.txt` file with your datas like in the first example of this Wiki tuto.
