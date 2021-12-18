# HW3 - Basic MongoDB Concepts, CRUD, Filters

1. Launch `mongo-ek` instance remotely on Gcloud with MongoDB v5.0 configured in the [homework 2](https://github.com/LirayKH/otus_mongodb_course/blob/main/HW2/README.md).

```
gcloud compute instances list
gcloud compute instances start mongo-ek
```

2. Connect to the remote server, start MongoDB. Add service to the autostart for future server restarts:
```
gcloud compute ssh mongo-ek --ssh-key-file=/Users/$USER/.ssh/id_ecdsa_sk
systemctl status mongod
systemctl start mongod
systemctl enable mongod
```

3. Logout from server, get `mongo-ek` remote server IP and connect to mongodb remotelly through MongoDB client:
```
gcloud compute instances describe mongo-ek --format='get(networkInterfaces[0].accessConfigs[0].natIP)'

mongo $(gcloud compute instances describe mongo-ek --format='get(networkInterfaces[0].accessConfigs[0].natIP)'):27001 -u mongo_db_admin -p mongo_db_admin_pwd --authenticationDatabase admin
```

4. Install [mongoexport](https://docs.mongodb.com/database-tools/mongoexport/) utill to MacOS
```
brew install mongodb/brew/mongodb-database-tools
```

5. Download [Mall_Customers dataset](https://www.kaggle.com/shwetabh123/mall-customers/download) from https://www.kaggle.com/shwetabh123/mall-customers

6. Import dataset:
```
mongoimport --host $(gcloud compute instances describe mongo-ek --format='get(networkInterfaces[0].accessConfigs[0].natIP)') --port 27001 -u mongo_db_admin -p mongo_db_admin_pwd --authenticationDatabase admin --drop --collection customers_info --db=mail_customers --type csv --headerline --file Mall_Customers.csv
```

7. Check database fillings:
```
use mail_customers;
show collections;
db.customers_info.find()
db.customers_info.countDocuments( { } );
200
```

8. Install Compas utility:
https://www.mongodb.com/try/download/compass

9. Connect from Compass APP (only for data representation):
```
gcloud compute instances describe mongo-ek --format='get(networkInterfaces[0].accessConfigs[0].natIP)'
```
```
mongodb://mongo_db_admin:mongo_db_admin_pwd@35.197.125.63:27001
```

10. Find:
```
use mail_customers
```
- Find all values:
```
db.customers_info.find( {} );
```
- Sort by Age:
```
db.customers_info.find().sort( { 'Age': 1 } )
```
`"1" (for ascending) or "-1" (for descending))`

- Find the richest persons and sort them by Age starting from young
```
db.customers_info.find().sort( { "Annual Income (k$)": -1, 'Age': 1 } )
```
- Find richest men:
```
db.customers_info.find( { "Genre" : "Male" } ).sort( { "Annual Income (k$)": -1, 'Age': 1 } )
```

11. Agregate users by age:
```
    db.customers_info.aggregate([
        { $group : { _id : "$Age", Total : { $sum : 1} } },
        { $sort : { "Total" : -1 } }
    ]
    )
```

12. Insert:
- Add 3 new person to the list:
```
> db.customers_info.insert([
... { "CustomerID" : 201, "Genre" : "Female", "Age" : 28, "Annual Income (k$)" : 138, "Spending Score (1-100)" : 77 },
... { "CustomerID" : 202, "Genre" : "Female", "Age" : 33, "Annual Income (k$)" : 143, "Spending Score (1-100)" : 88 },
... { "CustomerID" : 203, "Genre" : "Male", "Age" : 28, "Annual Income (k$)" : 142, "Spending Score (1-100)" : 69 } ])
```

- Find the richest persons:
```
db.customers_info.find().sort( { "Annual Income (k$)": -1, 'Age': 1 } ).limit( 3 )
```

13. Update:
- Find poorest person:
```
db.customers_info.find().sort( { "Annual Income (k$)": 1 } ).limit( 1 )
```

- Increase Annual Income for a poorest person:
```
db.customers_info.update({"Annual Income (k$)": 15}, {$set: {"Annual Income (k$)": 50}})
```

- Find all persons who Annual Income lower than 25:
```
db.customers_info.find( { "Annual Income (k$)": { $lt: 25 } } );
```

- Increase "Annual Income" +10 all persons whose "Annual Income" lower than 25:
```
db.customers_info.updateMany({"Annual Income (k$)": { $lt: 25 } }, {$inc: {"Annual Income (k$)": 10}})
```

14. Delete
- Count documents in the table:
```
db.customers_info.countDocuments( {} );
```
- Delete records where Annual Income lower than value 40
```
db.customers_info.deleteMany({"Annual Income (k$)": { $lt: 40 } })
```
- Recheck count documents in the table:

15. Stop GCP instance for reduce bill
```
gcloud compute instances stop mongo-ek
```

[Back to table of contents](../README.md)