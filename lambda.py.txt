import json
import datetime
import subprocess
import sys
import boto3
from io import StringIO

subprocess.call('pip install requests -t /tmp/ --no-cache-dir'.split(), stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
sys.path.insert(1, '/tmp/')

import requests

def lambda_handler(event, context):
    
    #example prompts: 
        #Identify if datasetID=DSET334 contains NPI fields or not
        #Identify if this data contains NPI or not "SSN=111-11-1111 is past due"
        #Identify if NPI exists in the attached file
    
    print("Received event: " + json.dumps(event))
    
    agent = event['agent']
    actionGroup = event['actionGroup']
    if event.get('apiPath'):
        apiPath = event.get('apiPath')
        print("found apiPath: " + apiPath)
    else:
        apiPath = ""
        
    if event.get('function'):
        function = event['function']
        print("found function: " + function)
    else:
        function = ""
        
    if event.get('parameters'):
        parameters = event['parameters']
        print("found parameter: ")
        print(parameters)
    else:
        parameters = []

    print("processing function or apiPath")
    print(function + " " + apiPath)
    
    if apiPath == "/get_data_from_API":
        print("in get_api")
        
        parameter1Val = parameters[0]['value']
        print("parameter1Val: " + parameter1Val)
        if (parameter1Val != "https://api.example.com/data"):
        
            USAurl="http://geodb-free-service.wirefreethought.com/v1/geo/countries/US"
            weburl = parameter1Val
            print("calling requests")
            result_text = requests.get(weburl).text
            print("result_text:" + result_text)
        
            if weburl == USAurl:
                result_text = result_text.replace("callingCode","SSN").replace("+1", "111-11-1111")
        else:
            result_text = ""
            
        print("result_text: " + result_text)
        
    elif apiPath == "/get_data_from_s3":
        print("in get_s3")
        bucketName= "expo-npi-detection"
        parameter1Val = parameters[0]['value']
        
        inputText = event['inputText']
        print(inputText)
        
        try:
            print("calling boto3 " + inputText.split("s3")[1])
            
            s3Client = boto3.client('s3')
            
            print("getting data from bucket: " + bucketName + " key: " + parameter1Val)
            object = s3Client.get_object(Bucket=bucketName, Key=parameter1Val)
            print("got object: ")
            print(object)
            
            body = object['Body']
            print("got body: ")
            print(body)

            result_text = body.read().decode('utf-8')
            print("object text:")
            print(result_text)

        except Exception as ex:
            print("in exception")
            print(ex)
            result_text=""

        
    elif function == 'npi_from_dataset':
        print("parameters: " + json.dumps(parameters))

        if ((len(parameters) > 0) and (parameters[0].get('value'))):
                datasetId = parameters[0]['value']
                print("parameter1: " + datasetId)
                
                #identify if dataset registered as NPI
                #if not NPI, get sample data
                response = {
                    "loanid": "string",
                    "ssn": "string",
                    "status": "approved"
                }
                
                if (datasetId == "DSET123"):
                    result_text = "Successfully identified that data contains NPI. NPI fields are: Credit Card Number, SSN. datasetID is " + datasetId + "."
                else:
                    result_text = "Successfully identified that data does not contain NPI. datasetID is " + datasetId + "."
        else:
            result_text = "Need datasetID as a parameter."
        
    else:
        result_text = "function npi_from_dataset was called successfully"
        
    print("responding with: " + result_text)

    responseBody = {
        "TEXT": {
            "body": result_text
        }
    }
    
    if function != "":
        action_response = {
            "actionGroup": actionGroup,
            "function": function,
            "functionResponse": {
                "responseBody": responseBody
            }
        }
    else:
        action_response = {
            "actionGroup": actionGroup,
            "apiPath": apiPath,
            "httpMethod": event['httpMethod'],
            "httpStatusCode": 200,
            "responseBody": responseBody

        }
    
    functionResponse = {
        'response': action_response,
        'messageVersion': "1.0"
    }
    
    return functionResponse
