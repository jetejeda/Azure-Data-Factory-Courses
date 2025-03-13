# Data source:

[Covid data](https://www.ecdc.europa.eu/en/covid-19/data)

## Working with Data Flows:

### The select activity:

1. Here you can select all the columns that you want from the previous activity.
2. You have access to different types of column mapping, for example:
   - By default, the Data Flow maps every input to an output
   - It has an auto mapping option, if enabled, if you have any drifted columns, the activity will also flow them through
   - There is a rule based mapping in which you can specify an expression and then you can evaluate and have separate fields based on that.
   - With the rule based mapping you can rename all column names:
     ```scala
     'your_prefix' + $$ #The dollars represent all columns
     ```
3. There are two options available:
   - Skip duplicate input columns: If you have a duplicate column it will only take the first one and then it'll ignore the next ones. This could happen if for example you had a join or lookup activity before the select.
   - Skip duplicate output columns: If you have two output columns with the same name, it will only take the top one.

### The Lookup activity:

It works very similar to a left outer join in a join activity. You have to select a primary stream and a Lookup stream, the match columns and it's Lookup conditions.

This will append all the columns from the Lookup stream onto the primary stream. If your lookup stream has multiple rows that will match with the primary stream you can specify if you want to keep all matches, create a match with any row or only with the first or last row.

It has an "Optimize" section in which you can Broadcast the data. As mentioned, everything we do on a data flow runs on a spark cluster. Spark works on a distributed computing infrastructure. If we set an Auto or Fixed broadcast it will push all of the data into every single spark node. Therefore, we have to make sure that the nodes have got enough memory so they can fit the lookup data (stream) entirely and get best performance outta them.

## HDInsight & Databricks

### Prepare Data for HDInsight & Databricks

The Data Flows from Azure Data Factory execute in Spark which works on distributed computing similar to HDInsight. In this section we will be learning about the HDInsight activity from ADF rather than learn about HDInsight or Hadoop as a whole

In order to work with those technologies we need to have our data written to folders as opposed to files. In HDInsight, every table points to a folder as opposed to a file.

In order to use this activity, we need to crate an HDInsight Cluster. These clusters are build even when they're not being used.

#### Create a HDInsight Cluster via the Azure Portal

Before we can create it, we will need to create a resource called managed identity. The cluster needs to be able to access the blob storage account, HDInsight only allows access to Data Lake Storage Gen2 accounts via a managed identity. The managed identity is a feature of Azure Active Directory that lets you assign an identity to various Azure resources without the need for an identity's credential. We can then provide access to the Azure resources via the managed identity.

The HDInsight cluster is created as a service (it's not inside ADF). The region for the HDInsight cluster has to be the same as the region for the Data Lake Storage Gen2.

When creating a HDInsight cluster you have to select the cluster type, depending on the type of cluster you selected, you will be asked to pick bigger nodes which will translate in you spending more money. These clusters are charged as soon as they are created and stops when the cluster is deleted.

HDInsight uses Ambary for the full management of our cluster. There you can look at every node, check for:

- Activities going on
- Add new metrics to a cluster dashboard
- Look at all the nodes from the cluster
- Check for background operations
- Stop/restart/start services
- Query Hive tables (you can also use different softwares to interact with the Hive tables, HDInsights supports JDBC connections)

## Data Transformation Assignment

### Explaining the Hive Script

[Script File](covid_transform_testing.hql)

1. We create the databases needed if they don't exist
2. We create the external tables (syntax similar to Synapse). Since they are external tables, they are not managed by Hive, but they are just a structure which is placed on top of the sources. The source of the data will be the folders from the Data Lake. They are just logical tables, therefore, if you drop the table, the data is not actually dropped.
3. We create the managed tables. With these type of tables, the data would get dropped if we drop the table.
4. Ingest the table with the data from select statement
5. Create the pipeline to execute the hql Script. Inside ADF there are multiple activities related to HDInsight which (by the time) are:
   - Hive
   - MapReduce
   - Pig
   - Spark
   - Streaming
6. Within the activity in ADF, you can create a link service to either use your own HDInsight cluster, or an On-demand cluster (will create a cluster and destroy it when it's done processing).
