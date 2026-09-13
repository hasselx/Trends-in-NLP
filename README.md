# Beyond Known Entities: Comparing BERT and Rule-Based Named Entity Recognition

**Naveen Krishnan Radhakrishna Pillai (1898670)**  
**Nithish Jegan Selvakumar (1903913)**

University of Trier — Trends in NLP

---

## Project Overview

Named Entity Recognition (NER) is the task of identifying and classifying named entities such as people, organizations, locations, and miscellaneous entities in text.

This project compares two different NER approaches:

1. A traditional **rule-based NER system**
2. A pre-trained **BERT-based NER system**

The main focus is on how well the two systems recognize **uncommon and unseen entity surface forms**.

The project uses the **CoNLL-2003** dataset and evaluates both systems on the test set using exact entity-span matching.

---

## Research Question

> **Does a context-aware BERT-based NER system identify uncommon and unseen named entities more accurately than a traditional rule-based NER system?**

---

## Hypothesis

A BERT-based NER system will outperform a traditional rule-based system, particularly for entity surface forms that are unseen during training, because contextual representations and subword tokenization can help identify entities without relying on exact dictionary matches.

---

## Dataset

The experiments use the **CoNLL-2003 Named Entity Recognition dataset**.

The dataset contains English newswire text and four entity categories:

- `PER` — Person
- `ORG` — Organization
- `LOC` — Location
- `MISC` — Miscellaneous

### Dataset Splits

| Split | Number of Sentences |
|---|---:|
| Training | 14,041 |
| Validation | 3,250 |
| Test | 3,453 |

---

## Entity Familiarity

Entity surface forms are categorized according to their frequency in the training split.

| Category | Training Frequency |
|---|---:|
| Common | ≥ 5 |
| Uncommon | 1–4 |
| Unseen | 0 |

An **unseen entity** therefore refers to an entity surface form that does not occur in our CoNLL-2003 training split.

> Important: "unseen" does not necessarily mean that the entity was unseen by the pre-trained BERT model.

---

# System A — Traditional Rule-Based NER

The traditional system uses information extracted from the CoNLL-2003 training data together with manually defined rules.

### Dictionary Matching

Entity dictionaries are constructed from the training data for:

- Person
- Organization
- Location
- Miscellaneous

Exact surface-form frequencies are also recorded.

### Regular Expression Rules

The system uses regular expressions for patterns including:

- Person titles followed by capitalized names
- Organization suffixes such as `Inc`, `Ltd`, `LLC`, `Corp`, `Corporation`, and `Company`

### Context Rules

Simple context-based rules are used to identify likely entities.

Examples include:

- Locations following words such as `in`, `from`, `near`, `to`, `towards`
- Persons following words such as `said`, `met`, `by`
- Organizations following words such as `company`, `organization`, `corporation`, `firm`

### Longest-Match Selection

When multiple dictionary matches are possible, the longest valid entity span is selected.

---

# System B — BERT-Based NER

The second system uses:

**`dslim/bert-base-NER`**

The model is loaded using Hugging Face Transformers.

### Processing

1. CoNLL-2003 sentences are provided to the BERT tokenizer.
2. Word-level alignment is preserved using word IDs.
3. BERT produces contextual token representations.
4. BIO predictions are generated for the tokenized input.
5. Subword predictions are aligned back to the original CoNLL-2003 words.
6. Entity spans are extracted from the resulting BIO labels.

---

## Evaluation

Both systems are evaluated on the **CoNLL-2003 test set**.

An entity is counted as correct only when:

- The predicted entity span exactly matches the gold span.
- The predicted entity type exactly matches the gold type.

### Metrics

The following metrics are reported:

- Precision
- Recall
- F1-score

In addition, recall is calculated separately for:

- Common entities
- Uncommon entities
- Unseen entities

---

# Results

## Overall Performance

| Metric | Traditional Rule-Based | BERT (`dslim/bert-base-NER`) |
|---|---:|---:|
| Gold entities | 5,648 | 5,648 |
| Predicted entities | 4,227 | 5,707 |
| Correct entities | 2,870 | 5,191 |
| **Precision** | **67.90%** | **90.96%** |
| **Recall** | **50.81%** | **91.91%** |
| **F1-score** | **58.13%** | **91.43%** |

### Performance Difference

| Metric | BERT Improvement |
|---|---:|
| Precision | +23.06 percentage points |
| Recall | +41.10 percentage points |
| F1-score | +33.30 percentage points |

---

## Recall by Entity Familiarity

| Entity Category | Traditional Rule-Based | BERT |
|---|---:|---:|
| **Common** | **92.60%** | **96.63%** |
| **Uncommon** | **92.46%** | **92.77%** |
| **Unseen** | **1.88%** | **87.81%** |

### Key Finding

The largest difference occurs for **unseen entity surface forms**.

BERT achieves:

**87.81% recall**

compared with:

**1.88% recall**

for the traditional rule-based system.

This represents an improvement of **85.92 percentage points**.

For uncommon entities, performance is almost identical:

- Rule-Based: **92.46%**
- BERT: **92.77%**

Therefore, the strongest advantage of BERT in this experiment is observed for **unseen entity surface forms**.

---

# Qualitative Example

The following example is taken from the CoNLL-2003 test set:

> But China saw their luck desert them in the second match of the group, crashing to a surprise 2-0 defeat to newcomers Uzbekistan.

### Gold Entities
- `China → LOC`
- `Uzbekistan → LOC`

### Traditional Rule-Based NER

- `China → LOC`

### BERT

- `China → LOC`
- `Uzbekistan → LOC`

The surface form **Uzbekistan** has a training frequency of **0**, making it an **unseen** entity according to our definition.

The rule-based system does not recognize it because its exact surface form is absent from the training dictionary, whereas BERT correctly identifies it.

---

# Discussion

The results support the hypothesis overall.

BERT achieves substantially higher performance than the traditional rule-based system across the overall evaluation:

- Higher precision
- Higher recall
- Higher F1-score

The most significant difference occurs for unseen entities, where BERT achieves **87.81% recall**, compared with only **1.88%** for the rule-based system.

However, the two systems perform similarly on uncommon entities. The difference is only **0.31 percentage points**.

The results indicate that dictionary-based approaches can perform well when entity surface forms are sufficiently represented in the training data, but their performance can deteriorate sharply when an entity surface form is absent from the dictionary.

---

# Limitations

- The rule-based system depends on hand-crafted rules and training-data dictionaries.
- Rule-based performance is therefore affected by dictionary coverage.
- "Unseen" means absent from our training split, not necessarily unseen by BERT.
- `dslim/bert-base-NER` is already a pre-trained and fine-tuned NER model.
- Therefore, this experiment is not a fully controlled comparison between a rule-based system and a BERT model trained from scratch on the same training split.
- A stronger controlled experiment would fine-tune BERT on the same CoNLL-2003 training split used for constructing the rule-based system.
- Evaluation uses exact entity-span matching.
- The experiment uses only the CoNLL-2003 English dataset and does not evaluate cross-domain performance.

---

# Conclusion

The experimental results support the hypothesis that a BERT-based NER system can outperform a traditional rule-based NER system, particularly for entity surface forms that are absent from the training split.

BERT achieves an overall F1-score of **91.43%**, compared with **58.13%** for the traditional rule-based system.

The largest difference is observed for unseen entities:

**BERT: 87.81% recall**  
**Rule-Based: 1.88% recall**

However, both systems achieve similar recall for uncommon entities.

Overall, the experiment demonstrates the strong advantage of contextual and subword-based representations over exact dictionary matching when dealing with entity surface forms that are not present in the training data.

---

# Repository Structure

```text
Trends-in-NLP/
│
├── README.md
├── System_A_Traditional.ipynb
└── System_B_BERT.ipynb
