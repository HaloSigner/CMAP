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

## 0. Process Overview (Pipeline)

> 전체 분석 흐름 (Process) 를 한 번에 정리하면 다음과 같습니다.

1. **Data download**
   - LINCS / CMap 2020 L1000 GCTX 파일과 메타데이터(`siginfo_beta.txt`, `geneinfo_beta.txt`)를 다운로드한다.

2. **Signature QC filtering**
   - `siginfo_beta.txt`에서 `pert_type == "trt_cp"` (compound treatment)만 남기고,  
     `qc_pass`, `median_recall_rank_spearman`, `cc_q75`, `pct_self_rank_q25`, `is_exemplar_sig` 등의 기준으로 **양질의 시그니처**만 필터링한다.

3. **Gene mapping**
   - 관심 유전자 심볼(예: `FXYD3`)을 `geneinfo_beta.txt`에서 찾아 **L1000 `gene_id` (rid)** 로 매핑한다.

4. **GCTX structure check**
   - `.gctx` 파일에서 사용 가능한 `cid`(signature ID)와 `rid`(row ID)를 확인하고,  
     QC를 통과한 `cid`와 관심 유전자 `rid`의 교집합만 추려서 사용할 준비를 한다.

5. **GCTX subset parsing**
   - `cmapPy.pandasGEXpress.parse`를 이용해 `.gctx` 파일에서  
     **(관심 시그니처 × 관심 유전자)** 부분만 서브셋으로 로딩한다.

6. **Attach metadata**
   - 로딩된 expression matrix에 대해 `siginfo_beta.txt`에서  
     `pert_dose`, `pert_time`, `cell_mfc_name`, `cmap_name`, `is_exemplar_sig` 등을 붙여  
     하나의 통합 `DataFrame`(`full_df`)으로 만든다.

7. **Cell line / drug subset**
   - 특정 셀 라인(예: `HEPG2`) 혹은 여러 셀 라인(`HEPG2`, `HUH7`, `PHH`)을 선택해 서브셋을 만들고,  
     필요 시 엑셀(`.xlsx`)로 저장한다.

8. **Visualization**
   - 유전자 하나(예: FXYD3의 `rid="5349"`)를 기준으로  
     - 여러 셀 라인 간 expression 비교  
     - 단일 셀 라인 내에서 여러 약물의 용량–시간 반응 패턴  
     - box + swarm plot 시각화  
     를 수행한다.

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
