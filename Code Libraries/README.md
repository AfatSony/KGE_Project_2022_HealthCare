This folder collects all the code libraries produced and/or adopted, during the KDI project to manage dta and knowledge resources.
(i.e., Pyhton libs, Shell scripts, Javascript, etc)
# shell script for fetching the top 10 patients from the synthea csv file obtained from 1K Sample Synthetic Patient Records, CSV from website #https://synthea.mitre.org/downloads
## 01_Patients10.list contains the id of 10 patients randomly taken from patients.csv.
while read line; do grep $line patients.csv | cut -f1,2,3,8,9,15 -d "," >> 02_Patients_names.csv; done < 01_Patients10.list 
