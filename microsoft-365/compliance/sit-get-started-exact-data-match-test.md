---
title: "Test an exact data match sensitive information type"
f1.keywords:
- NOCSH
ms.author: chrfox
author: chrfox
manager: laurawi
audience: Admin
ms.topic: how-to
ms.service: O365-seccomp
ms.date:
ms.localizationpriority: medium
ms.collection:
- M365-security-compliance
search.appverid:
- MOE150
- MET150
description: configure services
ms.custom: seo-marvel-apr2020
---

# Test an exact data match sensitive information type

After your exact data match (EDM) sensitive information type (SIT) has been created and an hour after verifying that your sensitive information table has finished uploading and indexing, you can test that it detects the information you want to detect by using the test function in the sensitive information types section in the Compliance center.
 
>[!NOTE:]
>Changes in an already created EDM SIT can take some time to propagate across the system. If you are making changes in an EDM sensitive information type for troubleshooting detection issues, make sure to wait at least one hour after making those changes before using the test function to validate their impact.

## Test your EDM SIT in the Compliance Center

1. Open **Compliance center** > **Data classification** > **Sensitive Information Types**.

2. Select your EDM SIT from the list and then select **Test** in the flyout pane. This option is only present under sensitive information types.
 
3. Upload an item that contains data you want to detect. For example, create an item that contains a subset of the rows in your sensitive information table. If you used the configurable match feature in your schema to define ignored delimiters, make sure the item includes examples with and without those delimiters.

4. After the file has been uploaded and scanned, check for matches to your EDM SIT.

5. If the **Test** function in the SIT detects a match, validate that it isn't trimming it or extracting it incorrectly. For example by extracting only a substring of the full string it's supposed to detect, or picking up only the first word in a multi-word string, or including extra symbols or characters in the extraction. See [Regular Expression Language - Quick Reference](/dotnet/standard/base-types/regular-expression-language-quick-reference) for the regular expression language reference. 

5. Alternatively, you can use the following PowerShell cmdlet:

```powershell
Test-DataClassification  -ClassificationNames “[Your EDM sensitive info type]” -TexttoClassify “[your own text to scan for matches]” 
```

> [!NOTE]
 When you create a or edit an EDM sensitive information type, or the primary SIT on which an EDM type is based, all new content and content that’s modified after the changes to the SITs will be crawled for text that matches the new definitions, but preexisting content won’t be crawled until modified or reindexed. 

To force re-crawling of existing content in a SharePoint site or library or in OneDrive, follow the instructions in [Manually request crawling and reindexing of a site, a library or a list](/sharepoint/crawl-site-content).

## Test your EDM SIT in MIP policies

You can see where your EDM SIT is being used and how accurate it is in production by using them in policies:

1. Create an [auto-labeling policy](apply-sensitivity-label-automatically.md#how-to-configure-auto-labeling-policies-for-sharepoint-onedrive-and-exchange) and run it in **Simulation overview**.

1. Add some content that will trigger the EDM SIT and some content that won't trigger the EDM SIT to a location that your policy is monitoring.

1. Open the **Items to review** tab to check the matches.

1. Tune your policies as appropriate. 

Once you're satisfied with the results of your testing and tuning, your EDM based custom SIT is ready for use in Information Protection policies, like:

- [DLP policies](create-test-tune-dlp-policy.md#create-test-and-tune-a-dlp-policy)
- [Auto-labeling policies](apply-sensitivity-label-automatically.md#how-to-configure-auto-labeling-for-office-apps)
- [Microsoft Cloud App Security file policies](/cloud-app-security/data-protection-policies)

## Troubleshooting tips

If you don't find any matches, here are some troubleshooting tips.


|Issue  |Troubleshooting tip  |
|---------|---------|
|No matches found     |  Confirm that your sensitive data was uploaded correctly using the commands explained in [Hash and upload the sensitive information source table for exact data match sensitive information types](sit-get-started-exact-data-match-hash-upload.md#hash-and-upload-the-sensitive-information-source-table-for-exact-data-match-sensitive-information-types)|
|No matches found   | Test the SIT you used when you configured the primary element in each of your patterns. This will confirm that the SIT is able to match the examples in the item. Using an incorrectly defined SIT as the classification element of an EDM Sensitive information type is the most common cause for detection failures in EDM.         |
|The SIT you selected for a primary element in the EDM type doesn't find a match in the item or finds fewer matches than you expected    |  Check that it supports the separators and delimiters that are in the content. Be sure to include the ignored delimiters defined in your schema.       |
|The primary element SIT finds matches in an item, but the EDM SIT doesn't.     | - Check your REGEX statements for starting or ending a capturing whitespace delimiter, like /s. The whitespace won't match the hashed value in the data table. Use a word delimiter like /b instead. </br> - Check your REGEX statements to ensure that they capture the whole string you want to capture, not just a substring. For example, this pattern for email addresses [a-zA-Z]{30}@[a-zA-Z]{20}.[a-zA-Z]{2,3} will match *user@contoso.com* and *user@contoso.co.jp*.  |
|An EDM SIT with primary elements and no secondary elements defined detects items, but doesn't detect, or detects fewer than expected, when primary and secondary elements are required.  | Make sure values for secondary evidence are composed of a single word or string that doesn't contain spaces or use REGEX statements that detect multi-word strings. For example, \b[A-Z][a-z]{1,25}([ -][A-Z][a-z]{1,25}){0,4}\b, which will match any sequence of one to five consecutive words that start with an uppercase character. Use this SIT as the classification element for the additional evidence conditions in your EDM sensitive info type XML. See [Create a rule package manually](sit-get-started-exact-data-match-create-rule-package.md#create-a-rule-package-manually)|
|SIT test function doesn't detect any matches at all.   | Check if the SIT you selected includes requirements for additional keywords or other validations. For the built-in SITs, see [Sensitive information type entity definitions](sensitive-information-type-entity-definitions.md#sensitive-information-type-entity-definitions) to verify what the minimum requirements are for matching each type.        |
|The Test functionality works but your SharePoint or OneDrive items aren't being detected in DLP or auto-labeling rules     | Check if the documents you would expect to match show up in Content Explorer. If they aren't there, remember that only content created after the changes to the sensitive information type will show as matches. You have to recrawl the sites and libraries for pre-existing items to show up. See [Manually request crawling and reindexing of a site, a library or a list](/sharepoint/crawl-site-content) for details on recrawling SharePoint and OneDrive.        |
|DLP or auto-labeling rules that require multiple matches don't trigger     |Check that the proximity requirements for both your EDM type and the base sensitive information types are met. For example, if the maximum distance of between the primary element and supporting keywords is 300 characters, but the keywords are only present in the first row of a long table, only the first few rows of matching values are likely to meet the proximity requirements. Modify your SIT definitions to support more relaxed proximity rules or use the anywhere in the document option for the additional evidence conditions.         |
|Detection of an EDM type is inconsistent or erratic     |Check that the sensitive information type you used as the base for the primary element in your EDM type isn't detecting unnecessary content. Using a SIT that matches too much unrelated content, like any word, any number, or all email addresses, might cause the service to saturate and ignore relevant matches. Check the number of content pieces that match the sensitive type you used for your primary elements in content explorer. </br> To estimate if the SIT is matching too much content: </br> - Dividing the number of content items in Content Explorer by the number of days since the sensitive type was created. </br> - If the number of matches per day is in the range of hundreds of thousands or millions, it's possible that the primary SIT is too broad. See [Learn about exact data match based sensitive information types](sit-learn-about-exact-data-match-based-sits.md#learn-about-exact-data-match-based-sensitive-information-types) for recommendations and best practices on selecting the right sensitive information type for an EDM type.         |