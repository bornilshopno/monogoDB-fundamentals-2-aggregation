```js
    db.test.aggregate([
    //Stage-1
    {$match: {gender:"Male", age:{$lt:30}}},
    //Stage-2
    {$addFields: {course:"Level2", eduTech:"Programming Hero"}},//only updates the pipeline. not original docs
    //Stage-3
    {$project: {gender:1, age:1}},
    //Stage-4
    // {$out: "newCourse"},//adds a new collection containing all elements in pipeline
    {$merge: "test"}//adds the addfields item in the extisting collection
    
    
    ])
    
    db.test.aggregate([
    {$group: { _id: "$gender",count:{$sum:1}}}
    ])
    
    //group with phone  
    db.test.aggregate([
    {$group: { 
        _id: "$gender",
        count:{$sum:1},
        contactNumber:{$push: "$phone"} // will provide a list as Array with all the phone numbers of the pipeline/group element 
    }}
    ])
    
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
//basic structure
db.test.aggregate([
    {$bucket: {
          groupBy: "$price",
          boundaries: [ 0, 200, 400 ],
          default: "Other",
          output: {
            "count": { $sum: 1 },
            "titles" : { $push: "$title" }
          }
        }}
    
    ])

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