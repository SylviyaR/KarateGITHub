Feature: EP - 863 Enhancements in Common Area -> Account Code, Custodian and Post Cash + Excluded Cash (C)

Background: Steps to execute before each scenarios
    # Select environment and provide other login credentials
    * url RB_Host
    * configure headers = read('../../helpers/headersfn.js')

    # Select schema file to validate API structure 
    * def schemaFile = read('../../feature/accountinstances/commonarea/testdata/schema-for-commonarea.json')

    # Select an account and get the account key dynamically
    * def accountID = 25
    * def result = call read('../../helpers/get-account-key.feature') { id : '#(accountID)'}
    * def key = $result[*].key
    * def endpoint = 'accountinstances/' + key + '/commonarea'

Scenario: DT - 9463 View Account Code, Custodian In Common Area (C) - Basic flow
 
    # Verify API response and verify whether the 3 new tags - custodianName,accountCode,postCashPlusExcludedCash are added to the API structure
    Given path endpoint
    When method GET
    And assert responseTime < 5000
    And match response == schemaFile


Scenario: Check the data for CustodianName
 
    # Verify the data displayed against the  new tag - custodianName
    Given path endpoint
    When method GET
    Then match response contains {custodianName:'FinPower'}

    #To Do: Verify the data displayed against the  new tags - accountCode(from accountinstances/Key/basics/update API), postCashPlusExcludedCash(Calculated value of Cash post $ and the excluded cash)


Scenario: Unauthorized- Wrong auth code (Status- 403 check) 
    
    # Select the file with wrong/expired authorization token
    * configure headers = read('../../helpers/wrongheadersfn.js')

    # Verify the message displayed when the user gives expired authorization token\
    Given path endpoint
    When method GET
    Then status 403 
     # Ideally it should throw 401 (Known issue for old endpoints. will be changed eventually)
    And match response.message contains ("Access Denied")


    
Scenario: Giving Invalid endpoint url (Status- 404 check)
       
    # Verify the message displayed when the user gives Invalid endpoint url
    * def endpoint = 'accountinstances/' + key + '/commonara'

    Given path endpoint
    When method GET
    Then status 404
    And match response.message contains ("No HTTP resource was found that matches the request")
