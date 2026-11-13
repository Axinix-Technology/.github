# AR.I.S — Internal Tools Overview

> **Document Reference:** AXN-DOCS-03-INT-001  
> **Last Updated:** 2026-11-11  
> **Repository:** `d:\Axinix Projects\AR.I.S`  
> **Implementation Status:** EARLY DEVELOPMENT  
> **Authored By:** Axinix Technologies Documentation System

---

## 1. Overview

**AR.I.S** (Artificial Intelligent System) is Axinix Technologies' proprietary neural AI substrate. It is a cognitive AI system being developed to support autonomous task reasoning, AST codebase inspections, and intelligent decision engines intended to eventually power all Axinix products.

According to the corporate portfolio, AR.I.S is classified as:
> *"Neural tensor and cognitive linguistic architecture supporting autonomous task reasoning, AST codebase inspections, and intelligent decision engines across Axinix products."*

**Current role:** Internal AI research project — not yet integrated into any production Axinix product.

---

## 2. Architecture

The AR.I.S system is organized as a Python package (`aris/`) with the following subsystems:

| Subsystem | Directory | Purpose | Status |
|---|---|---|---|
| **Core Engine** | `aris/core/` | Central cognitive processing | ✅ IMPLEMENTED |
| **Configuration** | `aris/configs/` | Configuration management | ✅ IMPLEMENTED |
| **Data Ingestion** | `aris/data/` | Dataset loading and preprocessing | ✅ IMPLEMENTED |
| **Evaluation** | `aris/evaluation/` | Model evaluation suite | ✅ IMPLEMENTED |
| **Inference** | `aris/inference/` | Inference engine | ✅ IMPLEMENTED |
| **Cognitive Linguistic Frame** | `aris/linguistic/` | CognitiveLinguisticFrame — neural tensor substrate for all linguistic decisions | ✅ IMPLEMENTED |
| **Memory System** | `aris/memory/` | Memory architecture | ✅ IMPLEMENTED |
| **Neural Models** | `aris/models/` | Model definitions | ✅ IMPLEMENTED |
| **Custom Tokenizer** | `aris/tokenizer/` | Custom tokenizer training and inference | ✅ IMPLEMENTED |
| **Training Pipeline** | `aris/training/` | Multi-corpus training pipeline | ✅ IMPLEMENTED |

---

## 3. Training Datasets

Multi-corpus training across 6 cognitive domains:

| Dataset | Category | Location | Purpose |
|---|---|---|---|
| **GSM8K** | Mathematical reasoning | `data/gsm8k/` | Grade school math problem solving |
| **MBPP** | Code generation | `data/mbpp/` | Python programming problem generation |
| **HotpotQA** | Multi-hop reasoning | `data/hotpotqa/` | Multi-step question answering |
| **qasrl_gs** | Linguistic / SRL | `data/qasrl_gs/` | Semantic role labeling, linguistic structure |
| **UltraChat 200k** | Conversation | `data/ultrachat_200k/` | Conversational AI training |
| **Vision** | Computer vision | `data/vision/` | Visual understanding |

---

## 4. Technology Stack

| Library | Version | Purpose |
|---|---|---|
| **PyTorch** | `>=2.0.0` | Neural network framework |
| **HuggingFace Tokenizers** | `>=0.20.0` | Tokenizer training and inference |
| **HuggingFace Datasets** | `>=2.14.0` | Dataset loading |
| **HuggingFace Hub** | `>=0.20.0` | Model and dataset downloads |
| **NumPy** | `>=1.24.0` | Numerical computing |
| **PyArrow** | `>=14.0.0` | Columnar data processing |
| **PyMySQL** | `>=1.1.0` | Training metadata storage |
| **tqdm** | `>=4.66.0` | Progress display |

---

## 5. CognitiveLinguisticFrame

> [!IMPORTANT]
> The `CognitiveLinguisticFrame` in `aris/linguistic/` is designated as the **neural tensor substrate for all linguistic decisions** across Axinix systems. Per engineering rules, no linguistic decision (such as intent detection, question classification, or text analysis) should use hardcoded string checks (`endswith("?")`, keyword lists) — all such decisions must route through the CognitiveLinguisticFrame.

This is the primary integration bridge between AR.I.S and future Workhub/E-commerce features.

---

## 6. Future Product Integration

| Integration | Target Product | Branch | Status |
|---|---|---|---|
| **Jarvis AI Assistant** | Workhub | `feat/jarvis-ai-assistant-6a898bc38f9330ae4f170153` | IN DEVELOPMENT |
| **AI Daily Work Status** | Workhub (DailyTracker) | Referenced in knowledge_brain | PLANNED |
| **AI-powered features** | E-Commerce | No branch detected | PROPOSED |

---

## 7. Git History

```
6f8276e feat: 7-pillar cognitive dataset expansion (GSM8K, MBPP, Vision)
a12baac chore(git): explicitly whitelist and prevent aris/models/ from being ignored
0829664 feat: modernize AR.I.S architecture with multi-corpus neural engine, alphabetical knowledge brain, and unified runtime
53a0361 feat: initial codebase setup and cognitive pipeline architecture
3b6bdea Initial commit
```

---

## 8. Source File References

| Component | File |
|---|---|
| Main entry point | [`run.py`](file:///d:/Axinix%20Projects/AR.I.S/run.py) |
| Requirements | [`requirements.txt`](file:///d:/Axinix%20Projects/AR.I.S/requirements.txt) |
| Environment template | [`.env-example`](file:///d:/Axinix%20Projects/AR.I.S/.env-example) |
| Knowledge Brain Index | [`knowledge_brain/`](file:///d:/Axinix%20Projects/AR.I.S/knowledge_brain/) (alphabetically organized A-V) |

---

## Related Documents

- [../../../00-discovery/PRODUCT-INVENTORY.md](../../00-discovery/PRODUCT-INVENTORY.md)
- [../../../05-architecture/ECOSYSTEM-ARCHITECTURE.md](../../05-architecture/ECOSYSTEM-ARCHITECTURE.md)

