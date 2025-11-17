# L1000 / LINCS 2020 – Single-Gene Drug Response Extraction & Visualization

This repository documents a small pipeline to:

1. Download LINCS 2020 L1000 signatures  
2. Perform basic **signature quality control (QC)**  
3. Map a **gene symbol → L1000 gene ID**  
4. Subset the large `.gctx` matrix (by signatures and by gene)  
5. Attach metadata (cell line, dose, time, compound name)  
6. Visualize **gene expression changes** across cell lines and drug treatments  

The example uses the gene **FXYD3** and LINCS 2020 **`trt_cp` (compound treatment)** signatures.

---

## 1. Data Download (LINCS 2020 / CMap)

We use the compressed L1000 connectivity matrix from the LINCS 2020 release:

- Source: CMap / LINCS 2020  
- Reference: <https://clue.io/data/CMap2020#LINCS2020>

Example download (takes ~10 min):

```bash
# From PowerShell (Windows)
cd C:\Users\USER\Desktop\L1000

aws s3 cp `
  s3://lincs-dcic/LINCS-sigs-2021/gctx/cd-coefficient/cp_coeff_mat.gctx `
  . --no-sign-request
