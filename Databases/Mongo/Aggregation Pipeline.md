[[Mongo]]
Documentation: https://www.mongodb.com/docs/manual/core/aggregation-pipeline/
``` toc
title: "## Table of Contents"
```
### Overview 
An aggregation pipeline consists of one or more [stages](https://www.mongodb.com/docs/manual/reference/operator/aggregation-pipeline/#std-label-aggregation-pipeline-operator-reference) that process documents:

Each stage performs an operation on the input documents. For example, a stage can filter documents, group documents, and calculate values. The documents that are output from a stage are passed to the next stage. An aggregation pipeline can return results for groups of documents. For example, return the total, average, maximum, and minimum values.

###  Aggregation Stages

| Operator | Description | Example |
| --- | --- | --- |
| $match | Matches documents under a condition (same as filter) | {\$match: {remote_id: "user-id-1"}} |
| $unwind | Deconstructs an array field from the input documents to output a document for _each_ element. | {\$unwind: {\$history}} |
| $sort | Reorders the document stream by a specified sort key | {\$sort: {timestamp: -1}} |
|  \$group | Groups input documents by a specified identifier expression into one document (or more if the \_id property is a variable). The output document(s) only contains the \_id field and specified accumulated fields. | `{$group: {_id: "$history.show_id", timestamp: {$max: "$history.timestamp"}}}` |

### Cheat-Sheet

##### Get values between two timestamps in subarray for single document
``` php
$this->collection->aggregate([  
    ['$match' => ['remote_id' => $userId]],  
    ['$project' => [  
	        'history' => [  
	            '$filter' => [  
	                'input' => '$history',  
	                'as' => 'history',  
	                'cond' => [  
	                    '$and' => [  
	                        ['$gte' => ['$$history.timestamp', $dateFrom->getTimestamp()]],  
	                        ['$lte' => ['$$history.timestamp', $dateTo->getTimestamp()]]  
	                    ]                
					]            
				]
			]    
		]
	]
]);
```

![[Example Document]]