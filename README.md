## Health Care Centre: RomeCNR
This repository is for KGE-Project developed during the KGE2022-23 course.

This project deals with patients data from Rome CNR and synthetic data from Synthea to integrate to help make a Knowledge Graph (KG).

- Such KGE might help how several entities involved in healthcare sector (the domain of the knowledge graph) and service such as patients, physicians, pharmacies access necessary information such as past disease history of patient, retrieve reports of patients, major diagnostics conducted, medications administered to patients, and allow hospital management manage patients based on severity of the their disease condition.
- Such KG could even help a given hospital share patient's information to other hospital in case the patient transfers to a new address or location into a new city/same city and needs urgent treatment for his disease condition (only after patient's consent to transfer his medical data). It can also help patient to get treated in another  superspeciality hospital (on a priority basis) far from his current address if his current hospital lacks services/equipments or medical professionals to deal with his urgent medical condition.
- Such KG could help the new hospital take necessary steps towards patient's fast treament without losing time. 


For this project we used iTelos Methodology and the phases are enlisted in order below:
1. Inception
2. Informal Modeling
3. Formal Modeling

The formal modelling consists of a well defined Entity Relationship (ER) model: 

![ER diagram](https://github.com/AfatSony/KGE_Project_2022_HealthCare/blob/main/ER%20Diagram_new.png)

4. Data Integration

Folders in the repository:
## Code Libraries: 

All the codes related to the project are present in the given hyperlink. The codes were written in python, bash scripts and R and can be found in the link below:
<https://github.com/AfatSony/KGE_Project_2022_HealthCare/tree/main/Code%20Libraries>
## Datasets:

The datasets used in the inception, informal modelling, formal modelling and data integration phases can be found in the given link:
<https://github.com/AfatSony/KGE_Project_2022_HealthCare/tree/main/Datasets>
## Documentation:

The project report and the final presentation can be found under this link:
<https://github.com/AfatSony/KGE_Project_2022_HealthCare/tree/main/Documentation>
## Teleologies:

The teleologies generated in each of the phases: inception, informal modelling, formal modelling and data integration, can be found in the given link:
<https://github.com/AfatSony/KGE_Project_2022_HealthCare/tree/main/Teleologies>



The knowledge graph creation was done in GraphDB and the querying of the data was done using GraphDB inbuilt SparQL. One such instance of the knowledge graph for understanding the patient's medication is shown in the figure below 
![GraphDb] (https://github.com/AfatSony/KGE_Project_2022_HealthCare/blob/main/medication.png)
