# empowclassifier filter plugin

- Plugin version:
- Released on:
- Changelog

# Getting Help
For questions about the plugin, open a topic in the [Discuss](http://discuss.elastic.co/) forums. 
For bugs or feature requests, open an issue in [Github](https://github.com/logstash-plugins/logstash-filter-fingerprint). 
For the list of Elastic supported plugins, please consult the [Elastic Support Matrix](https://www.elastic.co/support/matrix#matrix_logstash_plugins).

# Description

Returns classification information for attacks from the empow classification center, based on information in log strings.

# Configuration Options

Include the plugin in the filter section of your logstash config file like this:
 
```
filter {
    empowclassifier {
        username => "*****"
        password => "*********!"
        }
}
```


# Details

## cache_size
- Value type is: number
- Default value is:

The number of responses cached locally.

## pending_request_limit
- Value type is: number
- Default value is

Max number of requests pending response from the classification center 
## pending_request_timeout
- Value type is: number
- Default value is

Timeout for response from classification center.


## product_type_field
- Value type is: number
- Default value is
The name of the product type field in the log.

## product_name_field
- Value type is: string
- Default value is

The name of the product name field in the log.

## threat
- Value type is: string
- Default value is

The field containing the terms sent to the classification center

## src_internal
- Value type is: string
- Default value is

These field accepts true/false (Boolean) values that describes whether the source or dest are internal in the user's network. (an internal host/mail/user/app)

## dst_internal
- Value type is: string
- Default value is

These field accepts true/false (Boolean) values that describes whether the source or dest are internal in the user's network. (an internal host/mail/user/app)

## tag_on_error
- Value type is
- Default value is

['_empow_classifer_error']

## tag_on_timeout
- Value type is
- Default value is

[' _empow_classifer_timeout']

# Using the empowclassifier plugin

## Example

A log may look like this before the classification (in json representation, probably not ideal to show on the logstash manuals):
 
```
{
	"product_type": "IDS",
	"product_name": "snort",
	" threat": { "signature": "1:234" }
}
```
 
After filtering, using the plugin, the response would be contain these fields:
 
```
{
    "signatureTactics": [
        {
            "tactic": "Full compromise - active patterns",
            "attackStage": "Infiltration",
            "isSrcPerformer": true
        }
    ]
}
```
 

```signatureTactics``` is an array of the tactics classified by empow.

each result contains the actual tactic,

the attack stage empow classified for this log (determined by the tactic and whether the source and dest are within the user's network),

and whether the source was the performer or the victim of this attack.

 
