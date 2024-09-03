# Identifying Semantic Relationships Between Research Topics Using LLMs in a Zero-Shot Learning Setting

In this GitHub repository, you can access the Gold Standard and the code for identifying relationships between research topic pairs in the Gold Standard.

[![Python 3.12.0](https://img.shields.io/badge/python-3.12.0-blue.svg)](https://www.python.org/downloads/release/python-3120/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

# Abstract.
Knowledge Organization Systems (KOS), such as ontologies, taxonomies, and thesauri, play a crucial role in organising scientific knowledge. They help scientists navigate the vast landscape of research literature and are essential for building intelligent systems such as smart search engines, recommendation systems, conversational agents, and advanced analytics tools. However, the manual creation of these KOSs is costly, time-consuming, and often leads to outdated and overly broad representations. As a result, researchers have been exploring automated or semi-automated methods for generating ontologies of research topics. This paper analyses the use of large language models (LLMs) to identify semantic relationships between research topics.
We specifically focus on ``six open`` and ``lightweight`` LLMs (up to ``10.7`` billion parameters) and use two ``zero-shot reasoning`` strategies to identify four types of relationships: _broader_, _narrower_, _same-as_, and _other_. Our preliminary analysis indicates that ``Dolphin2.1-OpenOrca-7B`` performs strongly in this task, achieving a ``0.853`` F1-score against a gold standard of ``1,000`` relationships derived from the IEEE Thesaurus. These promising results bring us one step closer to the next generation of tools for automatically curating KOSs, ultimately making the scientific literature easier to explore.

![iamge](https://github.com/ImTanay/LLM-Semantic-Relationship-Analysis/assets/59340198/4bccbaa8-2c5f-462a-ac02-411411a90602)

**Figure 1**: Architecture of our two strategies. The first strategy (red dashed box) determines the relationship between 𝑡<sub>𝑎</sub> and 𝑡<sub>b</sub> in one way, whereas the second strategy (green dashed box) determines the relationship between pairs of topics in both ways.

# 📂 Folders
Below is the distribution and organisation of the folders in this repository.

## 📄 dataset
This folder contains the Gold Standard. Accessible [here](./dataset)

To create our gold standard dataset from the [IEEE Thesaurus](https://github.com/angelosalatino/ieee-taxonomy-thesaurus-rdf/blob/main/source/ieee-thesaurus_2023.pdf), we followed these steps:

1. **Data Extraction and Transformation**: We transformed the hierarchical structures and relationships from the original IEEE Thesaurus PDF into RDF format using a script that we developed, which is available [here](https://github.com/angelosalatino/ieee-taxonomy-thesaurus-rdf).

2. **Sampling for Gold Standard**: The relationships were represented using the ``SKOS-Notation``. Here are the relationships utilised:

   | SKOS Notation     | Relationship Type         |
   |-------------------|---------------------------|
   | `skos:broader`    | broader      |
   | `skos:narrower`   | narrower     |
   | `skos:altLabel`   | same-as         |
   | `skos:prefLabel`  | same-as          |

We randomly selected ``250`` relationships for each category: ``broader``, ``narrower``, ``same-as``, and ``other``. For the ``other`` category, the relationships were established by pairing topics at random, ensuring that they did not overlap with existing semantic relationships within the thesaurus.

This method ensured that our gold standard dataset of 1K semantic relationships was diverse and representative of the various types of relationships defined in the IEEE Thesaurus.

## 🖥 code

This folder contains the script that we used to identify the semantic relationships between pairs of research topics. 

The script (accessible [here](https://github.com/ImTanay/LLM-Semantic-Relationship-Analysis/blob/main/code/llm_relation_classifier.ipynb)) functions as follows:

#### Task Definition and Experiments

The task involves classifying the semantic relationship between pairs of research topics ($t_A$, $t_B$) into four categories essential for ontology generation:

- **broader**: $t_A$ is a parent topic of $t_B$. Example: ``ontological languages`` is broader than ``owl``.
- **narrower**: $t_A$ is a child topic of $t_B$. Example: ``nosql`` is a specific area within ``databases``.
- **same-as**: $t_A$ and $t_B$ can be used interchangeably to refer to the same concept. Example: ``haptic interface`` and ``haptic device``.
- **other**: $t_A$ and $t_B$ do not fit into the above categories. Example: ``blockchain`` and ``user interfaces``.

#### Experiment Strategies

The experiments are conducted using two strategies:

- **One-way Strategy**: Each pair of topics is processed once using a prompt template designed for classification by a language model.
  
- **Two-way Strategy**: Each pair is processed twice:
  1. First, the relationship between $t_A$ and $t_B$ is identified.
  2. Then, the relationship between $t_B$ and $t_A$ is identified in a separate context.
  
#### Prompt Template

A standardised prompt template is used across both strategies and all models.

#### Empirical Rules for Two-way Strategy Agreement

Empirical rules (cyan box in Figure 1) are employed to reconcile agreements and disagreements between the two branches of the two-way strategy:

1. broader :- f(broader) $\land$ s(narrower)
2. narrower :- f(narrower) $\land$ s(broader)
3. broader :- ((f(narrower) $\land$ s(narrower)) $\lor$ (f(broader) $\land$ s(broader))) $\land$ len($t_A$) $\leq$ len($t_B$)
4. narrower :- ((f(narrower) $\land$ s(narrower)) $\lor$ (f(broader) $\land$ s(broader))) $\land$ len($t_A$) $>$ len($t_B$)
5. same-as :- f(same-as) $\land$ s(same-as)
6. broader :- (f(broader) $\land$ s(other)) $\lor$ (f(other) $\land$ s(narrower))
7. narrower :- (f(narrower) $\land$ s(other)) $\lor$ (f(other) $\land$ s(broader))
8. :- f(X) 

---

| Rule Number | Rule Description                                                                 |
|-------------|---------------------------------------------------------------------------------|
| 1           | Assign **broader** if first branch ``(f)`` returns broader and second branch ``(s)`` returns narrower. |
| 2           | Assign **narrower** if ``(f)`` returns narrower and ``(s)`` returns broader.                 |
| 3           | Assign **broader** if both branches return narrower or broader and $t_A$ length $\leq$ $t_B$ length. |
| 4           | Assign **narrower** if both branches return narrower or broader and $t_A$ length > $t_B$ length. |
| 5           | Assign **same-as** if both branches return same-as.                               |
| 6           | Assign **broader** if ``(f)`` returns broader and ``(s)`` returns other, or f returns other and s returns narrower. |
| 7           | Assign **narrower** if ``(f)`` returns narrower and ``(s)`` returns other, or f returns other and s returns broader. |
| 8           | Relationship returned by LLM. |

These rules are applied sequentially to determine the final classification based on the outputs from the two branches of the two-way strategy (f and s), ensuring consistent and reasoned assignment of semantic relationships.

This approach allows for robust classification of semantic relationships between research topics, contributing to the development of ontologies in the field of interest.
