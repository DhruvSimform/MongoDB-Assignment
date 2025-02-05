# MongoDB Assignment

This repository demonstrates various MongoDB operations such as batch creation, batch update, indexing, and finding duplicates using aggregation.

## Setup

1. **Connect to MongoDB Cluster**  
   Use the following command to connect to your MongoDB cluster using the connection string in MongoDB shell:

   ```bash
   mongosh "mongodb+srv://mongodb-assignment.uoynp.mongodb.net/" --apiVersion 1 --username dhruvnpatel
   ```
   Enter the user password to connect.

2. **Check Current Database**  
   To check the current databases available in your MongoDB cluster, run:

   ```bash
   show dbs
   ```

3. **Create and Switch to Database**  
   To create and switch to the `mongoDB_task` database, run:

   ```bash
   use mongoDB_task
   ```

## Operations

### 1.  Batch Create with minimum 100 records in MongoDb (create batch).

The function creates more than 200 records in the `users` collection of MongoDB. It generates random user details such as `name`, `age`, `email`, `city`, and `status`, then inserts these records into the database. Additionally, a portion of the records are intentionally duplicated to simulate real-world scenarios where duplicate entries may occur.

- **Random User Data Generation**: The function generates a random user `name`, `age`, `email`, `city`, and `status` for each record. The `name` is uniquely generated based on the loop index, while other fields are randomly selected from predefined arrays (cities and statuses).
  
- **Duplicate Records**: For a portion of the records, the function randomly decides whether to create a duplicate of the current record. This helps in testing the system's behavior when duplicate data is inserted.

- **Insert Records**: All the generated records, including duplicates, are inserted into the `users` collection in MongoDB using the `insertMany()` method.


```javascript
const cities = ["New York", "Los Angeles", "Chicago", "Houston", "San Francisco"];
const statuses = ["active", "deactivated"];
const records = [];

for (let i = 1; i <= 200; i++) {
    const duplicateFlag = Math.round(Math.random()); // Randomly 0 or 1
    const record = {
        name: `User${i}`,
        age: Math.floor(Math.random() * 51) + 20, // Random age between 20-70
        email: `user${i}@example.com`,
        city: cities[Math.floor(Math.random() * cities.length)], // Random city
        status: statuses[Math.floor(Math.random() * statuses.length)], // Random status
    };

    records.push(record); // Insert normally

    if (duplicateFlag === 1) {
        records.push({ ...record }); // Insert duplicate
    }
}

db.users.insertMany(records)
```

### 2. Batch Update with minimum 100 records  in MongoDB (update batch).

This operation updates the `age` field of users with the status "active" by incrementing their age by 1.

```javascript
db.users.updateMany(
    {"status": "active"},
    {$inc: {age: 1}}
)
```

### 3. Perform indexing on particular 3 fields in MongoDB.

#### Before Indexing:

To analyze query performance before indexing, run the following command to explain how the query is executed:

```javascript
db.users.explain().find({"city": "Chicago", "status": "active", "age": { $gt: 40 ,$lt:60} })
```
#### Creating Indexes:

Create a compound index on the `city`, `status`, and `age` fields in a single command to improve query performance:

```javascript
db.users.createIndex({ city: 1, status: 1, age: 1 })
```
This command creates a compound index on the `city`, `status`, and `age` fields. The value `1` indicates ascending order, meaning the index will sort the fields in ascending order.

#### After Indexing:

Once the indexes are created, run the same query again to see how the query execution plan changes after indexing:

```javascript
db.users.explain().find({"city": "Chicago", "status": "active", "age": { $gt: 40 ,$lt:60} })
```

### 4. Find Duplicates Using Aggregation

This aggregation query groups users by their `name`, `age`, `email`, `city`, and `status` to identify any duplicates.

```javascript
db.users.aggregate([
    {
        $group: {
            _id: {
                name: "$name",      
                age: "$age",
                email: "$email",
                city: "$city",
                status: "$status"
            },
            count: { $sum: 1 }
        }
    },
    {
        $match: {
            count: { $gt: 1 }
        }
    },
    {
        $project: {
            _id: 0, 
            name: "$_id.name",
            age: "$_id.age",
            email: "$_id.email", 
            city: "$_id.city",
            status: "$_id.status",
            count: 1 
        }
    }
]);
```
