# Gift Entry Quick Start
1. Pre-req

   1. If they were using Batch Gift Entry -- process the open batches.

   1. Make sure My Domain is enabled.

1. Enable Advanced Mapping for Data Import & Gift Entry
   > Screenshot any errors.
   >
   > Example:  Payment Authorization Token (npsp__Payment_Authorization_Token__c) Authorization token for a transaction.

1. Enable Gift Entry

1. Have Dev do a deploy of the extra fields and permission set
`sfdx force:source:deploy -x ./manifest/package.xml -l RunSpecifiedTests -r MyProfilePageControllerTest -u [ALIAS]`
   > This uses a dummy test class which should exist in all orgs.
   > 
   > See: https://github.com/tschug/Gift-Entry-Quick-Start

1. If you want -- update page layouts for Opportunity and NPSP Data Import

1. Assign the "Gift Entry (Combined Updates)" permission set group to users
   > This should contain both the NPSP delivered version and items from ChapterSpot Permission Set (Gift Entry CSP)
   > 
   > You'll want to update the NPSP permission set to include [NPSP Data Import Batches].npsp__Batch_Table_Columns__c

1. Even if they didn't use it, remove Batch Gift Entry Tab for ALL USERS!
   > If they search "Gift Entry" in App Launcher, it will show up and they'll get confused.
   > 
   > Simply go to the Permission Set called "Batch Gift Entry" and make the Object Batch Gift Entry unavailable/not visible
   >
   > You may even want to go so far as to remove the tab visibility from the SysAdmin profiles.

1. Add `Batch Table Columns` to NPSP Gift Entry Recommended permission set (it's not currently there in releases).

1. Map the new fields.
   > If the client needs additional fields, you may map them now as well.  BUT REMEMBER to add those fields to the Permission Set `Gift Entry CSP` to ensure they can insert/edit the fields on NPSP Data Import AND the Object it maps too.

   | **Opportunity**                    |   |                                  |                                    |
   |------------------------------------|:-:|----------------------------------|------------------------------------|
   | Acknowledgement_Status__c          | → | npsp__Acknowledgment_Status__c   | Acknowledgement Status             |
   | Due_Date__c                        | → | quot__Due_Date__c                | Due Date (for cspb)                |
   | Honoree_Contact__c                 | → | npsp__Honoree_Contact__c         | Honoree Contact (lookup)           |
   | Honoree_Name__c                    | → | npsp__Honoree_Name__c            | Honoree Name (text if no lookup)   |
   | Invoice_Date__c                    | → | quot__Invoice_Date__c            | Invoice Date (for cspb)            |
   | Matching_Gift_Account__c           | → | npsp__Matching_Gift_Account__c   | Matching Gift Account (lookup)     |
   | Matching_Gift_Status__c            | → | npsp__Matching_Gift_Status__c    | Matching Gift Status               |
   | Matching_Gift_Employer__c          | → | npsp__Matching_Gift_Employer__c  | Matching Gift Employer             |
   | Opportunity_Currency_ISO_Code__c   | → | pymt__Currency_ISO_Code__c       | Currency ISO Code (for cspb)       |
   | Opportunity_Price_Book__c          | → | Pricebook2Id                     | Price Book (lookup) (for cspb)     |
   | Opportunity_Receivable_GAU__c      | → | cspb__Receivable_GAU__c          | Receivable GAU (lookup) (for cspb) |
   | Tribute_Type__c                    |   | npsp__Tribute_Type__c            | Tribute Type                       |

   |**Payment**                         |   |                                  |                                    |
   |------------------------------------|:-:|----------------------------------|------------------------------------|
   | Payment_Account__c                 | → | cspb__Account__c                 | Account (lookup) (for cspb)        |
   | npsp__Contact1Imported__c          | → | cspb__Payment_Contact__c         | Contact (lookup) (for cspb)        |
   | Credit_Card_Type__c                | → | cspb__Credit_Card_Type__c        | Credit Card Type (optional)        |
   | cspb__Deposit_Record_Id__c         | → | cspb__Deposit__c                 | Deposit (lookup) (for cspb)        |
   | npsp__Donation_Amount__c           | → | npe01__Payment_Amount__c         | Payment Amount                     |
   | npsp__Donation_Date__c             | → | npe01__Payment_Date__c           | Payment Date                       |
   | Payment_General_Accounting_Unit__c | → | cspb__General_Accounting_Unit__c | Payment GAU (lookup) (for cspb)    |
   | Payment_Record_Type_Name__c        | → | RecordTypeId                     | Payment Record Type (for cspb)     |
   | (Optional - See note)              |   |                                  |                                    |
   | npsp__Payment_Paid__c              | → | npe01__Paid__c                   | Paid indicator (T/F)               |

   > NOTE:  Fix Payment Creation ONLY WHEN CLIENT HAS TURNED OFF AUTO PAYMENT CREATION, and utilize the `Mark as Paid` fields to set the payment on NPSP Payments, otherwise it remains unpaid and can cause headaches.
   > Carefully make a back end update via Custom Metadata Type so that Payments are dependent on Opportunity, otherwise you MUST have Auto-Create Payments enabled to generate payment records.
   > 
   > **Custom Metadata**:  Data Import Object Mappings
   >
   > Take note of the `Data Import Object Mapping Name` for Opportunity.
   >
   > **Manage Record**:  Payment
   >
   > Fields:
   >
   >    **Predecessor ** = //Value for Opportunity mapping you noted above
   >
   >    **Relationship to Predecessor** = Child
   >
   >    **Relationship Field** = npe01__Opportunity__c

1. Setup a new Template that displays these fields if the Client wants them.

   1. Clone the `Default Gift Entry Template` to create a new one named `ChapterSpot Gift Entry Template`
   > This is a template used for Gift Entry and can be used to enter batches of gifts on the ChapterSpot Billing package.

   1. Add 3 new sections:
      - Matching
      - Tribute
      - Configuration

   1. Under Matching, add:
      - Matching Gift Status (Opportunity)
      - Matching Gift Account (Opportunity)
      - Matching Gift Name (Opportunity)

   1. Under Tribute, add:
      - Tribute Type (Opportunity)
      - Honoree Contact (Opportunity)
      - Honoree Name (Opportunity)

   1. Under Configuration, add:
      - Currency ISO Code (Opportunity)
      - Price Book ID (Opportunity)
      - Receivable GAU (Opportunity)
      - Record Type ID (Opportunity)
      - Stage (Opportunity)
      - Record Type ID (Payment)

   1. Under Donation Information, add:
      - Paid (Payment) // THIS MUST GO UNDER DONATION UNTIL SF FIXES DEFAULT CHECKBOX VALUES
      - Credit Card Type (Payment) 
      - Donor Notes
      - Acknowledgement Status

   > You may want to organize the order of fields under Donation Information.
   > Make sure that any other fields specific to the client are added at this time too.

   1. Organize the Batch Table Columns needed.

   1. Add the following fields to Batch Header:
      - Batch Status	// This is intended to prevent additional records being added when changing the page layout.
      - Custom Unique ID
      - Deposit General Accounting Unit // This will also be used to tie all Payments to this GAU
      - Deposit Source
      - Create Deposit // This will create a CSPB deposit and connect all payments to it.
      - Deposit // This can be used to link to an existing deposit and connect all payments to it.

   1. Add a better name if it would help your users.

1. Configure auto payment allocations in custom metadata Payment Allocation Config using the payment's Record Type Name.