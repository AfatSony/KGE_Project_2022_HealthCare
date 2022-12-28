This folder collects all the code libraries produced and/or adopted, during the KDI project to manage data and knowledge resources.
(i.e., Python libs, Shell scripts, Javascript, etc)
</br>
Python script for fetching the data from test.db obtained from CNR-Rome to generate csv file of the patient data.
</br>
Shell script for fetching the top 10 patients from the synthea csv file obtained from 1K Sample Synthetic Patient Records, CSV from website  https://synthea.mitre.org/downloads. 
</br>
1. 01_Patients10.list contains the id of 10 patients randomly taken from patients.csv.
```
while read line; do grep $line patients.csv | cut -f1,2,3,8,9,15 -d "," >> 02_Patients_names.csv; done < 01_Patients10.list 
```
2. Creating the phone numbers for each patients.
```
while read line; do grep "$line" patients.csv | cut -f8,9 -d "," | sed "s/[,]/\./g" | sed "s/.*/&@gmail\.com/g" >> 03_patients_email.list ; done < 01_Patients10.list
```
3. Create random phone numbers for the patients. 
```
shuf -i 10000-10000000000 -n 10 | sed "s/^/&+39/g" > 04_Phone_numbers.list
```
4. Create synthetic address of the patients using the website https://www.bestrandoms.com/random-address-in-it.
5. Fetch the encounters for each patient from encounters.csv file
```
while read line; do grep "$line" encounters.csv | cut -f2,4,8,10 -d"," | sed "s/\(T.*Z\)/\,\1/g" | sed -e "s/\,T/\,/1" |  sed -e "s/Z\,/\,/1" >> 05_Encounters.csv; done < 01_Patients10.list
```
We combined the data of 05_Encounters.csv and CNR-Rome as we need only those encounter IDs which map to a specific date in the 05_Encounters.csv for fetching several properties of the patients (such as healthissue, observations, diagnostics and medications of the patient). The file with data combined from 05_Encounters.csv and CNR-Rome patient is encounters_romecnr.csv.

6. Fetch the healthissue of each patient from the file conditions.csv based on specific date of visit present in encounters_romecnr.csv. 
```
while read line ; do date=`cat conditions.csv | grep "$line" | cut -f1,2 -d"," | tr '[,]' '[\n]' | sed '/^[[:space:]]*$/d'` ; issue=`cat conditions.csv | grep "$line" | cut -f1,2,6 -d "," | sed '/^[[:space:]]*$/d'`; myline=`cat encounters_romecnr.csv |  grep -w "$line"` ; newfile=`echo "$myline" | grep -w "$date"` ; echo "$newfile"",""$issue" >> 06_healthissue.csv; done < 01_Patients10.list
``` 
7. Fetch the diagnostics of each patient from the file procedures.csv based on specific date of visit present in encounters_romecnr.csv.
```
while read line ; do date=`cat procedures.csv | grep "$line" | cut -f1 -d"," | sed "s/T[0-9].*Z//g"`; test=`cat procedures.csv | grep "$line" | cut -f1,2,5 -d"," | sed "s/\(T.*Z\)/\,\1/g" | sed -e "s/\,T/\,/1" | sed -e "s/Z\,/\,/1"`; myline=`cat encounters_romecnr.csv |  grep -w "$line" | grep "$date" | cut -f1,2,3,4,6 -d","`; echo -e "$myline\n$test" >> 07_diagnostics.csv; done < 01_Patients10.list
```
8. Fetch the observations of each patient from the file observations.csv based on specific date of visit present in encounters_romecnr.csv.
```
while read line ; do date=`cat observations.csv | grep "$line" | cut -f1 -d"," | sed "s/T[0-9].*Z//g"`; test=`cat observations.csv | grep "$line" | cut -f1,2,5,6,7 -d"," | sed "s/\(T.*Z\)/\,\1/g" | sed -e "s/\,T/\,/1" | sed -e "s/Z\,/\,/1"`; myline=`cat encounters_romecnr.csv |  grep -w "$line" | grep "$date" | cut -f1,2,3,4,6 -d","`; echo -e "$myline\n$test" >> 08_observations.csv; done < 01_Patients10.list
```
9. Fetch the medications of each patient from the file medications.csv based on specific date of visit present in encounters_romecnr.csv
```
while read line ; do date=`cat medications.csv | grep "$line" | cut -f1 -d"," | sed "s/T[0-9].*Z//g"`; test=`cat medications.csv | grep "$line" | cut -f1,5,7,10 -d"," | sed "s/\(T.*Z\)/\,\1/g" | sed -e "s/\,T/\,/1" | sed -e "s/Z\,/\,/1"`; myline=`cat encounters_romecnr.csv |  grep -w "$line" | grep "$date" | cut -f1,2,3,4,6 -d","`; echo -e "$myline\n$test" >> 09_medications.csv; done < 01_Patients10.list
```

Renaming of values in the .csv of each dataset such as CityID, HealthCareCenterID, PatientID, PhysicianID, HealthIssueID,  DiagnosticsID, ObservationID, MedicationID, LocalPharmacyID, and NAs removal for data integration in Karma. 
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
