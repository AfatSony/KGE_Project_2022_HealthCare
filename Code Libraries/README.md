This folder collects all the code libraries produced and/or adopted, during the KDI project to manage data and knowledge resources.
(i.e., Python libs, Shell scripts, Javascript, etc)
</br>
**Shell script for fetching the top 10 patients from the synthea csv file obtained from 1K Sample Synthetic Patient Records, CSV from website  https://synthea.mitre.org/downloads**
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
