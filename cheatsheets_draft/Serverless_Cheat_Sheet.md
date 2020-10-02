# Serverless Cheat Sheet

## Introduction
Serverless refers to the model where your code is deployed dynamically as needed, and charged based on usage.  There is no server that is running continually, when a trigger occur the a server is spun up, performs its function and spins down.  The most common examples are AWS lambda and Azure functions.  Many aspects of secure coding do not change with serverless.  This guide focuses on the differences between serverless and traditional coding.

This cheat sheet provides guidance on how to code and deploy serverless functions in a secure manner.

## Contents

## What Does a Malicious Actor Want

In serverless the malcious actor only has a few interests.  The malicious actor will want to read your environmental variables, read your code, read anything left in your temporary storage, trigger your serverless function with malicious data, or just trigger your serverless function maliciously to cause denial of service or denail of wallet attacks. 

## Triggers

Triggers are events that start the serverless function.  The possible triggers vary on AWS lambdas and Azure functions but some possible triggers are emails, http requests, timers, logs, files, mqtt, object creation / database insertion, etc.  The trigger will generally provide data to the serverless function that is used in processing.  Valid the input data from the trigger, and validate the integrity of the trigger.

### Validate the integrity of the trigger

This will validate the trigger is from the correct source.  This should be done before any processing of the data.  The best way to validate the trigger is to use cryptographic hashing function to sign the trigger, and have the function verify the hash.  Let's look at a few examples.

#### Email as Trigger
If an email with a specific format is the trigger, a malicious actor could send an email with the correct format to trigger the event.  Spoofing email address is fairly easy so the sender email address does not guarentee integrity.  In this case, the email could be signed using PGP and the serverless function could verify the signature before any processing.

#### HTTP Request as Trigger

In this case the malicious actor could use a tool such burpsuite prozy or OWASP Zap to grab a request and modify it.  One way to add an integrity check would be to put the data inside a JWT and use the built in integrity checking.  Please check OWASP Cheat Sheet on JWTs for more details if necessary.  Briefly, always validate the JWT before using the data in the payload, do not accept any JWTs where algorithm is none.

### Validate the input from trigger

Any data used in the trigger should be validated using the same techniques generally used for input validation.  Verify the input makes sense.  Check for nulls, minimumum length, maximum length, type, size if a file or object, etc.  Please see input validation cheat sheet for more.


## Temporary Storage

When your serverless function is triggered a new server is not always spun up.  A server may stay frozen waiting to be reactivated by another triggering event.  As a defense in depth, delete any data in temproary storage at the end of your serverless function.  

## Logging

Always log trigger integrity check failue, input validation failures.  Make sure to have a maximumm log size, encode to stop XSS, and replace carriage returns to stop log forgeries.  Logging should always send the logs to centralized logging service for monitoring.  Monitoring should look for attempts to read the environtmental variable, temp directory or source code.

## Pseudocode Template

```pseudocode
lambda function(request, context):
    if (validateTriggerIntegrity(request) == false):
        log("Integrity Error:" + request)
        exit
    if (validateInput(request) == false):
        log("Input Error:" + request)
        exit
    Process(request)
    cleanTemporaryDirectory()

 log(unsafeStringToLog):
     safeStringToLog = null
     if (stringToLog is not null):
          safeStringToLog = Encode.forHtml(stringToLog.substring(0, Math.min(stringToLog, MAXLOGSIZE)).replace("\n","__n__").replace("\r","__r__))
     CentralizedLogger.log(safeStringToLog)
```

## Other Factors

## Good practices to keep in mind

