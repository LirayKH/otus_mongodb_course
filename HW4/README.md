# HW 4 - MongoDB optimization

1. Launch `mongo-ek` instance remotely on Gcloud with MongoDB v5.0 configured in the [homework 2](https://github.com/LirayKH/otus_mongodb_course/blob/main/HW2/README.md).

```
gcloud compute instances list
gcloud compute instances start mongo-ek
```

2. Get `mongo-ek` remote server IP and connect to mongodb remotelly through MongoDB client:
```
mongo $(gcloud compute instances describe mongo-ek --format='get(networkInterfaces[0].accessConfigs[0].natIP)'):27001 -u mongo_db_admin -p mongo_db_admin_pwd --authenticationDatabase admin
```

3. Check dataset after previous [HW](https://github.com/LirayKH/otus_mongodb_course/blob/main/HW3/README.md):
```
use mail_customers;
show collections;
db.customers_info.find()
db.customers_info.countDocuments( { } );
178
```

4. Check profiling status. Enable detailed ProfilingLevel. Set slowlog trashold (90 ms - 2nd parameter)
```
db.getProfilingStatus()
db.setProfilingLevel(2)
db.setProfilingLevel(1, 90)
```

5. Check last request statistic:
```
db.system.profile.find().sort({$natural:-1}); 
db.system.profile.find().pretty().sort({$natural:-1}); 
```
5. Get Indexes in the customers_info table
```
db.customers_info.getIndexes();
```

6. Find the richest persons and sort them by Age starting from young + add explain parameter for receive request statistic (we can execute request without explain and additionally execute point 5 instead)
```
 db.customers_info.find().sort( { "Annual Income (k$)": -1, 'Age': 1 } ).explain("executionStats")
```

<details> 
  <summary>Results</summary>

```
> db.customers_info.find().sort( { "Annual Income (k$)": -1, 'Age': 1 } ).explain("executionStats")
```

```json
{
	"explainVersion" : "1",
	"queryPlanner" : {
		"namespace" : "mail_customers.customers_info",
		"indexFilterSet" : false,
		"parsedQuery" : {

		},
		"maxIndexedOrSolutionsReached" : false,
		"maxIndexedAndSolutionsReached" : false,
		"maxScansToExplodeReached" : false,
		"winningPlan" : {
			"stage" : "SORT",
			"sortPattern" : {
				"Annual Income (k$)" : -1,
				"Age" : 1
			},
			"memLimit" : 104857600,
			"type" : "simple",
			"inputStage" : {
				"stage" : "COLLSCAN",
				"direction" : "forward"
			}
		},
		"rejectedPlans" : [ ]
	},
	"executionStats" : {
		"executionSuccess" : true,
		"nReturned" : 200,
		"executionTimeMillis" : 0,
		"totalKeysExamined" : 0,
		"totalDocsExamined" : 200,
		"executionStages" : {
			"stage" : "SORT",
			"nReturned" : 200,
			"executionTimeMillisEstimate" : 0,
			"works" : 403,
			"advanced" : 200,
			"needTime" : 202,
			"needYield" : 0,
			"saveState" : 0,
			"restoreState" : 0,
			"isEOF" : 1,
			"sortPattern" : {
				"Annual Income (k$)" : -1,
				"Age" : 1
			},
			"memLimit" : 104857600,
			"type" : "simple",
			"totalDataSizeSorted" : 44024,
			"usedDisk" : false,
			"inputStage" : {
				"stage" : "COLLSCAN",
				"nReturned" : 200,
				"executionTimeMillisEstimate" : 0,
				"works" : 202,
				"advanced" : 200,
				"needTime" : 1,
				"needYield" : 0,
				"saveState" : 0,
				"restoreState" : 0,
				"isEOF" : 1,
				"direction" : "forward",
				"docsExamined" : 200
			}
		}
	},
	"command" : {
		"find" : "customers_info",
		"filter" : {

		},
		"sort" : {
			"Annual Income (k$)" : -1,
			"Age" : 1
		},
		"$db" : "mail_customers"
	},
	"serverInfo" : {
		"host" : "mongo-ek",
		"port" : 27001,
		"version" : "5.0.2",
		"gitVersion" : "6d9ec525e78465dcecadcff99cce953d380fedc8"
	},
	"serverParameters" : {
		"internalQueryFacetBufferSizeBytes" : 104857600,
		"internalQueryFacetMaxOutputDocSizeBytes" : 104857600,
		"internalLookupStageIntermediateDocumentMaxSizeBytes" : 104857600,
		"internalDocumentSourceGroupMaxMemoryBytes" : 104857600,
		"internalQueryMaxBlockingSortMemoryUsageBytes" : 104857600,
		"internalQueryProhibitBlockingMergeOnMongoS" : 0,
		"internalQueryMaxAddToSetBytes" : 104857600,
		"internalDocumentSourceSetWindowFieldsMaxMemoryBytes" : 104857600
	},
	"ok" : 1
}
```
</details>

8. Find customers between 52 and 60 age old and get statistics (With explain() function inside of querry):
```
db.customers_info.explain("executionStats").find({"Age" : { $gt : 52, $lt : 60} });
```

<details> 
  <summary>Results</summary>

```
> db.customers_info.explain("executionStats").find({"Age" : { $gt : 52, $lt : 60} });
```

```json
{
	"explainVersion" : "1",
	"queryPlanner" : {
		"namespace" : "mail_customers.customers_info",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"$and" : [
				{
					"Age" : {
						"$lt" : 60
					}
				},
				{
					"Age" : {
						"$gt" : 52
					}
				}
			]
		},
		"maxIndexedOrSolutionsReached" : false,
		"maxIndexedAndSolutionsReached" : false,
		"maxScansToExplodeReached" : false,
		"winningPlan" : {
			"stage" : "COLLSCAN",
			"filter" : {
				"$and" : [
					{
						"Age" : {
							"$lt" : 60
						}
					},
					{
						"Age" : {
							"$gt" : 52
						}
					}
				]
			},
			"direction" : "forward"
		},
		"rejectedPlans" : [ ]
	},
	"executionStats" : {
		"executionSuccess" : true,
		"nReturned" : 16,
		"executionTimeMillis" : 0,
		"totalKeysExamined" : 0,
		"totalDocsExamined" : 200,
		"executionStages" : {
			"stage" : "COLLSCAN",
			"filter" : {
				"$and" : [
					{
						"Age" : {
							"$lt" : 60
						}
					},
					{
						"Age" : {
							"$gt" : 52
						}
					}
				]
			},
			"nReturned" : 16,
			"executionTimeMillisEstimate" : 0,
			"works" : 202,
			"advanced" : 16,
			"needTime" : 185,
			"needYield" : 0,
			"saveState" : 0,
			"restoreState" : 0,
			"isEOF" : 1,
			"direction" : "forward",
			"docsExamined" : 200
		}
	},
	"command" : {
		"find" : "customers_info",
		"filter" : {
			"Age" : {
				"$gt" : 52,
				"$lt" : 60
			}
		},
		"$db" : "mail_customers"
	},
	"serverInfo" : {
		"host" : "mongo-ek",
		"port" : 27001,
		"version" : "5.0.2",
		"gitVersion" : "6d9ec525e78465dcecadcff99cce953d380fedc8"
	},
	"serverParameters" : {
		"internalQueryFacetBufferSizeBytes" : 104857600,
		"internalQueryFacetMaxOutputDocSizeBytes" : 104857600,
		"internalLookupStageIntermediateDocumentMaxSizeBytes" : 104857600,
		"internalDocumentSourceGroupMaxMemoryBytes" : 104857600,
		"internalQueryMaxBlockingSortMemoryUsageBytes" : 104857600,
		"internalQueryProhibitBlockingMergeOnMongoS" : 0,
		"internalQueryMaxAddToSetBytes" : 104857600,
		"internalDocumentSourceSetWindowFieldsMaxMemoryBytes" : 104857600
	},
	"ok" : 1
}
```
</details>

9. In both cases - step 7 and 8 we received `"executionTimeMillis"` equal `0`

Lets upload bigger collection:

```
wget https://dl.dropboxusercontent.com/s/p75zp1karqg6nnn/stocks.zip
unzip stocks.zip
``` 

```
mongorestore --host=$(gcloud compute instances describe mongo-ek --format='get(networkInterfaces[0].accessConfigs[0].natIP)') --port=27001 -u mongo_db_admin -p mongo_db_admin_pwd --authenticationDatabase admin --db stocks_db -c stocks dump/stocks/values.bson
```

10. Connect to mongo and look the new collection:
```
mongo $(gcloud compute instances describe mongo-ek --format='get(networkInterfaces[0].accessConfigs[0].natIP)'):27001 -u mongo_db_admin -p mongo_db_admin_pwd --authenticationDatabase admin

use stocks_db;
db.stocks.find( {} ).limit( 10 );
```

<details> 
  <summary>Stocks collection</summary>

```json
{ "_id" : ObjectId("4d094f58c96767d7a0099d49"), "exchange" : "NASDAQ", "stock_symbol" : "AACC", "date" : "2008-03-07", "open" : 8.4, "high" : 8.75, "low" : 8.08, "close" : 8.55, "volume" : 275800, "adj close" : 8.55 }
{ "_id" : ObjectId("4d094f58c96767d7a0099d4a"), "exchange" : "NASDAQ", "stock_symbol" : "AACC", "date" : "2008-03-06", "open" : 9.03, "high" : 9.03, "low" : 8.41, "close" : 8.56, "volume" : 353600, "adj close" : 8.56 }
{ "_id" : ObjectId("4d094f58c96767d7a0099d4b"), "exchange" : "NASDAQ", "stock_symbol" : "AACC", "date" : "2008-03-05", "open" : 9.12, "high" : 9.17, "low" : 8.85, "close" : 9.12, "volume" : 156200, "adj close" : 9.12 }
{ "_id" : ObjectId("4d094f58c96767d7a0099d4c"), "exchange" : "NASDAQ", "stock_symbol" : "AACC", "date" : "2008-03-04", "open" : 9.05, "high" : 9.14, "low" : 8.73, "close" : 9.09, "volume" : 420700, "adj close" : 9.09 }
{ "_id" : ObjectId("4d094f58c96767d7a0099d4d"), "exchange" : "NASDAQ", "stock_symbol" : "AACC", "date" : "2008-03-03", "open" : 9.68, "high" : 9.69, "low" : 8.98, "close" : 9.15, "volume" : 407200, "adj close" : 9.15 }
{ "_id" : ObjectId("4d094f58c96767d7a0099d4e"), "exchange" : "NASDAQ", "stock_symbol" : "AACC", "date" : "2008-02-29", "open" : 9.52, "high" : 9.76, "low" : 9.25, "close" : 9.75, "volume" : 269400, "adj close" : 9.75 }
{ "_id" : ObjectId("4d094f58c96767d7a0099d4f"), "exchange" : "NASDAQ", "stock_symbol" : "AACC", "date" : "2008-02-28", "open" : 9.7, "high" : 10.1, "low" : 9.67, "close" : 9.7, "volume" : 150200, "adj close" : 9.7 }
{ "_id" : ObjectId("4d094f58c96767d7a0099d50"), "exchange" : "NASDAQ", "stock_symbol" : "AACC", "date" : "2008-02-27", "open" : 9.8, "high" : 10.25, "low" : 9.58, "close" : 9.76, "volume" : 190700, "adj close" : 9.76 }
{ "_id" : ObjectId("4d094f58c96767d7a0099d51"), "exchange" : "NASDAQ", "stock_symbol" : "AACC", "date" : "2008-02-26", "open" : 9.4, "high" : 9.94, "low" : 9.25, "close" : 9.91, "volume" : 205100, "adj close" : 9.91 }
{ "_id" : ObjectId("4d094f58c96767d7a0099d52"), "exchange" : "NASDAQ", "stock_symbol" : "AACC", "date" : "2008-02-25", "open" : 9.84, "high" : 10, "low" : 9.37, "close" : 9.79, "volume" : 352200, "adj close" : 9.79 }
```
</details>

Record structure:
```
db.stocks.find( {} ).pretty().limit( 1 );
{
	"_id" : ObjectId("4d094f58c96767d7a0099d49"),
	"exchange" : "NASDAQ",
	"stock_symbol" : "AACC",
	"date" : "2008-03-07",
	"open" : 8.4,
	"high" : 8.75,
	"low" : 8.08,
	"close" : 8.55,
	"volume" : 275800,
	"adj close" : 8.55
}
```

11. Get indexes and count documents
```
db.stocks.getIndexes();
[ { "v" : 2, "key" : { "_id" : 1 }, "name" : "_id_" } ]

db.stocks.countDocuments( { } );
4308303
```

12. Find deals in the time range 2008-02-13 - 2008-03-05
```
db.stocks.countDocuments( {"date" : { $gt : "2008-02-13", $lt : "2008-03-05"} } );
22346

db.stocks.find({"date" : { $gt : "2008-02-13", $lt : "2008-03-05"} });
```

13. Count executionTimeMillis without Indexes
```
db.stocks.explain("executionStats").find({"date" : { $gt : "2008-02-13", $lt : "2008-03-05"} });
```
<details> 
  <summary>Details</summary>

```json
{
	"explainVersion" : "1",
	"queryPlanner" : {
		"namespace" : "stocks_db.stocks",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"$and" : [
				{
					"date" : {
						"$lt" : "2008-03-05"
					}
				},
				{
					"date" : {
						"$gt" : "2008-02-13"
					}
				}
			]
		},
		"maxIndexedOrSolutionsReached" : false,
		"maxIndexedAndSolutionsReached" : false,
		"maxScansToExplodeReached" : false,
		"winningPlan" : {
			"stage" : "COLLSCAN",
			"filter" : {
				"$and" : [
					{
						"date" : {
							"$lt" : "2008-03-05"
						}
					},
					{
						"date" : {
							"$gt" : "2008-02-13"
						}
					}
				]
			},
			"direction" : "forward"
		},
		"rejectedPlans" : [ ]
	},
	"executionStats" : {
		"executionSuccess" : true,
		"nReturned" : 22346,
		"executionTimeMillis" : 3217,
		"totalKeysExamined" : 0,
		"totalDocsExamined" : 4308303,
		"executionStages" : {
			"stage" : "COLLSCAN",
			"filter" : {
				"$and" : [
					{
						"date" : {
							"$lt" : "2008-03-05"
						}
					},
					{
						"date" : {
							"$gt" : "2008-02-13"
						}
					}
				]
			},
			"nReturned" : 22346,
			"executionTimeMillisEstimate" : 327,
			"works" : 4308305,
			"advanced" : 22346,
			"needTime" : 4285958,
			"needYield" : 0,
			"saveState" : 4308,
			"restoreState" : 4308,
			"isEOF" : 1,
			"direction" : "forward",
			"docsExamined" : 4308303
		}
	},
	"command" : {
		"find" : "stocks",
		"filter" : {
			"date" : {
				"$gt" : "2008-02-13",
				"$lt" : "2008-03-05"
			}
		},
		"$db" : "stocks_db"
	},
	"serverInfo" : {
		"host" : "mongo-ek",
		"port" : 27001,
		"version" : "5.0.2",
		"gitVersion" : "6d9ec525e78465dcecadcff99cce953d380fedc8"
	},
	"serverParameters" : {
		"internalQueryFacetBufferSizeBytes" : 104857600,
		"internalQueryFacetMaxOutputDocSizeBytes" : 104857600,
		"internalLookupStageIntermediateDocumentMaxSizeBytes" : 104857600,
		"internalDocumentSourceGroupMaxMemoryBytes" : 104857600,
		"internalQueryMaxBlockingSortMemoryUsageBytes" : 104857600,
		"internalQueryProhibitBlockingMergeOnMongoS" : 0,
		"internalQueryMaxAddToSetBytes" : 104857600,
		"internalDocumentSourceSetWindowFieldsMaxMemoryBytes" : 104857600
	},
	"ok" : 1
}
```
</details>

```
 "executionTimeMillis" : 3217,
```

14. Add index by date column:
```
db.stocks.getIndexes();
[ { "v" : 2, "key" : { "_id" : 1 }, "name" : "_id_" } ]
```
```
db.stocks.createIndex({'date':1})
{
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"createdCollectionAutomatically" : false,
	"ok" : 1
}
```
```
db.stocks.getIndexes();
[
	{
		"v" : 2,
		"key" : {
			"_id" : 1
		},
		"name" : "_id_"
	},
	{
		"v" : 2,
		"key" : {
			"date" : 1
		},
		"name" : "date_1"
	}
]
```

15. Repeat query from step 13
```
db.stocks.explain("executionStats").find({"date" : { $gt : "2008-02-13", $lt : "2008-03-05"} });
```
<details> 
  <summary>Details</summary>

```json
{
	"explainVersion" : "1",
	"queryPlanner" : {
		"namespace" : "stocks_db.stocks",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"$and" : [
				{
					"date" : {
						"$lt" : "2008-03-05"
					}
				},
				{
					"date" : {
						"$gt" : "2008-02-13"
					}
				}
			]
		},
		"maxIndexedOrSolutionsReached" : false,
		"maxIndexedAndSolutionsReached" : false,
		"maxScansToExplodeReached" : false,
		"winningPlan" : {
			"stage" : "FETCH",
			"inputStage" : {
				"stage" : "IXSCAN",
				"keyPattern" : {
					"date" : 1
				},
				"indexName" : "date_1",
				"isMultiKey" : false,
				"multiKeyPaths" : {
					"date" : [ ]
				},
				"isUnique" : false,
				"isSparse" : false,
				"isPartial" : false,
				"indexVersion" : 2,
				"direction" : "forward",
				"indexBounds" : {
					"date" : [
						"(\"2008-02-13\", \"2008-03-05\")"
					]
				}
			}
		},
		"rejectedPlans" : [ ]
	},
	"executionStats" : {
		"executionSuccess" : true,
		"nReturned" : 22346,
		"executionTimeMillis" : 71,
		"totalKeysExamined" : 22346,
		"totalDocsExamined" : 22346,
		"executionStages" : {
			"stage" : "FETCH",
			"nReturned" : 22346,
			"executionTimeMillisEstimate" : 20,
			"works" : 22347,
			"advanced" : 22346,
			"needTime" : 0,
			"needYield" : 0,
			"saveState" : 22,
			"restoreState" : 22,
			"isEOF" : 1,
			"docsExamined" : 22346,
			"alreadyHasObj" : 0,
			"inputStage" : {
				"stage" : "IXSCAN",
				"nReturned" : 22346,
				"executionTimeMillisEstimate" : 4,
				"works" : 22347,
				"advanced" : 22346,
				"needTime" : 0,
				"needYield" : 0,
				"saveState" : 22,
				"restoreState" : 22,
				"isEOF" : 1,
				"keyPattern" : {
					"date" : 1
				},
				"indexName" : "date_1",
				"isMultiKey" : false,
				"multiKeyPaths" : {
					"date" : [ ]
				},
				"isUnique" : false,
				"isSparse" : false,
				"isPartial" : false,
				"indexVersion" : 2,
				"direction" : "forward",
				"indexBounds" : {
					"date" : [
						"(\"2008-02-13\", \"2008-03-05\")"
					]
				},
				"keysExamined" : 22346,
				"seeks" : 1,
				"dupsTested" : 0,
				"dupsDropped" : 0
			}
		}
	},
	"command" : {
		"find" : "stocks",
		"filter" : {
			"date" : {
				"$gt" : "2008-02-13",
				"$lt" : "2008-03-05"
			}
		},
		"$db" : "stocks_db"
	},
	"serverInfo" : {
		"host" : "mongo-ek",
		"port" : 27001,
		"version" : "5.0.2",
		"gitVersion" : "6d9ec525e78465dcecadcff99cce953d380fedc8"
	},
	"serverParameters" : {
		"internalQueryFacetBufferSizeBytes" : 104857600,
		"internalQueryFacetMaxOutputDocSizeBytes" : 104857600,
		"internalLookupStageIntermediateDocumentMaxSizeBytes" : 104857600,
		"internalDocumentSourceGroupMaxMemoryBytes" : 104857600,
		"internalQueryMaxBlockingSortMemoryUsageBytes" : 104857600,
		"internalQueryProhibitBlockingMergeOnMongoS" : 0,
		"internalQueryMaxAddToSetBytes" : 104857600,
		"internalDocumentSourceSetWindowFieldsMaxMemoryBytes" : 104857600
	},
	"ok" : 1
}
```
</details>

```
"executionTimeMillis" : 71,
```

16. Conclusions: Search by field with index increased "find" speed in 45 times
```
"executionTimeMillis" : 3217, --> "executionTimeMillis" : 71,
```

17. Stop GCP instance for reduce bill
```
gcloud compute instances stop mongo-ek
```

[Back to table of contents](../README.md)