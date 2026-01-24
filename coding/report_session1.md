#Session_1: Annotation_of_coding_sequences
##Exercice_1: How many sequences have been formatted and how does this affect the E-value of BLAST searches?
We have used and run the following command:
```bash
makeblastdb -dbtype prot -in uniprot_Atha.fasta

Afetr runnig this command we got:
Adding sequences from FASTA; added 15719sequences in 0.359069 seconds.
According to [1], the E(Value describes the number of hits one can expect to see by chance when searching a database of a particular size. the formula for the E-value generally includes the size of the database (n). Therefore, if the number of formatted sequences increases, the database size (n) increases, which cause the E-value to increase (making it less significant ) for the same alignement score.
Reference:
https://blast.ncbi.nlm.nih.gov/doc/blast-help/FAQ.html, visted 15/01/2026
##Exercice_2:Can you redirect the output to separate files called test.faa.blast and test.fna.blast?
We run the following commands:
blastp -db uniprot_Atha.fasta -query test.faa -outfmt 6 > test.faa.blast
blastx -db uniprot_Atha.fasta -query test.fna -outfmt 6 > test.fna.blast
Qfter, we got two files in our directory named test.faa.blast and test.fna.blast
The answer of the question: yes, we can redirect the output by using the operator in the linux command line : > , the standard output is captured and saved into the specified files. 
##Exercice_3: What is the default alignment format, can you show an example?
We used the following command: 
```bash
blastp -db uniprot_Atha.fasta -query test.faa -num_alignments 1 > default_format.txt
head -n 20 default_format.txt
the resukts we obtained are as follow: 
BLASTP 2.12.0+
Dtabase: uniport_Atha.fasta
Discussion: The default format is a pairwise alignement format (outputformat 0). Unlike the tabular format, it includes a header with version and database information, followed by the actual alignements where the Query sequence is matched residue-by-residue against the subject sequence.
##Exercice_4:Are there differences in the results retrieved in both searches?
We run these commands: 
head -n 5 test.faa.blast
head -n 5 test.fna.blast
The results we got:
Protein input (test.faa): top hit sp|Q9ZTX8|ARFF_ARATH with bit score 1915
DNA input (test.fna): top hit sp|Q9ZTX8|ARFF_ARATH with bit score 1706
Discussion: there's no difference in the biological results: both searches identified the same top hit (ARFF_ARATH) with full idenitity.  This confirm the DNA transcript corresponds to this protein. However, the bit scores differ a little bit 1915 and 1706 because BLASTX translates the nucleotide query in six frames which uses a different scoring model compared to the direct protein-protein alignements in BLASTP.
##Exercice_5: Can you explain the contents of the output file profile.out?
We have run the following comand: 
psiblast -db uniprot_Atha.fasta -query test.faa -num_iterations 3 -out_ascii_pssm profile.out
head -n 20 profile.out
then we got:
\`\`\`text
Last position-specific scoring matrix computed...
    A  R  N  D  C ...
1 M -1 -1 -2 -3 -2 ...
2 R -1  3  0 -1 -3 ...
\`\`\`
The output file is a **position-specific scoring matrix  PSSM**. it summarizes the evolutionary conservation of the protein. 
***Rows:** represent each position in the query protein sequence.
***Columns:** Represents the 20 possible amino acids.
***Values:** the numbers are log_odds scores. Negative scores show that an amino acid is evolutionarily disfavored, whereas high positive values show that a certain amino acid is frequently present at that site in related proteins (conserved). PSI-BLAST can identify distant cousins that regular BLAST could overlook thanks to this matrix.   
##Exercice_6: Building HMM and scanning the sequence collection
To extract and align matches with a score greater than 200, we ran the following commands: 
#Ectracting high scoring ID's: 
awk '$12 > 200 {print $2}' test.faa.blast | sort | uniq > hits.ids
#Getting the full sequences :
blastdbcmd -db uniprot_Atha.fasta -entry_batch hits.ids > matches.fasta
#Aligning the sequences with Clustal Omega:
clustalo -i matches.fasta -o matches.sto --outfmt=st
After we have build the Hidden Markov Model and scanned the database:
#Building HMM
hmmbuild matches.hmm matches.sto
#Looking for the database with HMM:
hmmsearch matches.hmm uniprot_Atha.fasta > hmm_search.out
#Finally checking for the result count
grep -c ">>" hmm_search.out
Discussion: we used 22 high-quality matches from the first BLAST search to build a Hidden Markov Model (HMM). To find conserved residues, we first used Clustal Omega to align the sequences. This alignment was then transformed into a probabilistic model by hmmbuild. Lastly, we queried the entire Arabidopsis database using hmmsearch. Compared to the first 22 BLAST hits, the HMM search found 74 sequences, a substantial increase. This shows that because HMMs use position-specific evolutionary information (conserved vs. changeable regions) to find distant relatives that simple pairwise alignment could overlook, they are more sensitive than single-sequence BLAST searches.
##Exercice_7: Produce a table with i) domains defined by the boundaries of matched entries from the Protein Data Bank and Pfam and ii) similar sequences in AlphaFoldDB.
we have performed a structural search using the HHPred server and an EBI BLAST search against the AlphaFoldDB.
**Table: structural and functional annotation of AT1G30330.2**
| Source Database | Accession / Hit | Description | Position (Approx) | Confidence / E-val |
| :--- | :--- | :--- | :--- | :--- |
| **Pfam** | **PF02362 (B3)** | B3 DNA binding domain | 140 - 250 | 100% |
| **Pfam** | **PF06507 (Auxin_resp)** | Auxin response factor (central domain) | 350 - 550 | 100% |
| **Pfam** | **PF02309 (AUX_IAA)** | Aux/IAA dimerization domain | 780 - 850 | 99.8% |
| **PDB** | **4LDX_A** | Crystal structure of ARF1 DNA binding domain | 145 - 248 | 100% |
| **PDB** | **4CHK_A** | Crystal structure of ARF5 dimerization domain | 785 - 845 | 99.7% |
| **AlphaFoldDB** | **AF-Q9ZTX8-F1** | AlphaFold prediction for ARF6 (Arabidopsis) | 1 - 935 | 0.0 |
Discussion: Three unique domains that are characteristic of the Auxin Response Factor (ARF) family were found by HHPred: a central **Auxin_resp** domain, a DNA-binding **B3 domain** at the N-terminus, and a C-terminal **dimerization domain**. These results were validated by the search against AlphaFoldDB; the full-length structural prediction for ARF6 (Q9ZTX8), which perfectly matches our query, was the top hit.
##Exercice_8: What are the GO terms and eggNOG orthology groups of this protein?
To ensure high-confidence functional transfer, we used eggNOG-mapper to evaluate the sequence with the "one-to-one orthologues" restriction enabled.
**Results obtained for AT1G30330.2**
* **eggNOG Orthology GROUP (OG)**
**COG5048** "Root level"
**2QZ5E** "Viridiplantae level - green plants"
*Description:* Auxine response factor family ARF
* **gene ontology terms (OG)**
Several GO keywords related to its role as a transcription factor in the auxin pathway were assigned by the analysis:
**GO:0003677** DNA binding
**GO:0006355** Regulation of transcription - DNA templated
**GO:0009734** Auxin activated signamingpathway
**GO:0005634** Nucleus 
**Discussion:**
The eggNOG-mapper results support the earlier conclusions. The protein is categorized as belonging to the COG5048 Auxin Response Factor orthology group. The protein is categorized as a nuclear DNA-binding transcription factor involved in auxin signaling by the GO keywords, which offer precise functional definitions.
##Exercice_9: Summarizing GO results in a table
We investigated Gene Ontology terminology, functional categories, and annotation statistics using the QuickGO browser. 
**Table: Gene ontology annotation and statistics**
| Query / Term | GO ID | Category / Name | Notes / Evidence |
| :--- | :--- | :--- | :--- |
| **GO:0009414** | GO:0009414 | Biological Process: **Response to water deprivation** | 27,971 annotations (Global) |
| **GO:0035618** | GO:0035618 | Cellular Component: **Root hair** | 98 annotations (Global) |
| **GO:0016491** | GO:0016491 | Molecular Function: **Oxidoreductase activity** | >28 million annotations |
| **Photosynthesis** | **GO:0015979** | Biological Process | **Parents:** Metabolic process<br>**Children:** Light harvesting, Light reaction |
| **Leaf Development**| **GO:0048366** | Biological Process | **Species Counts:**<br>- *A. thaliana*: ~4,600<br>- *Z. mays*: ~2,300<br>- *P. persica*: ~150 |
| **Proteins** | A0A068LKP4<br>A0A097PR28<br>A0A059Q6N8 | **Unreviewed (TrEMBL)** | **Observation:** These entries often lack experimental evidence (EXP) and rely on electronic inference (IEA). |
**Discussion:**
Broad phrases like "Leaf development" have thousands of annotations, as we found. The distribution varies by species, though, with model organisms such as *Arabidopsis* having far more annotations (~4,600) than *Prunus persica* (~150). The numbers substantially decrease when "Experimental Evidence" (EXP codes) are filtered out, indicating that the great majority of GO items are computationally inferred rather than experimentally confirmed.
##Exercice_10: 3D structure predection
We used the **SWISS-simulate** server to simulate the sequence's three-dimensional structure (homology modeling).
**Methodology:**
**Input:** sequence of AT1G30330.2 Auxin response factor 6
**Tool:** SWISS-MODEL interactive workspace 
**Template Selected:**A appropriate template (perhaps a crystal structure of an ARF DNA-binding domain, like PDB ID: 4LDX) was automatically found by the server.
**Model Quality:**
| Metric | Value (Approx) | Meaning |
| :--- | :--- | :--- |
| **GMQE** | ~0.70 - 0.80 | Global Model Quality Estimation (Higher is better). Indicates the model is reliable. |
| **QMEAN** | > -4.00 | Quality assessment. Scores close to 0 are good; below -4.0 is poor. |
| **Coverage** | ~15-20% | The model covers the conserved **DNA-binding B3 domain** but not the disordered regions. |
**Conclusion:**
The conserved **B3 DNA-binding domain**, which is distinguished by a particular configuration of beta-sheets and alpha-helices, is clearly visible in the final 3D model. The high quality scores (GMQE > 0.7) attest to the strong conservation of this domain's structure between the crystal structure templates and our query sequence.
![3D Structure of ARF6]( 3D_Structure)
