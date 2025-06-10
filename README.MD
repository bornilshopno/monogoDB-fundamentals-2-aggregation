

## 01--- $match

filters document with condition

```js
//will return all the docs of Male who are less than 30
db.collectionName.aggregate([
    {$match: {gender:"Male", age:{$lt:30}}}
])

```
--------------------------------------

## 02-- $group

Groups documents by a field and lets us do calculations
```js
//groups documnet by gender field and provide number of docs in each field value
db.test.aggregate([
    {$group: { _id: "$gender",count:{$sum:1}}}
    ])
    
//$push tag is used to show the field in the doc. here it will show all phone number of the specific array  
db.test.aggregate([
    {$group: { 
        _id: "$gender",
        count:{$sum:1},
        contactNumber:{$push: "$phone"} // will provide a list as Array with all the phone numbers of the pipeline/group element 
    }}
    ])
```
---------------------------------------

## $addFields

Adds new fields to documents.
```js
db.collectionName.aggregate([
    {
        $addFields:{
            totalPrice:{$multiplyL:["$price", "quantity"]}
        }
    },
    //create "newCollection" document adding totalPrice to collectionName
    {$out: "newCollection"},
    //updates existing collectionNme document with totalPrice
    {$merge: "collectionName"}
])

```
----------------------------------------

## $unwind

Flattens arrays (one item per document).Each array element becomes its own document.


```js
db.collectionName.aggregate([
    //stage-1
    {$unwind: "$interests"},//flattens the interests array
    //stage-2
    {$group:{_id:"$age", interest: {$push: "$interests"}}}//groups by age and shows interest of that age/range
])

```
-----------------------------------------

## $bucket

basic structure consists of groupBy, boundaries, default, output

```js
db.collectionName.aggregate([
    {$bucket: {
        groupBy: "$price",
        boundaries: [0, 100, 200],
        default: "Porsche Products",
        output:{
            count:{ $sum : 1 },
            titles: { $push: "titles"}
        }
    }}
])
```
-----------------------------------------

## $facet

$facet allows you to run multiple pipelines in parallel on the same input documents.

Each sub-pipeline can have different logic (e.g., filtering, grouping, counting).

Useful when you want to return multiple views of the data at once (like search results + stats).

```js
//basic structure 
db.collectionName.aggregate([
    {$facet:
    //pipeline-1
           friends:[],
    //pipeline-2
           skills:[],
    //pipeline-3
           languages:[]
    }
])

// Details Use Case
db.test.aggregate([
    {$facet:
    {
    //pipeline:1
        "friends":[
            {$unwind: "$friends"},
            {$group: { _id: "$friends", count:{$sum:1}}}
            ],
    //pipeline:2
        "skills":[
            {$unwind: "$skills"},
            {$group: { _id: "$skills",count:{$sum:1}}}
            ],            
    //pipeline:3     
        "languages":[
            {$unwind: "$languages"},
            {$group: { _id: "$languages", count:{$sum: 1}}}
            ]
    }    
    }
    ])

```


hello this is the begining



```js
 
    
    
    
    //group with full doc {$push: "$$ROOT"}
    db.test.aggregate([{
    $group: { 
        _id: "$gender",
        count:{$sum: 1},
        fullDoc:{$push: "$$ROOT"}
    }
},
    {$project: {
        "fullDoc.gender":1,
        "fullDoc.age":1,
       " fullDoc.email":1,
        "fullDoc.phone":1
    }}])
//aggregate by null, min,max,avg,calculation: add,subtract in project
  db.test.aggregate([
    {$group: { 
        _id: null, //captures all element in test
        totalSalary: {$sum: "$salary"},
        avgSalary: {$avg: "$salary"},
        maxSalary: {$max: "$salary"},
        minSalary: {$min: "$salary"},
    }
    },
    {$project: {
        totalSalary:1, 
        averageSalery:"$avgSalary",
        salaryRange:{$subtract: ["$maxSalary","$minSalary"]},
        addMinMax:{$add: ["$maxSalary","$minSalary"]}
    }}])
    
//unwind --array
db.test.aggregate([
    //stage-1
    {$unwind: "$friends"},
    //stage-2
    {$group: { _id: "$friends", count:{$sum:1}}    
    }
    ])
    
    
db.test.aggregate([
    //stage-1
    {$unwind: "$interests"},
    //stage-2
    {$group: { _id: "$age", interest:{$push:"$interests"}}    
    }
    ])
    
    
//bucket==>> groupBy,boundaries,default,output


//useCase
db.test.aggregate([
    //stage-1
    {$bucket: {
          groupBy: "$age",
          boundaries: [0, 20, 40, 60,80,100 ],
          default: "800 & Older than 80",
          output: {
            count: { $sum: 1 },
            age : { $push: "$age" }
          }
        }},
        {$sort: {count:1}},
        {$limit: 3},
        {$project: {count:1}}
    ])


```




## Further examples
```js

db.test.aggregate([
    {$facet:
    {
    //pipeline:1
        "friends":[
            {$unwind: "$friends"},
            {$group: { _id: "$friends", count:{$sum:1}}}
            ],
    //pipeline:2
        "skills":[
            {$unwind: "$skills"},
            {$group: { _id: "$skills",count:{$sum:1}}}
            ],
            
            
        "languages":[
            {$unwind: "$languages"},
            {$group: { _id: "$languages", count:{$sum: 1}}}
            ]
    }    
    }
    ])
    
    
    
//Basic Look Up Structure
db.orders.aggregate(
    {
        $lookup: {
               from: "<collection to join>",
               localField: "<field from the input documents>",
               foreignField: "<field from the documents of the from collection>",
               as: "<output array field>"
             }
    }
    )
    
//usecase
db.orders.aggregate(
    {
        $lookup: {
               from: "test",//looking up from test db
               localField: "userId",// searchparameters in orders db
               foreignField: "_id", //searchparameters in test db
               as: "customer_details" //naming of testdb information in aggregated orders
             }
    }
    )
    
//indexing
db.getCollection("massive-data").createIndex({email:1})
db.getCollection("massive-data").createIndex({about: "text"})
//collscan ixscan idhack
db.getCollection("massive-data").dropIndex({email:1})    

db.test.find({_id: new ObjectId("6406ad63fc13ae5a40000065")}).explain("executionStats")
db.getCollection("massive-data").find({$text: { $search: "dolor" }},{about:1})


```