# DEBUGGING PROCESS FOR THE WP GO MAPS WORDPRESS PLUGIN

## EXECUTIVE SUMMARY

-------------
### DEBUGGING RESULTS: 

Findings at the end of the debugguing process indicated that there was no problem with the JSON parser or with the source code of the plugin. The error message seen was as a result of an unsuccessful API call to the Airtable API, which was due to an authentication issue from a wrong API key / endpoint. 

The Airtable API responded with a 403 error of unauthorized, and a error response written in HTML syntax. The parser attempts to parse the HTML syntax returned, and it fails, because the file indeed is not a valid JSON syntax.

The error message is shown below

-------------
### ERROR SUMMARY: 

**Error Message**


*“Error parsing JSON: Syntax error”*


***Error Scenario***

A premium user got this error when trying to import marker data from an Airtable API into the WP Go Pro Plugin


*Error Screenshot*

![](./images/Screenshot%202022-10-04%20at%2017.32.31.png)

*Reason for Error*

The Airtable API returns HTML data instead of JSON data when an authentication fails. This HTML data cannot be parsed by the json_decode function.

-------------
### **RECOMMENDATIONS**: 

The code can be improved by catching the 403 unauthorized error which is returned from the Airtable API, when an incorrect API key is used. This can be used to create a custom error log for failed authentication. This can help the user know that the API key needs to be rechecked.

-------------

## DETAILED DEBUGGING PROCESS

CONTENT

1. Background Research / Documentation Study
2. Error Replication on Localhost
3. Scanning Codebase to Isolate Key Functional Component(s)
4. Understanding Working Principle of Functional Component
5. Testing the Airtable Endpoint using Curl
6. Testing the Airtable Endpoint using Nodejs
7. Testing the Airtable Endpoint using Postman
8. Testing a different Airtable Endpoint and API Key using Curl, Nodejs and Postman.
9. Testing the New Airtable Endpoint with the WP GO MAPS PRO plugin.
10. Conclusion
    
    -------

    ### 1. Background Research / Documentation Study
    An initial background study was done using the WP Go Maps documentation, to have a proper understanding of how the plugin works from an end user's point of view, and also to understand the various functionalities of the Pro Extension.  Once this was done, a quick Google search was carried out with the error message to discover if the error observed was a unique error or a common error.

    ![](./images/Screenshot%202022-10-05%20at%2010.47.23.png)

    Findings from this initial background study showed that the JSON syntax error was either as a result of the data structure being fed into the JSON Parser, or a buggy parsing algorithm.

     ### 2. Error Replication on Local Host
     To properly understand the error, the scenario had to be replicated in a test environment on Localhost. 
     
     - A MySQL server was setup and was started.
     - The latest version of WordPress was downloaded (version 6.0.2).
     - The the WP GO Maps Basic and Pro plugins were downloaded and installed.
     - PHP version 8 was used to for the testing of the plugin.
     - The Airtable data import was selected for the plugin.
     - The Airtable API endpoint was pasted in the input field, along side the API key.
     - The error message which was given in the error documentation sent was received as expected.
     - ![](./images/Screenshot%202022-10-04%20at%2017.32.31.png)
     - The error logs were properly studied to help isolate the component which would be responsible for throwing the error
     - ![](./images/Screenshot%202022-10-05%20at%2008.52.57.png)



    ### 3. Scanning the code base to isolate key functional Component(s)

    The error logs showed that the WP Go Maps Pro plugin could not parse the data returned from the Airtable API properly. This showed a problem with the data structure returned from the Airtable API, or a problem with the Parser.

       - The legacy code for the WP Go Maps Pro plugin was opened up in VS Code and the **ImportAIRTABLES** Class was studied. 
       - This was the component responsible for parsing the data returned from the Airtable API.
  
       - ![](./images/Screenshot%202022-10-05%20at%2009.00.12.png)
  

    ### 4. Understanding the Working Principle of the Functional Component.
    The functional component works by running the **json_decode** function on the data received from the airtable API. If that returns a null value, it throws out an error log which states
    
    ***"Failed to parse Airtable"***

        if( $this->file_data === null )
			{
				$this->log("Failed to parse Airtable");
				$this->log(json_last_error_msg());
			
				throw new \Exception( __('Error parsing Airtable: ', 'wp-google-maps') . json_last_error_msg() );
			}

    - This gave a suspicion that the airtable data returns a null value after it is attempted to be parsed by the json_decode function.

    - There was a need to find out what data was actually returned by the Airtable API, using a direct API call via POSTMAN or any other means


    ### 5. Testing the Airtable Endpoint Using Curl
    The curl command was used to make a get request on the API endpoint, using the authorization "Bearer APIKEY". However, this gave an error response of a HTML file and an authorization error

    ### 6. Testing the Airtable enpoint using Node JSON
    A simple node js Script was again used to query the enpoint with the authorization API key. 
    - ![](./images/Screenshot%202022-10-05%20at%2009.23.47.png)
    - The results of the API call showed an authorization error.
    - ![](./images/Screenshot%202022-10-05%20at%2009.26.01.png)
    - By this it was becoming obvious that there is an authorization issue with the API KEY for that endpoint

    ### 7. Testing the Airtable Endpoint Using Postman
    Finally, as a last test, the Airtable endpoint was tested using postman, and the response was the same.
    - it gave an authorization error and returned a HTML syntax as its error response
    - ![](./images/Screenshot%202022-10-05%20at%2009.31.47.png)

    ### 8. Testing a different Airtable endpoint and API Key using curl, Nodejs, Postman.
    To properly understand the issue, a different Airtable endpoint was created and used. This endpoint was to a base in my personal airtable account.An API Key was generated accordingly for that base.

    - This new endpoint and the API key was tested in both curl, nodejs, and postman.
    - The data received was a JSON file as expected. No error message was received.
    - ![](./images/Screenshot%202022-10-05%20at%2009.37.37.png)
    - -----
    - ![](./images/Screenshot%202022-10-05%20at%2009.39.57.png)


    ### 9. Testing the different Airtable endpoint with the WP GO MAPS PRO plugin.
    The New API endpoint and key was pasted in the input for Airtable imports in the WP Go Maps Pro plugin.

    - This worked perfectly, and no errors were observed
    - ![](./images/Screenshot%202022-10-05%20at%2008.51.27.png)

    ### 10. Conclusion
    The conclusion for the debugging process shows that the Parser works fine for JSON files, but does not work when HTML data is returned by the Airtable API. And the Airtable API returns this when there is an authorization error from the endpoint, which causes the parser to give an error.

    