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


``