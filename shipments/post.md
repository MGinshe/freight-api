# Create Shipments

    POST api/shipments

## Description
This method is used to create new shipments.

There are 2 pre-conditions that need to be considered before calling this method.
1. Does your workflow require you to display/calculate the freight costs and options prior to creating the shipment?
2. Does your workflow already know which carrier is to be used for creating the shipment?


#### 1. Display/Calculate rates first
In this scenario, you need to call the [AvailableRates](https://github.com/gosweetspot/freight-api/blob/master/ratesqueryv1/POST_availablerates.md) method first to resolve the different price options you may have.  Every option is accompanied by a **QuoteId** which needs to be retain to be used with the create shipment method.

#### 2. Create by Carrier Name and service
In this scenario, you won't know the cost of the freight for the shipment till after the shipment is created.  You will request the shipment to be used by specifying the carrier name, and service.


**Printing-**  if the access_key has an associated printer setup, the print job is activated by default.  This can be turned off by specifying *PrintToPrinter=false*

***

## Requires authentication
* A valid access Key must be provided in **access_key** request header.

***

## Parameters

You need to extend the AvailableRates post body to include other fields specified below.

When *QuoteId* is supplied, the *carrier* and *service* fields are ignored.  They should be used in exclusive pattern.

- **QuoteId** - unique quote identifier returned on the *availablerates* query.
- **Carrier** - carrier name, to select which carrier to create shipment for.
- **Service** - carrier service name, for which the shipment should be created for. If not supplied, the cheapest price option is selected.
- **origin** - blank when using site address, or provide one for freight forwards
- **destination** - JSON obect of recepient address details
- **packages** - JSON object array of package sizes
- **issaturdaydelivery** - true/false - is saturday delivery required
- **issignaturerequired** - true/false - is signature on delivery required
- **dutiesandtaxesbyreceiver**  - true/false - where duties are due, is the receiver paying for these
- **ruraloverride** - true/false - ignore all rural delivery validation and surcharges
- **deliveryreference** - string:50, order reference
- **commodities** - JSON object array of customs declared commoditiy information. Only required for international shipments.
- **printtoprinter** - true/false - if supplied, the print job is sent to the *access_token* desingated printer. For testing purpose this can be provided as "no"
- **outputs** - optional -  JSON string array. Returns output of label as a PNG or PDF. Acceptable values:  
      LABEL_PDF - label is presented on an A4 page  
      LABEL_PNG_100X175 - label is presented as a PNG image with dimension 100mm x 175mm  
      LABEL_PNG_100X150 - label is presented as a PNG image with dimension 100mm x 150mm  
      LABEL_PDF_100X175 - label is presented as a PDF with dimension 100mm x 175mm  
      LABEL_PDF_100X150 - label is presented as a PDF with dimension 100mm x 150mm  

      The 100x150 sizing is presently experimental and not available across all carriers.  
- **hasdg** - true/false - does your shipment contain any dangerous goods
- **dangerousgoods** - JSON object of dangerous goods details

*origin/destination Object*
- **name** - string:60, Company name or persons name
- **contactperson** - string:60, contact person at address, optional
- **phonenumber** - string:60, phone number, optional
- **isrural** - true/false, optional
- **deliveryinstructions** - string:120, special instructions to print on the label. For origin this field is ignored.
- **sendtrackingemail** - true/false, optional
- **costcentreid** - int, optional
- **explicitnotrural** - true/false, when true not rural surcharges are added.
- **address** - JSON Object as described below

*address Object*
- **buildingname** - string:60, property identifier, such as Unit 1, Level 10, Panasonic House, etc
- **streetaddress** - string:60, street number and name.
- **suburb** - string:60, suburb name
- **city** - string:60, city or state name. In countries where there are official states, use use use state abbreviations, such as California = CA, New South Wales = NSW, etc.
- **postcode** - string:10, postal code
- **countrycode** - string:2, ISO Alpha 2 country code, eg, NZ, AU, US, GB, CN, CA, etc.

*package Object*
- **name** - string:15, package custom name, if none available, use "custom"
- **length** - integer, package length in centimetres
- **width** - integer, package width in centimetres
- **height** - integer, package height  in centimetres
- **kg** - decimal, package weight in kilograms
- **type** - string:10, package type, eg, Box, Carton, Satchel, Bag, Pallet, etc
- **packagecode** - string:5, Trackpack codes, such as DLE, A5, A4 (please consult support before providing a value in this field) This feature is not available on all accounts.

*commodity Object*
- **description** - string50, description of good
- **harmonizedcode** - string50, customs related HS Code for goods.
- **units** - decimal, how many units
- **unitvalue** - decimal, value per unit
- **unitkg** - decimal, weight in kg per unit
- **currency** - string:3, declared value currency designation, eg NZD, AUD, USD
- **country** - string:2, ISO Alpha 2 country code for the Country of Manufacture of goods, eg, NZ, AU, US, GB, CN, CA, etc.

*dangerousgoods Object*
- **additionalhandinginfo** - string100
- **hazchemcode** - string50
- **isradioactive** - true/false, is goods radio active
- **cargoaircraftonly** - true/false, Cargo Aircraft Only
- **lineitems** - JSON object array of items

*dangerousgoods lineitems Object*
- **description** - string100
- **classordivision** - string50
- **unoridno** - string50
- **packinggroup** - string50
- **subsidaryrisk** - string50
- **packing** - string50
- **packinginstr** - string50
- **authorization** - string50

## Return format
A JSON object with the created shipment details.

- **carrier** - internal system carrier identifier
- **carriername** - courier provider name
- **isfreightforward** - is shipment a freight forward shipment.
- **message** - action outcome message
- **errors** - JSON string array of error messages
- **siteid** -  account site id
- **consignments** - JSON object arrary or consignments created. Generally only 1 is created, however were a split is required, more could be created

**Consignment**
- **connote** - courier provider connote number
- **trackingurl** - direct url to track and trace this shipment
- **cost** - total cost of shipment, excluding taxes
- **carriertype** - internal carrier classification
- **issaturdaydelivery** - true/false - whether saturday delivery service was applied
- **isrural** - true/false - whether delivery identified as rural
- **isovernight** - true/false - whether service is overnight where delivery is inter island
- **hastrackpaks** - true/false - whether this shipment has any trackpaks on it
- **outputs** - JSON object byte array with requested output file as Base64 encoded string

***


## Example
**Request**

    http://api.gosweetspot.com/ratesqueryv1/createandprint

*Headers*

    access_key: [access_key_for_site_account]
    Content-Type: application/json; charset=utf-8



*Body*
``` json
{
  "QuoteId": "f2878ff8-107f-4ca3-9d2e-3ca943b6d808",
  "DeliveryReference": "ORDER123",
  "Destination": {
    "Id": 0,
    "Name": "DestinationName",
    "Address": {
      "BuildingName": "",
      "StreetAddress": "DestinationStreetAddress",
      "Suburb": "Avonside",
      "City": "Christchurch",
      "PostCode": "8061",
      "CountryCode": "NZ"
    },
    "ContactPerson": "DestinationContact",
    "PhoneNumber": "123456789",
    "Email": "destinationemail@email.com",
    "DeliveryInstructions": "Desinationdeliveryinstructions"
  },
  "IsSaturdayDelivery": false,
  "IsSignatureRequired": true,
  "Packages": [
    {
      "Height": 1,
      "Length": 1,
      "Id": 0,
      "Width": 10,
      "Kg": 0.1,
      "Name": "GSS-DLE SATCHEL",
      "PackageCode": "DLE",
      "Type": "Box"
    }
  ],
  "PrintToPrinter": true,
  "Commodities": [
    {
      "Description": "Apparel",
      "UnitKg": 0.33,
      "UnitValue": 1,
      "Units": 1,
      "Country": "NZ",
      "Currency": "NZD"
    }
  ],
  "Outputs": [
    "LABEL_PDF"
  ]
}



```


**Response**

``` json
{
  "CarrierId": 102,
  "CarrierName": "Post Haste",
  "IsFreightForward": false,
  "IsOvernight": true,
  "IsSaturdayDelivery": false,
  "IsRural": false,
  "HasTrackPaks": true,
  "Message": "Connote created and print queued.",
  "Errors": [

  ],
  "SiteId": 4180,
  "Consignments": [
    {
      "Connote": "SSPOT012864",
      "TrackingUrl": "http://gosweetspot.com/track/4180-SSPOT012864",
      "Cost": 9.22,
      "CarrierType": 13,
      "IsSaturdayDelivery": false,
      "IsRural": false,
      "IsOvernight": true,
      "HasTrackPaks": true,
      "ConsignmentId": 1830322,
      "OutputFiles": {
        "LABEL_PDF": [
          "JVBERi0xLjQKJdP0zOEKMSAwIG9iago8PAovQ3JlYXRpb25EYXRlKEQ6MjAxNDA3MDEyMTU4NTYrMTInMDAnKQovVGl0bGUoU3dlZXRTcG90IEZyZWlnaHQgTGFiZWwpCi9BdXRob3IoU3dlZXRTcG90R3JvdXAuY28ubnopCi9TdWJqZWN0KExhYmVsKQovS2V5d29yZHMoU3dlZXRTcG90R3JvdXAgU3dlZXRTcG90KQovQ3JlYXRvcihQREZzaGFycCAxLjMyLjMwNTctZyBcKHd3dy5wZGZzaGFycC5uZXRcKSkKL1Byb2R1Y2VyKFBERnNoYXJwIDEuMzIuMzA1Ny1nIFwod3d3LnBkZnNoYXJwLm5ldFwpKQo+PgplbmRvYmoKMiAwIG9iago8PAovVHlwZS9DYXRhbG9nCi9QYWdlcyAzIDAgUgo+PgplbmRvYmoKMyAwIG9iago8PAovVHlwZS9QYWdlcwovQ291bnQgMQovS2lkc1s0IDAgUl0KPj4KZW5kb2JqCjQ3ysLfKWv7dlnb10vCb+23PO2u9ZOHQ0ZQ\/evR+A\/Pdtm58bUSMtj2mcHSsE2a1h\/fGzSC9J8Nxt6zWp6vrug\/Tu69kJOzkl2W3YlU+Zkz5\/i5bxVFn\/LasmW7sJ79xo1bubkr3Vkjz2\/frqXeyxs8fvxkEBS9ClD0gCGsCVVZC8lnUR\/FhYKt6PXpEaQ07tWDqukpPeZk1+pkMrlq1NhxuIyVL4p\/nn2WXy+j9q2DYWJyyYeB4W2Xfvglcab2D+Ub0VPhT1OWX7rcfSyeaJl7bNk26xQW3w+W18mO0ZmmdsPAAAAAMCqUBgFHIo0y8XxBQAAAAAA64LCKDCjsy\/99RGxWBxfAAAAAACwTiiMAjMq2pW+Oo\/xOL4AAAAAAGCdUBgFAAAAAAAAsONQGAUAAAAAAACw41AYBQAAAAAAALDjUBgFAAAAAAAAsONQGAUAAAAAAACw41AYBQAAAAAAALDjUBgFAAAAAAAAsONQGAUAAAAAAACw41AYBQAAAAAAALDjUBgFAAAAAAAAsONQGAUAAAAAAACw41AYBQAAAAAAALDjUBgFAAAAAAAAsONQGAUAAAAAAACw41AYBQAAAAAAALDjUBgFAAAAAAAAsONQGAUAAAAAAACw41AYBQAAAAAAALDjUBgFAAAAAAAAsONQGAUAAAAAAACw41AYBQAAAAAAALDjUBgFAAAAAAAAsONQGAUAAAAAAACw41AYBQAAAAAAALDjUBgFAAAAAAAAsONQGAUAAAAAAACw41AYBQAAAAAAALDDTCb\/P2S20KsfKAYdAAAAAElFTkSuQmCC"
        ]
      }
    }
  ],
  "Downloads": [

  ],
  "CarrierType": 13,
  "AlertPath": null
}
```
Base64 encoding string has been trimmed for presentation.