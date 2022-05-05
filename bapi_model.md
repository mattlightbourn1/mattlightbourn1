# BAPI IAG Technical Guide

#### Version 0.1

Created by Matt Lightbourn

## Introduction

This page details a concept data payload for the exchange of data between various internal and external parties within the lifecycle of an insurance quote request right through to when that quote is bound and becomes an active policy. It also then includes any alterations, cancellations and renewals of that policy.

### <a name="integrationObjects"></a>Integration Related Data Objects

These are the identifiers as used in the **header** and url that control how the interaction between consumer and insurer work and in the context of specific quotes or policies.

Object Property | Property Type | Description | Originating Operation
:------ | :-------- | :-------- | :--------------------
`X-Iag-Correlation-Id` | `string` | Used to tie together request and response messages for async operations. This is unique per request and returned back in the response. | `request`
`X-B3-GlobalTransactionId` | `string` | This is the unique message identifier for each and every request and response. | `request` `response`
`quoteid` | `string` | The quote identifier is used in the url for any update quote operations for new business, alteration and cancel. When a quote requires an update, the `quote_id` from [Policy object](#policyObject) will be required. | `request`
`policyid` | `string` | The policy identifier is used in the url for close after a bind has successfully produced a policy from a quote. When a policy lifecycle needs to close, the `policy_id` from [Policy object](#policyObject) will be required. | `request`

### <a name="apiOperations"></a>Supported API Operations

The API comes with the below standard operations

Operation | Type | Description
------ | -------- | --------
`QuoteRequest` | `POST` | Create a New Business Quote Request using Quote Request](#quoteRequest)
`UpdateQuote` | `PUT` | Referencing an existing `quoteid` add, remove or change the content of the original quote request.
`BindQuote` | `PUT` | Referencing an existing `quoteid` commit to making a quote into a policy with a bind request.
`CloseCycleRequest` | `PUT` | Once a quote has been a policy, the policy schedule is to be attached the request and referencing the `opportunity_id` in order to complete the request lifecycle.

# Operation based Payloads

## New Business Process
```mermaid
  graph TD;
      createQuoteForBusinessPackProduct-->QuoteResponse_Quoted;
      createQuoteForBusinessPackProduct-->QuoteResponse_Refer;
      createQuoteForBusinessPackProduct-->QuoteResponse_Declined;
      QuoteResponse_Quoted-->updateQuoteForBusinessPackProduct;
      updateQuoteForBusinessPackProduct-->QuoteResponse_Quoted;
      QuoteResponse_Refer-->createReferralForBusinessPackProduct;
      createReferralForBusinessPackProduct-->QuoteResponse_Quoted;
      createReferralForBusinessPackProduct-->QuoteResponse_Declined;
      QuoteResponse_Quoted-->createBindForBusinessPackProduct;
      createBindForBusinessPackProduct-->BindResponse;
      BindResponse-->closeBusinessPackProduct;
      closeBusinessPackProduct-->CloseResponse;
      QuoteResponse_Quoted-->notifyLossForBusinessPackProduct
```
## New Business Process
```mermaid
  graph TD;
      createQuoteForBusinessPackProduct-->QuoteResponse;
      createAlterationForBusinessPackProduct-->QuoteResponse;
      createCancellationForBusinessPackProduct-->QuoteResponse;
      QuoteResponse-->updateQuoteForBusinessPackProduct;
      updateQuoteForBusinessPackProduct-->QuoteResponse;
      QuoteResponse-->createReferralForBusinessPackProduct;
      QuoteResponse-->createBindForBusinessPackProduct;
      createBindForBusinessPackProduct-->BindResponse;
      BindResponse-->closeBusinessPackProduct;
      closeBusinessPackProduct-->CloseResponse;
      QuoteResponse-->notifyLossForBusinessPackProduct
```

# Testing Sequence diagram

```mermaid
sequenceDiagram
    participant broker
    participant insurer
    participant underwriter
    broker->>insurer: New Business Quote Request
    insurer->>broker: Quote Response - Quoted
    broker->>insurer: Bind Request
    insurer->>broker: Bind Response
```
# Flowchart
```mermaid
  flowchart LR;
      A[CI MULTI CHAPTCHA]-->B{Select captcha service by developer?};
      classDef green color:#022e1f,fill:#00f500;
      classDef red color:#022e1f,fill:#f11111;
      classDef white color:#022e1f,fill:#fff;
      classDef black color:#fff,fill:#000;
      B--YES-->C[How to use?]:::green;
      
      C-->U[I choose recaptcha.]:::green;
      U--Views-->Q["echo CIMC_JS('recaptcha');\n echo CIMC_HTML(['captcha_name'=>'recaptcha']);"]:::green;
      U--Controller-->W["CIMC_RULE('recaptcha');"]:::green;
      
      C-->I[I choose arcaptcha.]:::white;
      I--Views-->O["echo CIMC_JS('arcaptcha');\n echo CIMC_HTML(['captcha_name'=>'arcaptcha']);"]:::white;
      I--Controller-->P["CIMC_RULE('arcaptcha');"]:::white;
      
      C-->X[I choose bibot.]:::red;
      X--Views-->V["echo CIMC_JS('bibot');\n echo CIMC_HTML(['captcha_name'=>'bibot']);"]:::red;
      X--Controller-->N["CIMC_RULE('bibot');"]:::red;
      
      B--NO-->D[How to use?]:::black;
      D---Views:::black-->F["echo CIMC_JS('randomcaptcha');\n echo CIMC_HTML(['captcha_name'=>'randomcaptcha']);"]:::black; 
      D---Controller:::black-->T["CIMC_RULE('archaptcha,recaptcha,bibot');"]:::black;
```
