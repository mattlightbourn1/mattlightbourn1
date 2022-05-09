# BAPI IAG Technical Guide

#### Version 0.1

Created by Matt Lightbourn

## Introduction

This page details a concept data payload for the exchange of data between various internal and external parties within the lifecycle of an insurance quote request right through to when that quote is bound and becomes an active policy. It also then includes any alterations, cancellations and renewals of that policy.

# First request based workflow - Create and update quote requests to bind and close
The diagram below shows the various operations required from the initial request of a quote through to binding and closing on the happy path. Where there are other additional requests required to change the outcome (example, **referRequest**) these additional paths will be documented using a variation of this workflow. 
```mermaid
  flowchart LR;
      A[createQuote]:::green--Quoted-->B[quoteResponse_Quoted];
      classDef black color:#fff,fill:#000;
      classDef green color:#022e1f,fill:#00f500;
      B--Lost-->L[notifyLoss]:::green;
      B--Bind-->C[bindRequest]:::green;
      C--Bound-->G[closeRequest]:::green;
      C-->I[error]:::black;
      B--Refer-->D[referRequest]:::green;
      A--Refer-->E[quoteResponse_Refer];
      A--Declined-->F[quoteResponse_Declined];
      E-->D;
      H[updateQuote]:::green--Quoted-->B;
      H--Refer-->E;
      H--Declined-->F;
      F--Refer-->D;
      F-->J[abandon]:::black;
```
## New Quote Lifecycle Requests

### Create Quote
The beginning of a policy lifecycle starts with the creating of a quote in the context of new business, alteration, cancellation and renewal. The difference will be in whether it is related to an existing `policy` or `quote`. Each new quote is the start of a new **opportunity** along with the first **thread**.
Request Type | Lifecycle Stage | Endpoint | Operation | Operation Type | URL
----|----|----|---|---|---
createQuote | New Business | /quotes | createQuoteForBusinessPackProduct | POST | https://product-services-dev.ff-dev.iagcloud.net/services/v1/product/commercial/business/quotes
createQuote | Alteration | /policies/**{policyId}**/policy-changes | createAlterationForBusinessPackProduct | POST | https://product-services-dev.ff-dev.iagcloud.net/services/v1/product/commercial/business/policies/{policyId}/policy-changes/
createQuote | Cancellation | /policies/**{policyId}**/cancellations/ | createCancellationForBusinessPackProduct | POST | https://product-services-dev.ff-dev.iagcloud.net/services/v1/product/commercial/business/policies/{policyId}/cancellations/

### Update Quote
A requestor can resubmit request based upon a previously submitted quote as an update. This may then impact the insurer response after processing. An updated quote will be received as a response and will now have a new `quote_id` and `quote_number`. Where there are more than one quote in a state of **QUOTED**, either of these can be bound. 
Request Type | Lifecycle Stage | Endpoint | Operation | Operation Type | URL
----|----|----|---|---|---
updateQuote | New Business | /quotes/**{quoteId}** | updateQuoteForBusinessPackProduct | PUT| https://product-services-dev.ff-dev.iagcloud.net/services/v1/product/commercial/business/quotes/{quoteId}
updateQuote | Alteration | /policies/**{policyId}**/policy-changes/**{quoteId}** | updateAlterationForBusinessPackProduct | PUT | https://product-services-dev.ff-dev.iagcloud.net/services/v1/product/commercial/business/policies/{policyId}/policy-changes/{quoteId}/
updateQuote | Cancellation | /policies/**{policyId}**/cancellations/**{quoteId}** | updateCancellationForBusinessPackProduct | PUT | https://product-services-dev.ff-dev.iagcloud.net/services/v1/product/commercial/business/policies/{policyId}/cancellations/{quoteId}/

## Operations based upon Quote Responses

### Bind and Close
If a quote is in a `QUOTED` state then it can be bound and the opportunity closed as long as the quote has not expired.
Request Type | Lifecycle Stage | Endpoint | Operation | Operation Type | URL
----|----|----|---|---|---
bindRequest | QUOTED | /bind-and-issue | bindAndIssueForBusinessPackProduct | POST | https://product-services-dev.ff-dev.iagcloud.net/services/v1/product/commercial/business/bind-and-issue/
closeRequest | BOUND | /close | closeBusinessPackProduct | POST | https://product-services-dev.ff-dev.iagcloud.net/services/v1/product/commercial/business/close/

### Notify Loss
This is to complete the opportunity as lost.
Request Type | Lifecycle Stage | Endpoint | Operation | Operation Type | URL
----|----|----|---|---|---
notifyLoss | QUOTED | /notify-loss | notifyLossForBusinessPackProduct | POST | https://product-services-dev.ff-dev.iagcloud.net/services/v1/product/commercial/business/notify-loss/

### Refer
This is where the broker has decided to refer to an underwriter which can be done where the current state of a request due to the latest response of `QUOTED`, `REFER` or `DECLINED`.
A referral is the same payload as a quote request but includes the following additional values:
Request Type | Lifecycle Stage | Endpoint | Operation | Operation Type | URL
----|----|----|---|---|---
referRequest | any | /referrals | createReferralForBusinessPackProduct | POST | https://product-services-dev.ff-dev.iagcloud.net/services/v1/product/commercial/business/referrals/

# Second Request based workflow - Refer and supply additional info to bind and close
These are the steps beyond the first round of workflow where the broker submits a new related request off the back of a response from the insurer. Note, for supply additional information, you will be expecting back a new quoteResponse_...... of some sort. The next steps are articulated in both the first and second round of workflow.
```mermaid
  flowchart LR;
      classDef black color:#fff,fill:#000;
      classDef green color:#022e1f,fill:#00f500;
      B[quoteResponse_Quoted]--Refer-->D[referRequest]:::green;
      E[quoteResponse_Refer]--Refer-->D;
      F[quoteResponse_Declined]--Refer-->D;
      D--Quoted-->W[quoteResponse_Quoted];
      D--Conditional-->X[quoteResponse_Conditional];
      D--AdditionalInfo-->Y[quoteResponse_AdditionalInfo];
      D--Declined-->Z[quoteResponse_Declined];
      W--Bind-->C[bindRequest]:::green;
      C--Bound-->G[closeRequest]:::green;
      C-->I[error]:::black;
      W--Lost-->L[notifyLoss]:::green;
      X--Update-->H[updateQuote]:::green;
      Y--Supply-->A[supplyInfo]:::green;
      A-->K[quoteResponse_???];
      Z-->J[abandon]:::black;
      Y-->J[abandon]:::black;
```

## Additional Operations based upon Second Request based workflow
### Conditional
A conditional quote requires the broker to update quote as described in first request based workflow. This might be to add or remove specific sections and/or coverages in order to be quoted, for example.

### Supply Additional Info
Request Type | Lifecycle Stage | Endpoint | Operation | Operation Type | URL
----|----|----|---|---|---
supplyInfo | ADDITIONAL INFO | /supply-additional-info | supplyAdditionalInfoForBusinessPackProduct | POST | https://product-services-dev.ff-dev.iagcloud.net/services/v1/product/commercial/business/supply-additional-info/

# Constructing Quote Request Payload

## Header

### <a name="integrationObjects"></a>Integration Related Identifiers
These are the identifiers as used in the **header** and url that control how the interaction between consumer and insurer work.

Object Property | Property Type | Validation | Description | Originating Operation
:------ | :-------- | :-------- | :--- | :--------------------
`X-Iag-Correlation-Id` | `string` | Mandatory | Used to tie together request and response messages for async operations. This is unique per request and returned back in the response. | `request`
`X-B3-GlobalTransactionId` | `string` | Mandatory | This is the unique message identifier for each and every request and response. | `request` `response`

### <a name="documentIdentifiers"></a>Document Identifiers
These are the identifiers that are required on specific operation where either a quote or policy or both already exist. Example, an update to a quote will require the quoteid supplied in the url.
Object Property | Property Type | Description | Originating Operation
:------ | :-------- | :-------- | :--------------------
`quoteid` | `string` | The quote identifier is used in the url for any update quote operations for new business, alteration and cancel. When a quote requires an update, the `quote_id` from [Policy object](#policyObject) will be required. | `request`
`policyid` | `string` | The policy identifier is used in the url for close after a bind has successfully produced a policy from a quote. When a policy lifecycle needs to close, the `policy_id` from [Policy object](#policyObject) will be required. | `request`

## Body
### <a name="opportunityIdentifiers"></a>Opportunity Identifiers
These are the identifiers that allow you to identify an opportunity and thread. An opportunity can have may threads if a quote is duplicated. An opportunity must be unique at the beginning of the lifecycle for **crerateQuote** otherwise it will be rejected.

Object Property | Property Type | Validation | Description | Originating Operation
:------ | :-------- | :-------- | :----------| :----------
`opportunity_id` | `string` | Mandatory | This is the identifier for the sales opportunity which starts with a create quote. This should be a UUID.| `request` `response`
`thread_id` | `string` | Mandatory | For each opportunity, we support multiple threads to allow for multiple scenarios to be quoted at the same time. All threads will have unique thread identifiers but share the same opportunity identifier. This should be a UUID. | `request` `response`
```json
{
  "opportunity_id": "aeda01d8-f48e-4bb6-a842-ba6e4f85dfe2",
  "thread_id": "c222cecf-af18-455a-9596-e76807d7c9be"
}
```

### <a name="messageSenderObject"></a>Message Sender object 

Object Property | Property Type | Validation | Description | Originating Operation
:------ | :-------- | :-------- | :---- | :--------------------
`full_name` | `string` | Mandatory | This is the full name of the system operator that submitted the request or is set to Automated system response if a part of the response. This value should originate from the system user profile. | `request` `response`
`email_address` | `string` | Optional | The email contact details of the system operator. This value should originate from the system user profile. | `request` `response`
`phone_number` | `string` | Optional | The phone number of the system operator. This value should originate from the system user profile. | `request`
```json
  "message_sender": {
    "full_name": "John Smith",
    "email_address": "js@abc.com.au",
    "phone_number" : "03 6073000"
  }
```
### <a name="distributorDetailsObject"></a>Distributor Details object
This contains information about the intermediary organisation transacting with the insurer and includes what trading platform is being used to originate the request.

Object Property | Property Type | Description | Originating Operation
:------ | :-------- | :-------- | :--------------------
`organisation_name` | `string` | This is the broker's organisation name. | `request`
`office_name` | `string` | This is the broker's organisation's branch or site. | `request`
`organisation_identifier` | `string` | This is the broker's organisation code. | `request`
`office_identifier` | `string` | This is the broker's organisation's branch or site code. | `request`
`trading_platform_channel` | `string` | This is the trading platform's channel where the request originated as a data payload. | `request`

```json
  "distributor_details": {
    "trading_platform_channel": "BROKER1_CW"
    "organisation_details": {
      "office": {
        "office_identifier": "Melbourne",
        "office_name": "Melbourne Office"
      },
      "organisation_identifier": "INTERRISK",
      "organisation_name": "Broker One Australia Pty Ltd"
    },
  }
```
### Insured Party
Every request related to a policy requires at least one insured party. Refer to [Insured Parties](#insuredParties)

### Broker Agent or Broker Client
The `distributor_details` identifies where a request is originating. As a part of the correspondence preferences, either a [Broker Agent](#brokerAgent) or [Broker Client](#brokerClient) will be required along with either their email or address.  

### Interested Parties
Where there is the need to include one or more [Interested Party](#interestedParties) in a quote request.

### Business Details
This is information about occupation, turnover, staff and other characteristics of the business. 

### Situations
For each business premises to be added to the policy, there will be a [Situation](#situation) added for each. This includes the physical address, information about the building, security and safety. Refer to the document section Add Situation.

### Sections
There needs to be at least one [Section](#section) in a quote request at policy level or at least one per situation added. A section relates to one or more coverages and each section has their own set of acceptance questions.

### Excesses
There are some [Excesses](#excesses) that the broker can request and then others that are applied by the insurer. In addition to the variable amount, there is also an amount that can be imposed by the underwriter and then the total excess. All excesses are in their respective sections within the request and response.

### Notes
There is the option to add notes to the policy, situation or section which are either [Printable Note](#printableNote) in the policy schedule or [Non-Printable Note](#nonPrintableNote). 

# Parties
A party is required for each insured and interested party related to the policy. Each party required a unique identifier (UUID) since it is used as a foreign key in the payload to allocate a `party_role` and assigning the `interested_parties` to the policy and/or specific situations.

In addition to defining the various parties, this is also where `party_history_disclosures` are to be supplied. Where there have been claims within the last three years, for each claim you can supply information.
```mermaid
erDiagram
    parties ||--|{ organisations : contains
    parties ||--|{ individuals : contains
    organisations ||--|{ party : contains
    party {
        string party_id
    }
    party_i ||--|{ names_i : contains
    party ||--|{ addresses : contains
    addresses {
        string building_name
        string address_line1
        string address_line2
        string locality_name
        string city
        string state
        string country
    }
    addresses ||--|| geo_location : contains
    geo_location {
        string gnaf_pid
        int latitude
        int longitude
    }
    party ||--|{ email_contacts : contains
    email_contacts {
        string email_address
    }
    party ||--|{ names : contains
    names {
        string name
        string type
    }
    party ||--|{ phone_contacts : contains
    phone_contacts {
        string area_code
        string number
        string type
    }
    individuals ||--|{ party_i : contains
    party_i {
        string party_id
    }
    party_i ||--|{ addresses : contains

    party_i ||--|{ email_contacts : contains
    names_i {
        string preferred_name
    }
    party_i ||--|{ phone_contacts : contains
 
    parties ||--|{ party_roles : contains
    party ||--|{ registered_numbers : contains
    registered_numbers {
        string number
        string type
    }
    party_roles {
        string party_id
        string role
    }
    parties ||--|| party_history_disclosures : contains
    party_history_disclosures ||--|| bankruptcy : contains
    bankruptcy {
        string description
        string declared_bankrupt
    }
    bankruptcy ||--|| details : contains
    details {
        string description
    }
    details ||--|| date : contains
    date {
        int day
        int month
        int year
    }
    party_history_disclosures ||--|| civil_offence_or_pecuniary_penalty : contains
    civil_offence_or_pecuniary_penalty {
        string description
        string liability_of_penalty
    }
    civil_offence_or_pecuniary_penalty ||--|| details : contains
    party_history_disclosures ||--|| criminal_conviction : contains
    criminal_conviction {
        string description
        string convicted_for_offence
    }
    criminal_conviction ||--|| details : contains
    party_history_disclosures ||--|| claims_history : contains
    claims_history {
        string claims_made_in_last_three_years
    }
    claims_history ||--|{ claims_last_three_years : contains
    claims_last_three_years {
        int claim_amount
        string date_of_loss
        string description
        string preventive_or_corrective_action
        string sections
    }
    line_of_businesses ||--|| commercial_operations : contains
    line_of_businesses ||--|{ commercial_situations : contains
    commercial_operations ||--|{ interested_parties : contains
    commercial_situations ||--|{ interested_parties : contains
    commercial_situations {
        string asset_id
    }
    interested_parties {
        string nature_of_interest
        string party_id
        string sections
    }
```
## <a name="insuredParties"></a> Insured Parties
These are the named parties that are to be insured by the policy.

### Party Primary Policy Holder

```json
{
"parties" : {
     "organisations": [ 
       {
         "addresses": [
           {
              "address_line_1": "181 William Street",
              "address_line_2": "Melbourne 2000",
              "building_name": "Tower Two",
              "country": "AUS",
              "geo_location": {
                 "gnaf_pid": "GANSW716798454",
                 "latitude": -31.7708963,
                 "longitude": 115.84100663
              },
              "locality_name": "Lane Cove",
              "postcode": "3000",
              "state": "VIC"
           }
        ],
        "email_contacts": {
          "emails": [
            {
              "email_address": "abc@xyz.com"
            }
          ]
        },
        "insured": {
          "broker_client_id": "CLNT1234567890",
          "insurer_client_id": "CLNT1234567890"
        },
        "names": [
          {
            "name": "Business Name Limited.",
            "type": "TRADING"
          },
          {
            "name": "The Name to Insure Under",
            "type": "REGISTERED"
          }
        ],
        "party_id": "insured_party_uuid",
        "phone_contacts": {
          "phone_numbers": [
            {
              "area_code": "03",
              "number": "96024650",
              "type": "PRIMARY"
            }
          ]
        },
       "registered_numbers": [
          {
            "number": "123 455 678",
            "type": "ABN"
          }
        ]
     }
   ]
  }
}
```
### Party Role - Primary Policy Holder
In order to use this party as insured, there is the need to assign a party role to the payload using the `party_id` as shown above (**INSURED1**) which should be a UUID. Reference data for [Party Roles](#partyRoles)
```json
  {
      "party_roles": [
        {
          "party_id": "INSURED1",
          "role": "PRIMARY_POLICY_HOLDER"
        }
      ]
    }
```
## Correspondence Preferences
If the correspondence preference is **broker agent**, then the party can contain any of the following information with a minimum of `party_id`, `email_address` and/or `address` depending on whether `preferred_communication_mode is "EMAIL" or "MAIL". 

## <a name="brokerAgent"></a> Party - Broker Agent
Payload for a broker agent.
```json
  {
    "parties": {
      "individuals": [
        {
          "addresses": [
            {
              "address_line_1": "181 William Street",
              "address_line_2": "Melbourne 2000",
              "building_name": "Tower Two",
              "country": "AUS",
              "locality_name": "Lane Cove",
              "postcode": "3000",
              "state": "VIC"
            }
          ],
          "email_contacts": {
            "emails": [
              {
                "email_address": "abc@xyz.com"
              }
            ]
          },
          "party_id": "BROKER1",
          "phone_contacts": {
            "phone_numbers": [
              {
                "area_code": "03",
                "number": "96024650",
                "type": "PRIMARY"
              }
            ]
          }
        }
      ]
    }
  }
```
### Party Role - Broker Agent
In order to use this party as broker agent, there is the need to assign a party role to the payload using the `party_id` as shown above (**BROKER1**) which should be a UUID. Reference data for [Party Roles](#partyRoles)
```json
  {
      "party_roles": [
        {
          "party_id": "BROKER1",
          "role": "BROKER_AGENT"
        }
      ]
    }
```
## <a name="brokerClient"></a> Party - Broker Client
Payload for a broker client.
```json
  {
    "parties": {
      "individuals": [
        {
          "addresses": [
            {
              "address_line_1": "181 William Street",
              "address_line_2": "Melbourne 2000",
              "building_name": "Tower Two",
              "country": "AUS",
              "locality_name": "Lane Cove",
              "postcode": "3000",
              "state": "VIC"
            }
          ],
          "email_contacts": {
            "emails": [
              {
                "email_address": "abc@xyz.com"
              }
            ]
          },
          "names" : {
            "details" : [
              {
                "title" : "MR"
              }],
              "preferred_name" : "John Smith"
          },
          "party_id": "CLIENT1",
          "phone_contacts": {
            "phone_numbers": [
              {
                "area_code": "03",
                "number": "96024650",
                "type": "PRIMARY"
              }
            ]
          }
        }
      ]
    }
  }
```
### Party Role - Broker Client
In order to use this party as broker client, there is the need to assign a party role to the payload using the `party_id` as shown above (**CLIENT1**) which should be a UUID. Reference data for [Party Roles](#partyRoles)
```json
  {
      "party_roles": [
        {
          "party_id": "CLIENT1",
          "role": "BROKER_CLIENT"
        }
      ]
    }
```
## <a name="interestedParties"></a> Party - Interested Party
If a Quote request and subsequent policy include the addition of interested parties, they are to be added as a party as shown below.
```json
{
"parties" : {
     "organisations": [
       {
         "addresses": [
           {
              "address_line_1": "181 William Street",
              "address_line_2": "Melbourne 2000",
              "building_name": "Tower Two",
              "country": "AUS",
              "geo_location": {
                 "gnaf_pid": "GANSW716798454",
                 "latitude": -31.7708963,
                 "longitude": 115.84100663
              },
              "locality_name": "Lane Cove",
              "postcode": "3000",
              "state": "VIC"
           }
        ],
        "email_contacts": {
           "emails": [
             {
               "email_address": "abc@abcbankplc.com"
             }
           ]
        },
        "names": [
          {
            "name": "ABC Bank plc",
            "type": "TRADING"
          }
        ],
        "party_id": "INTEREST1",
        "phone_contacts": {
           "phone_numbers": [
             {
               "area_code": "03",
               "number": "96024650",
               "type": "PRIMARY"
             }
           ]
         }
       }
      ]
    }
  }
```
### Party Role - Interested Party
In order to use this party as broker client, there is the need to assign a party role to the payload using the `party_id` as shown above (**INTEREST1**) which should be a UUID. Reference data for [Party Roles](#partyRoles)
```json
  {
      "party_roles": [
        {
          "party_id": "INTEREST1",
          "role": "INTERESTED_PARTY"
        }
      ]
    }
```
### Assigning Interested Party - Policy Level
Using the `party_id` of each interested party, where policy level sections exist, assign them a `nature_of_interest` and list the `sections` as per the accepted reference data options. [Nature of Interest](#natureOfInterest) and [Interested Party Sections](#interestedPartyections)
```json
"commercial_operations" : [
    {
      "interested_parties": [
        {
          "nature_of_interest": "MORTGAGEE",
          "party_id": "INTEREST1",
          "sections": ["BUSINESS_INTERRUPTION","PUBLIC_LIABILITY"]
        }
     ],
   }]
```
### Assigning Interested Party - Situation Level
Using the `party_id` of each interested party, where situation level sections exist, assign them a `nature_of_interest` and list the `sections` as per the accepted reference data options for each situation applicable. [Nature of Interest](#natureOfInterest) and [Interested Party Sections](#interestedPartyections)
```json
"commercial_situations" : [
    {
      "interested_parties": [
        {
          "nature_of_interest": "MORTGAGEE",
          "party_id": "INTEREST1",
          "sections": ["BUSINESS_PROPERTY","ELECTRONIC_EQUIPMENT"]
        }
     ],
   }]
```

# Reference Data

### <a name="partyRoles"></a>Party Roles
| Party Role |
:----
| PRIMARY_POLICY_HOLDER |
| INTERESTED_PARTY |
| BROKER_AGENT |
| BROKER_CLIENT |

### <a name="natureOfInterest"></a>Nature of Interest
| Nature of Interest |
:----
| MORTGAGEE |
| LOCAL_GOVERNMENT_AUTHORITY |
| LANDLORD |
| LEASE |
| PREMIUM_FUNDER |
| PRINCIPAL |
| FRANCHISOR |
| LENDOR |
| OTHER |

### <a name="interestedPartyections"></a>Interested Party Sections
| Interested Party Section |
:----
| PUBLIC_AND_PRODUCTS_LIABILITY |
| GENERAL_PROPERTY |
| EMPLOYEE_DISHONESTY |
| TAX_INVESTIGATION |

# Additional Payloads

## Bind Request

### Bind with no update to insured party ABN details
```json
{
    "opportunity_id" :  "3aacb472-4f44-4385-96c7-f7605707a5ab",
    "thread_id" :  "bcf46dab-2ac0-4797-a0fd-65a040e480c1",
    "quote_id" :  "99a6b43b-7b08-444f-af41-5d337011bcda",
    "message_sender" : {
         "full_name" : "John Smith",
         "email_address" : "js@abc.com.au",
         "phone_number" : "03 6073000"
    },
    "distributor_details" : {
        "trading_platform_channel" : "AON_CW",
        "organisation_details" : {
            "organisation_identifier" : "INTERRISK",
            "organisation_name" :  "Interrisk Australia Pty Ltd)",
            "office" : {
                "office_identifier" : "Melbourne",
                "office_name" : "Melbourne Office",
            }
        }
    },
    "policy_dates" : {
        "message_sent_date" :  "2020-06-30T15:47:55.123+10:00"
    }
}
```
### Bind with ABN update for insured parties
```json
{
    "opportunity_id" :  "3aacb472-4f44-4385-96c7-f7605707a5ab",
    "thread_id" :  "bcf46dab-2ac0-4797-a0fd-65a040e480c1",
    "quote_id" :  "99a6b43b-7b08-444f-af41-5d337011bcda",
    "message_sender" : {
         "full_name" : "John Smith",
         "email_address" : "js@abc.com.au",
         "phone_number" : "03 6073000"
    },
    "distributor_details" : {
        "trading_platform_channel" : "AON_CW",
        "organisation_details" : {
            "organisation_identifier" : "INTERRISK",
            "organisation_name" :  "Interrisk Australia Pty Ltd)",
            "office" : {
                "office_identifier" : "Melbourne",
                "office_name" : "Melbourne Office",
            }
        }
    },
    "policy_dates" : {
        "message_sent_date" :  "2020-06-30T15:47:55.123+10:00"
    },
    "parties" : {
        "organisations" : [
            {
                "party_id" :  "PRTY123",
                "registered_numbers" : [
                    {
                        "number" :  "123 455 678",
                        "type" : "ABN"
                    }
                ]
            }
        ]
    }
}
```

