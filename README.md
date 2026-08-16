# UnaniMedKG-GraphRAG

**Bilingual English–Urdu Knowledge Graph and GraphRAG framework for evidence-grounded question answering in Unani medicine**

UnaniMedKG-GraphRAG is a research implementation for organizing Unani medicine knowledge as a bilingual Neo4j knowledge graph and using that structured knowledge to support Retrieval-Augmented Generation (GraphRAG) with large language models. The repository contains the bilingual dataset, graph-construction logic, Cypher-based retrieval code, LoRA fine-tuning notebooks for Llama 3 and Mistral, and English/Urdu evaluation sets and evaluation pipelines.

> **Research-use notice:** This project is intended for research, knowledge preservation, and computational evaluation. It is **not** a clinical decision-support system and should not be used as a substitute for professional medical advice, diagnosis, or treatment.

---

## Project Overview

The implementation follows this pipeline:

```text
Unani medicine data
        |
        v
Preprocessing and normalization
        |
        v
English -> Urdu translation (NLLB-200)
        |
        v
Bilingual Unani dataset
        |
        v
Neo4j knowledge graph (UnaniMedKG)
        |
        +-----------------------------+
        |                             |
        v                             v
Semantic entity retrieval        Cypher graph traversal
(multilingual MPNet)                  |
        |                             |
        +-------------+---------------+
                      |
                      v
               Graph context
                      |
                      v
          Llama 3 / Mistral LLM
             + optional LoRA
                      |
                      v
      English / Urdu question answering
                      |
                      v
     METEOR / BERTScore / other metrics
```

The implementation supports both **single-hop** and **multi-hop** Unani medicine questions.

---

## Main Components

### 1. Bilingual Unani Dataset

The included dataset is:

```text
Dataset/data.csv
```

It contains **5,040 rows** and includes English and Urdu fields for Unani formulations and related information.

Important fields include:

| English field | Urdu counterpart | Description |
|---|---|---|
| `unani_herb__formulations` | `unani_herb__formulations_ur` | Unani formulation/herb name |
| `botanic_name` | `botanic_name_ur` | Botanical name |
| `parts_used` | `parts_used_ur` | Plant part used |
| `temperament` | `temperament_ur` | Unani temperament / mizāj information |
| `ingredients` | `ingredients_ur` | Active/associated ingredients |
| `diseases` | `diseases_ur` | Diseases/conditions associated with treatment |
| `symptoms_relieved` | `symptoms_relieved_ur` | Symptoms relieved |
| `dosage` | `dosage_ur` | Dosage information |
| `treatment` | `treatment_ur` | Treatment description |

The included CSV is already bilingual and can be used directly for the Neo4j construction stage.

---

## Repository Structure

```text
.
├── Dataset/
│   └── data.csv
│
├── Evaluation queries/
│   ├── final.json
│   └── final_urdu.json
│
├── PREPROCESSING,NEO4J,TRANSLATION.ipynb
├── FINETUNE_LLAMA_MODEL.ipynb
├── FINETUNE_MISTRAL_MODEL.ipynb
├── GRAPHRAG_LLAMA3B_Final.ipynb
└── GRAPHRAG_MISTRAL_Final.ipynb
```

### What each notebook does

| File | Purpose |
|---|---|
| `PREPROCESSING,NEO4J,TRANSLATION.ipynb` | Data cleaning, normalization, English-to-Urdu translation with NLLB-200, Neo4j constraints/indexes, node and relationship creation, and graph statistics |
| `FINETUNE_LLAMA_MODEL.ipynb` | Parameter-efficient fine-tuning of `meta-llama/Meta-Llama-3-8B-Instruct` using LoRA |
| `FINETUNE_MISTRAL_MODEL.ipynb` | Parameter-efficient fine-tuning of `mistralai/Mistral-7B-Instruct-v0.3` using LoRA |
| `GRAPHRAG_LLAMA3B_Final.ipynb` | Llama 3 GraphRAG pipeline, bilingual query handling, Neo4j retrieval, response generation, and English/Urdu evaluation |
| `GRAPHRAG_MISTRAL_Final.ipynb` | Mistral GraphRAG pipeline and English/Urdu evaluation |
| `Evaluation queries/final.json` | English benchmark queries |
| `Evaluation queries/final_urdu.json` | Urdu benchmark queries |

---

## Reproducibility Resources

The repository maps the reproducibility material to the following files:

| Reproducibility item | Location |
|---|---|
| Sample bilingual dataset | `Dataset/data.csv` |
| Graph schema definitions | `PREPROCESSING,NEO4J,TRANSLATION.ipynb` |
| Neo4j constraints and indexes | `PREPROCESSING,NEO4J,TRANSLATION.ipynb` |
| Cypher graph-construction queries | `PREPROCESSING,NEO4J,TRANSLATION.ipynb` |
| GraphRAG retrieval Cypher | `GRAPHRAG_LLAMA3B_Final.ipynb`, `GRAPHRAG_MISTRAL_Final.ipynb` |
| Fine-tuning implementation | `FINETUNE_LLAMA_MODEL.ipynb`, `FINETUNE_MISTRAL_MODEL.ipynb` |
| English evaluation dataset | `Evaluation queries/final.json` |
| Urdu evaluation dataset | `Evaluation queries/final_urdu.json` |
| Evaluation implementation | `GRAPHRAG_LLAMA3B_Final.ipynb`, `GRAPHRAG_MISTRAL_Final.ipynb` |

---

## Knowledge Graph Schema

The Neo4j construction code creates the following main node labels:

- `UnaniFormulation`
- `BotanicalName`
- `PlantPart`
- `Temperament`
- `Ingredient`
- `Disease`
- `Symptom`
- `Dosage`
- `Treatment`

Each bilingual node stores a `language` property to distinguish English and Urdu entities.

### Main relationships

```text
(UnaniFormulation)-[:HAS_BOTANICAL_NAME]->(BotanicalName)
(UnaniFormulation)-[:USES_PLANT_PART]->(PlantPart)
(UnaniFormulation)-[:HAS_TEMPERAMENT]->(Temperament)
(UnaniFormulation)-[:CONTAINS_INGREDIENT]->(Ingredient)
(UnaniFormulation)-[:TREATS_DISEASE]->(Disease)
(UnaniFormulation)-[:RELIEVES_SYMPTOM]->(Symptom)
(UnaniFormulation)-[:HAS_DOSAGE]->(Dosage)
(UnaniFormulation)-[:HAS_TREATMENT]->(Treatment)

(Disease)-[:HAS_SYMPTOM]->(Symptom)
(Ingredient)-[:FOUND_IN_PART]->(PlantPart)

(English entity)-[:TRANSLATES_TO]->(Urdu counterpart)
(Urdu entity)-[:TRANSLATES_TO]->(English counterpart)
```

The construction notebook also creates uniqueness constraints and full-text/language indexes to support reliable graph creation and retrieval.

---

## Translation Pipeline

The preprocessing notebook uses:

```text
facebook/nllb-200-distilled-600M
```

for English-to-Urdu translation.

The translation code uses:

```text
src_lang = eng_Latn
tgt_lang = urd_Arab
```

and saves intermediate/final translated files during the translation workflow.

### Important note

The preprocessing section references an earlier raw input path:

```text
sample_data/5000done.csv
```

That raw file is **not included in the provided repository package**. The repository already contains the resulting bilingual dataset at:

```text
Dataset/data.csv
```

Therefore, users who only want to build and evaluate UnaniMedKG can start from `Dataset/data.csv`. Re-running the full preprocessing/translation stage requires the original raw source file or an equivalent dataset with the expected columns.

---

## Neo4j Setup

A running Neo4j database is required.

Configure:

```bash
export NEO4J_URI="neo4j+s://YOUR_INSTANCE"
export NEO4J_USER="neo4j"
export NEO4J_PASSWORD="YOUR_PASSWORD"
```

For notebook environments such as Google Colab, set the same values securely in environment variables or notebook secrets.

**Do not commit Neo4j passwords, Hugging Face tokens, or other credentials to GitHub.**

The Neo4j construction notebook currently contains credential placeholders that should be populated securely before execution.

### Building the graph

1. Open:

   ```text
   PREPROCESSING,NEO4J,TRANSLATION.ipynb
   ```

2. Point the Neo4j import step to:

   ```text
   Dataset/data.csv
   ```

3. Configure the Neo4j connection.

4. Run:
   - constraint creation,
   - index creation,
   - node creation,
   - relationship creation,
   - graph statistics.

---

## GraphRAG Retrieval

The GraphRAG notebooks combine structured Neo4j retrieval with multilingual semantic matching.

### Semantic retriever

The semantic component uses:

```text
sentence-transformers/paraphrase-multilingual-mpnet-base-v2
```

The implementation:

1. detects query language,
2. extracts candidate English/Urdu terms,
3. embeds query terms and graph entities,
4. computes cosine similarity,
5. performs semantic entity linking,
6. retrieves graph information using Cypher,
7. supplies the retrieved graph context to the LLM.

The implementation includes embedding caching and cached Neo4j query execution to reduce repeated computation.

### Graph context

For identified formulations, the retriever can collect:

- botanical names,
- temperament,
- plant parts,
- ingredients,
- dosage,
- treatments,
- diseases,
- relieved symptoms.

This graph evidence is then inserted into an English or Urdu prompt before response generation.

---

## Language Support

The GraphRAG implementation supports:

- **English**
- **Urdu**

Urdu handling includes:

- Unicode-based language detection,
- Arabic-script preprocessing,
- Arabic reshaping,
- bidirectional text handling,
- Urdu-specific prompt construction.

---

## LLMs

Two instruction-tuned models are used:

### Llama

```text
meta-llama/Meta-Llama-3-8B-Instruct
```

### Mistral

```text
mistralai/Mistral-7B-Instruct-v0.3
```

Access to some model checkpoints may require accepting the corresponding Hugging Face model license/terms and authenticating with a Hugging Face account.

---

## LoRA Fine-Tuning

Both fine-tuning notebooks use Parameter-Efficient Fine-Tuning (PEFT) with LoRA.

Common settings in the implementation include:

```text
LoRA rank (r):          16
LoRA alpha:             32
LoRA dropout:           0.05
Epochs:                 3
Learning rate:          2e-5
Weight decay:           0.01
Evaluation steps:       50
Save steps:             50
Logging steps:          10
Warmup ratio:           0.1
Maximum gradient norm:  0.3
Maximum sequence length: 1024
Optimizer:              paged_adamw_8bit
Scheduler:              cosine
```

The implementation attempts memory-efficient model loading with 4-bit or 8-bit quantization where supported.

### LoRA target modules

The Llama notebook targets:

```text
q_proj
k_proj
v_proj
o_proj
gate_proj
up_proj
down_proj
```

The Mistral notebook targets:

```text
q_proj
k_proj
v_proj
o_proj
```

---

## Fine-Tuning Data Format

The fine-tuning notebooks expect a JSON file named:

```text
unani_train_en_ur.json
```

with a list of samples containing at least:

```json
[
  {
    "instruction": "Question in English",
    "output": "Reference answer in English",
    "instruction_ur": "Optional Urdu question",
    "output_ur": "Optional Urdu answer"
  }
]
```

### Reproducibility note

`unani_train_en_ur.json` is referenced by both fine-tuning notebooks but is **not present in the supplied implementation archive**. To reproduce the fine-tuning stage exactly, add this training JSON to the public repository or document how it is generated from the released data.

---

## Running Fine-Tuning

### Llama 3

Open and run:

```text
FINETUNE_LLAMA_MODEL.ipynb
```

Before training:

1. configure Hugging Face authentication securely;
2. ensure `unani_train_en_ur.json` is available;
3. confirm GPU/CUDA availability;
4. set or verify the output directory.

The adapter is saved to a timestamped directory similar to:

```text
unani-lora-ft-<timestamp>
```

### Mistral

Open and run:

```text
FINETUNE_MISTRAL_MODEL.ipynb
```

Follow the same preparation steps.

---

## Running GraphRAG

### Llama 3 GraphRAG

Use:

```text
GRAPHRAG_LLAMA3B_Final.ipynb
```

Set:

```text
NEO4J_URI
NEO4J_USER
NEO4J_PASSWORD
LORA_MODEL_PATH
```

If a LoRA adapter is supplied, the notebook loads it on top of the base model. If no adapter path is supplied, the system can operate with the base model.

### Mistral GraphRAG

Use:

```text
GRAPHRAG_MISTRAL_Final.ipynb
```

Configure the same Neo4j credentials and the relevant Mistral LoRA adapter path.

---

## Example Query Flow

Example English query:

```text
What is the botanical name of Gaozaban-e-Hindi?
```

Conceptually, the system performs:

```text
Question
  -> language detection
  -> candidate entity extraction
  -> multilingual semantic matching
  -> Neo4j graph retrieval
  -> evidence/context assembly
  -> LLM generation
  -> final answer
```

A multi-hop query can involve several constraints, such as a formulation, disease, temperament, symptom, ingredient, or treatment relationship.

---

## Evaluation Data

### English

```text
Evaluation queries/final.json
```

Contains:

- **52 single-hop questions**
- **47 multi-hop questions**
- **99 English questions total**

The JSON structure is:

```json
{
  "single_hop": [
    {
      "instruction": "...",
      "input": "",
      "output": "...",
      "type": "single-hop",
      "herb": "..."
    }
  ],
  "multi_hop": [
    {
      "instruction": "...",
      "input": "",
      "output": "...",
      "type": "multi-hop",
      "herbs": ["...", "..."]
    }
  ]
}
```

### Urdu

```text
Evaluation queries/final_urdu.json
```

Contains:

- **56 single-hop questions**
- **33 multi-hop questions**
- **89 Urdu questions total**

Together, the provided evaluation files contain **188 benchmark questions**.

---

## Evaluation

The evaluation code in the GraphRAG notebooks compares generated answers with reference answers.

Metrics loaded/used by the implementation include:

- BLEU
- ROUGE
- METEOR
- BERTScore Precision
- BERTScore Recall
- BERTScore F1
- heuristic answer relevance

The notebooks contain evaluation logic for:

- base model responses,
- fine-tuned model responses,
- GraphRAG responses,
- GraphRAG with fine-tuned models,
- single-hop queries,
- multi-hop queries,
- English queries,
- Urdu queries.

Example output files referenced by the notebooks include:

```text
unani_comprehensive_evaluation_llama_results.json
graphrag_evaluation_llama_results.json
unani_comprehensive_evaluation_results.json
graphrag_evaluation_results.json
```

---

## Installation

A GPU-enabled environment is recommended for fine-tuning and efficient inference.

Create an environment:

```bash
python -m venv .venv
source .venv/bin/activate
```

Install the core packages used by the notebooks:

```bash
pip install \
  torch torchvision torchaudio \
  transformers accelerate datasets \
  sentence-transformers \
  peft bitsandbytes trl \
  neo4j \
  pandas numpy scikit-learn \
  evaluate rouge_score bert_score nltk \
  tqdm tenacity \
  langdetect \
  arabic-reshaper python-bidi \
  langchain langchain-community \
  python-dotenv sentencepiece protobuf \
  jupyter
```

The notebooks contain experiment-specific installation/version cells. For strict reproduction, follow the versions recorded in the relevant notebook and use a CUDA-compatible PyTorch build.

---

## Recommended Execution Order

If you are using the **already included bilingual `Dataset/data.csv`**:

```text
1. Configure Neo4j
2. Run the Neo4j construction section in PREPROCESSING,NEO4J,TRANSLATION.ipynb
3. Add/configure the fine-tuning dataset if reproducing model adaptation
4. Run FINETUNE_LLAMA_MODEL.ipynb and/or FINETUNE_MISTRAL_MODEL.ipynb
5. Set the LoRA adapter path in the corresponding GraphRAG notebook
6. Run GRAPHRAG_LLAMA3B_Final.ipynb or GRAPHRAG_MISTRAL_Final.ipynb
7. Evaluate with Evaluation queries/final.json
8. Evaluate with Evaluation queries/final_urdu.json
```

If you are rebuilding from the original raw English data:

```text
1. Provide the raw source dataset expected by the preprocessing notebook
2. Run preprocessing
3. Run English-to-Urdu NLLB translation
4. Export the bilingual dataset
5. Construct the Neo4j graph
6. Continue with fine-tuning and GraphRAG evaluation
```

---

## Environment Variables and Secrets

Recommended configuration:

```bash
export NEO4J_URI="..."
export NEO4J_USER="..."
export NEO4J_PASSWORD="..."
export HF_TOKEN="..."
```

For Google Colab, use Colab Secrets rather than hard-coding tokens.

### Security warning

Before publishing the notebooks, search all cells for:

```text
HF_TOKEN
login(token=...)
NEO4J_PASSWORD
PASSWORD
```

and remove any real credential values. If a real access token has ever been committed or shared publicly, revoke it and create a new one.

---

## Current Reproducibility Notes

To make the repository fully self-contained for external reviewers/researchers, verify the following before release:

1. `Dataset/data.csv` is publicly accessible.
2. `Evaluation queries/final.json` and `final_urdu.json` are committed.
3. The referenced `unani_train_en_ur.json` is added, or its generation procedure is documented.
4. Neo4j credentials are replaced with environment-variable configuration.
5. Hugging Face tokens are removed from notebook cells.
6. Fine-tuned adapter paths are documented or adapters are released separately.
7. Any raw dataset required to reproduce the translation stage is released where licensing permits, or its provenance/generation procedure is documented.
8. Notebook file paths are updated to match the public repository layout.
9. A stable repository release/tag or archival DOI is used in the manuscript.

---

## Troubleshooting

### Neo4j connection errors

Confirm that:

- the Neo4j instance is running,
- `NEO4J_URI` is correct,
- the database accepts remote connections,
- username/password are valid,
- firewall/network settings allow access.

### Hugging Face model access

Some model repositories require authentication or acceptance of model terms. Authenticate through an environment variable or notebook secret rather than embedding the token in source code.

### GPU memory errors

The implementation uses quantization and LoRA to reduce memory use. If memory remains insufficient:

- reduce generation length,
- reduce training batch size,
- increase gradient accumulation,
- use 4-bit model loading,
- restart the runtime between large model runs.

### Urdu reshaping issues

The notebooks depend on `arabic-reshaper` and `python-bidi`. If an Urdu formatting API differs across installed versions, install a compatible version and verify Urdu output rendering before evaluation.

---

## Research Scope

This repository is designed to support research on:

- computational preservation of Unani medical knowledge,
- bilingual English–Urdu knowledge representation,
- domain-specific knowledge graphs,
- Neo4j-based structured retrieval,
- GraphRAG,
- multilingual semantic retrieval,
- low-resource medical NLP,
- LLM adaptation with LoRA,
- single-hop and multi-hop question answering.

---

## Citation

If you use this repository in academic work, please cite the associated paper:

```bibtex
@article{unanimedkg_graphrag,
  title   = {GraphRAG-Enhanced Bilingual Knowledge Graph LLM for Unani Medicines},
  author  = {Ashraf, Ayesha and Waheed, Talha and Khurshid, Khaldoon S. and Javed, Abqa and Haris, Muhammad},
  year    = {2026},
  note    = {Manuscript / publication details to be updated}
}
```

Update the BibTeX entry with the final journal, volume, DOI, and publication details when available.

---

## License

A license file was not present in the supplied implementation package. Add an appropriate `LICENSE` file before public redistribution if required by your project/data/model usage conditions.

---

## Acknowledgment

This implementation brings together Unani medicine knowledge representation, bilingual data processing, Neo4j graph modeling, multilingual semantic retrieval, GraphRAG, and parameter-efficient LLM adaptation to support transparent and reproducible research on English–Urdu Unani medicine question answering.
