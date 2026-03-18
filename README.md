Below is a clean one-page handout style summary you could bring to the discussion (or use as your own notes). It focuses on the problem, proposed architecture, and practical implementation, without exposing your internal repo or prompts. The tone is neutral and collaborative so it won’t sound like you’re proposing to replace their system—just helping refine it.

Evidence-Aligned CSR Generation

Conceptual Architecture for Reliable AI-Assisted CSR Development

Problem
Current CSR automation attempts appear to rely heavily on:

CSR Template + Guidance → LLM Generation

However, CSR templates are intentionally sparse and flexible, because section content varies based on:

study phase

indication

compound

endpoints

population

statistical methods

As a result, the model lacks sufficient source evidence context and may produce:

incomplete sections

hallucinated interpretations

unsupported statements

inconsistent numerical reporting

The core challenge is not necessarily model capability but alignment between CSR sections and their primary evidence sources.

Key Principle
In regulatory writing:

Every CSR statement should be traceable to a primary study document or dataset.

Therefore, automated generation should follow the same rule.

Instead of treating the template as the main source of content, each section should be generated from the correct underlying evidence source.

Evidence-Aligned Generation Workflow
CSR Template (section structure) ↓ Identify CSR section ↓ Evidence Mapping Layer ↓ Retrieve correct source documents ↓ Section-specific generation ↓ Verification against source evidence ↓ Human MW review

The template acts as the output scaffold, while source documents provide the content authority.

Example Evidence Mapping
CSR Section Primary Evidence Source

Study objectives Protocol (CSP) Study design Protocol (CSP) Inclusion/exclusion criteria Protocol (CSP) Statistical methods SAP Subject disposition TLFs / subject listings Baseline characteristics TLFs Efficacy results efficacy TLFs Safety results safety TLFs PK results PK TLFs PD / biomarkers PD / biomarker TLFs Protocol deviations study conduct documentation

Each section retrieves the appropriate evidence source before generation.

Role of Previously Approved CSRs
Approved CSRs are valuable but should be used as:

Style and structural examples

Examples of:

phrasing patterns

section composition

narrative structure

They should not be treated as verification sources, because study-specific variables change across trials.

Generation Architecture
A robust system introduces three distinct layers:

Evidence Routing
Determine which source documents contain the authoritative information for each CSR section.

Section-Specific Generation
Use targeted prompts for each section type (e.g., study design, efficacy results, safety summary).

Verification
Check generated text against the same source evidence.

Examples of verification:

numerical fidelity checks for TLF-derived text

semantic alignment checks for protocol/SAP-derived sections

citation support checks for interpretive sections

Example Workflow (Efficacy Results Section)
CSR Section: Efficacy Results ↓ Retrieve efficacy TLFs ↓ Generate narrative summary ↓ Verify numerical consistency with TLFs ↓ Human MW review

Example Workflow (Study Design Section)
CSR Section: Study Design ↓ Retrieve relevant CSP sections ↓ Generate narrative description ↓ Verify alignment with protocol text ↓ Human MW review

Benefits of Evidence-Aligned Generation
This architecture helps address key challenges in automated CSR drafting:

reduces hallucination risk

improves traceability to study documentation

ensures numerical accuracy

supports modular generation of individual sections

aligns with regulatory writing practices

Conceptual Summary
Template defines structure Evidence defines content Model generates draft Verification ensures fidelity Human review finalizes narrative

One-sentence explanation for discussion

The template provides the structural scaffold for the CSR, but the actual section content should be generated from the appropriate underlying evidence source and verified against that same source.# Mit-const
