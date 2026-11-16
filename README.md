# CS 5787 Final Project

**Abishek Chozhan (ac3388), Nishant Ray (nr487), Ethan Nguyen (eln39)**

## File Descriptions

| Notebook | Description |
|---|---|
| `concept_test.ipynb` | First exploratory analysis. Reads the annotation files from the BoneMarrowWSI-PediatricLeukemia dataset to check whether there is enough data to use, and selects the chronic-phase chronic myeloid leukemia (CP-CML) patients, the subset with the most cell annotations. CP-CML patients also have every stage of Granulocytic precursors available in the dataset, providing a good basis for classification and diffusion pseudotime. Proof of concept. |
| `download_pipeline.ipynb` | Downloads the whole-slide images (several GB per file) and cuts each annotated cell into a 320 px crop. |
| `dpt_dinobloom_embeddings.ipynb` | Checks all crops against the whole-slide image annotations, computes DinoBloom embeddings, and computes diffusion pseudotime (DPT) for the 46 CP-CML patients. Compares crop types to find which best represents the cell (ultimately, 5% letterbox crop was chosen). |
| `stain_normalization.ipynb` | Tests two stain normalizations to see whether DPT was picking up staining as a main component. |
| `classification_benchmark.ipynb` | Evaluates cell-type classification with frozen DinoBloom embeddings on the pediatric dataset (patient-level splits) and compares the results with the published DinoBloom benchmark on adult bone marrow. |
