


## $project, $limit, $sort, $skip, $count  in aggregation

```js
//projects

db.collectionName.aggregate([

{
      $sort: { price: 1} //+1 for ascending -1 for descending
},
{ 
      $project: { name: 1, price: 1, _id: 0 }  //includes name and price but excludes _id
},
{
      $skip: 10  // Skip first 10 documents.
},
{
      $limit: 3 // show only 3 results
},
{
       $count: "totalDocuments" // output: [{ totalDocuments: 3 }]

}
])



```
## $match

filters document with condition

```js
//will return all the docs of Male who are less than 30
db.collectionName.aggregate([
    {$match: {gender:"Male", age: { $lt : 30}}}
])

```
--------------------------------------

## $group

Groups documents by a field and lets us do calculations
```js
//groups documnet by gender field and provide number of docs in each field value
db.test.aggregate([
    {$group: { _id: "$gender",count: {$sum : 1}}}
    ])
    
//$push tag is used to show the field in the doc. here it will show all phone number of the specific array  
db.test.aggregate([
    {$group: { 
        _id: "$gender",
        count:{$sum:1},
        contactNumber:{$push: "$phone"} // will provide a list as Array with all the phone numbers of the pipeline/group element. {$push: "$$ROOT"} will provide the full documents with group element
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
            totalPrice: {$multiply: ["$price", "quantity"]}
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
    {$group:{_id: "$age", interest: {$push: "$interests"}}}//groups by age and shows interest of that age/range
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

facet allows you to run multiple pipelines in parallel on the same input documents.

Each sub-pipeline can have different logic e.g. filtering, grouping, counting.

Useful when you want to return multiple views of the data at once like search results + stats.

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

----------------------------------------------

## $LookUp
consists of an object with form , localField , foreignField, as.

matches with connected parameter from a foreign collection and pulls related data to the local collection.

```js

db.orders.aggregate(
    {
        $lookup: {
               from: "test",//looking up from test collection
               localField: "userId",// searchparameters in orders db
               foreignField: "_id", //searchparameters in test db
               as: "customer_details" //naming of testdb information in aggregated orders collection
             }
    }
    )

```
-----------------------------------------------

## aggregate by null, min,max,avg,add,subtract calculation: in project
```js
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
```
---------------------------------------------
## group with full doc {$push: "$$ROOT"} and use of $project 

```js
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

```
--------------------------------------------------
## Further examples

```js
//indexing with one field
db.getCollection("massive-data").createIndex({email:1})
db.getCollection("massive-data").dropIndex({email:1})   

//for query status
db.test.find({_id: new ObjectId("6406ad63fc13ae5a40000065")}).explain("executionStats")

//for search fileds with text
db.getCollection("massive-data").createIndex({about: "text"})
db.getCollection("massive-data").find({$text: { $search: "dolor" }},{about:1})

```
-------------------------------------------------

### Basics

collscan === page by page scan (timeconsuming)

ixscan=== index scan (time saving but db memory using) 

idhack=== mongoDBs own indexing with _id

### Referncing vs Embedding

### Embedding

Store related data inside the same document.

Best for: One-to-few or tightly related data.
```js
{
  name: "Alice",
  address: {
    city: "Dhaka",
    zip: "1200"
  }
}
```
Pros:
Fast reads (everything in one place).

Simple to retrieve.

Cons:
Document size limit (16MB).

Hard to update if nested data is large or reused elsewhere.

### Referencing

Store related data in separate documents, and link them using an ID.
```js
// User document
{ _id: 1, name: "Ashraf", addressId: 101 }

// Address document
{ _id: 101, city: "Dhaka", zip: "1200" }

```

Pros:

Flexible and normalized.

Better for one-to-many or many-to-many relationships.

Cons:

Requires extra queries or $lookup to fetch related data.

Slower reads if many joins.

--------------------------------------------------

## $substrBytes

Syntax:

```
$substrBytes: [ <string>, <start>, <length> ]
```
string: The string field or value to extract from.

start: The position (0-based) to start the substring from.

length: How many bytes to extract.

```js
//In $substrBytes, setting the length to -1 means:take everything from the start index to the end of the string.
{$substrBytes: ["$balance", 1, -1] } //output: "1893.92" when { "balance": "$1893.92" }
{ $substrBytes: ["Hello", 1, -1] }  // → "ello"
{ $substrBytes: ["abcdef", 0, 3] }  // → "abc"


```

--------------------------------------------

### Further Practice Problems [Practice Problem - 2]: 

```js
//1. Retrieve the count of individuals who are active (isActive: true) for each gender.

db.indiviualInfo.aggregate([
    {$match: {isActive: true}},
    {$group: {_id: "$gender", count:{$sum:1}}}
    ])
    
//optional for aggregating with 2 fields
db.indiviualInfo.aggregate([
    {$group: {_id: {active:"$isActive", gender:"$gender"}, count:{$sum:1}}}
    ])
    
//2. Retrieve the names and email addresses of individuals who are active (`isActive: true`) and have a favorite fruit of "banana".
db.indiviualInfo.aggregate([
    {$match: {favoriteFruit : "banana" , isActive: true}},
    {$project: {name:1, email:1,isActive:1, favoriteFruit:1}}
    ])
    
//3. Find the average age of individuals for each favorite fruit, then sort the results in descending order of average age
db.getCollection("indiviualInfo").aggregate([
    {$group: { _id: "$favoriteFruit", aveAge:{$avg: "$age"}}},
    {$sort: {aveAge: -1}}
    ])

//4. Retrieve a list of unique friend names for individuals who have at least one friend, and include only the friends with names starting with the letter "W".
//couldnot do it yet
    
//5.  Use $facet to separate individuals into two facets based on their age:
//those below 30 and those above 30. Then, within each facet, bucket theindividuals into age ranges (e.g., 20-25, 26-30, etc.) and sort them by age within each bucket.
db.indiviualInfo.aggregate([
    {$facet: {
        //pipeline-1
        "YOUNGERS":[
            {$match: {age: { $lte: 30}}},
          {$bucket:
                {
                groupBy: "$age",
                boundaries: [6,11,16,21,26],
                default: "0-5",
                output: {count: {$sum :1}}
                    
                }
            },
            {$sort:{count:1}}
            ],
        //pipeline-2
        "OLDERS":[
            {$match: {age: {$gt : 30}}},
            {$bucket:{
                groupBy: "$age",
                boundaries:[31,36,41],
                default:"46Y+",
                output: {count: {$sum:1}}
            }},
            {$sort:{count:1}}
            ]
    }}])
    
//6. Calculate the total balance of individuals for each company and displaythe company name along with the total balance. Limit the result to show only the top two companies with the highest total balance
db.indiviualInfo.aggregate([
    {$project:{company:1, 
    balance:{ $toDouble: {$substrBytes: ["$balance", 1, -1]}}}},
    {$group: { _id: "$company", totalBalance: {$sum:"$balance"}, count:{$sum:1}} },
    {$sort: {totalBalance:-1}},
    {$limit: 2}
    ])
```

---------------------------------------------------------------------