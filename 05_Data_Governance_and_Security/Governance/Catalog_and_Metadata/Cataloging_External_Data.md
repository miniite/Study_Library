---
id: "05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Cataloging_External_Data"
level: 5
type: "concept"
topic: "05_Data_Governance_and_Security/Governance/Catalog_and_Metadata"
tools: ["AWS"]
---

# Cataloging External Data

**L5 Expert** · [↑ Catalog and Metadata](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Deep] [An interviewer probe: you said you'd auto-tag PII. What happens when your classifier tags a hashed/tokenized column as PII and blocks a legitimate pipeline? How does your design avoid that failure mode?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11144_An_interviewer_probe_you_said_you_d_auto-tag_PII_What_happen.md>)
- [Deep] [Design how you'd expose lineage and metadata to non-technical stakeholders (auditors, compliance, execs) versus engineers. What's the right abstraction and UI so each audience gets a trustworthy, non-overwhelming view?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11170_Design_how_you_d_expose_lineage_and_metadata_to_non-technica.md>)
- [Deep] [Ownership metadata rots the fastest — people leave, teams reorg. Design a system that keeps 'who owns this dataset' trustworthy over years without constant manual upkeep.](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11196_Ownership_metadata_rots_the_fastest_people_leave_teams_reorg.md>)
- [Expert] [You want to bring LLMs into the catalog — auto-generating descriptions, answering 'which table for X', semantic search. Where do LLMs genuinely help, where do they create governance risk, and how do you keep hallucinated metadata out of the system of record?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11148_You_want_to_bring_LLMs_into_the_catalog_auto-generating_desc.md>)
- [Expert] [How do you support 'data products' as a first-class catalog concept in a mesh — bundling schema, SLAs, owner, quality, docs, and access — and what metadata does a consumer need to trust and depend on a product?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11165_How_do_you_support_data_products_as_a_first-class_catalog_co.md>)
- [Expert] [You are consolidating five siloed catalogs (Glue, Collibra, Alation, an internal wiki, and Confluence pages) into one org-wide metadata platform. Walk me through your target-state architecture and how you decide what becomes the system of record for each metadata type.](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11187_You_are_consolidating_five_siloed_catalogs_Glue_Collibra_Ala.md>)
- [Expert] [Compare OpenMetadata, DataHub, Amundsen, and a commercial catalog on their metadata models and extensibility. If you were standardizing across an org, which would you pick and why — or would you build?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11201_Compare_OpenMetadata_DataHub_Amundsen_and_a_commercial_catal.md>)
- [Expert] [How do you catalog and govern derived/semantic-layer assets (metrics, dimensions defined in a metrics layer)? Should the metric definition itself be a first-class cataloged entity, and how does that relate to physical lineage?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11220_How_do_you_catalog_and_govern_derived_semantic-layer_assets_.md>)

## Related concepts

- [Glossary Metadata and Contracts](<../Lineage/Glossary_Metadata_and_Contracts.md>) *(Lineage)*
- [Governing Real-time Features](<../Stewardship/Governing_Real-time_Features.md>) *(Stewardship)*
- [Platform-level Data Quality](<../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Platform-level_Data_Quality.md>) *(Data Platform Strategy)*

## References

- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Give Data Products a Frontend with Latent Documentation
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Metadata Service(s) as a Core Component of the Data Platform
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Modern Metadata for the Modern Data Stack
- *Building Data Integration Solutions* — Jay Borthen — Key Concepts in Data Integration
- *Data Engineering Best Practices* — Richard J. Schiller, David Larochelle — Summary
- *Data Engineering Best Practices* — Richard J. Schiller, David Larochelle — Metadata strategy
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Metadata Management, Data Flow, and Lineage
- *Designing Financial Data Architectures* — Tamer Khraisha — Financial Data Architectures

