# CarePath SL – Pseudocode

## Constants and variables

| Name | Type | Purpose |
|------|------|---------|
| SUPPORTED_LANGUAGES | Constant: list of text | English and Krio |
| FRESHNESS_LIMIT_HOURS | Constant: number | Proposed rule for flagging records that need re-verification (value to be agreed with health authorities) |
| language | Text | Language selected by the user |
| internetAvailable | Boolean | True if an internet connection is available |
| location | Text | Area selected by the user |
| requestedService | Text | Healthcare service being searched for |
| possibleEmergency | Boolean | True when the user indicates possible emergency warning signs |
| facilityRecords | List of records | Facility service, availability, verification time, location, contact |
| matchingFacilities | List of records | Facilities recorded as offering the requested service in the area |
| searchAgain | Boolean | Whether the user wants another search |

## Main pseudocode

```
BEGIN CAREPATH_SL

    language ← SelectLanguage(SUPPORTED_LANGUAGES)
    internetAvailable ← CheckInternetConnection()

    IF internetAvailable = TRUE THEN
        facilityRecords ← RetrieveLatestFacilityRecords()
    ELSE
        facilityRecords ← LoadSavedFacilityRecords()
        DisplayOfflineWarning()
    END IF

    searchAgain ← TRUE

    WHILE searchAgain = TRUE DO

        location ← GetUserLocationOrArea()
        requestedService ← GetRequestedService()

        IF ValidateInputs(location, requestedService) = FALSE THEN
            DisplayMissingInformationMessage()
            searchAgain ← AskWhetherToTryAgain()
            CONTINUE WHILE
        END IF

        possibleEmergency ← CheckForPossibleWarningSigns()

        IF possibleEmergency = TRUE THEN
            DisplayUrgentClinicalAssessmentGuidance()
            DisplayVerifiedEmergencyContactsIfAvailable()
            DisplayMessage("Do not delay seeking urgent help because of payment.")
        ELSE
            matchingFacilities ← FindFacilities(facilityRecords, location, requestedService)

            FOR EACH facility IN matchingFacilities DO
                CheckAndLabelInformationFreshness(facility)
            END FOR

            IF matchingFacilities IS NOT EMPTY THEN
                RankFacilitiesBy(serviceSuitability, verifiedAvailability, distance)
                DisplayFacilityOptions(matchingFacilities)
            ELSE
                DisplayNoRecordedMatchMessage()
                DisplayVerifiedReferralOrHealthWorkerContactIfAvailable()
            END IF
        END IF

        searchAgain ← AskWhetherToSearchAgain()

    END WHILE

    DisplayClosingMessage()

END CAREPATH_SL
```

## Supporting functions

| Function | Purpose |
|----------|---------|
| SelectLanguage | Returns the language chosen from the supported list |
| CheckInternetConnection | Returns TRUE if a usable connection is available, otherwise FALSE |
| RetrieveLatestFacilityRecords | Gets records from an authorised source and records retrieval time |
| LoadSavedFacilityRecords | Loads locally saved records for offline use |
| ValidateInputs | Checks that location and service are present and usable |
| CheckForPossibleWarningSigns | Uses a clinician-reviewed safety prompt; does not diagnose |
| FindFacilities | Returns facilities recorded as offering the service in the area |
| CheckAndLabelInformationFreshness | Compares verification time with the approved freshness rule and labels the record |
| RankFacilitiesBy | Orders results by service suitability, verified availability, then distance |
| DisplayFacilityOptions | Shows results, verification time, limitations and contact details |
| AskWhetherToSearchAgain | Returns TRUE if the user wants another search |
