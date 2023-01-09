This folder collects all the code libraries produced and/or adopted, during the KDI project to manage data and knowledge resources.
(i.e., Python libs, Shell scripts, Javascript, etc)
</br>
Python script for fetching the data from test.db obtained from CNR-Rome to generate csv file of the patient data.
</br>
```
import pandas as pd
from sqlalchemy import create_engine
from sqlite3 import connect
conn = connect(':memory:')
df = pd.DataFrame(data=[[0, '10/11/12'], [1, '12/11/10']],
                  columns=['int_column', 'date_column'])
df.to_sql('test_data', conn)
pd.read_sql('SELECT int_column, date_column FROM test_data', conn)
import os
os.getcwd()
os.chdir("/Users/sony/Desktop/KGE/KGE_Project") # get the directory path in which the file is located according to your working machine
import sqlite3
from sqlite3 import OperationalError
conn = sqlite3.connect('test.db')
c = conn.cursor()
fd = open('KG_Trento.db.sql', 'r')
sqlFile = fd.read()
fd.close()

# all SQL commands (split on ';')
sqlCommands = sqlFile.split(';')
# Execute every command from the input file
for command in sqlCommands:
    # This will skip and report errors
    # For example, if the tables do not yet exist, this will skip over
    # the DROP TABLE commands
    try:
        c.execute(command)
    except OperationalError as msg:
        print ("Command skipped: ", msg)
 # SQLAlchemy connectable
cnx = create_engine('sqlite:///test.db').connect()
# Create a Pandas Excel writer using XlsxWriter as the engine.
writer = pd.ExcelWriter('rome_cnr.xlsx', engine='xlsxwriter')

for table in ['Patient', 'Physician', 'Encounter','Healthissue','Observation','MedicationAdmin']:
    
    df = pd.read_sql_table(table, cnx) ;
   # df.to_excel("output%s.xlsx" % num,sheet_name="Sheet_1");
   # Write each dataframe to a different worksheet.
    df.to_excel(writer, sheet_name='%s' %table)

# Close the Pandas Excel writer and output the Excel file.
writer.save()
conn.commit()
c.close()

```
Shell script for fetching the top 10 patients from the synthea csv file obtained from 1K Sample Synthetic Patient Records, CSV from website  https://synthea.mitre.org/downloads. 
</br>
1. 01_Patients10.list contains the id of 10 patients randomly taken from patients.csv.
```
while read line; do grep $line patients.csv | cut -f1,2,3,8,9,15 -d "," >> 02_Patients_names.csv; done < 01_Patients10.list 
```
2. Fetching patient's full name, gender, date of birth, date of death, encounter id.
```
 while read line; do grep $line patients.csv | cut -f1,2,3,8,9,15 -d "," >> 02_Patients_names.csv; done < 01_Patients10.list
```
3. Creating the email addresses for each patients.
```
while read line; do grep "$line" patients.csv | cut -f8,9 -d "," | sed "s/[,]/\./g" | sed "s/.*/&@gmail\.com/g" >> 03_patients_email.list ; done < 01_Patients10.list
```
4. Create random phone numbers for the patients. 
```
shuf -i 10000-10000000000 -n 10 | sed "s/^/&+39/g" > 04_Phone_numbers.list
```
5. Create synthetic address of the patients using the website https://www.bestrandoms.com/random-address-in-it.
6. Fetch the encounters for each patient from encounters.csv file
```
while read line; do grep "$line" encounters.csv | cut -f2,4,8,10 -d"," | sed "s/\(T.*Z\)/\,\1/g" | sed -e "s/\,T/\,/1" |  sed -e "s/Z\,/\,/1" >> 05_Encounters.csv; done < 01_Patients10.list
```
We combined the data of 05_Encounters.csv and CNR-Rome as we need only those encounter IDs which map to a specific date in the 05_Encounters.csv for fetching several properties of the patients (such as healthissue, observations, diagnostics and medications of the patient). The file with data combined from 05_Encounters.csv and CNR-Rome patient is encounters_romecnr.csv.

7. Fetch the healthissue of each patient from the file conditions.csv based on specific date of visit present in encounters_romecnr.csv. 
```
while read line ; do date=`cat conditions.csv | grep "$line" | cut -f1,2 -d"," | tr '[,]' '[\n]' | sed '/^[[:space:]]*$/d'` ; issue=`cat conditions.csv | grep "$line" | cut -f1,2,6 -d "," | sed '/^[[:space:]]*$/d'`; myline=`cat encounters_romecnr.csv |  grep -w "$line"` ; newfile=`echo "$myline" | grep -w "$date"` ; echo "$newfile"",""$issue" >> 06_healthissue.csv; done < 01_Patients10.list
``` 
8. Fetch the diagnostics of each patient from the file procedures.csv based on specific date of visit present in encounters_romecnr.csv.
```
while read line ; do date=`cat procedures.csv | grep "$line" | cut -f1 -d"," | sed "s/T[0-9].*Z//g"`; test=`cat procedures.csv | grep "$line" | cut -f1,2,5 -d"," | sed "s/\(T.*Z\)/\,\1/g" | sed -e "s/\,T/\,/1" | sed -e "s/Z\,/\,/1"`; myline=`cat encounters_romecnr.csv |  grep -w "$line" | grep "$date" | cut -f1,2,3,4,6 -d","`; echo -e "$myline\n$test" >> 07_diagnostics.csv; done < 01_Patients10.list
```
9. Fetch the observations of each patient from the file observations.csv based on specific date of visit present in encounters_romecnr.csv.
```
while read line ; do date=`cat observations.csv | grep "$line" | cut -f1 -d"," | sed "s/T[0-9].*Z//g"`; test=`cat observations.csv | grep "$line" | cut -f1,2,5,6,7 -d"," | sed "s/\(T.*Z\)/\,\1/g" | sed -e "s/\,T/\,/1" | sed -e "s/Z\,/\,/1"`; myline=`cat encounters_romecnr.csv |  grep -w "$line" | grep "$date" | cut -f1,2,3,4,6 -d","`; echo -e "$myline\n$test" >> 08_observations.csv; done < 01_Patients10.list
```
10. Fetch the medications of each patient from the file medications.csv based on specific date of visit present in encounters_romecnr.csv
```
while read line ; do date=`cat medications.csv | grep "$line" | cut -f1 -d"," | sed "s/T[0-9].*Z//g"`; test=`cat medications.csv | grep "$line" | cut -f1,5,7,10 -d"," | sed "s/\(T.*Z\)/\,\1/g" | sed -e "s/\,T/\,/1" | sed -e "s/Z\,/\,/1"`; myline=`cat encounters_romecnr.csv |  grep -w "$line" | grep "$date" | cut -f1,2,3,4,6 -d","`; echo -e "$myline\n$test" >> 09_medications.csv; done < 01_Patients10.list
```
R script for renaming of values in the .csv of each dataset such as CityID, HealthCareCenterID, PatientID, PhysicianID, HealthIssueID,  DiagnosticsID, ObservationID, MedicationID, LocalPharmacyID, and NAs removal for data integration in Karma. 
```
library(readxl)

city_data <- read_excel("FINAL_DATSET_KGE.xlsx",sheet = 1)
city_data$ID <- paste0("City", city_data$ID)
View(city_data)
write.csv(city_data,"city_data.csv")

HCC_data <- read_excel("FINAL_DATSET_KGE.xlsx",sheet = 2)
HCC_data$City_Id <- paste0("City", HCC_data$City_Id)
HCC_data$ID <- paste0("HCC", HCC_data$ID)
View(HCC_data)

write.csv(HCC_data,"HCC_data.csv")

patient_data <- read_excel("FINAL_DATSET_KGE.xlsx",sheet = 3)
patient_data <- patient_data[,-1]
patient_data$`First Name` <- gsub("[[:digit:]]", "", patient_data$`First Name`)  
patient_data$`Last name` <- gsub("[[:digit:]]", "", patient_data$`Last name`)  
patient_data$`id-patient_karma` <- paste0("Patient", patient_data$`id-patient_karma`)
View(patient_data)

write.csv(patient_data,"patient_data.csv")

physician_data <- read_excel("FINAL_DATSET_KGE.xlsx",sheet = 4)
physician_data$`id-physician` <- paste0("P", physician_data$`id-physician`)
View(physician_data)
write.csv(physician_data,"physician_data.csv")

encounter_data <- read_excel("FINAL_DATSET_KGE.xlsx",sheet = 5)
encounter_data$`id-patient_karma` <- paste0("Patient", encounter_data$`id-patient_karma`)
encounter_data$`id-physician` <- paste0("P", encounter_data$`id-physician`)
encounter_data[is.na(encounter_data)] <-  " "
encounter_data$HCCname <-  "CNR-Rome"
View(encounter_data)
write.csv(encounter_data,"encounter_data.csv")

healthissue_data <- read_excel("FINAL_DATSET_KGE.xlsx",sheet = 6)
healthissue_data$`id-patient_karma` <- paste0("Patient", healthissue_data$`id-patient_karma`)
healthissue_data$id <- paste0("HI", healthissue_data$id)
View(healthissue_data)
write.csv(healthissue_data,"healthissue_data.csv")

diagnostic_data <- read_excel("FINAL_DATSET_KGE.xlsx",sheet = 7)
diagnostic_data$`id-patient_karma` <- paste0("Patient", diagnostic_data$`id-patient_karma`)
diagnostic_data$Id <-  paste0("Diagnos", diagnostic_data$Id)
View(diagnostic_data)
write.csv(diagnostic_data,"diagnostic_data.csv")

observation_data <- read_excel("FINAL_DATSET_KGE.xlsx",sheet = 8)
observation_data$`id-patient_karma` <- paste0("Patient", observation_data$`id-patient_karma`)
observation_data$id <-  paste0("Obs", observation_data$id)
View(observation_data)
write.csv(observation_data,"observation_data.csv")

medication_data <- read_excel("FINAL_DATSET_KGE.xlsx",sheet = 9)
medication_data$`id-patient_karma` <- paste0("Patient", medication_data$`id-patient_karma`)
medication_data$id <- paste0("Med", medication_data$id)
View(medication_data)
write.csv(medication_data,"medication_data.csv", na = "")

pharmacy_data <- read_excel("FINAL_DATSET_KGE.xlsx",sheet = 10, na = "")
pharmacy_data$ID <- paste0("LP", pharmacy_data$ID)
View(pharmacy_data)
write.csv(pharmacy_data,"pharmacy_data.csv", na = "")

pathology_data <- read_excel("FINAL_DATSET_KGE.xlsx",sheet = 11)
pathology_data$ID <- paste0("Pathology", pathology_data$ID)
View(pathology_data)
write.csv(pathology_data,"pathology_data.csv")

drugDB_data <- read_excel("FINAL_DATSET_KGE.xlsx",sheet = 12)
drugDB_data[is.na(drugDB_data)] <-  ""
View(drugDB_data)
write.csv(drugDB_data,"drugDB_data.csv")
```
SPARQL QUERY CODES FOR COMPETENCY QUESTIONS (CQs)

CQ2. Hospital staff retrieves Andre email address to send him the diagnostic test results.
```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#type>
PREFIX etype:<http://knowdive.disi.unitn.it/etype#>
select * where
{
  ?patientId a etype:Patient_GID-55936.
    optional{?patientId etype:has_PatientFirstName_GID-2_Type-55936 ?firstName.}
    filter(?firstName ="Andre").
  ?patientId etype:has_PatientEmail_GID-105296_Type-55936 ?patientEmail.
}
```
CQ3. Hsiu is diagnosed with obesity and connects with receptionist to schedule an appointment with the clinician for routine follow up.
```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#type>
PREFIX etype:<http://knowdive.disi.unitn.it/etype#>
select * where
{
  ?patientId a etype:Patient_GID-55936.
    optional{?patientId etype:has_PatientFirstName_GID-2_Type-55936 ?firstName.}
    filter(?firstName ="Hsiu")
   ?patientId etype:has_visited_GID-45307_Type-55936 ?encId.
   ?encId etype:has_EncounterDate_GID-80737_Type-6350 ?encDate.
   ?encId etype:has_diagnosedWith_GID-103532_Type-6350 ?heaIssId.
    ?heaIssId etype:has_HealthIssueDescription_GID-74784_Type-74784 ?heaDesc.
    ?heaIssId etype:has_HealthIssueStatus_GID-74023_Type-74784 ?heaStatus
    filter(?heaStatus="active")
}
```
CQ4. Physician wants to refer to the last medication prescribed to Jayson who is diagnosed with Acute bronchitis. 
```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#type>
PREFIX etype:<http://knowdive.disi.unitn.it/etype#>
select ?firstName ?encDate ?heaIssId ?heaDesc ?heaStatus ?MedDesc where
{
  ?patientId a etype:Patient_GID-55936.
    optional{?patientId etype:has_PatientFirstName_GID-2_Type-55936 ?firstName.}
    filter(?firstName ="Jayson")
   ?patientId etype:has_visited_GID-45307_Type-55936 ?encId.
   ?encId etype:has_EncounterDate_GID-80737_Type-6350 ?encDate.
   ?encId etype:has_diagnosedWith_GID-103532_Type-6350 ?heaIssId.
   ?heaIssId etype:has_HealthIssueDescription_GID-74784_Type-74784 ?heaDesc.
    ?heaIssId etype:has_HealthIssueStatus_GID-74023_Type-74784 ?heaStatus.
    filter(?heaStatus="active")
    
    optional{ 
        ?MedId etype:has_prescribedTo_GID-34156_Type-3601 ?encId.
        ?MedId etype:has_MedicationDescription_GID-20520_Type-3601 ?MedDesc.}
}ORDER BY DESC(?encDate) limit 1
```
CQ5. Clinician searches through the list of approved drugs that are available in the local pharmacy for hypertension.
```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#type>
PREFIX etype:<http://knowdive.disi.unitn.it/etype#>
select * where
{
  ?pharmacyid a etype:Local_Pharmacy_GID-17637.
   ?pharmacyid etype:has_treatmentOf_GID-3415 ?disease.
    filter(?disease= <http://localhost:8080/source/Hypertension>)
        ?pharmacyid etype:has_PharmacyDrugName_GID-17633_Type-17637 ?drug.
        ?pharmacyid etype:has_PharmacyDrugAvailability_GID-26190_Type-17637 ?availability.
    optional{ 
        ?pharmacyid  etype:has_conditions_GID-36419_Type-17637 ?drugDB.
        ?drugDB etype:has_DrugAuthorization_GID-28495_Type-35583 ?Approval.
    }        
}
```
CQ6. Physician searches through database for a drug to prescribe to Hsiu who is diagnosed with Coronary Heart Disease and has active Prediabetes comorbidity.
```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#type>
PREFIX etype:<http://knowdive.disi.unitn.it/etype#>
select ?firstName ?encDate ?heaDesc ?heaStatus where
{
  ?patientId a etype:Patient_GID-55936.
    optional{?patientId etype:has_PatientFirstName_GID-2_Type-55936 ?firstName.}
    filter(?firstName ="Hsiu")
   ?patientId etype:has_visited_GID-45307_Type-55936 ?encId.
   ?encId etype:has_EncounterDate_GID-80737_Type-6350 ?encDate.
   ?encId etype:has_diagnosedWith_GID-103532_Type-6350 ?heaIssId.
   ?heaIssId etype:has_HealthIssueDescription_GID-74784_Type-74784 ?heaDesc.
   ?heaIssId etype:has_HealthIssueStatus_GID-74023_Type-74784 ?heaStatus.
   filter(?heaDesc = "Coronary Heart Disease" || ?heaDesc = "Prediabetes")
}Order BY DESC (?encDate) 

select ?drugname ?disease ?conditions where
{
  ?drugid a etype:Drug_Database_GID-35583.
  ?drugid etype:has_treatmentOf_GID-3415 ?disease.
  filter(?disease= <http://localhost:8080/source/Myocardial%20Infarction;%20Acute%20Coronary%20Syndrome;%20Peripheral%20Vascular%20Diseases;%20Stroke>)
  ?drugid etype:has_DrugName_GID-17633_Type-35583 ?drugname.
  ?drugid etype:has_DrugConditions_GID-36419_Type-35583 ?conditions.
}limit 1
```
CQ7. Physician checks if Carvedilol, a drug for Hypertension is available in the Local pharmacy or not.
```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#type>
PREFIX etype:<http://knowdive.disi.unitn.it/etype#>
select * where
{
  ?pharmacyid a etype:Local_Pharmacy_GID-17637.
   ?pharmacyid etype:has_treatmentOf_GID-3415 ?disease.
    filter(?disease= <http://localhost:8080/source/Hypertension>)
   ?pharmacyid etype:has_PharmacyDrugName_GID-17633_Type-17637 ?drug.
   filter(?drug="Carvedilol")
   ?pharmacyid etype:has_PharmacyDrugAvailability_GID-26190_Type-17637 ?availability.
}
```
CQ8. Physician wants to search through drug database for Saxenda prescribed in which comorbidity conditions to avoid side effects of the drug
```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#type>
PREFIX etype:<http://knowdive.disi.unitn.it/etype#>
select ?drugname ?disease ?conditions where
{
  ?drugid a etype:Drug_Database_GID-35583.
  ?drugid etype:has_treatmentOf_GID-3415 ?disease.
  ?drugid etype:has_DrugName_GID-17633_Type-35583 ?drugname.
    filter(?drugname ="Saxenda")
  ?drugid etype:has_DrugConditions_GID-36419_Type-35583 ?conditions.
}
```
CQ9. Test Examiner needs to look for the type of test to conduct for Andre on 2017-02-04. 
```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#type>
PREFIX etype:<http://knowdive.disi.unitn.it/etype#>
select ?firstName ?encDate ?DiagDesc where
{
  ?patientId a etype:Patient_GID-55936.
    optional{?patientId etype:has_PatientFirstName_GID-2_Type-55936 ?firstName.}
    filter(?firstName ="Andre")
   ?patientId etype:has_visited_GID-45307_Type-55936 ?encId.
   ?encId etype:has_EncounterDate_GID-80737_Type-6350 ?encDate.
    filter(?encDate="2017-02-04")
   ?encId etype:has_conduct_GID-103826_Type-6350 ?DiagId.
   ?DiagId etype:has_DiagnosticsDescription_GID-3_Type-99155 ?DiagDesc.
}
```
CQ11. Vennie has Preeclampsia and is referred to the test examiner to conduct Evaluation of uterine fundal height.
```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#type>
PREFIX etype:<http://knowdive.disi.unitn.it/etype#>
select ?firstName ?heaDesc ?heaStatus ?DiagDesc where
{
  ?patientId a etype:Patient_GID-55936.
    optional{?patientId etype:has_PatientFirstName_GID-2_Type-55936 ?firstName.}
    filter(?firstName ="Vennie")
   ?patientId etype:has_hasActive_GID-81706_Type-55936 ?heaId.
    ?heaId etype:has_HealthIssueDescription_GID-74784_Type-74784 ?heaDesc.
    ?heaId etype:has_HealthIssueStatus_GID-74023_Type-74784 ?heaStatus.
    filter(?heaDesc="Preeclampsia" && ?heaStatus="open")
   ?encId etype:has_diagnosedWith_GID-103532_Type-6350 ?heaId.
   ?encId etype:has_conduct_GID-103826_Type-6350 ?DiagId.
   ?DiagId etype:has_DiagnosticsDescription_GID-3_Type-99155 ?DiagDesc.
    filter(?DiagDesc="Evaluation of uterine fundal height")
}
```
CQ12. Clinician looks into the recorded observations of Patient 1 on his visit to MMG on 2019-01-02. 
```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#type>
PREFIX etype:<http://knowdive.disi.unitn.it/etype#>
select * where
{
   ?patientId a etype:Patient_GID-55936.
    filter(?patientId =<http://localhost:8080/source/Patient1>)
   ?patientId etype:has_visited_GID-45307_Type-55936 ?encId.
   ?encId etype:has_EncounterDate_GID-80737_Type-6350 ?encDate.
   filter(?encDate = "2019-01-02")
   ?encId etype:has_EncounterDescription_GID-38695_Type-6350 ?EncDesc.
   ?obsid etype:has_recorded_GID-35630_Type-6350 ?encId.
   ?obsid etype:has_ObservationType_GID-5157 ?obstype.
   ?obsid etype:has_ObservationValue_GID-31912_Type-5157 ?obsvalue.
}
```
CQ13. Pharmacist is looking for an active component of the prescribed drug for the treatment of hypertension. 
```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#type>
PREFIX etype:<http://knowdive.disi.unitn.it/etype#>
select ?drugname ?disease ?activeComponent where
{
  ?drugid a etype:Drug_Database_GID-35583.
  ?drugid etype:has_treatmentOf_GID-3415 ?disease.
    filter(?disease= <http://localhost:8080/source/Hypertension>)
    ?drugid etype:has_DrugName_GID-17633_Type-35583 ?drugname.
  ?drugid etype:has_DrugActiveIngredient_GID-19510_Type-35583 ?activeComponent.
}limit 5
```
CQ14. Pharmacist searches through database if Glidipion, a drug for Diabetes Mellitus Type 2 still available in the market or withdrawn. 
```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#type>
PREFIX etype:<http://knowdive.disi.unitn.it/etype#>
select * where
{
  ?drugid a etype:Drug_Database_GID-35583.
  ?drugid etype:has_treatmentOf_GID-3415 ?disease.
    filter(?disease= <http://localhost:8080/source/Diabetes%20Mellitus%2C%20Type%202>)
    ?drugid etype:has_DrugName_GID-17633_Type-35583 ?drugname.
    filter(?drugname="Glidipion (previously Pioglitazone Actavis Group)")
  ?drugid etype:has_DrugAuthorization_GID-28495_Type-35583 ?authorization.
}
```
CQ15. Pharmacist wants to check the dosage amount of blopress prescribed to patient ID-1. 
```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#type>
PREFIX etype:<http://knowdive.disi.unitn.it/etype#>
select * where
{
   ?patientId a etype:Patient_GID-55936.
    filter(?patientId =<http://localhost:8080/source/Patient1>)
   ?patientId etype:has_visited_GID-45307_Type-55936 ?encId.
   ?encId etype:has_EncounterDate_GID-80737_Type-6350 ?encDate.
   ?MedId etype:has_prescribedTo_GID-34156_Type-3601 ?encId.
   ?MedId etype:has_MedicationDescription_GID-20520_Type-3601 ?MedDesc.
    filter(?MedDesc ="blopress")
   ?MedId etype:has_MedicationDosage_GID-73216_Type-3601 ?MedDosage.
} ORDER BY DESC(?encDate)
```
CQ16. Pharmacist in Paris city is looking for an alternative of Tranquillante prescribed to an Italian citizen who is diagnosed with hypertension.
```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#type>
PREFIX etype:<http://knowdive.disi.unitn.it/etype#>
select ?disease ?drugname ?alternateName ?activeIngredient where
{
  ?drugid a etype:Drug_Database_GID-35583.
  ?drugid etype:has_treatmentOf_GID-3415 ?disease.
    filter(?disease= <http://localhost:8080/source/Hypertension>)
    ?drugid etype:has_DrugName_GID-17633_Type-35583 ?drugname.
       optional{filter(?drugname="tranquillante")}
    ?drugid etype:has_DrugActiveIngredient_GID-19510_Type-35583 ?activeIngredient.
    ?drugid etype:has_Drug_alternateName_GID-31595_Type-35583 ?alternateName.
} limit 5
```
CQ17. Clyde visited Rome CNR on 2009-07-18 to undergo a test and wants to know what healthissue is she diagnosed with.
```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#type>
PREFIX etype:<http://knowdive.disi.unitn.it/etype#>
select * where
{
  ?patientId a etype:Patient_GID-55936.
    optional{?patientId etype:has_PatientFirstName_GID-2_Type-55936 ?firstName.}
    filter(?firstName ="Clyde")
   ?patientId etype:has_visited_GID-45307_Type-55936 ?encId.
   ?encId etype:has_EncounterDate_GID-80737_Type-6350 ?encDate.
    filter(?encDate = "2009-07-18")
    ?encId etype:has_diagnosedWith_GID-103532_Type-6350 ?heaIssId.
    ?heaIssId etype:has_HealthIssueDescription_GID-74784_Type-74784 ?heaDesc.
}
``` 
CQ19. Tia who diagnosed with Idiopathic atrophic hypothyroidism records her BP level observations.
``` 
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#type>
PREFIX etype:<http://knowdive.disi.unitn.it/etype#>
Select * where
{
  ?patientId a etype:Patient_GID-55936.
    optional{?patientId etype:has_PatientFirstName_GID-2_Type-55936 ?firstName.}
    filter(?firstName ="Tia")
   ?patientId etype:has_hasActive_GID-81706_Type-55936 ?heaId.
    ?heaId etype:has_HealthIssueDescription_GID-74784_Type-74784 ?heaDesc.
    ?heaId etype:has_HealthIssueStatus_GID-74023_Type-74784 ?heaStatus.
    filter(?heaDesc="Idiopathic atrophic hypothyroidism")
    ?encId etype:has_diagnosedWith_GID-103532_Type-6350 ?heaId.
    optional{?obsId etype:has_recorded_GID-35630_Type-6350 ?encId.
    ?obsId etype:has_ObservationType_GID-5157 ?obstype.
        ?obsId etype:has_ObservationValue_GID-31912_Type-5157 ?obsvalue.}
} 
``` 
—---for different health issue not in CQ19—---------
``` 
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#type>
PREFIX etype:<http://knowdive.disi.unitn.it/etype#>
Select * where
{
  ?patientId a etype:Patient_GID-55936.
    optional{?patientId etype:has_PatientFirstName_GID-2_Type-55936 ?firstName.}
    filter(?firstName ="Tia")
   ?patientId etype:has_hasActive_GID-81706_Type-55936 ?heaId.
    ?heaId etype:has_HealthIssueDescription_GID-74784_Type-74784 ?heaDesc.
    ?heaId etype:has_HealthIssueStatus_GID-74023_Type-74784 ?heaStatus.
    filter(?heaDesc="Streptococcal sore throat (disorder)")
    ?encId etype:has_diagnosedWith_GID-103532_Type-6350 ?heaId.
    optional{?obsId etype:has_recorded_GID-35630_Type-6350 ?encId.
    ?obsId etype:has_ObservationType_GID-5157 ?obstype.
        ?obsId etype:has_ObservationValue_GID-31912_Type-5157 ?obsvalue.}
}
``` 
CQ22. Patient 1 records routine health observations and wants to track the whole record.
```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#type>
PREFIX etype:<http://knowdive.disi.unitn.it/etype#>
select ?encDate ?obstype ?obsvalue where
{
   ?patientId a etype:Patient_GID-55936.
    filter(?patientId =<http://localhost:8080/source/Patient1>)
   ?patientId etype:has_record_GID-35578_Type-55936 ?obsId.
   ?obsId etype:has_ObservationType_GID-5157 ?obstype.
   ?obsId etype:has_ObservationValue_GID-31912_Type-5157 ?obsvalue.
   ?obsId etype:has_recorded_GID-35630_Type-6350 ?encId.
   ?encId etype:has_EncounterDate_GID-80737_Type-6350 ?encDate.
}ORDER BY DESC (?encDate) limit 10
```
