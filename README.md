## Microhack 3: Advanced KQL, Policies, Security 

This Microhack is organised into the following 3 challenges:
- Challenge 1: Materialized views, Functions, External Tables
- Challenge 2: Caching and retention policies
- Challenge 3: Control commands
- Challenge 4: Cluster autoScaling
- Challenge 5: Security


#### Challenge 1: Materialized views, Functions, External Tables

##### Task 1: Create LKV materialized view

Instead of writing a query every time to retrieve the last known value for every device, create a materialized view containing the last known value for every device

[Materialized views - Azure Data Explorer | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/materialized-views/materialized-view-overview)
[Create materialized view - Azure Data Explorer | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/materialized-views/materialized-view-create)

##### Task 2: Create materialized view of aggregated data



#### Challenge 2: Caching and retention policies

##### Task 1: Cache and retention policies - concepts

Among the different policies you can set to the ADX cluster, two policies are of particular importance: retention policy (retention period) and cache policy (cache period).
First, a policy, is what’s  used to enforce and control the properties of the cluster (or the database/table.)
  
The **retention** policy is the time span, in days, for which it’s guaranteed that the data is kept available for querying. The time span is measured from the time that the records are ingested. When the period expires , the records  will not be available for querying any more. In other words, the retention policy defines the period in which the data available to query, measured since ingestion time. Notice that a large retention period may impact the cost. 

The **cache** policy, is the time span, in days, for which to keep recently ingested data (which is usually the frequently queried data) available in the hot cache rather than in long term storage (this is also known as cold cache. Specifically, it is Azure blob storage). Data stored in the hot cache is actually stored in local SSD or the RAM of the machine, very close to the compute nodes. Therefore, more readily available for querying. The availability of data in hot cache improves query performance but can potentially increase the cluster cost (as more data is being stored, more VMs are required to store it). In other words, the caching policy defines the period in which data is kept in the hot cache. 

All the data is always stored in the cold cache, for the duration defined in the retention policy. Any data whose age falls within the hot cache policy will also be stored in the hot cache. If you query data from cold cache, it’s recommended to target a small specific range in time (“point in time”) for queries to be efficient.

##### Task 2: Change the cache policy via the Azure portal (data base level)
Go to your Azure Data Explorer cluster resource in the Azure portal. Click on the “Databases” blade

![Screen capture 1](/assets/images/MH3-Challenge2-Task2-Pic1.png)

Click on the database name. The database page opens. Select "Edit" from the top menu. The side pane allows you to edit the retention and caching periods (policies) of the database. Change the retention to 365 days and the cache to 31 days, and save.

![Screen capture 1](/assets/images/MH3-Challenge2-Task2-Pic2.png)
 
##### Task 3: change the cache policy via commands (data base or table level)

Database policies can be overridden per table using a KQL control command.
ADX cluster and database are Azure resources. A database is a sub-resource of the cluster, so it can be edited from the portal. Tables are not considered an Azure resource, so they cannot be managed in the portal but via a KQL command.    
You can always use KQL commands to alter the policies of the entire Cluster/Database/tables. Table level cache policy takes precedence over database level which precedence over cluster level.

Alter the cache policy of the table LogisticsTelemetryExtended to 60 days.

[.alter table cache policy command - Azure Data Explorer | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/alter-table-cache-policy-command)

##### Task 4: Query cold data with hot windows
Although querying cold data is possible, the data is queried faster when it's in local SSD (the hot cache), particularly for range queries that scan large amounts of data. 

To query cold data, ADX process a loading step that requires accessing a storage tier with much higher latency than the local disk. When the query is limited to a small time window, often called "point-in-time" queries, the amount of data to be retrieved will usually be small, and the query will complete quickly. For example, forensic analyses querying telemetry on a given day in the past fall under this category. The impact on the query duration depends on the size of data that is pulled from storage, and can be significant. 

But, if you're scanning a large amount of cold data, query performance could benefit from using the ‘hot windows’ feature, which lets you efficiently query cold data.

Hot windows are part of the cache policy commands syntax and are set with the .alter policy caching command.

To try out this feature, set a hot_window between datetime(2021-01-01) .. datetime(2021-02-01)

[Use hot windows for infrequent queries over cold data in Azure Data Explorer | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-explorer/hot-windows)

#### Challenge 3: Control commmands

##### Task 1: .show/diagnostic logs/Insights
Control commands used to manage Azure Data Explorer. Control commands are requests to the service to retrieve information that is not necessarily data in the database tables, or to modify the service state, etc.
The first character of the text of a request determines if the request is a control command or a query. Control commands must start with the dot (.) character, and no query may start by that character.

The ‘.show queries’ command returns a list of queries that have reached a final state, and that the user invoking the command has access to see.
The ‘.show commands command returns a table of the admin commands that have reached a final state.  The commands table has two columns with resources consumption details of every completed command.
TotalCpu - The total CPU clock time (User mode + Kernel mode) consumed by this command.
ResourceUtilization - Contains all resource use information related to that command, including the TotalCpu.

##### Task 2: more .show commands

Write a command to count the number commands of your user id, from the past 7 day.

Reference:
[.show queries](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/queries)

[.show commands](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/commands)

[Management (control commands) overview](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/)

##### Task 3: Table details

Write a control command to show details on all tables in the database.

[.show tables](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/show-tables-command)

#### Challenge 4: Cluster Autoscaling

Sizing a cluster appropriately is critical to the performance of Azure Data Explorer. A static cluster size can lead to under-utilization or over-utilization, neither of which is ideal. Because demand on a cluster can’t be predicted with absolute accuracy, it's better to scale a cluster, adding and removing capacity and CPU resources with changing demand.

There are two workflows for scaling an Azure Data Explorer cluster:

Horizontal scaling, also called scaling in and out.
Vertical scaling, also called scaling up and down. 

##### Task 1: Manage cluster horizontal scaling (scale in/out)
By using horizontal scaling, you can scale the instance count automatically, based on predefined rules and schedules. To specify the autoscale settings for your cluster:

In the Azure portal, go to your Azure Data Explorer cluster resource. Under Settings, select Scale out.

In the Scale out window, you can select the autoscale method that you want: Manual scale, Optimized autoscale, or Custom autoscale.
Optimized autoscale is the recommended autoscale method. This method optimizes cluster performance and costs. If the cluster approaches a state of under-utilization, it will be scaled in. This action lowers costs but keeps performance level. If the cluster approaches a state of over-utilization, it will be scaled out to maintain optimal performance. 
To configure Optimized autoscale: select the Optimized autoscale option. Then, select a minimum instance count and a maximum instance count. The cluster auto-scaling ranges between those two numbers, based on load.

[Optimized Autoscale](https://docs.microsoft.com/en-us/azure/data-explorer/manage-cluster-horizontal-scaling#optimized-autoscale)

##### Task 2: Manage cluster vertical scaling (scale up/down)

To Configure vertical scaling, in the Azure portal, go to your Azure Data Explorer cluster resource. Under Settings, select Scale up.

In the Scale up window, you will see a list of available SKUs for your cluster.
Please note that the vertical scaling process can take up to 30 minutes, and during that time your cluster will be suspended.
Scaling down can harm your cluster performance. Each SKU offers a distinct SSD and CPU ratio to help you correctly size their deployment and build cost optimal solutions for their enterprise analytical workload.  There are four types of SKUs: types of storage optimized, compute optimized, heavy compute and isolated compute.

[Choosing Cluster SKU](https://docs.microsoft.com/en-us/azure/data-explorer/manage-cluster-choose-sku)

#### Challenge 5: Security


## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft 
trademarks or logos is subject to and must follow 
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.
