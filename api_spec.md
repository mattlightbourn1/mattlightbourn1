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

Object | Path | Description | Originating Operation
------ | -------- | -------- | --------------------
`policy` | `ROOT` | This contains all policy related objects for this request. | `request`
`policy_notes` | `policy` | These are the notes that are added to request by the **consumer** as either a `PRINTABLE_NOTE` if it is to appear on the schedule or for information purposes only is a `NON_PRINTABLE_NOTE` | `request`
`acceptance_messages` | `policy` | This is where any policy related issues were found when processing a quote request. | `response`
`endorsement_clauses` | `policy` | This is where an underwriter imposed or automated endorsement clauses have been added to a request based upon the outcome of processing a quote request. | `response`
`policy_premiums` | `policy` | These are the policy premiums which are the total of all `situation` and `section` based premiums contained within the request. | `response`
`party` | `policy` | Represents all parties related to the processing of the request including the insured, the broker agent and interested parties | `request`
`lines_of_business` | `policy` | Each request can have one or more lines of business. Each will have details of the business, related sections and situations. | `request`
`business_details` | `policy.lines_of_business` | This object contains all aspects of business activities and details about how the business operates | `request`

### <a name="partyDetails"></a>Party Details

Object |  Description | Originating Operation
------ | -------- | --------------------
`party_roles` | This is where the assignment of one or more role as `PRIMARY_POLICY_HOLDER`, `INTERESTED_PARTY` and `BROKER_AGENT` | `request`
`party_interests` | If `party_roles` includes `INTERESTED_PARTY` then `party_interests` are to be set | `request`

### <a name="partyDetails"></a>Party Interests

Object |  Description | Originating Operation
------ | -------- | --------------------
`policy_interests` | For all non situation based interests of one or more `sections` to be added to the array | `request`
`situation_interests` | For interests of one or more `sections` to be added to the array per situation in the request. For each situation, the `situation_id` will be required. | `request`

### <a name="businessDetails"></a>Business Details

Object |  Description | Originating Operation
------ | -------- | --------------------
`license_details` | Relates to any specific business related license holders required to perform specific business activities. Currently relates to `VIC_PLUMBERS` and `QLD_ELECTRICIANS` but extensible for others in the future. | `request`
`overseas_activities` | This is to describe what overseas activities the business performs and specify the turnover that such activities generates. | `request`
`employee_by_location` |  This is to describe what responsibilities staff have in various states. This is related to information required for **Employee Dishonesty** coverages. | `request`
`import_goods_details`  | This where the description of goods, where they are imported and the turnover generated is specified. | `request`
`export_usa_canada_details`  | This where the description of goods exported to USA or canada and the turnover generated is specified. | `request`
`seasonality`  | This is where the start of each business season and the impact to turnover is supplied. | `request`
