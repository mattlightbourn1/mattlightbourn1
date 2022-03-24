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
`transaction_activity_log` | `array objects` | This contains information about the exchange of information between consumer and insurer. All entries here are added by the insurer including where activity is from the consumer. | `response`

### <a name="policyObject"></a>Policy object

This is where all information about the policy is contained and it relates to one or more lines of business

Object Property | Property Type | Description | Originating Operation
------ | -------- | -------- | --------------------
`quote_id` | `string` | This is the system generated unique identifier related to a quote after it has been delivered back to the requestor. | `response`
`policy_id` | `string` | This is the system generated unique identifier related to a policy after it has been bound. It relates to the `policy_number` | `response`
`opportunity_id` | `string` | This is to group the requests and responses into an opportunity. | `request`
`thread_id` | `string` | A thread is a branch of an opportunity, where there may be different quotes requested for the same broker client where only one of them has the potential of being bound. | `request`
`quote_number` | `string` | The quote number is the human readable reference used between a consumer and their client and for all non system to system quote related engagement. | `response`
`policy_number` | `string` | The policy number is the human readable reference used between a consumer and their client and for all non system to system policy related engagement. | `response`
`policy_status` | `string` | This is the status of the entire policy based upon the statuses of each section and situation within the request | `response`
`parties` | `object` | Represents all parties related to the processing of the request including the insured, the broker agent and interested parties using [Parties object](#partiesObject) | `request`
`policy_dates` | `object` | Text using [Policy Dates object](#policyDatesObject) | `request`
`policy_wording` | `object` | Text using [Policy Wording object](#policyWordingObject) | `request`
`policy_premiums` | `array objects` | These are the policy premiums which are the total of all `situation` and `section` based premiums contained within the request using [Premiums object](#premiumsObject). | `response`
`commission_rate` | `string` | Text | `response`
`acceptance_messages` | `array objects` | This is where any policy related issues were found when processing a quote request using [Acceptance Messages object](#acceptanceMessagesObject) | `response`
`endorsement_clauses` | `array objects` | This is where an underwriter imposed or automated endorsement clauses have been added to a request based upon the outcome of processing a quote request using [Endorsement Clauses object](#endorsementClausesObject). | `response`
`policy_notes` | `array objects` | This is where a consumer can add notes which are either to be added to the schedule or is simply information for the benefit of the consumer or underwriter only using [Notes](#notesObject). | `request`
`lines_of_business` | `array objects` | Each request can have one or more lines of business. Each will have details of the business, related sections and situations using [Lines of Business object](#linesOfBusinessObject). | `request`
`policy_changes` | `object` | This is used when an alteration or cancellation is done to an existing policy using [Policy Changes object](#policyChangesObject) | `request`
`policy_remarks` | `object` | Two way information regarding benefits, conditions, declined reasons and consumer originated remarks along with attachments using [Policy Remarks object](#policyRemarksObject). | `request` `response`

### <a name="partiesObject"></a>Parties object
Represents all parties related to the processing of the request including the insured, the broker agent and interested parties.

Object Property | Property Type | Description | Originating Operation
------ | -------- | -------- | --------------------
`party_history` | `object`| Text | `request`
`preferred_correspondence` | `string` | This is to assign which party role represents the insured corrspondence in the policy process, either `PRIMARY_POLICY_HOLDER` or `BROKER_AGENT` | `request`
`party_details` | `array objects` | Details all the information about a specific party involved in the policy process using the [Party object](#partyObject) | `request`

### <a name="partyDetails"></a>Party object
Details all the information about a specific party involved in the policy process

Object Property | Property Type | Description | Originating Operation
------ | -------- | -------- | --------------------
`party_type` | `string` | The options are either **ORGANISATION** or **INDIVIDUAL** | `request`
`party_names` | `array object` | This is to capture both the trading name and the name it is to be insured under. | `request`
`party_contacts` | `array object` | This is to capture one or more related individual contact details. | `request`
`party_roles` | `array` | This is where the assignment of one or more role as `PRIMARY_POLICY_HOLDER`, `INTERESTED_PARTY` and `BROKER_AGENT`. | `request`
`party_interests` | `object` | If `party_roles` includes `INTERESTED_PARTY` then `party_interests` are to be set uing the [Party Interests object](#partyInterestsObject) | `request`
`party_address` | `object` | This is the physical address of the party using [Address object](#addressObject). | `request`
`website_address`
`broker_client_id` | `string` | If the party represents the insured then the BTP originated client identifier should exist here. | `request`

### <a name="partyInterestsObject"></a>Party Interests object
If `party_roles` includes `INTERESTED_PARTY` then `party_interests` are to be set.

Object Property | Property Type |  Description | Originating Operation
------ | ------ |-------- | --------------------
`nature_of_interest` | `string` | Sets the nature of interest by this party based on the [Nature of Interest](#natureOfInterestOptions) | `request`
`policy_interests` | `object` | For all non situation based interests of one or more `sections` to be added to the array | `request`
`situation_interests` | `object` | For interests of one or more `sections` to be added to the array per situation in the request. For each situation, the `situation_id` will be required. | `request`

### <a name="addressObject"></a>Address object
This is to capture the physical address of a location of the business, the party or a situation.

Object Property | Property Type |  Description | Originating Operation
------ | ------ |-------- | --------------------
`building_name` | `string` | Text | `request`
`address_line_1` | `string` | Text | `request`
`address_line_2` | `string` | Text | `request`
`locality` | `string` | Text | `request`
`state` | `string` | Text | `request`
`postcode` | `string` | Text | `request`
`geo_coordinates` | `object` | Geo-coordinates are specifically for each situation address. If they cannot be supplied, the insurer will enrich the payload by looking up valid address parameters. | `request`

### <a name="policyDatesObject"></a>Policy Dates object
This is to capture all the relevant dates in all types of request

Object Property | Property Type |  Description | Originating Operation
------ | ------ |-------- | --------------------
`transaction_effective_date` | `string` | Definition to be confirmed | `request`
`term_inception_date` | `date` | This is when the policy would be set to start. | `request`
`term_expiry_date` | `date` | This is when the policy would be set to expire. | `request`
`transaction_message_sent` | `string` | This is the date that was produced by the payload constructing service indicating when it was submitted to the insurer. | `request`

### <a name="policyWordingObject"></a>Policy Wording object
This is where the policy wording from the insurer response is to be retrieved via a url along with related metadata

Object Property | Property Type |  Description | Originating Operation
------ | ------ |-------- | --------------------
`policy_wording_url` | `string` | Text | `response`
`policy_wording_version` | `string` | Text | `response`
`policy_url` | `string` | Text | `response`
`product_name` | `string` | Text | `response`
`product_version` | `string` | Text | `response`

### <a name="policyChangesObject"></a>Policy Changes object
This is where an existing policy is being altered or cancelled.

Object Property | Property Type |  Description | Originating Operation
------ | ------ |-------- | --------------------
`change_type` | `string` | This is either `CANCELLATION` or `ALTERATION` | `request`
`change_effective_date` | `date` | When this change comes into effect. | `request`
`change_reason` | `date` | Currently only applicable to **CANCELLATION** using [Change Reasons object](#changeReasonsObject). | `request`
`change_sub_reason` | `date` | An extension of `change_reason` * using [Change Reasons object](#changeReasonsObject). | `request`

### <a name="policyRemarksObject"></a>Policy Remarks object
This is utilised in both the `REQUEST` and `RESPONSE` and contains objects specific to each.

Object Property | Property Type |  Description | Originating Operation
------ | ------ |-------- | --------------------
`benefits` | `array string` | Where a response includes quoted sections, for each, there may be some benefits listed | `response`
`conditions` | `array string` | One or mode conditions imposed automatically or from an underwriter. | `response`
`decline` | `string` | Where a response concludes that the quote request was **DECLINED** supplies additional information. | `response`
`remark` | `array objects` | Used to add remarks during the lifecycle of the quote including one or many attached files using [Policy Remark object](#policyRemarkObject). | `request`

### <a name="policyRemarkObject"></a>Policy Remark object
This is specifically how the consumer supplies a remark and any related attachment(s).

Object Property | Property Type |  Description | Originating Operation
------ | ------ |-------- | --------------------
`attachments` | `array objects` | Add one or more file path and category for one or more attachments | `request`
`description` | `string` | Add a description (subject) for referral and supplying of additional information to underwriter. | `request`
`message` | `array objects` | Include a message between the consumer and the insurer to provide reason for referral, supplying additional information or commentary around attached files/ | `request`

### <a name="linesOfBusinessObject"></a>Lines of Business object
This is where an entire business related to the policy is defined.

Object Property | Property Type |  Description | Originating Operation
------ | ------ |-------- | --------------------
`line_of_business_type` | `string` | This is to identify what type of line of business this is. Currently the default value for CGU is **COMMERCIAL_PACK** | `request`
`business_details` | `object` | This is where all details related to the business activities and the parameters of the business operation are defined using [Business Details object](#businessDetailsObject). | `request`
`business_characteristics` | `array object` | This is to capture variable information about the business in addition to the core. For example, acceptance questions are to be captured using data driven objects where the maintenance of what can be supplied in a payload is done through configuration. This is using [Characteristics object](#characteristicsObject). | `request`
`sections` | `array object` | This is for the policy level sections which contain coverages, excesses and their own sets of common response object. This is using [Section object](#sectionObject). | `request`
`situations` | `array object` | This is for all the situations related to the business for the policy and its related sections using [Situation object](#situationObject). | `request`

### <a name="businessDetailsObject"></a>Business Details object

Object Property | Property Type |  Description | Originating Operation
------ | ------ |-------- | --------------------
`license_details` | `object` | Relates to any specific business related license holders required to perform specific business activities. Currently relates to `VIC_PLUMBERS` and `QLD_ELECTRICIANS` but extensible for others in the future. | `request`
`overseas_activities` | `object` | This is to describe what overseas activities the business performs and specify the turnover that such activities generates. | `request`
`employee_by_location` | `object`|  This is to describe what responsibilities staff have in various states. This is related to information required for **Employee Dishonesty** coverages. | `request`
`import_goods_details`  | `object`| This where the description of goods, where they are imported and the turnover generated is specified. | `request`
`export_usa_canada_details`  | `object`| This where the description of goods exported to USA or canada and the turnover generated is specified. | `request`
`seasonality`  | `object`| This is where the start of each business season and the impact to turnover is supplied. | `request`

### <a name="characteristicsObject"></a>Characteristics object
This is used for variable information using `category` and `type` combination to allow for a configuration driven solution for add, remove and change to acceptance questions.

Object Property | Property Type |  Description | Originating Operation
------ | ------ |-------- | --------------------
`category` | `string` | This is a grouping mechanism where a single category may have many types. A type (child) value may also be used as a category so that it can also have many types to allow for nested question sets. These two values are used to control a recursive model. | `request`
`type` | `string` | This is the specific definition of the value that has been captured. | `request`
`description` | `string` | To assist with readability, a description allows for a more human readable version which has been assigned a category and type. This would not be required from a consumer (broker) but will be supplied to a consumer if additional questions are required by the insurer which are not supported by the broker trading platform. | `request`
`value` | `string` | This is the specific definition of the value that has been captured. | `request`
`values` | `array string` | Specifically for acceptance questions where the answer might be from a multi-select set of options. | `request`
`details` | `string` | This is to allow for additional information tp be supplied along with the value which might include category or reason details. | `request`

### <a name="sectionObject"></a>Section object
This is for sections in policy or for each situation. Each section also has its own sets of coverages, cover extensions along with common response objects.

Object Property | Property Type |  Description | Originating Operation
------ | ------ |-------- | --------------------
`section_type` | `string` | This is to identify what type of section is within this array so that all objects and properties are in the correct context using [Section and Coverage Options](#sectionCoverageOptions) | `request`
`section_quote_status` | `string` | This is the overall status of the section which is the roll-up of all related coverages. | `response`
`section_status` | `string` This is to indicate whether a this section was already **EXISTING** or **ADDED** or **REMOVED** from an existing policy. | `request`
`acceptance_messages` | `array objects` | This is where any section related issues were found when processing a quote request using [Acceptance Messages object](#acceptanceMessagesObject) | `response`
`endorsement_clauses` | `array objects` | This is where an underwriter imposed or automated endorsement clauses have been added to a request based upon the outcome of processing a quote request using [Endorsement Clauses object](#endorsementClausesObject). | `response`
`section_notes` | `array objects` | This is for consumer added notes using [Notes](#notesObject). | `request`
`coverages`
`excesses`
`section_premiums` | `array objects` | These are the section premiums which are the total of all `coverage`  premiums contained within the request using [Premiums object](#premiumsObject). | `response`

### <a name="coverageObject"></a>Coverages object
This is for the individual coverages and their relationship to different [Section object](#sectionObject).

Object Property | Property Type |  Description | Originating Operation
------ | ------ |-------- | --------------------
`coverage_category` | `string` | This is the category of the coverage using [Section and Coverage Options](#sectionCoverageOptions). | `request`
`coverage_type` | `string` | This is type of coverage using [Section and Coverage Options](#sectionCoverageOptions). | `request`
`coverage_selected` | `boolean` | This indicates whether a coverage or additional benefit has been selected. | `request`
`coverage_qty` | `integer` | This is relevant to product offering where a number of units being covered is stated otherwise processed as single. | `request`
`coverage_value` | `integer` | This is the exposure value which is related to `value_type`, `exposure_type` and `basis_of_settlement`. | `request`
`value_type` | `string` | This is to indicate what `coverage_value is. The options are **CURRENCY** or **PERC** for percentage. | `request`
`basis_of_settlement` | `string` | The current options are **REPLACEMENT** or **INDEMNITY**. | `request`
`exposure_type` | `string` | Depending on the type of converage, this is either **SUM_INSURED** or **LIMIT**. | `request`
`coverage_extensions` | `array objects` | This is for future coverages which has a total exposure value as the sum of all related coverage extensions. | `request`
`specified_items` | `array objects` | This is for listing specified item where the total exposure value as the sum of all related specified items using [Specified Items](#specifiedItemsObject). | `request`

### Coverage Example
```json
          "coverage_category": "ADDITIONAL_BENEFIT",
          "coverage_type": "WORKS_OF_ART",
          "coverage_selected": true,
          "coverage_details": "Building Cover",
          "coverage_qty": 1,
          "coverage_value": 5000,
          "value_type": "CURRENCY",
          "basis_of_settlement" : "REPLACEMENT",
          "exposure_type": "SUM_INSURED",
          "coverage_extensions": []
```

### <a name="situationObject"></a>Situation object
This is for sections in policy or for each situation. Each section also has its own sets of coverages, cover extensions along with common response objects.

Object Property | Property Type |  Description | Originating Operation
------ | ------ |-------- | --------------------
`situation_identifier` | `string` | Text | `request`
`situation_type` | `string` | Text | `request`
`situation_quote_status` | `string` | Text | `response`
`situation_status` | `string` | Used to identify whether the situation was already **EXISTING** or **ADDED** or **REMOVED** from the policy. | `request`
`situation_occupation` | `object` | Identifies what primary occupation is performed at this situation. | `request`
`situation_location`
`situationn_characteristics` | `array object` | This is to capture variable information about a situation in addition to the core. For example, acceptance questions are to be captured using data driven objects where the maintenance of what can be supplied in a payload is done through configuration. This is using [Characteristics object](#characteristicsObject). | `request`
`acceptance_messages` | `array objects` | This is where any section related issues were found when processing a quote request using [Acceptance Messages object](#acceptanceMessagesObject) | `response`
`endorsement_clauses` | `array objects` | This is where an underwriter imposed or automated endorsement clauses have been added to a request based upon the outcome of processing a quote request using [Endorsement Clauses object](#endorsementClausesObject). | `response`
`situation_notes` | `array objects` | This is for consumer added notes using [Notes](#notesObject). | `request`
`situation_premiums` | `array objects` | These are the situation premiums which are the total of all `section` premiums contained within the request using [Premiums object](#premiumsObject). | `response`

### <a name="acceptanceMessagesObject"></a>Acceptance Messages object
This is where any issues were found when processing a quote request.

Object Property | Property Type |  Description | Originating Operation
------ | ------ |-------- | --------------------
`code` | `string` | This is the code for any common acceptance messages | `request`
`message` | `string` | This is the acceptance message itself | `request`
`recovery` | `string` | In some cases, a message may also come along with an action of how to recover from the issue. | `request`

### <a name="endorsementClausesObject"></a>Endorsement Clauses object
This is where an automated or underwriter imposed endorsement clauses.

Object Property | Property Type |  Description | Originating Operation
------ | ------ |-------- | --------------------
`code` | `string` | This is the code for any common acceptance messages | `request`
`description` | `string` | This is the definition of the endorsement clause that has been applied. | `request`
`title` | `string` | Short description of the endorsement clause. | `request`

### <a name="notesObject"></a>Notes object
This is an array of objects originated by the consumer and is available at the policy, situation and section level..

Object Property | Property Type |  Description | Originating Operation
------ | ------ |-------- | --------------------
`note_identifier` | `string` | A unique identifier for each note | `request`
`note_type` | `string` | This is either `PRINTABLE_NOTE` (to add to schedule) or `NON_PRINTABLE_NOTE` for information only. | `request`
`recovery` | `string` | In some cases, a message may also come along with an action of how to recover from the issue. | `request`

### <a name="premiumsObject"></a>Premiums object
This is an array of premium objects where there may be more than one in any given response, depending on how the request originated.

Object Property | Property Type |  Description | Originating Operation
------ | ------ |-------- | --------------------
`base_premium` | `integer` | This is the code for any common acceptance messages | `response`
`commission` | `integer` | This is the definition of the endorsement clause that has been applied. | `response`
`commission_gst` | `integer` | Short description of the endorsement clause. | `response`
`esl` | `integer` | Short description of the endorsement clause. | `response`
`gst` | `integer` | Short description of the endorsement clause. | `response`
`stamp_duty` | `integer` | Short description of the endorsement clause. | `response`
`total_premium` | `integer` | Short description of the endorsement clause. | `response`
`type` | `string` | This is the type of premium specified using [Premium Type Options](#premiumTypeOptions). | `response`

```json
    "section_premiums": [
	{
	    "stamp_duty": 29.74,
	    "total_premium": 344.65,
	    "base_premium": 286.27,
	    "gst": 28.64,
	    "commission": 0.0,
	    "commission_gst": 0.0,
	    "type": "TRANSACTION",
	    "esl": 0.0
	},
	{
	    "stamp_duty": 0.0,
	    "total_premium": 0.0,
	    "base_premium": 0.0,
	    "gst": 0.0,
	    "commission": 0.0,
	    "commission_gst": 0.0,
	    "type": "CURRENT_TERM",
	    "esl": 0.0
	},
	{
	    "stamp_duty": 29.74,
	    "total_premium": 344.65,
	    "base_premium": 286.27,
	    "gst": 28.64,
	    "commission": 0.0,
	    "commission_gst": 0.0,
	    "type": "ANNUALISED",
	    "esl": 0.0
	}
    ]
```


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

### <a name="premiumTypeOptions"></a>Premium Type Options

Option Name | Option Code | Option Description 
------ | ------ | -----------
Transaction | TRANSACTION	| Text
Annualised | ANNUALISED | Text
Current Term | CURRENT_TERM	| Text

### <a name="sectionCoverageOptions"></a>Section and Coverage Options

Section Name | Coverage Category |  Coverage Type | Object Usage
------ | ------ |----------- | -------
LIABILITY |	COVERAGE | PUBLIC_AND_PRODUCTS_LIABILITY	
LIABILITY |	ADDITIONAL_BENEFIT |	DESIGNATED_CONTRACTS	
LIABILITY	| ADDITIONAL_BENEFIT |	VIC_PLUMBERS_WARRANTY	
LIABILITY	| ADDITIONAL_BENEFIT |	QLD_ELECTRICIANS
LIABILITY	| ADDITIONAL_BENEFIT |	MOTOR_TRADE | turnover
LIABILITY	| ADDITIONAL_BENEFIT |	MOTOR_FAULTY_WORK_RECTIFICATION | turnover
LIABILITY	| ADDITIONAL_BENEFIT |	MOTOR_VEHICLE_INSPECTION | turnover
LIABILITY	| ADDITIONAL_BENEFIT |	VEHICLES_WATERCRAFT_PHYSICAL_LEGAL_CONTROL | sum insured
LIABILITY	| ADDITIONAL_BENEFIT |	PROPERTY_IN_PHYSICAL_LEGAL_CONTROL
PROPERTY | COVERAGE |	BUILDING | sum insured
PROPERTY | COVERAGE |	CONTENTS | sum insured
PROPERTY | COVERAGE |	CONTENTS_AND_STOCK | sum insured
PROPERTY | COVERAGE |	STOCK | sum insured
PROPERTY | COVERAGE |	SPECIFIED_ITEMS	
PROPERTY | ADDITIONAL_BENEFIT |	MORTGAGE_PROTECTION	
PROPERTY | ADDITIONAL_BENEFIT |	EXTRA_COST_REINSTATEMENT
PROPERTY | ADDITIONAL_BENEFIT |	REMOVAL_OF_DEBRIS
PROPERTY | ADDITIONAL_BENEFIT |	FLOOD
PROPERTY | ADDITIONAL_BENEFIT |	PLAYING_SURFACES
PROPERTY | ADDITIONAL_BENEFIT |	REWRITING_OF_RECORDS
PROPERTY | ADDITIONAL_BENEFIT |	RENT_LOSS_OR_PAYABLE
PROPERTY | ADDITIONAL_BENEFIT |	MALICIOUS_DAMAGE_BY_TENANTS
THEFT |	COVERAGE |	CONTENTS_INCLUDING_STOCK |	sum_insured	
THEFT |	COVERAGE |	CONTENTS_EXCLUDING_STOCK	sum_insured	
THEFT |	COVERAGE |	STOCK_IN_TRADE |	sum_insured	
THEFT |	COVERAGE |	CAGARETTES_TOBACCO |	sum_insured	
THEFT |	COVERAGE |	ALCOHOL |	sum_insured	
THEFT |	COVERAGE |	LIQUOR_AND_TOBACCO |	sum_insured	
THEFT |	COVERAGE |	REWRITING_OF_RECORDS |	sum_insured	
THEFT |	COVERAGE |	SPECIFIED_ITEMS |	sum_insured	
THEFT |	ADDITONAL_BENEFIT |	DAMAGE_TO_RENTAL_PREMISES |	sum_insured	
THEFT |	ADDITIONAL_BENEFIT |	THEFT_OF_PROPERTY_OPEN_AIR |	sum_insured	
THEFT |	ADDITIONAL_BENEFIT |	THEFT_WITHOUT_FORCEFUL_ENTRY |	sum_insured	
THEFT |	ADDITIONAL_BENEFIT |	BUSINESS_RECORDS |	sum_insured	
THEFT |	ADDITIONAL_BENEFIT |	DAMAGE_TO_PREMISES |	sum_insured	
THEFT |	ADDITIONAL_BENEFIT |	SEASONAL_INCREASES |	selected	
MONEY |	COVERAGE |	BLANKET_COVER |	sum_insured	
MONEY |	COVERAGE |	MONEY_IN_TRANSIT |	sum_insured	
MONEY |	COVERAGE |	MONEY_IN_BUILDING_DURING_BUSINESS_HOURS |	sum_insured	
MONEY |	COVERAGE |	MONEY_IN_BUILDING_OUTSIDE_BUSINESS_HOURS |	sum_insured	
MONEY |	COVERAGE |	MONEY_IN_BUILDING_LOCKED_IN_SAFE |	sum_insured	
MONEY |	COVERAGE |	MONEY_IN_PERSONAL_CUSTODY |	sum_insured	
MONEY |	COVERAGE |	MONEY_IN_PRIVATE_RESIDENCE |	sum_insured	
MONEY |	COVERAGE |	DAMAGE_TO_SAFES |	sum_insured	
MONEY |	COVERAGE |	SPECIFIED_ITEMS |	sum_insured	
MONEY |	ADDITIONAL_BENEFIT |	PERSONAL_MONEY_EXTENSION |	selected	
ELECTRONIC |	COVERAGE |	SPECIFIED_ITEMS	
ELECTRONIC |	ADDITONAL_BENEFIT |	ACCIDENTAL_DAMAGE	
ELECTRONIC |	ADDITONAL_BENEFIT |	FIRE_AND_PERILS	
ELECTRONIC |	ADDITIONAL_BENEFIT |	BREAKDOWN	
ELECTRONIC | COVERAGE | BUSINESS_INTERRUPTION	
ELECTRONIC |	ADDITIONAL_BENEFIT |	RESTORATION_OF_DATA |	sum_insured	
ELECTRONIC |	ADDITIONAL_BENEFIT |	INCREASED_COST_OF_WORKING |	sum_insured	
ELECTRONIC | ADDITIONAL_BENEFIT |	GROSS_INCOME |	sum_insured	
ELECTRONIC |	ADDITIONAL_BENEFIT |	GOODS_IN_TRANSIT |	sum_insured	
MACHINERY_BREAKDOWN	| COVERAGE |	BLANKET_COVER_AND_MACHINERY	
MACHINERY_BREAKDOWN	| COVERAGE |	SPECIFIED_MACHINERY	
MACHINERY_BREAKDOWN	| COVERAGE |	SPECIFIED_PRESSURE_EQUIPMENT	
MACHINERY_BREAKDOWN	| ADDITIONAL_BENEFIT |	DETERIORATION_OF_STOCK	

