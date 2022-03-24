# Standardise Insurance Data Exchange Format

#### Version 0.1

Created by Matt Lightbourn

## Introduction

This page details a concept data payload for the exchange of data between various internal and external parties within the lifecycle of an insurance quote request right through to when that quote is bound and becomes an active policy. It also then includes any alterations, cancellations and renewals of that policy.

### <a name="apiOperations"></a>Supported API Operations

The API comes with the below standard operations

Operation | Type | Description
------ | -------- | --------
`QuoteRequest` | `POST` | Create a New Business Quote Request
`UpdateQuote` | `PUT` | Referencing an existing `quoteid` add, remove or change the content of the original quote request.
`BindQuote` | `PUT` | Referencing an existing `quoteid` commit to making a quote into a policy with a bind request.
`CloseCycleRequest` | `PUT` | Once a quote has been a policy, the policy schedule is to be attached the request and referencing the `policyid` in order to complete the request lifecycle.

### <a name="integrationObjects"></a>Integration Related Data Objects

Placeholder for identifiers, etc

### <a name="integrationDetails"></a>Integration Details

Placeholder for sequencing of different types of API operations and what is required with each.

### <a name="insuranceObjects"></a>Insurance Related Data Objects

**This data model represents the best of the learnings from the project so far.** The model as shown below maintains all the existing operation level values and reuses most of the existing objects that were built in the original schema design. 

Where it differs, we will migrate any/all information about the business or an individual situation's location or building out of the sections which enables sections to be generic and driven by section_type value. In addition, everything inside of a section is generic including the coverages and any cover extensions and specified Items.

### <a name="rootObject"></a>Root object

Object Property | Property Type | Description | Originating Operation
------ | -------- | -------- | --------------------
`distributor_details` | `object` | This contains information about the intermediary organisation transacting with the insurer and includes what trading platform is being used to originate the request. | `request`
`policy` | `object` | This contains all policy related objects for this request using [Policy object](#policyObject) | `request`

### <a name="policyObject"></a>Policy object

Object Property | Property Type | Description | Originating Operation
------ | -------- | -------- | --------------------
`quote_id` | `string` | Text | `response`
`policy_id` | `string` | Text | `response`
`opportunity_id` | `string` | Text | `request`
`thread_id` | `string` | Text | `request`
`quote_number` | `string` | Text | `response`
`policy_number` | `string` | Text | `response`
`parties` | `object` | Represents all parties related to the processing of the request including the insured, the broker agent and interested parties using [Parties object](#partiesObject) | `request`
`policy_notes` | `array objects` | These are the notes that are added to request by the **consumer** as either a `PRINTABLE_NOTE` if it is to appear on the schedule or for information purposes only is a `NON_PRINTABLE_NOTE` using [Policy Notes object](#policyNotesObject) | `request`
`policy_dates` | `object` | Text using [Policy Dates object](#policyDatesObject) | `request`
`policy_wording` | `object` | Text using [Policy Wording object](#policyWordingObject) | `request`
`policy_premiums` | `array objects` | These are the policy premiums which are the total of all `situation` and `section` based premiums contained within the request using [Premiums object](#ppremiumsObject). | `response`
`commission_rate` | `string` | Text | `response`
`acceptance_messages` | `array objects` | This is where any policy related issues were found when processing a quote request using [Acceptance Messages object](#acceptanceMessagesObject) | `response`
`endorsement_clauses` | `array objects` | This is where an underwriter imposed or automated endorsement clauses have been added to a request based upon the outcome of processing a quote request using [Endorsement Clauses object](#endorsementClausesObject). | `response`
`lines_of_business` | `array objects` | Each request can have one or more lines of business. Each will have details of the business, related sections and situations using [Lines of Business object](#linesOfBusinessObject). | `request`
`policy_changes` | `object` | Text | `request`
`policy_remarks` | `object` | Text | `request` `response`

### <a name="partiesDetails"></a>Parties object
Represents all parties related to the processing of the request including the insured, the broker agent and interested parties.

Object Property | Property Type | Description | Originating Operation
------ | -------- | -------- | --------------------
`party_history` | `object`| Text | `request`
`preferred_correspondence` | `string` | This is to assign which party role represents the insured corrspondence in the policy process, either `PRIMARY_POLICY_HOLDER` or `BROKER_AGENT` | `request`
`party_details` | `array objects` | Details all the information about a specific party involved in the policy process using the [Party object](#partyObject) | `request`

### <a name="partyDetails"></a>Party object
Details all the information about a specific party involved in the policy process

Object |  Description | Originating Operation
------ | -------- | --------------------
`party_roles` | This is where the assignment of one or more role as `PRIMARY_POLICY_HOLDER`, `INTERESTED_PARTY` and `BROKER_AGENT` using the **Party Roles object** | `request`
`party_interests` | If `party_roles` includes `INTERESTED_PARTY` then `party_interests` are to be set uing the [Party Interests object](#partyInterestsObject) | `request`

### <a name="partyInterestsObject"></a>Party Interests object
If `party_roles` includes `INTERESTED_PARTY` then `party_interests` are to be set.

Object Property | Property Type |  Description | Originating Operation
------ | ------ |-------- | --------------------
`nature_of_interest` | `string` | Sets the nature of interest by this party based on the [Nature of Interest](#natureOfInterestOptions) | `request`
`policy_interests` | `object` | For all non situation based interests of one or more `sections` to be added to the array | `request`
`situation_interests` | `object` | For interests of one or more `sections` to be added to the array per situation in the request. For each situation, the `situation_id` will be required. | `request`

### <a name="linesOfBusinessObject"></a>Lines of Business object
This is where an entire business related to the policy is defined.

Object Property | Property Type |  Description | Originating Operation
------ | ------ |-------- | --------------------
`line_of_business_type` | `string` | This is to identify what type of line of business this is. Currently the default value for CGU is **COMMERCIAL_PACK** | `request`
`business_details` | `object` | This is where all details related to the business activities and the parameters of the business operation are defined using [Business Details object](#businessDetailsObject). | `request`
`business_characteristics` | `array object` | This is to capture variable information about the business in addition to the core. For example, acceptance questions are to be captured using data driven objects where the maintenance of what can be supplied in a payload is done through configuration. This is using [Characteristics object](#characteristicsObject). | `request`
`sections` | `array object` | This is for the policy level sections which contain coverages, excesses and their own sets of common response object. This is using [Section object](#sectionObject). | `request`
`situations` | `array object` | This is for the policy level sections which contain coverages, excesses and their own sets of common response object. This is using [Section object](#sectionObject). | `request`

### <a name="businessDetailsObject"></a>Business Details object

Object Property | Property Type |  Description | Originating Operation
------ | ------ |-------- | --------------------
`license_details` | `object` | Relates to any specific business related license holders required to perform specific business activities. Currently relates to `VIC_PLUMBERS` and `QLD_ELECTRICIANS` but extensible for others in the future. | `request`
`overseas_activities` | `object` | This is to describe what overseas activities the business performs and specify the turnover that such activities generates. | `request`
`employee_by_location` | `object`|  This is to describe what responsibilities staff have in various states. This is related to information required for **Employee Dishonesty** coverages. | `request`
`import_goods_details`  | `object`| This where the description of goods, where they are imported and the turnover generated is specified. | `request`
`export_usa_canada_details`  | `object`| This where the description of goods exported to USA or canada and the turnover generated is specified. | `request`
`seasonality`  | `object`| This is where the start of each business season and the impact to turnover is supplied. | `request`

## Reference Data

### <a name="natureOfInterestOptions"></a>Natures of Interest

Option Name | Option Code |  Option Description 
------ | ------ |-----------
Mortgagee	| MORTGAGEE
Local Government Authority | LOCAL_GOVERNMENT_AUTHORITY
Landlord	| LANDLORD
Lease	| LEASE
Premium Funder |	PREMIUM_FUNDER
Principal	| PRINCIPAL
Franchisor |	FRANCHISOR
Lendor |	LENDOR
Other |	OTHER
