# Nextshipping API Documentation

Implementation documentation for the Nextshipping logistics service

# Changelog
|Date|Description|
|---|---|
|21.1.2020|Initial release|

# Table of contents

- [Nextshipping API Documentation](#nextshipping-api-documentation)
- [Changelog](#changelog)
- [Table of contents](#table-of-contents)
- [Generic](#generic)
- [Abbreviations and terms used in the service description](#abbreviations-and-terms-used-in-the-service-description)
- [Environments](#environments)
  * [Testing](#testing)
  * [Production](#production)
- [Request Authentication](#request-authentication)
  * [Hmac Example](#hmac-example)
- [General APIs](#general-apis)
  * [List available shipping methods](#list-available-shipping-methods)
    + [Request](#request)
    + [Response](#response)
  * [Get information about shipping method](#get-information-about-shipping-method)
    + [Request](#request-1)
    + [Response](#response-1)
  * [List avaible additional services [OBSOLETE]](#list-avaible-additional-services--obsolete-)
    + [Request](#request-2)
    + [Response](#response-2)
  * [Search pickup points](#search-pickup-points)
    + [Request](#request-3)
    + [Response](#response-3)
    + [Defining a pickup point in Prinetti API](#defining-a-pickup-point-in-prinetti-api)
  * [Get info about a single pickup point](#get-info-about-a-single-pickup-point)
    + [Request](#request-4)
    + [Response](#response-4)
  * [Get Shipment status](#get-shipment-status)
    + [Request](#request-5)
    + [Response](#response-5)
    + [Shipment status codes](#shipment-status-codes)
  * [Cancel shipment](#cancel-shipment)
    + [Request](#request-6)
    + [Cancel shipment](#cancel-shipment-1)
    + [Response](#response-6)
  * [Callback](#callback)
    + [Server request](#server-request)
    + [Client response](#client-response)
- [Prinetti API](#prinetti-api)
  * [Create Shipment](#create-shipment)
    + [Minimal XML example](#minimal-xml-example)
    + [Full example with additional services](#full-example-with-additional-services)
    + [Request](#request-7)
  * [Response](#response-7)
    + [Response](#response-8)
  * [Fetch Shipping Label](#fetch-shipping-label)
    + [Request](#request-8)
    + [Response](#response-9)
- [Information and support](#information-and-support)

# Generic
This document describes APIs available to Nextshippings resellers and customers.

All Nextshipping specific APIs are implemeted using REST like schema to post or get information. Usage of the API requires contract between user and Nextshipping and all features might not be available to every user.

All data sent should be UTF-8 unless mentioned otherwise.

# Abbreviations and terms used in the service description
|Term|Description|
|---|---|
|API|Application programming interface. See more information from https://en.wikipedia.org/wiki/Application_programming_interface|
|SHA256|SHA-256 is a hash functions computed with 32-bit words. See more information from https://en.wikipedia.org/wiki/SHA-2|
|HMAC|Keyed-hash message authentication code.See more information from https://en.wikipedia.org/wiki/Hash-based_message_authentication_code|
|AN|Alphanumeric - Digit or a number|
|UNIX TIME|Unix time is defined as the number of seconds that have elapsed since 00:00:00 Coordinated Universal Time (UTC), Thursday, 1 January 1970. See more information from https://en.wikipedia.org/wiki/Unix_time|
|PRINETTI|Prinetti is address label printing software for Posti's domestic and international deliveries. Posti Group Oyj owns trademark "Prinetti" (Trademark number: 227572)|
|UUID|Universal unique identifier. See more information from https://en.wikipedia.org/wiki/Universally_unique_identifier|
|URL|Uniform Resource Locator. See more information from https://en.wikipedia.org/wiki/Uniform_Resource_Locator|

# Environments
## Testing

Not available at the moment

## Production
|Parameter|Value|Type|
|---|---|---|
|API endpoint|https://nextshipping.posti.fi/|URL|
|Merchant API key||STRING|
|Merhcnat secret||AN 80|

# Request Authentication
With the exception of Prinetti API all request are signed with an sha256 hmac calculated by using the request parameters, sorted alphabetically by parameter key and separated by an &, as the message. Calculated HMAC is placed in the hash -parameter.

## Hmac Example

```php
$secret = 'a1b2c3d4e5f6';

$post_params = [
	'name'  => 'Some name',
	'date'  => '2016-02-01',
	'char'  => 'C',
];

ksort($post_params);

$post_params['hash'] =  hash_hmac('sha256', join('&', $post_params), $secret);


$ch = curl_init($url);
curl_setopt($ch, CURLOPT_POST, 1);
curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query($post_params));
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);

$output = curl_exec($ch);
```

# General APIs
## List available shipping methods
### Request
Lists shipping methods currently supported by Nextshipping.fi API. Use the shipping_method_code in <Consignment.Product></Consignment.Product> element to choose your shipping method.
POST: /shipping-methods/list
|Param name|Type|Required|Content|
|---|---|---|---|
|api_key|UUID|x||	
|timestamp|UNIX TIME|x||
|language|AN2|FI, SE or EN. If not specified, FI is assumed.|
|hash|AN64|x||

Example:
```php
$post_params = [
	'api_key' 		=> '00000000-0000-0000-0000-000000000000',
	'timestamp' 	=> time(),
];

$post_params['hash'] =  hash_hmac('sha256', join('&', $post_params), $secret);
```

### Response
Maximum dimensions for icon are 120px width and 55px height.

Example:
```json
[{
	"name": "Bussipaketti",
	"shipping_method_code": 90010,
	"description": null,
	"service_provider": "Matkahuolto",
	"supported_countries": ["FI", "AX"],
	"has_pickup_points": false,
	"home_delivery": false,
	"additional_services": [{
		"name": "Postiennakko",
		"service_code": "3101",
		"specifiers": [{
			"name": "amount",
			"type": "number",
			"step": "0.01",
			"min": "0"
		}, {
			"name": "account",
			"type": "text"
		}, {
			"name": "reference",
			"type": "text"
		}, {
			"name": "codbic",
			"type": "text"
		}]
	}, {
		"name": "S\u00e4rkyv\u00e4",
		"service_code": "3104",
		"specifiers": null
	}],
	"icon": "https:\/\/nextshipping.posti.fi\/logos\/matkahuolto\/logo.jpg"
}, {
	"name": "EMS",
	"shipping_method_code": 2017,
	"description": null,
	"service_provider": "Posti",
	"supported_countries": [],
	"has_pickup_points": false,
	"home_delivery": false,
	"additional_services": [],
	"icon": "https:\/\/nextshipping.posti.fi\/logos\/posti\/1.1_Posti_logo_Posti_Orange_rgb.png"
}, {
	"name": "Euro Business",
	"shipping_method_code": 70010,
	"description": null,
	"service_provider": "GLS",
	"supported_countries": ["EE", "HU", "CZ", "DK", "CH", "SI", "SK", "RS", "DE", "SE", "RO", "FR", "PL", "PT", "NO", "MT", "LU", "LT", "LV", "HR", "GR", "AT", "IT", "GB", "IE", "NL", "ES", "BG", "BE", "CY"],
	"has_pickup_points": false,
	"home_delivery": false,
	"additional_services": []
}, {
	"name": "Euro Business Pallet",
	"shipping_method_code": 70030,
	"description": null,
	"service_provider": "GLS",
	"supported_countries": [],
	"has_pickup_points": false,
	"home_delivery": false,
	"additional_services": []
}, {
	"name": "Express",
	"shipping_method_code": 70050,
	"description": null,
	"service_provider": "GLS",
	"supported_countries": [],
	"has_pickup_points": false,
	"home_delivery": false,
	"additional_services": []
}, {
	"name": "Express-paketti",
	"shipping_method_code": 2102,
	"description": null,
	"service_provider": "Posti",
	"supported_countries": ["FI", "AX"],
	"has_pickup_points": false,
	"home_delivery": true,
	"additional_services": [],
	"icon": "https:\/\/nextshipping.posti.fi\/logos\/posti\/1.1_Posti_logo_Posti_Orange_rgb.png"
}, {
	"name": "Global Economy",
	"shipping_method_code": 70070,
	"description": null,
	"service_provider": "GLS",
	"supported_countries": [],
	"has_pickup_points": false,
	"home_delivery": false,
	"additional_services": []
}, {
	"name": "Jakopaketti",
	"shipping_method_code": 90030,
	"description": null,
	"service_provider": "Matkahuolto",
	"supported_countries": ["FI", "AX"],
	"has_pickup_points": false,
	"home_delivery": true,
	"additional_services": [{
		"name": "Postiennakko",
		"service_code": "3101",
		"specifiers": [{
			"name": "amount",
			"type": "number",
			"step": "0.01",
			"min": "0"
		}, {
			"name": "account",
			"type": "text"
		}, {
			"name": "reference",
			"type": "text"
		}, {
			"name": "codbic",
			"type": "text"
		}]
	}, {
		"name": "S\u00e4rkyv\u00e4",
		"service_code": "3104",
		"specifiers": null
	}],
	"icon": "https:\/\/nextshipping.posti.fi\/logos\/matkahuolto\/logo.jpg"
}, {
	"name": "Kotipaketti",
	"shipping_method_code": 2104,
	"description": null,
	"service_provider": "Posti",
	"supported_countries": ["FI", "AX"],
	"has_pickup_points": false,
	"home_delivery": true,
	"additional_services": [{
		"name": "LQ L\u00e4hetys",
		"service_code": "3143",
		"specifiers": null
	}, {
		"name": "Postiennakko",
		"service_code": "3101",
		"specifiers": [{
			"name": "amount",
			"type": "number",
			"step": "0.01",
			"min": "0"
		}, {
			"name": "account",
			"type": "text"
		}, {
			"name": "reference",
			"type": "text"
		}, {
			"name": "codbic",
			"type": "text"
		}]
	}, {
		"name": "S\u00e4rkyv\u00e4",
		"service_code": "3104",
		"specifiers": null
	}, {
		"name": "Suuri",
		"service_code": "3174",
		"specifiers": null
	}],
	"icon": "https:\/\/nextshipping.posti.fi\/logos\/posti\/1.1_Posti_logo_Posti_Orange_rgb.png"
}, {
	"name": "L\u00e4hi-\/Verkkopaketti",
	"shipping_method_code": 90080,
	"description": null,
	"service_provider": "Matkahuolto",
	"supported_countries": ["FI", "AX"],
	"has_pickup_points": true,
	"home_delivery": false,
	"additional_services": [{
		"name": "Noutopiste",
		"service_code": "2106",
		"specifiers": [{
			"name": "pickup_point_id",
			"type": "number"
		}]
	}],
	"icon": "https:\/\/nextshipping.posti.fi\/logos\/matkahuolto\/logo.jpg"
}, {
	"name": "Noutopistepaketti",
	"shipping_method_code": 80010,
	"description": null,
	"service_provider": "DB Schenker",
	"supported_countries": ["FI", "AX"],
	"has_pickup_points": true,
	"home_delivery": false,
	"additional_services": [{
		"name": "Noutopiste",
		"service_code": "2106",
		"specifiers": [{
			"name": "pickup_point_id",
			"type": "number"
		}]
	}],
	"icon": "https:\/\/nextshipping.posti.fi\/logos\/dbschenker\/DB_Schenker_Noutopiste_-logo.png"
}, {
	"name": "Palautus",
	"shipping_method_code": 2108,
	"description": "",
	"service_provider": "Posti",
	"supported_countries": ["FI", "AX"],
	"has_pickup_points": false,
	"home_delivery": false,
	"additional_services": [{
		"name": "LQ L\u00e4hetys",
		"service_code": "3143",
		"specifiers": null
	}],
	"icon": "https:\/\/nextshipping.posti.fi\/logos\/posti\/1.1_Posti_logo_Posti_Orange_rgb.png"
}, {
	"name": "Pikkupaketti",
	"shipping_method_code": 2461,
	"description": null,
	"service_provider": "Posti",
	"supported_countries": ["FI", "AX"],
	"has_pickup_points": false,
	"home_delivery": true,
	"additional_services": [],
	"icon": "https:\/\/nextshipping.posti.fi\/logos\/posti\/1.1_Posti_logo_Posti_Orange_rgb.png"
}, {
	"name": "Postipaketti",
	"shipping_method_code": 2103,
	"description": null,
	"service_provider": "Posti",
	"supported_countries": ["FI", "AX"],
	"has_pickup_points": true,
	"home_delivery": false,
	"additional_services": [{
		"name": "Henkil\u00f6kohtaisesti luovutettava",
		"service_code": "3163",
		"specifiers": null
	}, {
		"name": "LQ L\u00e4hetys",
		"service_code": "3143",
		"specifiers": null
	}, {
		"name": "Noutopiste",
		"service_code": "2106",
		"specifiers": [{
			"name": "pickup_point_id",
			"type": "number"
		}]
	}, {
		"name": "Postiennakko",
		"service_code": "3101",
		"specifiers": [{
			"name": "amount",
			"type": "number",
			"step": "0.01",
			"min": "0"
		}, {
			"name": "account",
			"type": "text"
		}, {
			"name": "reference",
			"type": "text"
		}, {
			"name": "codbic",
			"type": "text"
		}]
	}, {
		"name": "S\u00e4hk\u00f6inen saapumisilmoitus",
		"service_code": "3139",
		"specifiers": null
	}, {
		"name": "S\u00e4ilytysajan pidennys",
		"service_code": "3165",
		"specifiers": null
	}, {
		"name": "S\u00e4rkyv\u00e4",
		"service_code": "3104",
		"specifiers": null
	}, {
		"name": "Suuri",
		"service_code": "3174",
		"specifiers": null
	}],
	"icon": "https:\/\/nextshipping.posti.fi\/logos\/posti\/1.1_Posti_logo_Posti_Orange_rgb.png"
}, {
	"name": "Priority",
	"shipping_method_code": 2015,
	"description": null,
	"service_provider": "Posti",
	"supported_countries": [],
	"has_pickup_points": false,
	"home_delivery": false,
	"additional_services": [],
	"icon": "https:\/\/nextshipping.posti.fi\/logos\/posti\/1.1_Posti_logo_Posti_Orange_rgb.png"
}]
```

## Get information about shipping method

### Request
Get informationa bout a shipping method. This information is the same as in list-shipping-methods but returns information about the selected one.

POST: /shipping-methods/get
|Param name|Type|Required|Content|
|---|---|---|---|
|api_key|UUID|x||
|timestamp|UNIX TIME|x||
|shipping_method_code|N 6|x||
|language|AN 2||FI, SE or EN. If not specified, FI is assumed.|
|hash|AN 64|x||

Example:
```php

$post_params = [
	'api_key' 		=> '00000000-0000-0000-0000-000000000000',
	'shipping_method_code'	=> 90010,
	'timestamp' 	=> time(),
];

$post_params['hash'] =  hash_hmac('sha256', join('&', $post_params), $secret);
```

### Response

Maximum dimensions for icon are 120px width and 55px height.

Example:
```json
{
	"name": "Bussipaketti",
	"shipping_method_code": 90010,
	"description": null,
	"service_provider": "Matkahuolto",
	"supported_countries": ["FI", "AX"],
	"has_pickup_points": false,
	"home_delivery": false,
	"additional_services": [{
		"name": "Postiennakko",
		"service_code": "3101",
		"specifiers": [{
			"name": "amount",
			"type": "number",
			"step": "0.01",
			"min": "0"
		}, {
			"name": "account",
			"type": "text"
		}, {
			"name": "reference",
			"type": "text"
		}, {
			"name": "codbic",
			"type": "text"
		}]
	}, {
		"name": "S\u00e4rkyv\u00e4",
		"service_code": "3104",
		"specifiers": null
	}],
	"icon": "https:\/\/nextshipping.posti.fi\/logos\/matkahuolto\/logo.jpg"
}
```

## List avaible additional services [OBSOLETE]

Lists possible additional services that are available for a shipping method. 
Obsolete information

Information this API provides can be found from "list available shipping methods" or by "get information about shipping method".

### Request

POST: /additional-services/list

|Param name|Type|Required|Content|
|---|---|---|---|
|api_key|UUID|x||	
|timestamp|UNIX TIME|x||	
|language|AN 2||FI, SE or EN. If not specified, FI is assumed.|
|hash|AN 64|x||
List shipping methods 
```php
$post_params = [
	'api_key' 		=> '00000000-0000-0000-0000-000000000000',
	'timestamp' 	=> time(),
];

$post_params['hash'] =  hash_hmac('sha256', join('&', $post_params), $secret);
```

### Response
Response 
```json
[
	{
		"shipping_method_code":2103,
		"name":"Postiennakko",
		"service_code":"3101",
		"description":""
	},
	{
		"shipping_method_code":2103,
		"name":"Särkyvä",
		"service_code":"3104",
		"description":"Erilliskäsiteltävä"
	},
	{
		"shipping_method_code":90010,
		"name":"Särkyvä",
		"service_code":"3104",
		"description":"Varoen käsiteltävä"
	}
]
```
## Search pickup points
List possible pickup points for the delivery.
### Request

POST: /pickup-points/search

|Param name     |Type   |Required       |Content|
|---|----|----|----|
|api_key        |UUID   |x      ||
|address        |AN 400 |       ||
|postcode       |AN 5   |       ||
|country        |AN 2   |       |ISO 3166 style. If not specified, FI is used.|
|service_provider       |AN 10  |       |Comma separeted list of products. Product codes can be obtained using "list available shipping methods" API call.|
|limit  |N      |       |Limit search results. Defaults to 5. Possible values are from 1 to 15.|
|query  |AN     |       |Address string query. If used address and postcode params are ignored and geolocation search is done by just using this string.|
|timestamp      |UNIX TIME      |x      ||
|hash   |AN 64  |x      ||

Example:
```php
$post_params = [
    'api_key'       => $api_key,
    'address'       => 'Itsenäisyydenkatu 2'
    'postcode'      => '33100',
    'timestamp'     => time()
];

ksort($post_params);

$post_params['hash'] =  hash_hmac('sha256', join('&', $post_params), $secret);
```

Example:
```php
// you can also use the just query param 
$post_params = [
    'api_key'       => $api_key,
    'query'         => 'Itsenäisyydenkatu 2, 33100 Tampere',
    'timestamp'     => time()
];

ksort($post_params);

$post_params['hash'] =  hash_hmac('sha256', join('&', $post_params), $secret);
```

### Response
Maximum dimensions for provider logo are 120px width and 55px height.

```json
[{
	"provider": "Posti",
	"pickup_point_id": 356,
	"name": "Posti, Keskusta",
	"street_address": "Tullikatu 6",
	"postcode": "33100",
	"city": "TAMPERE",
	"country": "FI",
	"description": "ma-pe 8.00 - 20.00, la 10.00 - 15.00",
	"provider_logo": "https://nextshipping.posti.fi/icons/posti.png",
	"map_longitude": "23.777128",
	"map_latitude": "61.4978247"
}, {
	"provider": "Matkahuolto",
	"pickup_point_id": "5410",
	"name": "SIWA RAUTATIEKATU\/TAMPERE",
	"street_address": "RAUTATIENKATU 25",
	"postcode": "33100",
	"city": "TAMPERE",
	"country": "FI",
	"description": null,
	"provider_logo": "https://nextshipping.posti.fi/icons/mh.png",
	"map_longitude": null,
	"map_latitude": null
},  {
	"provider": "Matkahuolto",
	"pickup_point_id": "5038",
	"name": "SIWA AMURI",
	"street_address": "PUUVILLATEHTAANK. 12",
	"postcode": "33210",
	"city": "TAMPERE",
	"country": "FI",
	"description": null,
	"provider_logo": "https://nextshipping.posti.fi/icons/mh.png",
	"map_longitude": null,
	"map_latitude": null
}, {
	"provider": "DB Schenker",
	"pickup_point_id": "6679",
	"name": "R-KIOSKI TRE MINIMARKET",
	"street_address": "KYLLIKINKATU 11",
	"postcode": "33100",
	"city": "TAMPERE",
	"country": null,
	"description": "V0730-2200L0730-2200S0900-2200",
	"provider_logo": "https://nextshipping.posti.fi/icons/db.png",
	"map_longitude": null,
	"map_latitude": null
}]
```

### Defining a pickup point in Prinetti API

In the original Prinetti API SmartPOST product code (2106) was used, along with using the pickup points address as receiver, to direct a packet to a parcel locker of choice. With Nextshipping.fi implementation this method is obsolete and would only work when the parcel was shipped with Posti. To accommodate multiple parcel service providers a new AdditionalService.ServiceCode (2106) is added to mark a parcel going to alternate pickup point, with the pickup point id as AdditionalService.Specifier.

If you have implemented a pickup point search API directly with a packet service provider supported by Nextshipping.fi (Matkahuolto for example) you can use the point id fetched via your implementation. With the pickup point API Nextshipping.fi functions only as a technical wrapper to different parcel service provider between you and the service provider. 

Additional Service example:
```xml
<Consignment.AdditionalService>
    <AdditionalService.ServiceCode>2106</AdditionalService.ServiceCode>
    <AdditionalService.Specifier name="pickup_point_id">5410</AdditionalService.Specifier>
</Consignment.AdditionalService>
```

## Get info about a single pickup point 
### Request

POST: /pickup-point/info

|Param name	|Type	|Required	|Content|
|---	|---	|---	|---	|
|api_key	|UUID|	x|	|	|
|point_id	|AN 9|	x|Id of the pick up point|
|service_code	|AN 5|	x|	|Service product code. If not specified, service_provider must be specified.|
|service_provider	|AN 50	|x	|Obsolete: Limits search to a service provider(s), possible values can be| obtained using "list available shipping methods" API call. If empty, service_code must be specified and will be used.|
|timestamp	|UNIX TIME	|x	||
|hash	|AN 64	|x	||

Example:
```php
$url = 'https://nextshipping.posti.fi/pickup-point/info';
$secret = '1234567890ABCDEF';

$post_params = [
	'api_key'          => $api_key,
	'point_id'         => '905253201',
	'service_code'     => '2103',
	'service_provider' => 'Posti',
	'timestamp' => time(),
];

ksort($post_params);

$post_params['hash'] =  hash_hmac('sha256', join('&', $post_params), $secret);

$ch = curl_init($url);
curl_setopt($ch, CURLOPT_POST, 1);
curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query($post_params));
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
$output = curl_exec($ch);
```

### Response
Maximum dimensions for provider logo are 120px width and 55px height.
Response 
```json
{
	"provider": "Posti",
	"provider_code": "Posti",
	"provider_logo": "https:\/\/nextshipping.posti.fi\/logos\/posti\/1.1_Posti_logo_Posti_Orange_rgb.png",
	"pickup_point_id": "905253201",
	"name": "Pakettiautomaatti, Sale Toppila",
	"street_address": "Paakakatu 2",
	"postcode": "90525",
	"city": "OULU",
	"country": "FI",
	"description": "ma-la 7.00 - 22.00, su 9.00 - 22.00",
	"map_longitude": "25.433573289",
	"map_latitude": "65.045625281",
	"point_type": "PARCEL_LOCKER"
}
```

## Get Shipment status
### Request

POST: /shipment/status

|Param name	|Type	|Required|
|---	|---	|---	|
|api_key	|UUID	|x	|
|tracking_code	|AN 100	|x	|
|timestamp	|UNIX TIME	|x	|
|hash	|AN 64	|x	|

Example:
```php
$post_params = [
    'api_key'       => $api_key,
    'tracking_code' => 'JJFI64574900000137203',
    'timestamp'     => time()
];

ksort($post_params);

$post_params['hash'] =  hash_hmac('sha256', join('&', $post_params), $secret);
```

### Response

Example:
```json
[
	{
		"message_created":"2016-09-30 17:37:00",
		"reference":"1475233831",
		"tracking_code":"JJFI64574900000137203",
		"measured_weight":"0",
		"measured_volume":"0",
		"status_code":"68",
		"receiver_name":"",
		"event_timestamp":"2016-09-30 17:34:00",
		"postcode":"87700",
		"post_office":"KAJAANI"
	}
]
```
### Shipment status codes

Status codes are most part same as described in implementation guide of IFTSTA transport status documentation by Posti.

|Status Code    |Message FI     |Message EN|
|---|---|---|
|13     |Lähetys on noudettu lähettäjältä       |Item is collected from sender - picked up|
|20     |Lähetylle on tehty poikkeama   |Exception|
|22     |Lähetys on luovutettu vastaanottajalle |Item has been handed over to the recipient|
|31     |Lähetys on kuljetuksessa       |Item is in transport|
|38     |Lähetykseen liittyvä postiennakkomaksu on suoritettu   |C.O.D payment is paid to the sender|
|45     |Lähetyksestä on lähetetty saapumisilmoitus     |Informed consignee of arrival|
|48     |Lähetys on lastattu runkokuljetukseen  |Item is loaded onto a means of transport|
|56     |Lähetystä ei ole toimitettu – luovutusyritys on tehty  |Item not delivered – delivery attempt made|
|68     |Lähetyksen EDI-tiedot vastaanotettu lähettäjältä       |Pre-information is received from sender|
|71     |Lähetys on lastattu jakelukuljetukseen |Item is ready for delivery transportation|
|77     |Lähetys palautuu lähettäjälle  |Item is returning to the sender|
|91     |Lähetys on saapunut postitoimipaikkaan |Item is arrived to a post office|
|99     |Lähetys lähdössä ulkomaille    |Outbound|

## Cancel shipment

Cancels the shipment. Also refunds customers if it's possible. Only shipments that haven't been sent, can be cancelled.

### Request

POST: /shipment/cancel

|Param name     |Type   |Required       |Content|
|---|---|---|---|
|api_key        |UUID   |x      ||
|shipment_id    |UUID   |x      |Original shipments UUID (as in response.reference@uuid)|
|timestamp      |UNIX TIME      |x      ||
|hash   |AN 64  |x      ||

### Cancel shipment 

Example:
```php
$post_params = [
    'api_key'       => $api_key,
    'shipment_id'   => 'df202295-b950-46a9-8125-0e7921924620',
    'timestamp'     => time()
];

ksort($post_params);

$post_params['hash'] =  hash_hmac('sha256', join('&', $post_params), $secret);
```

### Response

Example:
```json
{
	"success": true
}
```

Example:
```
{
	"success": false
}
```

## Callback

Callback service is used to push notifications from the tracking data back to client system.

### Server request

POST: callback url

|Param name     |Type   |Required       |Details|
|---|---|---|---|
|api_key        |UUID   |x      ||
|tracking_code  |AN 100 |x      ||
|timestamp      |UNIX TIME      |x      ||
|event  |N      |x      |See "Shipment status codes"|
||
|object |JSON   |x      |JSON presentation of the event. Object is same which is returned from shipment/status API.|
|hash   |AN 64  |x      ||

Hash calculation example:
```php
$post_params = [
    'api_key' => $api_key,
    'tracking_code' => 'JJFI64574900000137203',
	'event'	=> 68,
	'object' => '{"message_created":"2016-09-30 17:37:00", "reference":"1475233831", "tracking_code":"JJFI64574900000137203", "measured_weight":"0", "measured_volume":"0", "status_code":"68", "receiver_name":"", "event_timestamp":"2016-09-30 17:34:00", "postcode":"87700", "post_office":"KAJAANI"}',
    'timestamp' => time()
];

ksort($post_params);

$post_params['hash'] =  hash_hmac('sha256', join('&', $post_params), $secret);
```

### Client response

Client have to respond with HTTP status code 200 and with a valid JSON -object. If the client system does not respond in 10 seconds or responds in some other than correct message and status server will try to send data again after 15 minutes. Server will try to send notification to client multiple times before it stops trying.

JSON response example:
```json
{
	"success": true
}
```

Full HTTP response example response:
```
HTTP/1.1 200 OK
Date: Tue, 12 Sep 2017 10:54:56 GMT
Server: Apache
Last-Modified: Tue, 12 Sep 2017 10:54:43 GMT
Accept-Ranges: bytes
Content-Length: 18
Connection: close
Content-Type: application/json

{
	"success": true
}
```

# Prinetti API

Prinetti API is an implementation of a service that was hosted by the Finnish post office but with some altered, missing  or added functionality.

A working implementation of a Prinetti client should be compatible with Nextshipping.fi API with without major changes. To use the Nextshipping.fi API the client must change the end point URLs to match the URLs below and set the client to use the Account id and Secret key provided by Nextshipping.fi.

Data can be sent as ISO-8859-1 or UTF-8. If using ISO-8859-1, XML encoding attribute has to be set correctly or result will be invalid XML document.

## Create Shipment

POST: /prinetti/create-shipment
 
|Element        |Presence       |Definition     |Data type      |Optional / Mandatory|
|---------------|---------------|---------------|---------------|--------------------|
|eChannel       |1      |Root element of the document   |       ||
|ROUTING        |1      |       |       ||
|Routing.Account        |1      |User account   |UUID   ||
|Routing.Key    |1      |Shared secret  |String ||
|Routing.Id     |1      |Unique id of the request       |Numeric        ||
|Routing.Time   |1      |Timestamp of the request, in format YYYYMMDDHHMMSS     |Timestamp      ||
|Shipment       |1      |       |       ||
|Shipment.Sender        |1      |Information about the Sender   |       ||
|Sender.Name1   |1      |Name of the Sender     |String ||
|Sender.Name2   |1      |Additional name of the Sender  |String ||
|Sender.Addr1   |1      |Address of the Sender  |String ||
|Sender.Addr2   |1      |Additional address of the Sender       |String ||
|Sender.Addr3   |1      |Additional address of the Sender       |String ||
|Sender.Postcode        |1      |Postcode of the Sender |String ||
|Sender.City    |1      |City of the Sender     |String ||
|Sender.Country |1      |Country of the Sender  |String ||
|Sender.Vatcode |1      |Vat code of the Sender |String ||
|Shipment.Recipient     |1      |       |       ||
|Recipient.Name1        |1      |       |String ||
|Recipient.Name2        |1      |       |String ||
|Recipient.Addr1        |1      |       |String ||
|Recipient.Addr2        |1      |       |       ||
|Recipient.Addr3        |1      |       |       ||
|Recipient.Postcode     |1      |       |String ||
|Recipient.City |1      |       |String ||
|Recipient.Country      |1      |2 letter country code  |String ||
|Recipient.Phone        |1      |       |String ||
|Recipient.Email        |1      |       |String ||
|Shipment.Consigment    |1      |       |       ||
|Consigment.Reference   |1      |       |Numeric        ||
|Consigment.Product     |1      |       |String ||
|Consignment.AdditionalService  |0...N  |Additional Services    |       ||
|AdditionalService.ServiceCode  |1      |       |String ||
|AdditionalService.Specifier[@name]     |0...N  |Attribute name can have values depending on the service code   |String ||
|Consigment.Parcel      |1...N  |       |       ||
|Parcel.Packagetype     |1      |       |String ||
|Parcel.Weight  |1      |       |Decimal        ||
|Parcel.Volume  |1      |       |Decimal        ||
|Parcel.ReturnService   |1      |Creates return label if specified. "2108" for Posti, "80020" for DB Schenker.  |Numeric        |O|
|Parcel.Contents        |1      |       |String ||
|Parcel.contentline     |0...N  |Description of the content, required for non-EU shipments      |       |O / M|
|contentline.description        |1      |Description of the content     |String |M|
|contentline.quantity   |1      |Quantity of the content        |Numeric        |M|
|contentline.currency   |1      |3 letter currency code, EUR    |String |M|
|contentline.netweight  |1      |Netweight in grams     |Numeric        |M|
|contentline.value      |1      |Value of the content in currency. Decimal point is "." |Decimal        |M|
|contentline.countryoforigin    |1      |2 letter country code, FI      |String |M|
|contentline.tariffcode |1      |Customs tariff -code, required for non-EU shipments    |Numeric        |O / M|

### Additional Services

|Service	|Code	|Attribute	|Type	|Description|
|---|---|---|---|---|
|Cash on delivery (Postiennakko, Bussiennakko)|	3101	|amount	N	|Amount in eurocents|
|||account|IBAN||
|||codbic|BIC||	
|||reference|AN|Finnish bank reference number with valid check digit|
|Multipacket service	|3102	|count	|N	||
|Shipment contract	|3103	|	|	|Not in use|
|Fragile	|3104		|	|	||
|Sähköinen saapumisilmoitus	|3139	|||		
|Henkilökohtaisesti luovutettava	|3164|||			
|Säilytysajan pidennys	|3165|||			
|Large shipment	|3174|||			
|Pickup location	|2106	|pickup_point_id	|N	|Pickup point ID fetched from the "Fetch shipping label" -API|
|LQ shipment|3143	|lqcount	|N	|Limited quantity of dangerous goods|
|||lqweight	|N	|Weight in kilograms|
|Tracking callback URL	|9901	|url	|URL	|URL where Nextshipping posts data when new tracking event occurs. See details from "Callback service".|
|Label code	|9902	|	|	|If defined, generates unique code for the label that can be used to send the shipment without printing the label. (Posti Easy Sending Code /Matkahuolto activation code) Using this feature can have performance impact.|

### Minimal XML example

A single packet, no additional services.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<eChannel>
    <ROUTING>
	<Routing.Account>00000000-0000-0000-0000-000000000000</Routing.Account>
	<Routing.Key>1234567890ABCDEF</Routing.Key>
	<Routing.Id>1464524676</Routing.Id>
	<Routing.Name>puitajamuttereita.fi</Routing.Name>
	<Routing.Time>20160529152436</Routing.Time>
    </ROUTING>
    <Shipment>
	<Shipment.Sender>
	    <Sender.Name1>Sender name</Sender.Name1>
	    <Sender.Addr1>Some address</Sender.Addr1>
	    <Sender.Postcode>33100</Sender.Postcode>
	    <Sender.City>Tampere</Sender.City>
	    <Sender.Country>FI</Sender.Country>
	    <Sender.Vatcode>1234567-8</Sender.Vatcode>
	</Shipment.Sender>
	<Shipment.Recipient>
	    <Recipient.Name1>Receiver name</Recipient.Name1>
	    <Recipient.Addr1>Some address</Recipient.Addr1>
	    <Recipient.Postcode>33100</Recipient.Postcode>
	    <Recipient.City>Tampere</Recipient.City>
	    <Recipient.Country>FI</Recipient.Country>
	    <Recipient.Phone>123456789</Recipient.Phone>
	    <Recipient.Email>someone@example.com</Recipient.Email>
	</Shipment.Recipient>
	<Shipment.Consignment>
	    <Consignment.Reference>123456879</Consignment.Reference>
	    <Consignment.Product>2103</Consignment.Product>
	    <Consignment.Parcel>
		<Parcel.Packagetype>PC</Parcel.Packagetype>
		<Parcel.Weight>1.2</Parcel.Weight>
		<Parcel.Volume>0.06</Parcel.Volume>
		<Parcel.Contents>Nuts and Bolts</Parcel.Contents>
	    </Consignment.Parcel>
	</Shipment.Consignment>
    </Shipment>
</eChannel>
```

### Full example with additional services

Multipacket shipment via Matkahuolto, cash on delivery. Has multiple elements from the original Prinetti API that are not used with Nextshipping.fi implementation.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<eChannel>

    <ROUTING>
	<Routing.Target>1</Routing.Target><!-- Ignored -->
	<Routing.Source>505</Routing.Source><!-- Ignored -->
	<Routing.Account>00000000-0000-0000-0000-000000000000</Routing.Account>
	<Routing.Key>1234567890ABCDEF</Routing.Key>
	<Routing.Id>1479032410</Routing.Id>
	<Routing.Name>Testisanoma</Routing.Name>
	<Routing.Time>20161113122010</Routing.Time>
	<Routing.Version>1</Routing.Version><!-- Ignored -->
	<Routing.Mode>0</Routing.Mode><!-- Ignored -->
	<Routing.Comment>Comment</Routing.Comment><!-- Ignored -->
    </ROUTING>

    <Shipment>
	<Shipment.Sender>
	    <Sender.Contractid>123456</Sender.Contractid><!-- Ignored -->
	    <Sender.Name1>Nuts and bolts Ltd</Sender.Name1>
	    <Sender.Name2></Sender.Name2>
	    <Sender.Addr1>Somestreet 123</Sender.Addr1>
	    <Sender.Addr2></Sender.Addr2>
	    <Sender.Addr3></Sender.Addr3>
	    <Sender.Postcode>04320</Sender.Postcode>
	    <Sender.City>Tuusula</Sender.City>
	    <Sender.Country>FI</Sender.Country>
	    <Sender.Phone></Sender.Phone>
	    <Sender.Vatcode></Sender.Vatcode>
	</Shipment.Sender>

	<Shipment.Recipient>
	    <Recipient.Code></Recipient.Code><!-- Ignored -->
	    <Recipient.Name1>John Doe</Recipient.Name1>
	    <Recipient.Name2></Recipient.Name2>
	    <Recipient.Addr1>Doe street 123 A 123</Recipient.Addr1>
	    <Recipient.Addr2></Recipient.Addr2>
	    <Recipient.Addr3></Recipient.Addr3>
	    <Recipient.Postcode>33100</Recipient.Postcode>
	    <Recipient.City>Tampere</Recipient.City>
	    <Recipient.Country>FI</Recipient.Country>
	    <Recipient.Phone>123123123</Recipient.Phone>
	    <Recipient.Vatcode></Recipient.Vatcode>
	    <Recipient.Email>john@doe.com</Recipient.Email>
	</Shipment.Recipient>

	<Shipment.Consignment>
	    <Consignment.Reference>3211479032410</Consignment.Reference>
	    <Consignment.Product>90010</Consignment.Product>
	    <Consignment.Contentcode>D</Consignment.Contentcode>
	    <Consignment.ReturnInstruction>E</Consignment.ReturnInstruction>
	    <Consignment.Invoicenumber/>
	    <Consignment.Merchandisevalue>100</Consignment.Merchandisevalue>

	    <Consignment.AdditionalService><!-- Cash on delivery, Postiennakko/Bussiennakko -->
		<AdditionalService.ServiceCode>3101</AdditionalService.ServiceCode>
		<AdditionalService.Specifier name="amount">150</AdditionalService.Specifier>
		<AdditionalService.Specifier name="account">FI2180000012345678</AdditionalService.Specifier>
		<AdditionalService.Specifier name="codbic">DABAFIHH</AdditionalService.Specifier>
		<AdditionalService.Specifier name="reference">12344</AdditionalService.Specifier>
	    </Consignment.AdditionalService>

			<Consignment.AdditionalService>
				<AdditionalService.ServiceCode>2106</AdditionalService.ServiceCode>
				<AdditionalService.Specifier name="pickup_point_id">5410</AdditionalService.Specifier>
			</Consignment.AdditionalService>

	    <Consignment.Currency>EUR</Consignment.Currency>
	    <Consignment.AdditionalInfo>
		<AdditionalInfo.Text>Puita ja muttereita</AdditionalInfo.Text>
	    </Consignment.AdditionalInfo>

	    <Consignment.Parcel type="normal">
		<Parcel.Reference>123456</Parcel.Reference>
		<Parcel.Packagetype>PC</Parcel.Packagetype>
		<Parcel.Weight unit="kg">1</Parcel.Weight>
		<Parcel.Volume unit="m3">0.03</Parcel.Volume>
		<Parcel.Infocode>1012</Parcel.Infocode>
		<Parcel.Contents>Puita ja muttereita</Parcel.Contents>
		<Parcel.ReturnService>123</Parcel.ReturnService>
		<Parcel.contentline><!-- Customs declaration info -->
		    <contentline.description>Puita</contentline.description>
		    <contentline.quantity>1</contentline.quantity>
		    <contentline.currency>EUR</contentline.currency>
		    <contentline.netweight>1</contentline.netweight>
		    <contentline.value>100</contentline.value>
		    <contentline.countryoforigin>FI</contentline.countryoforigin>
		    <contentline.tariffcode>9608101000</contentline.tariffcode>
		</Parcel.contentline>
	    </Consignment.Parcel>

	    <Consignment.Parcel type="normal">
		<Parcel.Reference>1234567</Parcel.Reference>
		<Parcel.Packagetype>PC</Parcel.Packagetype>
		<Parcel.Weight unit="kg">2</Parcel.Weight>
		<Parcel.Volume unit="m3">0.04</Parcel.Volume>
		<Parcel.Infocode>1012</Parcel.Infocode>
		<Parcel.Contents>Muttereita ja puita</Parcel.Contents>
		<Parcel.ReturnService>123</Parcel.ReturnService>
		<Parcel.contentline>
		    <contentline.description>Muttereita</contentline.description>
		    <contentline.quantity>1</contentline.quantity>
		    <contentline.currency>EUR</contentline.currency>
		    <contentline.netweight>1</contentline.netweight>
		    <contentline.value>100</contentline.value>
		    <contentline.countryoforigin>FI</contentline.countryoforigin>
		    <contentline.tariffcode>9608101000</contentline.tariffcode>
		</Parcel.contentline>
	     </Consignment.Parcel>

	 </Shipment.Consignment>
     </Shipment>
</eChannel>
```

### Request

Create shipment 

Example:
```php
$url = 'https://nextshipping.posti.fi/prinetti/create-shipment';
$secret = '1234567890ABCDEF';


$xml = simplexml_load_file("prinetti_example.xml");


$account = (string)$xml->ROUTING->{"Routing.Account"};
$id = (string)$xml->ROUTING->{"Routing.Id"};

$xml->ROUTING->{'Routing.Key'} = md5($account .$id .$secret);

$ch = curl_init($url);

curl_setopt($ch, CURLOPT_POST, 1);
curl_setopt($ch, CURLOPT_HTTPHEADER, array('Content-Type: text/xml'));
curl_setopt($ch, CURLOPT_POSTFIELDS, $xml->asXML());
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);

$output = curl_exec($ch);
```

## Response

|Element	|Presence	|Definition	|Data type	|Mandatory	|Parent|
|---|---|---|---|---|---|
|Response	|1	|	|	|x	||
|response.status	|1	|Status code	|N	|x	|Response|
|response.message	|1	|Status code in text	|AN 1000	|x	Response|
|response.reference	|0...1	|Consignment.Reference	|AN 100	|o	|Response|
|@uuid	|1	|Unique ID for the shipment. Not in original Prinetti API specification.	|UUID	|o	|response.reference|
|response.trackingcode	|0...n	|Tracking code for each parcel in shipment	|AN 32	|o	|Response|
|@uuid	|1	|Unique ID for the parcel. Not in original Prinetti API specification.	|UUID	|o	|response.trackingcode|
|@labelcode	|0....1	|Unique code for the label. Not in original Prinetti API specification.	|AN 20	|o	|response.trackingcode|

Response status codes

|Code	|Definition|
|---	|---	|
|0	|OK|
|100	|Unkown error|
|160	|Error in saving label|
|230	|XML is invalid|
|240	|System failure|

### Response

Example:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Response>
    <response.status>0</response.status>
    <response.message>OK</response.message>
    <response.reference uuid="00000000-0000-0000-0000-000000000000">1479034267</response.reference>
    <response.trackingcode labelcode="12345" uuid="00000000-0000-0000-0000-000000000000">JJFI64574900000202361</response.trackingcode>
</Response>
```

## Fetch Shipping Label

POST: /prinetti/get-shipping-label

XML example:
```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<eChannel>
<ROUTING>
	<Routing.Account>00000000-0000-0000-0000-000000000000</Routing.Account>	
	<Routing.Key>1234567890ABCDEF</Routing.Key>
	<Routing.Id>1479035179</Routing.Id>
	<Routing.Name>Testisanoma</Routing.Name>
	<Routing.Time>20161113130618</Routing.Time>
</ROUTING>
    <PrintLabel combine="true" responseFormat="File"><!-- "File" and "inline" are supported. "File" is default -->
	<Reference>1479034267</Reference>
	<TrackingCode>JJFI64574900000202361</TrackingCode>
    </PrintLabel>
</eChannel>
```

### Request

Example:
```php
$url = 'https://nextshipping.posti.fi/prinetti/get-shipping-label';
$secret = '1234567890ABCDEF';

$xml = simplexml_load_file("prinetti_example.xml");

$account = (string)$xml->ROUTING->{"Routing.Account"};
$id = (string)$xml->ROUTING->{"Routing.Id"};

$xml->ROUTING->{'Routing.Key'} = md5($account .$id .$secret);

$ch = curl_init($url);

curl_setopt($ch, CURLOPT_POST, 1);
curl_setopt($ch, CURLOPT_HTTPHEADER, array('Content-Type: text/xml'));
curl_setopt($ch, CURLOPT_POSTFIELDS, $xml->asXML());
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);

$output = curl_exec($ch);	
```

### Response

If responseFormat was File the following xml is returned.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Response>
    <response.status>0</response.status>
    <response.message>OK</response.message>
    <response.file encode="base64">JVBERi0xLjcKJeLjz9MKNiAwIG9i...</response.file>
</Response>
```

# Information and support

Support is available through normal channels.
