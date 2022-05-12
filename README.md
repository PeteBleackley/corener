CoReNer
==============================

[![Python Version](https://img.shields.io/badge/python-3.7+-blue?logo=python)](https://www.python.org/)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

A multi-task model for named-entity recognition, relation extraction, entity mention detection and coreference resolution.

## Installation

```bash
cd corener-mtl
pip install -e .
```

## Training

Training CLI example:

```bash
python train.py --train-path path/to/train.json \
  --val-path path/to/val.json \
  --types-path path/to/types.json \
  --n-epoch 30 \
  --train-batch-size 16 \
  --lr 5e-5 \
  --model-name-or-path roberta-base \
  --artifact-path path/to/artifacts \
  --do-eval
```

## Inference

Inference example and output.

```bash
python inference.py 
  --artifact-path path/to/artifacts \ 
  --input "Steve Jobs was a charismatic pioneer of the personal computer era. With Steve Wozniak, Jobs founded Apple Inc. in 1976 and transformed the company into a world leader in telecommunications. Widely considered a visionary and a genius, he oversaw the launch of such revolutionary products as the iPod and the iPhone."
```

Output:

```json5
[
  {
    "tokens": [
      "Steve",
      "Jobs",
      "was",
      "a",
      ...
    ],
    "entities": [
      {
        "type": "PERSON",
        "start": 0,
        "end": 2,
        "span": [
          "Steve",
          "Jobs"
        ],
        "score": 0.9999884366989136,
        "start_char": 0,
        "end_char": 10
      },
      ...
    ],
    "relations": [
      {
        "type": "Work_For",
        "head": 0,
        "tail": 3,
        "head_span": [
          "Steve",
          "Jobs"
        ],
        "tail_span": [
          "Apple",
          "Inc."
        ],
        "score": 0.9500910043716431
      }
    ],
    "mentions": [
      {
        "type": "MENTION",
        "start": 0,
        "end": 2,
        "span": [
          "Steve",
          "Jobs"
        ],
        "score": 0.9999381303787231,
        "start_char": 0,
        "end_char": 10
      },
      ...
    ],
    "references": [
      {
        "type": "COREF",
        "head": 2,
        "tail": 0,
        "head_span": [
          "Jobs"
        ],
        "tail_span": [
          "Steve",
          "Jobs"
        ],
        "score": 0.9999998807907104
      },
      ...
    ],
    "clusters": [
      [
        {
          "start": 16,
          "end": 17,
          "span": [
            "Jobs"
          ],
          "cluster_id": 0
        },
        {
          "start": 41,
          "end": 42,
          "span": [
            "he"
          ],
          "cluster_id": 0
        },
        {
          "start": 0,
          "end": 2,
          "span": [
            "Steve",
            "Jobs"
          ],
          "cluster_id": 0
        }
      ],
      ...
    ]
  }
]
```

## Data

Training data is a `json` file of the following form:

```json5
[
  {
    "tokens": ["John", "met", "Jane", ".", "He", "asked", "her", "a", "question", "."],
    "entities": [
      {"type": "PERSON", "start": 0, "end": 1}, // John
      {"type": "PERSON", "start": 2, "end": 3}  // Jane
    ],
    "relations": [
      {"type": "MET", "head": 0, "tail": 1}  // "head"/"tail" is the index of the head/tail entities.
    ],
    "mentions": [
      {"type": "MENTION", "start": 0, "end": 1}, // John
      {"type": "MENTION", "start": 2, "end": 3}, // Jane
      {"type": "MENTION", "start": 4, "end": 5}, // he
      {"type": "MENTION", "start": 6, "end": 7}  // her
    ],
    "references": [
      {"type": "COREF", "head": 2, "tail": 0}, // He -> John
      {"type": "COREF", "head": 3, "tail": 1} // her -> Jane
    ],
    "is_ner": 1, // boolean for whether the doc is labeled for the NER task
    "is_emd": 1, // boolean for whether the doc is labeled for the EMD task
    "is_re": 1, // boolean for whether the doc is labeled for the relation extraction task
    "is_cr": 1, // boolean for whether the doc is labeled for the co-reference task
  },
  {
    // second document.
  }
]
```

In addition, you will need to provide a `types.json` file will all entity/relation types presented in the training data. For example, to train CoReNer on the Ontonotes + Conll04 datasets we use the following file:

```json5
{
  "entities": {
    "ORG": {"short": "ORG", "verbose": "ORGANIZATION"},
    "PERSON": {"short": "PER", "verbose":"PERSON"},
    "NORP": {"short": "NORP", "verbose":"Nationalities or religious or political groups"},
    "FAC": {"short": "FAC", "verbose":"Buildings, airports, highways, bridges"},
    "GPE": {"short": "GPE", "verbose":"Countries, cities, states."},
    "LOC": {"short": "LOC", "verbose":"LOCATION"},
    "PRODUCT": {"short": "PROD", "verbose": "PRODUCT"},
    "DATE": {"short": "DATE", "verbose": "DATE"},
    "TIME": {"short": "TIME", "verbose": "TIME"},
    "PERCENT": {"short": "PERCENT", "verbose": "PERCENT"},
    "MONEY": {"short": "MONEY", "verbose": "MONEY"},
    "QUANTITY": {"short": "QUANTITY", "verbose": "QUANTITY"},
    "ORDINAL": {"short": "ORDINAL", "verbose": "ORDINAL"},
    "CARDINAL": {"short": "CARDINAL", "verbose": "CARDINAL"},
    "EVENT": {"short": "EVENT", "verbose": "EVENT"},
    "WORK_OF_ART": {"short": "WORK_OF_ART", "verbose": "WORK_OF_ART"},
    "LAW": {"short": "LAW", "verbose": "LAW"},
    "LANGUAGE": {"short": "LANGUAGE", "verbose": "LANGUAGE"}
  },
  "relations": {
    "Work_For": {"short": "Work", "verbose": "Work for", "symmetric": false},
    "Kill": {"short": "Kill", "verbose": "Kill", "symmetric": false},
    "OrgBased_In": {"short": "OrgBI", "verbose": "Organization based in", "symmetric": false},
    "Live_In": {"short": "Live", "verbose": "Live in", "symmetric": false},
    "Located_In": {"short": "LocIn", "verbose": "Located in", "symmetric": false}
  },
  "references": {
    "COREF": {"short": "COREF", "verbose": "COREF"}
  },
  "mentions": {
    "MENTION": {"short": "MENTION", "verbose": "MENTION"}
  }
}
```