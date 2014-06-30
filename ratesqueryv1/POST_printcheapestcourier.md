# Print Cheapest Courier 

    POST ratesquery/printcheapestcourier

## Description
This interface allows you to create a shipment with the cheapest courier option available.

Upon creating the shipment, the print command can also be automatically initiated. This print command requires the Print Agent to be installed and available.

***

## Requires authentication
* A valid access Key must be provided in **access_key** request header.

***

## Parameters
- **origin** - Specify the unique order number from your source system, that was used as the packing slip no when the order was published
- **destination** - recepient name
- **packages** - address line 1, eg, building identifier, like Level 1, Fisher House, etc.
- **issaturdaydelivery** - address line 2, street name
- **issignaturerequired** - suburb name
- **dutiesandtaxesbyreceiver**  - post code, for NZ addresses, this can be left blank if unknown.
- **ruraloverride** - ISO Alpha 2 country code, eg NZ, AU, US, UK, CN
- **deliveryreference** - Order number, or customer reference for this order. 
- **printtoprinter** - yes/no - if supplied, the print job is sent to the *access_token* desingated printer. For testing purpose this can be provided as "no"

*origin/destination Object*
- **name** - string:60, Company name or persons name
- **contactperson** - string:60, contact person at address, optional
- **phonenumber** - string:60, phone number, optional
- **isrural** - true/false, optional
- **deliveryinstructions** - string:120, special instructions to print on the label. For origin this field is ignored.
- **sendtrackingemail** - true/false, optional
- **costcenterid** - int, optional
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
- **length** - decimal, package length in centimetres
- **width** - decimal, package width in centimetres
- **height** - decimal, package height  in centimetres
- **kg** - decimal, package weight in kilograms
- **type** - string:10, package type, eg, Box, Carton, Satchel, Bag, Pallet, etc
- **packagecode** - string:5, Trackpack codes, such as DLE, A5, A4 (please consult support before providing a value in this field) This feature is not available on all accounts.


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

***

## Errors
If there are any validation errors these are reported via the *errors* property.

***

## Example
**Request**

    http://api.gosweetspot.com/ratesqueryv1/printcheapestcourier

*Headers*

    access_key: [access_key_for_site_account]
    Content-Type: application/json; charset=utf-8

    

*Body*
``` json
{
	"Origin": null,
	"Destination": {
		"Id": 0,
		"Name": "DestinationName",
		"Address": {
			"BuildingName": "",
			"StreetAddress": "DestinationStreetAddress",
			"Suburb": "Oamaru",
			"City": "Oamaru",
			"PostCode": "7980",
			"CountryCode": "NZ"
		},
		"Email": "destinationemail@email.com",
		"ContactPerson": "DestinationContact",
		"PhoneNumber": "123456789",
		"IsRural": false,
		"DeliveryInstructions": "Desinationdeliveryinstructions",
		"SendTrackingEmail": false,
		"CostCentreId": 0,
		"ExplicitNotRural": false
	},
	"Packages": [
		{
			"Id": 0,
			"Name": "Custom",
			"Length": 10,
			"Width": 10,
			"Height": 10,
			"Kg": 10,
			"Type": “Box”,
			“PackageCode” : “A5”
		}
	],
	"IsSaturdayDelivery": false,
	"IsSignatureRequired": true,
	"IsUrgentCouriers": false,
	"DutiesAndTaxesByReceiver": false,
"RuralOverride": false,
    "DeliveryReference": "ORDER123"
}
```


**Response** 
``` json
{
	"CarrierId": 201,
	"CarrierName": "NZPost Easytrack",
	"IsFreightForward": false,
	"Message": "Connote create and print queued.",
	"Errors": [],
	"SiteId": 108633,
	"Consignments": [
		{
			"Connote": "NP8102203",
			"TrackingUrl": "http://gosweetspot.com/track/108633-NP8102203",
			"Cost": 16.25,
			"CarrierType": 21,
			"IsSaturdayDelivery": false,
			"IsRural": false,
			"IsOvernight": false,
			"HasTrackPaks": false
		}
	]
}
```
