# AI Exposure Score (AIES) for Korean Occupations

This repository provides the derived data accompanying

> Lee and Hwang, "Artificial Intelligence Exposure and Wage Effects in the Korean Labor Market", Journal of Asian Economics (under review).

It contains (a) the IPC codes used to identify AI-related patents, (b) the crosswalk between KSCO-7 and SOC-2018, and (c) the occupation-level AIES. Raw microdata from KLIPS, KIPRIS, and O\*NET are not redistributed here (see Data Sources).

## Files

| File                                 | Content                                                              | Rows  |
| ------------------------------------ | -------------------------------------------------------------------- | ----- |
| `IPC code of AI-related patents.csv` | AI-related IPC codes                                                 | 23    |
| `SOC-KSCO crosswalk.csv`             | KSCO-7 detailed occupations matched to SOC-2018 detailed occupations | 1,227 |
| `occupation-level AIES 2018.csv`     | AIES for each O\*NET-SOC occupation, Eq. (2) of the paper            | 873   |


## (a) IPC code of AI-related patents.csv

AI-related IPC codes following the KIPO guideline in Patent Statistics Related to the 4th Industrial Revolution (2020). A patent application is classified as AI-related if it contains at least one of these IPC codes.

| Variable | Description |
|---|---|
| `Sub-class` | IPC sub-class (e.g., G06N) |
| `Group` | IPC main group (e.g., G06N 3/00); all sub-groups are included |
| `Description` | IPC group title |

Patent applications filed in 2010 and 2018 were collected through the KIPRIS Open API. Of 268,719 applications, 20,434 AI patents with English titles were used (7,285 in 2010 and 13,149 in 2018).

## (b) SOC-KSCO crosswalk.csv

Each KSCO-7 detailed occupation (5-digit) is matched to the SOC-2018 detailed occupation (6-digit) whose title has the highest cosine similarity of Sentence-BERT (`all-mpnet-base-v2`) embeddings. The mapping is many-to-one.

| Variable       | Description                                                        |
| -------------- | ------------------------------------------------------------------ |
| `kscoCode`     | KSCO-7 detailed occupation code (5-digit)                          |
| `kscoTitle`    | KSCO-7 title (Korean)                                              |
| `kscoTitleEng` | KSCO-7 title (English)                                             |
| `similarity`   | Cosine similarity between `kscoTitleEng` and `socTitle` embeddings |
| `socCode`      | Matched SOC-2018 detailed occupation code (6-digit)                |
| `socTitle`     | SOC-2018 title                                                     |


## (c) occupation-level AIES 2018.csv

AIES for each O\*NET-SOC 2019 occupation $k$, computed from O\*NET task statements and AI patent titles filed in 2018:

```math
$$
\text{AIES}_k = \sum_{l=1}^{L_k} w_{k,l}\, s_{k,l}, \qquad
s_{k,l} = \max \frac{\mathbf{p} \cdot \mathbf{q}_{k,l}}{\lVert \mathbf{p} \rVert \lVert \mathbf{q}_{k,l} \rVert}, \qquad
w_{k,l} = \frac{\text{importance}_{k,l}}{\sum_{l} \text{importance}_{k,l}}
$$
```

where $`\mathbf{q}_{k,l}`$ is the embedding of task $`l`$ of occupation $`k`$, $`\mathbf{p}`$ is the embedding of an AI patent title, and $`\text{importance}_{k,l}`$ is the O\*NET task importance rating.

| Variable     | Description                                                                          |
| ------------ | ------------------------------------------------------------------------------------ |
| `code`       | O*NET-SOC 2019 code, written as `XX-XXXX-XX` (corresponds to `XX-XXXX.XX` in O\*NET) |
| `occupation` | O\*NET-SOC title                                                                     |
| `AIES`       | $`\text{AIES}_k`$                                                                      |

Of the 923 O\*NET-SOC occupations with task information, 50 occupations without task importance ratings are excluded, because the importance ratings serve as the weights $`w_{k,l}`$ in the AIES calculation. The file therefore contains 873 occupations.

## From occupation-level AIES to KSCO-7 minor groups

The AIES used in the wage regressions is obtained in two further steps:

1. Average $`\text{AIES}_k`$ within each SOC-2018 detailed occupation $`d`$ (first 7 characters of `code`):
   $`\text{AIES}_d = \frac{1}{\sum_k \mathbb{1}[k \in d]} \sum_{k \in d} \text{AIES}_k`$
2. Assign $`\text{AIES}_d`$ to KSCO-7 detailed occupations via the crosswalk, then average within each KSCO-7 minor group $m$ (first 3 digits of `kscoCode`):
   $`\text{AIES}_m = \frac{1}{\sum_d \mathbb{1}[d \in m]} \sum_{d \in m} \text{AIES}_d`$

## Data Sources

- Patents: KIPRIS Open API, Korean Intellectual Property Office
- Task statements and importance ratings: O\*NET U.S. Department of Labor (CC BY 4.0)
- Occupational classification: KSCO-7 (Statistics Korea, 2018), SOC-2018 (U.S. Bureau of Labor Statistics)
