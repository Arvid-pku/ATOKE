# ATOKE
This is the repository for our paper [AAAI 2024] [History Matters: Temporal Knowledge Editing in Large Language Model](https://arxiv.org/abs/2312.05497).







## Datasets

### Overview
ATOKE dataset contains three temporal knowledge editing datasets.

The datasets are included in `datasets/`. There are three files:
* `AToKe-SE.json`: Single Edit dataset.
* `AToKe-ME.json`: Multiple Edits dataset.
* `AToKe-EE.json`: Extending Edit dataset.

### Data format

The dataset is saved as a list of dicts, each of which represents a data instance. We take `AToKe-ME`, the most complex format, as an example, while `AToKe-SE` and `AToKe-EE` are both simplifications of `AToKe-ME`. An example in `AToKe-ME` is shown below.

```
{
    "case_id": 3,
    "requested_rewrite": [
        {   ### Note: Multi-step editing - first step
            "prompt": "{}'s team is",
            "time_prompt": "From 1947 to 1948, {} is a player of",
            "target_new": {"str": "Southampton F.C.", "id": "<1!Phb4wLjn>"},
            "target_true": {"str": "Manchester United F.C.", "id": "<zUrH0rVXjH>"
            },
            "subject": "Billy Wrigglesworth",
            "relation": "<playsFor>", "relation_id": "<Xu7>",
            "time_new": {"since": 1947, "until": 1948, "gptj": false},
            "time_true": {"since": 1937, "until": 1947, "gptj": true},
            "evaluation": {
                "completion": "Billy Wrigglesworth's team is",
                "questions": "Which club does Billy Wrigglesworth play for?",
                "paraphrase_completion": "Billy Wrigglesworth plays for",
                "time_completion": "From 1947 to 1948, Billy Wrigglesworth is a player of",
                "time_questions": "Which club does Billy Wrigglesworth affiliate with from 1947 to 1948?",
                "paraphrase_time_completion": "From 1947 to 1948, Billy Wrigglesworth plays for"
            }
        },
        {   ### Note: Multi-step editing - second step
            "prompt": "{}'s team is",
            "time_prompt": "From 1948 to 1953, {} is a player of",
            "target_new": {"str": "Arsenal F.C.", "id": "<6zNA4ZR7x?>"},
            "target_true": {"str": "Southampton F.C.", "id": "<1!Phb4wLjn>"},
            "subject": "Billy Wrigglesworth",
            "relation": "<playsFor>", "relation_id": "<Xu7>",
            "time_new": {"since": 1948, "until": 1953, "gptj": false},
            "time_true": {"since": 1947, "until": 1948, "gptj": false},
            "evaluation": {
                "completion": "Billy Wrigglesworth's team is",
                "questions": "Which club does Billy Wrigglesworth play for?",
                "paraphrase_completion": "Billy Wrigglesworth plays for",
                "time_completion": "From 1948 to 1953, Billy Wrigglesworth is a player of",
                "time_questions": "Which club does Billy Wrigglesworth affiliate with from 1948 to 1953?",
                "paraphrase_time_completion": "From 1948 to 1953, Billy Wrigglesworth plays for"
            }
        }
    ],
    "history_evaluation": [
        {
            "completion": "Billy Wrigglesworth used to play for",
            "questions": "Which team did Billy Wrigglesworth play for before?",
            "paraphrase_completion": "Billy Wrigglesworth's previous team was",
            "time_completion": "From 1937 to 1947, Billy Wrigglesworth's team was",
            "time_questions": "Which club did Billy Wrigglesworth play for from 1937 to 1947?",
            "paraphrase_time_completion": "From 1937 to 1947, Billy Wrigglesworth played for"
        },
        {
            "completion": "Billy Wrigglesworth used to play for",
            "questions": "Which team did Billy Wrigglesworth play for before?",
            "paraphrase_completion": "Billy Wrigglesworth's previous team was",
            "time_completion": "From 1947 to 1948, Billy Wrigglesworth's team was",
            "time_questions": "Which club did Billy Wrigglesworth play for from 1947 to 1948?",
            "paraphrase_time_completion": "From 1947 to 1948, Billy Wrigglesworth played for"
        }
    ],
    "answer": [["Manchester United F.C."], ["Southampton F.C."]],
    "answer_alias": [["Manchester Red Devils", ...],["Soton FC", ...]],
    "new_answer": [["Southampton F.C."], ["Arsenal F.C."]],
    "new_answer_alias": [["Soton FC", ...],["Arsenal football club", ...]],
}
```
* `requested_rewrite`: The list of editing facts we want to inject into the model: only one element in `AToKe-SE` and `AToKe-EE`; multiple consecutive-time fact lists in `AToKe-ME`. (In general, we follow the format of the [`MQuAKE`](https://github.com/princeton-nlp/MQuAKE/blob/main/datasets/MQuAKE-CF-3k.json) dataset. In particular, we place the question of the current time here.)
    * `time_true` and `time_new`: The current time interval of the model and the new time interval we want to inject into the model. "gptj: true" indicates the current time interval before editing the model.
    * `evaluation`: includes both completion (cloze-style for GPT-j) and natural language QA formats, and includes both direct questions about the **current time** and questions that include current time intervals. `paraphrase_completion` is used to eliminate the effects of different textual expressions
* `history_evaluation`: includes both completion and natural language QA formats, and includes both direct questions about **past time** and questions that include past time intervals
* `answer` and `answer_alias`: the gold answer **before** injecting new facts into language models. `answer_alias` is a list of aliases of the answer.
* `new_answer` and `new_answer_alias`: the gold answer **after** injecting new facts into language models. 

*For AToKe-EE only*:
- The object before and after editing is **unchanged** and the time interval is **extended**.
- There is no `history_evaluation`, because the facts don't change, just the time interval of the facts.


## Evaluation
We evaluate the model's ability to edit temporal knowledge in terms of two dimensions: the time at which the model is questioned (historical, current) and the form of the time at which it is questioned (relative, explicit).

|          | Historical                                    | Current                                    |
| -------- | --------------------------------------------- | ------------------------------------------ |
| Relative | Historical Relative time Question Score (**HRS**) | Current Relative time Question Score (**CRS**) |
| Explicit | Historical Explicit time Question Score (**HES**) | Current Explicit time Question Score (**CES**) |

In addition, we eliminate the effect of question description by paraphrasing the question. We measure this as the **Paraphrase Question Score**.

Examples are as follows:
- time:
    - historical: "Billy Wrigglesworth used to play for"
    - current: "Billy Wrigglesworth's team is"
- time form:
    - relative: "Billy Wrigglesworth's team is"
    - explicit: "From 1947 to 1948, Billy Wrigglesworth is a player of"
    - paraphrase: "From 1947 to 1948, Billy Wrigglesworth plays for"


## METO



## Issues or Questions?
If you come across any issues while using the datasets or have any questions regarding the repository or the paper, please don't hesitate to reach out. You can contact Xunjian Yin at (xjyin@pku.edu.cn) or create an issue.

## Citation
If you use our code in your research, please cite our work:
```bibtex
@article{yin2023history,
  title={History Matters: Temporal Knowledge Editing in Large Language Model},
  author={Yin, Xunjian and Jiang, Jin and Yang, Liming and Wan, Xiaojun},
  journal={arXiv preprint arXiv:2312.05497},
  year={2023}
}
```
