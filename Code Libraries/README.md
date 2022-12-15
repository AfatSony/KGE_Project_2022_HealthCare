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
8. 
