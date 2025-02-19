---
ArtifactType: ARM templates, scripts etc.
Documentation: Below
Language: bash etc.
Tags: lma,telemetry,networkcloud,nc,aods,workbooks,alerts
---

# AODS/Network Cloud Logging, Monitoring, and Alerting

## Overview

This folder contains sample workbooks and alert rules related to AODS/Network Cloud Logging, Monitoring and Alerting.
It provides a variety of ARM templates for use by customers to deploy selected resources.

### Prerequisites

The samples require one or more AODS instances connected with a Log Analytics workspace. An AODS instance will deploy
all of the necessary Prometheus exporters which will provide the metrics supporting these workbooks and alert rules.

## Azure Workbooks

These instructions will let you deploy an Azure Workbook within your Log Analytics workspace as mentioned through the
parameter. You will have the ability to select the Arc-enabled Cluster associated to that Log Analytics workspace and
see the data for a particular cluster. The timerange can also be selected to see data for a particular timeframe.

In the workbooks/templates folder, there is a workbook for Hardware Validation. This workbook should be deployed on 
Log Analytics Workspace related to Cluster Manager (and not Cluster). One Hardware validation workbook per Cluster Manager. 
This workbook allows you to view data collected from executions of Hardware validation from among all the clusters 
associated to this Cluster Manager instance.
### Workbooks Folder Structure

AzureMonitorCommunity repository path:  **`Workbooks/Preview/armTemplates`**

- `deployWorkbooks.sh` - Shell script utility for deploying all workbooks to a log analytics workspace
- `/templates` - Folder contains workbook templates

The following sections describe these resources in greater detail.

### Deployment

To use the Azure CLI to deploy a workbook ARM template, login to the Azure CLI and set your subscription.

``` sh
    az login
    az account set -s "<SUBSCRIPTION_NAME_OR_ID>"
```
where 
- **<SUBSCRIPTION_NAME_OR_ID>** = name or ID of subscription where deployment will be created

Deploy a single workbook in your resource group using the following command:

```sh
    az deployment group create \
        --name "<DEPLOYMENT_NAME>" \
        --resource-group "<RESOURCE_GROUP>" \
        --template-file "<PATH_TO_TEMPLATE_FILE>" \
        --parameters workspaceLAW="<WORKSPACE_LAW>" \
        --parameters workbookLocation="<AZ_LOCATION>"
```
where
- **<DEPLOYMENT_NAME>** = name for the deployment
- **<RESOURCE_GROUP>** = resource group name
- **<PATH_TO_TEMPLATE_FILE>** = path to selected workbook ARM template in `templates/`
- **<WORKSPACE_LAW>** = Log Analytics workspace name 
- **<AZ_LOCATION>** = Region in which to create the workbook

### Scripting
A sample shell script (`deployWorkbooks.sh`) can be used to deploy all of the sample workbooks.
The script may be invoked with the following environment variables set or passed as arguments.  If a value is not set
the script will prompt for it.

- `RESOURCE_GROUP` - The resource group in which the Log Analytics workspace is located
- `WORKSPACE_LAW` - The name of the Log Analytics workspace within the resource group
- `AZ_LOCATION` - *Optional* - The region in which the workbooks should be created.  Defaults to same region as Log
Analytics workspace.

The script can be run from the workbooks folder directory:

```sh
  ./deployWorkbooks.sh $RESOURCE_GROUP $WORKSPACE_LAW $AZ_LOCATION
```

### Built With

<https://docs.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-automate>

## Alert Rules

The Alerts ARM templates subfolder contains sample scripts, ARM templates and associated parameter files which can be
run to create alert rules.  

### Alert Rules Folder Structure

AzureMonitorCommunity repository path:  **`Alerts/Preview/armTemplates`**

- `deployScheduledQueryRules.sh` - Shell script utility for creating log alert rules on a log analytics workspace
- `deployMetricAlerts.sh` - Shell script utility for creating metric alert rules on an AODS cluster
- `/templates` - Folder contains ARM templates for use in deploying 
[alert rule types](https://docs.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-types).  
These templates expose parameters that specify alert rule settings that can be set using parameter files or directly 
on the command line.
  - `metricAlerts.bicep` - For Metric alert rules
  - `scheduledQueryRules.bicep` - For Log alert rules.  This template also defines some Kusto 
[user-defined functions](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/functions/user-defined-functions)
used by queries in the parameter files to encapsulate some common approaches for 
[querying Prometheus metrics](https://docs.microsoft.com/en-us/azure/azure-monitor/containers/container-insights-log-query#query-prometheus-metrics-data) 
scraped by Azure Container Insights.
- `/scheduledQueryRules` - Folder contains parameter files each containing settings for sample log alert rule
- `/metricAlerts` - Folder contains parameter files each containing settings for sample metric alert rule

The following sections describe these resources in greater detail.

### Create Action Groups

When Azure Monitor data indicates that there might be a problem with your infrastructure or application, an alert is
triggered. Azure Monitor then use action groups to notify users about the alert and take an action. An action group is
a collection of notification preferences that are defined by the owner of an Azure subscription.

Operators can create an Action Group that they deem suitable for their alert rules. They can then provide that during
deployment to the alert rules in the next steps.

Documentation to create Action Group: https://docs.microsoft.com/en-us/azure/azure-monitor/alerts/action-groups

### Deployment

To use the Azure CLI to deploy an alert rule ARM template, login to the Azure CLI and set your subscription.

``` sh
    az login
    az account set -s "<SUBSCRIPTION_NAME_OR_ID>"
```
where 
- **<SUBSCRIPTION_NAME_OR_ID>** = name or ID of subscription where deployment will be created

#### **Log Alert Rules**

![AODS Logging](AODSLogs.png)

Log query alert rules are applied to a Log Analytics workspace where logs from one or more AODS clusters are collected.
These alert rules rely on log query results to trigger and attach to the appropriate resource producing the log.
Deploy a sample log query alert rule using the following command:

```sh
  az deployment group create \
    --name "<DEPLOYMENT_NAME>" \
    --resource-group "<RESOURCE_GROUP>" \
    --template-file "<PATH_TO_TEMPLATE_FILE>" \
    --parameters @"<PATH_TO_PARAMETER_FILE>" \
      resourceId="<LAW_RESOURCE_ID>" location="<AZ_LOCATION>" \
      actionGroupIds="<ACTION_GROUP_IDS>" \
      <PARAM_NAME>="<PARAM_VALUE>"...
```

where
- **<DEPLOYMENT_NAME>** = name for the deployment
- **<RESOURCE_GROUP>** = resource group where the alert rule will be created
- **<PATH_TO_TEMPLATE_FILE>** = path to `templates/scheduledQueryRules.bicep`
- **<PATH_TO_PARAMETER_FILE>** = path to parameter file in `scheduledQueryRules/` for alert rule to be created
- **<LAW_RESOURCE_ID>** = Full Resource ID of the AODS Log Analytics workspace
- **<AZ_LOCATION>** = Region in which to create the log alert rule
- **<ACTION_GROUP_IDS>** = Optional comma-separated list of action group resource IDs to be associated to the alert
rule.
- **<PARAM_NAME>="<PARAM_VALUE>"** = Optional name/value pairs which can override other parameter file values 

### Scripting
A sample shell script (`deployScheduledQueryRules.sh`) can be used to deploy all of the sample log query alert rules.
The script may be invoked with the following environment variables set or passed as arguments.  If a value is not set
the script will prompt for it.

- `RESOURCE_GROUP` - The resource group in which the Log Analytics workspace is located
- `WORKSPACE_LAW` - The name of the Log Analytics workspace within the resource group
- `AZ_LOCATION` - *Optional* - The region in which the alerts should be created.  Defaults to same region as Log
Analytics workspace.
- `ACTION_GROUP_IDS` - *Optional* - Comma-separated list of action group resource IDs

The script can be run from the alert rules folder directory:

```sh
  ./deployScheduledQueryRules.sh $RESOURCE_GROUP $WORKSPACE_LAW $LOCATION $ACTION_GROUP_IDS
```

#### **Metric Alert Rules**

Metric alert rules are applied to an AODS cluster which emits metrics that the alert rules monitor. Deploy a sample 
metric alert rule using the following command:

```sh
  az deployment group create \
    --name "<DEPLOYMENT_NAME>" \
    --resource-group "<RESOURCE_GROUP>" \
    --template-file "<PATH_TO_TEMPLATE_FILE>" \
    --parameters @"<PATH_TO_PARAMETER_FILE>" \
      resourceId="<CLUSTER_RESOURCE_ID" \
      actionGroupIds="<ACTION_GROUP_IDS>" \
      <PARAM_NAME>="<PARAM_VALUE>"...
```

where
- **<DEPLOYMENT_NAME>** = name for the deployment
- **<RESOURCE_GROUP>** = resource group where the alert rule will be created
- **<PATH_TO_TEMPLATE_FILE>** = path to `templates/metricAlerts.bicep`
- **<PATH_TO_PARAMETER_FILE>** = path to parameter file in `metricAlerts/` for alert rule to be created
- **<CLUSTER_RESOURCE_ID>** = Full Resource ID of the AODS cluster emitting the metric
- **<ACTION_GROUP_IDS>** = Optional comma-separated list of action group resource IDs to be associated to the alert
rule.
- **<PARAM_NAME>="<PARAM_VALUE>"** = Optional name/value pairs which can override other parameter file values 

### Scripting
A sample shell script (`deployMetricAlerts.sh`) can be used to deploy all of the sample metric alert rules.
The script may be invoked with the following environment variables set or passed as arguments.  If a value is not set
the script will prompt for it.

- `RESOURCE_GROUP` - The resource group in which the Log Analytics workspace is located
- `CLUSTER_NAME` - The name of the AODS cluster in the resource group that will emit the metrics
- `ACTION_GROUP_IDS` - *Optional* - Comma-separated list of action group resource IDs

The script can be run from the alert rules folder directory:

```sh
  ./deployMetricAlerts.sh $RESOURCE_GROUP $CLUSTER_NAME $ACTION_GROUP_IDS
```

## Testing

The workbooks provide you an ability to filter the data across Timeframe, selected Arc-enabled Cluster and the 
Hostname. Please make sure data is getting populated in these parameters to show the charts properly.

Alert rules may be viewed in the Azure portal when viewing a resource group by selecting *Alerts* from the left 
navigation pane and then selecting *Alert rules* from the top of the Alerts view.