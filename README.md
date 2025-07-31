# Okta Rest Collector
----

## About this Pack

This pack is designed to handle JSON data collected from the Okta System Log API endpoint. The JSON is parsed and the timestamp normalized from the proper field within each event. The pack offers currently two different optional methods of reduction:

1. Drop, Sample, or Suppress based off of eventType. You can target individual eventTypes by modifying the provided okta-event-types.csv lookup under Knowledge > Lookups.
2. Remove nested null value fields.

The pack also currently includes three forms of outputs:

1. Normalized JSON
2. OCSF - Primarily meant for Amazon Security Lake, the pack can normalize the data into the proper OCSF categories based on eventType. OCSF category can also be managed on an individual eventType basis through the okta-event-types.csv lookup.

## Deployment

The Okta Rest pack allows for events to be sent from the System Log API endpoint and normalized into the proper format for the required destinations. To use this pack, follow these steps:

1. Configure the Pack
This pack includes several functions that can help reduce events. Please make sure you evaluate the functions before enableling to ensure vital data is not missed.

Additionally, several output formats are available to be selected. Please only enable one output, as enabling multiple may break the output formatting.

2. Configure the Event Breaker Rule
In order to separate the individual events from the API, please add the provided Event Breaker Rule to your stream instance under Processing > Knowledge > Event Breaker Rules. The JSON for the rule is provided in Appendix A

3. Configure the Rest Collector Source
Once the Event Breaker Rule is created, copy the JSON from Appendix B and place it under Data > Sources > Collectors > REST. Make sure to replace both Domain and Token with proper values from your environment.

4. Tie the Pack to the Okta Rest Collector

## Upgrades

Upgrading certian Cribl Packs using the same Pack ID can have unintended consequences. See [Upgrading an Existing Pack](https://docs.cribl.io/stream/packs#upgrading) for details.

## Release Notes

### Version 0.1.0
Initial release

## Contributing to the Pack

To contribute to the Pack, please connect with us on [Cribl Community Slack](https://cribl-community.slack.com/). You can suggest new features or offer to collaborate.

## License
This Pack uses the following license: [Apache 2.0](https://github.com/criblio/appscope/blob/master/LICENSE).

## Appendix A
```
{
  "id": "Okta-API",
  "lib": "custom",
  "description": "Event breakers for Okta logs",
  "rules": [
    {
      "condition": "true",
      "type": "json_array",
      "timestampAnchorRegex": "/published\":\"/",
      "timestamp": {
        "type": "format",
        "length": 150,
        "format": "%Y-%m-%dT%H:%M:%S.%LZ"
      },
      "timestampTimezone": "utc",
      "timestampEarliest": "-420weeks",
      "timestampLatest": "+1week",
      "maxEventBytes": 51200,
      "disabled": false,
      "jsonExtractAll": false,
      "eventBreakerRegex": "/[\\n\\r]+(?!\\s)/",
      "name": "System Logs",
      "jsonTimeField": "published"
    }
  ]
}
```

## Appendix B
```
{
  "type": "collection",
  "ttl": "4h",
  "removeFields": [],
  "resumeOnBoot": false,
  "schedule": {},
  "streamtags": [],
  "workerAffinity": false,
  "collector": {
    "conf": {
      "discovery": {
        "discoverType": "none"
      },
      "collectMethod": "get",
      "pagination": {
        "type": "response_header_link",
        "nextRelationAttribute": "next",
        "maxPages": 0,
        "curRelationAttribute": "self"
      },
      "authentication": "none",
      "timeout": 0,
      "useRoundRobinDns": false,
      "disableTimeFilter": false,
      "safeHeaders": [],
      "collectUrl": "'https://<Domain| This is your Okta Domain, if you need help here is the [Okta Docs](https://developer.okta.com/docs/guides/find-your-domain/main/)>.okta.com/api/v1/logs'",
      "collectRequestParams": [
        {
          "name": "since",
          "value": "`${new Date((earliest * 1000 || Date.now()-(7*24*60*60*1000))).toISOString()}`"
        },
        {
          "name": "until",
          "value": "`${new Date((latest * 1000 || Date.now())).toISOString()}`"
        }
      ],
      "collectRequestHeaders": [
        {
          "name": "Authorization",
          "value": "`SSWS <SSWS Key| This is the Okta API Key, more infor can be found in the [Okta Docs](https://developer.okta.com/docs/guides/create-an-api-token/main/)>`"
        }
      ]
    },
    "destructive": false,
    "type": "rest"
  },
  "input": {
    "type": "collection",
    "staleChannelFlushMs": 10000,
    "sendToRoutes": true,
    "preprocess": {
      "disabled": true
    },
    "throttleRatePerSec": "0",
    "breakerRulesets": [
      "Okta-API"
    ]
  },
  "id": "Okta-API"
}
```
