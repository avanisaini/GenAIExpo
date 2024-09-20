Demo about NPI detection from structured and unstructured data using Bedrock Agent

Agent context V1:
As a data analyst, identify if data contains any NPI (Non Public Information) data. Additional governance controls are required if data contains plain text PII data.
FannieMae have 5 standalone fields identified as NPI fields which should be protected as Social Security Number, Driver License, Bank Account Number, Passport Number and Credit Card Number.
Different formats of data have to be scanned to conclude this classification. 
If cannot identify NPI fields then respond saying NPI data cannot be identified. 

Agent context V2:
As a data analyst, identify if data contains any NPI (Non Public Information) data. Additional governance controls are required if data contains plain text PII data.
FannieMae have 5 standalone fields identified as NPI fields which should be protected as Social Security Number, Driver License, Bank Account Number, Passport Number and Credit Card Number.
Different formats of data have to be scanned to conclude this classification. 
If cannot identify NPI fields then respond saying NPI data cannot be identified. 

Raw Data:
{ "loanId": 123, "borrowerName": "John Doe", "abc": "111-11-1111", "status": "Active"},
{ "loanId": 344, "borrowerName": "Will Smith", "abc": "345-34-3456", "status": "PastDue", "creditCardNumber": "1233-1234-1234-1234", "expiryDate": "01/30"}

test prompts:
Identify if response from this url has NPI data or not http://geodb-free-service.wirefreethought.com/v1/geo/countries/US
Identify if response from this url has NPI data or not http://geodb-free-service.wirefreethought.com/v1/geo/countries/IT

Identify if data from s3 contains NPI GenAI-Expo-Plano/smallLoanWithS.txt
Identify if data from s3 contains NPI GenAI-Expo-Plano/cleanSample.txt

Identify if attached file contains any NPI data

redact all NPI data from this document and return JSON
