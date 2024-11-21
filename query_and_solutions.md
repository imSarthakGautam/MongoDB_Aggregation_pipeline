1. How many users are active ?

```js
db.users.aggregate([

    //stage 1
    {
        $match : {
            isActive: true
        }
    },

    //output of stage 1 is passed to stage 2--

    //Therefore now the db is only active users for stage 2
    //let's say count the active users in stage 2
    {
        $count : 'activeUsers'
        //Provide the field name for the count as activeUsers
    }

])
```

2. What is the average age of all the users ?

```js
db.users.aggregate([

    //stage 1
    {
        $group : {
            _id: null, // a singel large group
            average : {
                $avg: '$age' //avg is accumulator
            }

        }
    },
    {}
])
```
 This accumulator objects must be one of the few operators: like `$avg, $sum, $top`

3. Average age of males and females ?
```js
db.users.aggregate([

    //stage 1
    {
        $group : {
            _id: "$gender", //groups users on the basis of gender : male, female
            average : {
                $avg: '$age'
            }

        }
    },

])
```
4. List top 5 common fruit among all the users

```js
db.users.aggregate([

    //stage 1
    { 
        $group : {

            _id : "$favouriteFruit",
            count : { 
                $sum: 1 // adds +1 whenever you find one instance
             }        
        }
    },

    //stage 2 
    {  $sort :{
        count : -1
    }},

    { $limit : 5}
])
```

6. Find total numbers of males and females.

```js
db.users.aggregate([ 
    { 
    $group :{
        _id : '$gender'
        count : {
            $sum : 1
        }    }
    }
])
```

7. Which country has highest number of registered users

```js
db.users.aggregate([ 

    {
        $group: {
            _id: "$company.location.country"
            count : {
                $sum : 1
            }
        }

    },

    { $sort : {
        count : -1
    }},
    { 
        $limit : 1
    }
])
```

8. List all the unique eye colors present.
```js
db.users.aggregate([ 
    {
        $group : {
            _id : '$eyeColor'
        }
    }
])
```

9. What is the average numnber of tags per user ?

- to spread the array we use unwind, for an array with 3 elements it creates 3 documents.

```js
db.users.aggregate([

    {
        $unwind : '$tags',  
    },
    {
        $group : {
            _id : '$_id'
            numberOfTags: { 
                $sum : 1 
                }
        }
    }
    {
        $group : {
            _id: null,
            avgNoOftags: {
                $avg: "$numberOfTags"
            }
        }
    }

 ])
```
Approach 2 : addFields
```js

db.users.aggregate([
{
    $addFileds : {
        numberOfTags : {
            $size : {$ifNull :[`$tags`, []]}
        }
    }
},{
     $group : {
            _id: null,
            avgNoOftags: {
                $avg: "$numberOfTags"
            }
        }

}
```

10. How many users have 'enim' as one of their tags
```js
db.users.aggregation([
    {
        $match :
        {
            //query
            tags: "enim"
        }
    },
    {
        $group : {
            _id: null,
            totalEnims: {
                $sum: 1
                    }
        }
    }
])
```

11. what are the names and age of users who are inactive and have velit as a tag ?

projects can filter out the feilds (new,old) to pass on to next stage 
```js
db.users.aggregation([
    {
        $match :
        {
            //query
            tags: "velit"
        }
    },
    {
        $project : {
            _id:0,
            name:1,
            age:1
        }

    }
    ])
```

12. how many users have phone no starting with '+1 (940)'
```js
db.users.aggregation([
    {
        $match :
        { //regex
        "company.phone": /^\+1 \(940\)/
        }
    }, {
        $count: 'Users with the sp phone no starting with +1 (940)'
    }
    ])
```

13. who has registered more recently
```js
db.users.aggregation([
    {
        $sort :{
            registered: -1
        }
    },
    {
      $limit : 4   
    },{
        $project : {
            _id:0,
            name:1
        }

    }
])
```
```js
db.users.aggregation([
{
$group : {
  _id:'$favoriteFruit',
  
  totalUsers:{
    $push :"$name" 
  }
}
}])
```
14. How many users have 'ad' as the second tag in their tags
```js
db.users.aggregation([
{
    $match :{
        "tags.1" : "ads"
    }

},
{
  $count : 'secondTag'
}
])
```





### Lookup
```js
db.books.aggregation([
  {
    $lookup: {
      from: 'authors',
      localField: 'author_id',
      foreignField: '_id',
      as: 'author_details'
    }
  },
  {
    $addFields: {
      author_details: {
        $first: '$author_details'
    }
  }
}
])
```