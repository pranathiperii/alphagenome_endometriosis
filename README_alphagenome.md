# What do endometriosis GWAS risk variants do? Sequence-based prediction with AlphaGenome

Using a sequence-to-function model to propose mechanisms for replicated endometriosis risk variants.

## Question

GWAS has located endometriosis risk loci, but most lead variants are non-coding and their effects are unknown. For each variant, can a sequence-based model predict a specific, testable molecular effect?

## Variants

- The eight replicated loci from Rahmioglu et al. 2014 (*Human Reproduction*): WNT4, GREB1, VEZT, FN1, 7p15.2, CDKN2B-AS1, ID4 and 2p14.
- **rs3820282** at WNT4, a fine-mapping candidate whose T allele creates an oestrogen-receptor (ERα) binding site (Pavličev et al. 2024, *Nature Communications*).
- Positions and alleles come from Ensembl (GRCh38) in code. Risk alleles come from the 2014 meta-analysis (Table II); where the risk allele is the reference, the predicted direction is flipped so that every effect is stated for the risk allele.

## Method

1. **AlphaGenome** (`alphagenome==0.7.0`) compares the reference and alternative sequence in a **1 Mb window**, using all recommended scorers: expression (GeneMaskLFC, GeneMaskActive), splicing (SpliceJunction and splice-site scorers) and local signals (CenterMask: accessibility, histone marks, TF binding, CAGE).
2. **Ranking by extreme-track counts.** Each variant is scored on 4,800 to 13,000 expression tracks, so its single highest score is close to 1 by chance. Variants are instead ranked by how many expression tracks pass |quantile| > 0.999, compared with the roughly 1 in 1,000 expected by chance.
3. **Targeted checks:** reproductive-tissue tracks for each locus's own gene; whether GREB1's strongest splicing changes sit at the variant; ESR1 binding at rs3820282; whether VEZT falls inside smaller windows.

Code: [`alphagenome_endometriosis.ipynb`](alphagenome_endometriosis.ipynb).

## Results

**Only the fine-mapped WNT4 variant stands out in expression tracks.**

| Variant | Locus | Expression tracks beyond \|0.999\| | Expected by chance |
| --- | --- | --- | --- |
| rs3820282 | WNT4 (fine-mapped) | **132** | 13.4 |
| rs1250248 | FN1 | 10 | 7.0 |
| rs7521902 | WNT4 (lead) | 2 | 13.4 |
| rs12700667 | 7p15.2 | 1 | 7.4 |
| rs10859871, rs1537377, rs13394619, rs4141819, rs7739264 | VEZT, CDKN2B-AS1, GREB1, 2p14, ID4 | 0 | 4.8 to 11.1 |

**Locus by locus**

- **WNT4, rs3820282 (T).** About ten times more extreme expression tracks than chance, far more than the lead SNP at the same locus. The model predicts increased ESR1 binding at the variant (MCF-7, quantile 0.82), consistent with the knock-in result. Predicted WNT4 expression falls in myometrial and uterine smooth-muscle cells and rises in ovary; the model has no hormone-stimulated endometrial-stroma track, where the knock-in showed WNT4 rising after the oestrogen peak.
- **GREB1, rs13394619.** The variant is the last base of a GREB1 intron (`c.1160-1G>A`, splice acceptor). **All ten strongest splicing predictions are changes in junctions ending exactly at the variant** (from 11,580,832 and 11,585,905 to 11,587,381), including in ovary and uterus, while GREB1 expression barely changes. The endometriosis **risk allele is G, the intact acceptor**; the non-risk A is predicted to disrupt it.
- **VEZT, rs10859871.** The variant lies about 17 kb from VEZT, outside a 16 kb window. In the 1 Mb window, the risk allele C is predicted to lower VEZT slightly (larger in myometrial cells), with no extreme tracks; the strongest nearby change is in the microRNA MIR3685.
- **FN1, rs1250248.** A small but consistent predicted change in FN1 across reproductive tissues; the risk allele A predicts slightly higher FN1. Its extreme-track count is close to chance.

## How this revised my first screen

A first pass used a 16 kb window and ranked loci by each variant's single highest score, which put GREB1 and VEZT on top. Two problems changed that picture: the top score across thousands of tracks is close to 1 for almost any variant, and VEZT sat outside the 16 kb window, so its signal came from local regulation and a neighbouring microRNA. Ranking by extreme-track counts at 1 Mb, and adding the fine-mapped WNT4 variant, gives the results above. A comparison of lead SNPs with nearby control SNPs (8 vs 8) did not separate them (p = 0.14), as expected for so few independent variants.

## Limitations

- These are predictions, not measurements, and the predicted effects are small (raw log fold changes mostly below 0.01), as expected for common variants.
- The model's tracks do not include hormone-stimulated endometrial stroma or menstrual-cycle states.
- Extreme-track counts cover expression only; GREB1's signal is in splicing, which this count does not capture.
- Other splice predictors do not flag rs13394619; the splicing hypothesis needs SpliceAI and splicing-QTL (sQTL) data from uterine tissue.
- Risk alleles for the four additional loci were not assigned, so only their rankings are reported.

## Next steps

1. Score the fine-mapped credible sets from Rahmioglu et al. 2023 (*Nature Genetics*) at 1 Mb.
2. Colocalise with endometrial eQTL and sQTL data.
3. Test rs3820282 in oestrogen-treated endometrial stromal cells carrying each allele, and GREB1 splicing with a minigene assay.
