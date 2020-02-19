### Insight example: 

```sh
[
  {
    "ResourceProvider": "Microsoft.Web",
    "ResourceType": "sites",
    "JsonPath": "value[*].properties.alwaysOn",
    "DataSource": "ARM",
    "Title": "AlwaysOn disabled, End users may experience longer response time while site resumes.",
    "Category": "Performance Impact",
    "Reference": "https://docs.microsoft.com/en-us/azure/app-service/faq-availability-performance-application-issues#how-do-i-decrease-the-response-time-for-the-first-request-after-idle-time",
    "Info": "If the site is inactive for an extended period of time, the process associated with it is shut down to conserve resources. Subsequent requests following a long idle time, may take longer to respond as the process has to be re-started and the site re-initialized.",
    "Conditions": [
      {
        "OldValue": "false",
        "NewValue": "true"
      }
    ]
  }
]
```
### Insight configuration: 
 
 - **ResourceProvider**: Azure Resource Provider Name.
 - **ResourceType**: Azure Resource Provider Tracked Resource type name.
 - **JsonPath**: The json path of the property to match on (Wild cards supported in array indexes "value[*].properties.alwaysOn")
 - **DataSource**: The first three characters of the ChangeSetId.
 - **Title** : The title of the recommended insight.
 - **Category**: Insight category.
 - **Reference** : Link to a suggested article or reference material.
 - **Info**: Short description of the insights or suggestion.


### Conditions: 
- Each insight can have multiple conditions. 
- All conditions should be evaluated as a 'match' in order to attach an insight to a change.

**Required properties** :
- JToken **NewValue** - the value to apply against after change value (either the NewValue in change object or snapshot)
- JToken **OldValue** - the value to apply against before change value (either the OldValue in change object or snapshot)

**Optional properties** : 

**ComparisonType**: the type of comparison operator to execute in the insight condition check. 

```
 StringCompare -  Checks if the string value of JTokens is equal. (**default** value of ComparisonType if omitted) 
 StringContains - Checks if the string value of first JToken, contains a string value of a second JToken.
 StringContainsAny -  Checks if the string value of the first JToken, contains any of the string values specified in the second JToken array.
```

**ContentSource** : 
The content source that should be used for evaluation of insight condition.
```
Change - The condition check expression should be applied to a change object that is currently processed. (**default** value of ContentSource if omitted) 
Snapshot - The condition check expression should be applied to a snapshot from where change was extracted from. 

```
 **Path** : 
 The JsonPath is applied to evaluated Json content (Change [Old or New Value] or Snapshot). The discovered JTokens are compared with Value field. The comparison is done by using specified ComparisonType operator. (**default** value is null, which means no json path is applied and the full value of Change NewValue or OldValue is used.)
 
 ### Examples
 
```sh
 {
    "ResourceProvider": "Microsoft.Network",
    "ResourceType": "networkSecurityGroups",
    "JsonPath": "properties.securityRules[*].properties.access",
    "DataSource": "ARG",
    "Description": "The suggestion is validating whether existing security rule .access value was modified. We first check whether whether Change OldValue equals to null or 'Allow' and Change NewValue equals to 'Deny'. Secondly we check whether the parent object of .access property in after change snapshot has a JsonPathFilter .destinationPortRanges matching any of the ports specified in the ExpectedValue",
    "Title": "Firewall network security rule blocking access to critical ports",
    "Category": "Availability",
    "Reference": "https://docs.microsoft.com/en-us/azure/virtual-machines/troubleshooting/troubleshoot-rdp-nsg-problem",
    "Info": "Firewall network security rule blocking access to critical ports, it might affect the VMs accessing internet.",
    "Conditions": [
      {
        "OldValue": [ "Allow", null ],
        "NewValue": ["Deny"],
        "Comparison": "StringContainsAny"
      },
      {
        "NewValue": [ "3389", "22", "443", "80", "8080", "*" ],
        "Source": "Snapshot",
        "Comparison": "StringContainsAny",
        "Path": "properties.securityRules[*].destinationPortRanges"
      }
    ]
  }
```

The insight above has 2 matching conditions - it would first match a first condition and check whether change OldValue contains either "Allow" or null, and NewValue has value "Deny". Secondary it would check whether the content of properties.securityRules[*].destinationPortRanges json path in the "after change" snapshot json object would contain any of the values specified in the NewValue field.

**Insights Editing and Testing Tools**:
https://aca-insights.azurewebsites.net


# Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
