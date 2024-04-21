# Database-NOSQL-Project

# MongoDB Query Examples

This repository contains MongoDB query examples for various analytical tasks. Each query is designed to answer specific questions related to the database's data, such as vendor registration, customer analysis, revenue tracking, and database optimization.

## Queries Overview

#### 1. **How many vendors do you have registered?**
    ```mongodb
    db.VendorAuthentication.countDocuments()
    ```

#### 2. **Number of active vs inactive vendors this month**
    ```mongodb
    db.vendor.aggregate([
    {
        $match: {
        $expr: {
            $and: [
            { $eq: [{ $month: { $toDate: "$register_timestamp" } }, 3] }, // Assuming March is represented by 3
            { $eq: [{ $year: { $toDate: "$register_timestamp" } }, { $year: new Date() }] }
            ]
        }
        }
    },
    {
        $group: {
        _id: "$Status", // Group by the 'Status' field
        count: { $sum: 1 } // Count the occurrences of each status
        }
    },
    {
        $group: {
        _id: null, // Group by a constant value to merge all documents into one
        Active_Vendors: { $sum: { $cond: [{ $eq: ["$_id", 1] }, "$count", 0] } }, // Sum the count of active vendors
        Inactive_Vendors: { $sum: { $cond: [{ $eq: ["$_id", 0] }, "$count", 0] } } // Sum the count of inactive vendors
        }
    },
    {
        $project: {
        _id: 0 // Exclude the '_id' field
        }
    }
    ]);
    ```

#### 3. **Total number of customers registered from the beginning**
    ```mongodb
    db.vendor.aggregate([
    {
        $match: {
        $expr: {
            $gte: [{$toDate: "$register_timestamp"}, new Date('2021-01-01')]
        }
        }
    },
    {
        $group: {
        _id: null,
        count: {$sum: 1}
        }
    },
    {
        $project: {
        _id: 0,
        Total_Customer_registered_from_beginning: "$count"
        }
    }
    ]);
    ```

#### 4. **Vendor with the most number of listings**
    ```mongodb
    db.vendor.aggregate([
    {
        $lookup: {
        from: "inventory_item",
        localField: "vendor_email",
        foreignField: "vendor_email",
        as: "inventory"
        }
    },
    {
        $addFields: {
        num_listings: { $size: "$inventory" }
        }
    },
    {
        $sort: { num_listings: -1 }
    },
    {
        $limit: 1
    },
    {
        $project: {
        _id: 0,
        vendor_email: "$vendor_email",
        vendor_name: { $concat: [ "$First_Name", " ", "$Last_Name" ] },
        num_listings: 1
        }
    }
    ]);
    ```

#### 5. **Customer with the most number of orders**
    ```mongodb
    db.order.aggregate([
    {
        $lookup: {
        from: "customer",
        localField: "cust_email",
        foreignField: "cust_email",
        as: "customer"
        }
    },
    {
    $unwind: "$customer"
    },
    {
        $group: {
        _id: {
            cust_email: "$cust_email",
            customer_name: { $concat: ["$customer.First_name", " ", "$customer.Last_name"] }
        },
        num_orders: { $sum: 1 }
        }
    },
    {
        $sort: { num_orders: -1 }
    },
    {
        $group: {
        _id: "$num_orders",
        customers: { $push: { customer_email: "$_id.cust_email", customer_name: "$_id.customer_name", num_orders: "$num_orders" } }
        }
    },
    {
        $sort: { "_id": -1 }
    },
    {
        $limit: 1
    },
    {
        $unwind: "$customers"
    },
    {
        $project: {
        _id: 0,
        customer_email: "$customers.customer_email",
        customer_name: "$customers.customer_name",
        num_orders: "$customers.num_orders"
        }
    }
    ]);
    ```

#### 6. **Top 5 vendors by revenue by month (January, February, March)**
    ```mongodb
    // Top 5 Vendors by Revenue by Jan

    db.order.aggregate([
    {
        $match: {
        $expr: { $eq: [{ $month: { $toDate: "$Order_Placed_timestamp" } }, 1] }
        }
    },
    {
        $lookup: {
        from: "order_details",
        localField: "order_id",
        foreignField: "order_id",
        as: "order_details"
        }
    },
    {
        $unwind: "$order_details"
    },
    {
        $lookup: {
        from: "item",
        localField: "order_details.item_id",
        foreignField: "item_id",
        as: "item"
        }
    },
    {
        $unwind: "$item"
    },
    {
        $lookup: {
        from: "inventory_item",
        localField: "item.item_id",
        foreignField: "item_id",
        as: "inventory_item"
        }
    },
    {
        $unwind: "$inventory_item"
    },
    {
        $lookup: {
        from: "vendor",
        localField: "inventory_item.vendor_email",
        foreignField: "vendor_email",
        as: "vendor"
        }
    },
    {
        $unwind: "$vendor"
    },
    {
        $group: {
        _id: {
            vendor_email: "$vendor.vendor_email",
            vendor_name: { $concat: ["$vendor.First_Name", " ", "$vendor.Last_Name"] },
            month: { $month: { $toDate: "$Order_Placed_timestamp" } }
        },
        Revenue: { $sum: { $multiply: ["$item.price", "$order_details.item_qty"] } }
        }
    },
    {
        $sort: { Revenue: -1 }
    },
    {
        $limit: 5
    },
    {
        $project: {
        _id: 0,
        Vendor_Name: "$_id.vendor_name",
        vendor_email: "$_id.vendor_email",
            Month: {
        $switch: {
            branches: [
            { case: { $eq: ["$_id.month", 1] }, then: "Jan" },
            { case: { $eq: ["$_id.month", 2] }, then: "Feb" },
            { case: { $eq: ["$_id.month", 3] }, then: "Mar" },
            ],
            default: "Unknown"
        }
        },
        Revenue: 1
        }
    }
    ]);


    // Top 5 Vendors by Revenue by Feb

    db.order.aggregate([
    {
        $match: {
        $expr: { $eq: [{ $month: { $toDate: "$Order_Placed_timestamp" } }, 2] }
        }
    },
    {
        $lookup: {
        from: "order_details",
        localField: "order_id",
        foreignField: "order_id",
        as: "order_details"
        }
    },
    {
        $unwind: "$order_details"
    },
    {
        $lookup: {
        from: "item",
        localField: "order_details.item_id",
        foreignField: "item_id",
        as: "item"
        }
    },
    {
        $unwind: "$item"
    },
    {
        $lookup: {
        from: "inventory_item",
        localField: "item.item_id",
        foreignField: "item_id",
        as: "inventory_item"
        }
    },
    {
        $unwind: "$inventory_item"
    },
    {
        $lookup: {
        from: "vendor",
        localField: "inventory_item.vendor_email",
        foreignField: "vendor_email",
        as: "vendor"
        }
    },
    {
        $unwind: "$vendor"
    },
    {
        $group: {
        _id: {
            vendor_email: "$vendor.vendor_email",
            vendor_name: { $concat: ["$vendor.First_Name", " ", "$vendor.Last_Name"] },
            month: { $month: { $toDate: "$Order_Placed_timestamp" } }
        },
        Revenue: { $sum: { $multiply: ["$item.price", "$order_details.item_qty"] } }
        }
    },
    {
        $sort: { Revenue: -1 }
    },
    {
        $limit: 5
    },
    {
        $project: {
        _id: 0,
        Vendor_Name: "$_id.vendor_name",
        vendor_email: "$_id.vendor_email",
            Month: {
        $switch: {
            branches: [
            { case: { $eq: ["$_id.month", 1] }, then: "Jan" },
            { case: { $eq: ["$_id.month", 2] }, then: "Feb" },
            { case: { $eq: ["$_id.month", 3] }, then: "Mar" },
            ],
            default: "Unknown"
        }
        },
        Revenue: 1
        }
    }
    ]);


    // Top 5 Vendors by Revenue by Mar

    db.order.aggregate([
    {
        $match: {
        $expr: { $eq: [{ $month: { $toDate: "$Order_Placed_timestamp" } }, 3] }
        }
    },
    {
        $lookup: {
        from: "order_details",
        localField: "order_id",
        foreignField: "order_id",
        as: "order_details"
        }
    },
    {
        $unwind: "$order_details"
    },
    {
        $lookup: {
        from: "item",
        localField: "order_details.item_id",
        foreignField: "item_id",
        as: "item"
        }
    },
    {
        $unwind: "$item"
    },
    {
        $lookup: {
        from: "inventory_item",
        localField: "item.item_id",
        foreignField: "item_id",
        as: "inventory_item"
        }
    },
    {
        $unwind: "$inventory_item"
    },
    {
        $lookup: {
        from: "vendor",
        localField: "inventory_item.vendor_email",
        foreignField: "vendor_email",
        as: "vendor"
        }
    },
    {
        $unwind: "$vendor"
    },
    {
        $group: {
        _id: {
            vendor_email: "$vendor.vendor_email",
            vendor_name: { $concat: ["$vendor.First_Name", " ", "$vendor.Last_Name"] },
            month: { $month: { $toDate: "$Order_Placed_timestamp" } }
        },
        Revenue: { $sum: { $multiply: ["$item.price", "$order_details.item_qty"] } }
        }
    },
    {
        $sort: { Revenue: -1 }
    },
    {
        $limit: 5
    },
    {
        $project: {
        _id: 0,
        Vendor_Name: "$_id.vendor_name",
        vendor_email: "$_id.vendor_email",
            Month: {
        $switch: {
            branches: [
            { case: { $eq: ["$_id.month", 1] }, then: "Jan" },
            { case: { $eq: ["$_id.month", 2] }, then: "Feb" },
            { case: { $eq: ["$_id.month", 3] }, then: "Mar" },
            ],
            default: "Unknown"
        }
        },
        Revenue: 1
        }
    }
    ]);
    ```

#### 7. **Top 5 customers by revenue this year**
    ```mongodb
    db.order.aggregate([
    {
        $match: {
        $expr: {
            $eq: [{ $year: { $toDate: "$Order_Placed_timestamp" } }, { $year: new Date() }]
        }
        }
    },
    {
        $lookup: {
        from: "order_details",
        localField: "order_id",
        foreignField: "order_id",
        as: "order_details"
        }
    },
    {
        $unwind: "$order_details"
    },
    {
        $lookup: {
        from: "item",
        localField: "order_details.item_id",
        foreignField: "item_id",
        as: "item"
        }
    },
    {
        $unwind: "$item"
    },
    {
        $lookup: {
        from: "customer",
        localField: "cust_email",
        foreignField: "cust_email",
        as: "customer"
        }
    },
    {
        $unwind: "$customer"
    },
    {
        $group: {
        _id: {
            cust_email: "$customer.cust_email",
            Customer_Name: { $concat: ["$customer.First_name", " ", "$customer.Last_name"] },
            year: { $year: { $toDate: "$Order_Placed_timestamp" } }
        },
        Revenue: { $sum: { $multiply: ["$item.price", "$order_details.item_qty"] } }
        }
    },
    {
        $sort: { Revenue: -1 }
    },
    {
        $limit: 5
    },
    {
        $project: {
        _id: 0,
        Customer_Name: "$_id.Customer_Name",
        cust_email: "$_id.cust_email",
        year: "$_id.year",
        Revenue: 1
        }
    }
    ]);
    ```

#### 8. **Revenue comparison between last month and the same month last year**
    ```mongodb
    db.order.aggregate([
    {
        $match: {
        $expr: {
            $in: [
            {
                $dateToString: {
                format: "%Y-%m",
                date: { $toDate: "$Order_Placed_timestamp" }
                }
            },
            [
                '2024-03', '2023-03'
            ]
            ]
        }
        }
    },
    {
        $lookup: {
        from: "order_details",
        localField: "order_id",
        foreignField: "order_id",
        as: "order_details"
        }
    },
    {
        $unwind: "$order_details"
    },
    {
        $lookup: {
        from: "item",
        localField: "order_details.item_id",
        foreignField: "item_id",
        as: "item"
        }
    },
    {
        $unwind: "$item"
    },
    {
        $group: {
        _id: {
            yearmonth: {
            $dateToString: {
                format: "%Y-%m",
                date: { $toDate: "$Order_Placed_timestamp" } // Convert string date to date object
            }
            }
        },
        revenue: { $sum: { $multiply: ["$item.price", "$order_details.item_qty"] } }
        }
    },
    {
        $sort: { "_id.yearmonth": -1 }
    },
        {
        $limit: 1
    },
    {
        $project: {
        _id: 0,
        metrics: {
            $cond: [
            { $eq: ["$_id.yearmonth", "2024-03"] },
            "Increased",
            "Not Increased"
            ]
        }
        }
    }
    ]);
    ```

#### 9. **Most expensive order among my orders**
    ```mongodb
    db.order.aggregate([
    {
        $match: {
        "cust_email": "rahul.sharma@gmail.com"
        }
    },
    {
        $lookup: {
        from: "order_details",
        localField: "order_id",
        foreignField: "order_id",
        as: "order_details"
        }
    },
    {
        $unwind: "$order_details"
    },
    {
        $lookup: {
        from: "item",
        localField: "order_details.item_id",
        foreignField: "item_id",
        as: "item"
        }
    },
    {
        $unwind: "$item"
    },
    {
        $lookup: {
        from: "customer",
        localField: "cust_email",
        foreignField: "cust_email",
        as: "customer"
        }
    },
    {
        $unwind: "$customer"
    },
    {
        $group: {
        _id: {
            Customer_Name: { $concat: ["$customer.First_name", " ", "$customer.Last_name"] },
            cust_email: "$cust_email",
            order_id: "$order_id"
        },
        expense: { $sum: { $multiply: ["$item.price", "$order_details.item_qty"] } }
        }
    },
    {
        $sort: { "expense": -1 }
    },
    {
        $limit: 1
    }
    ]);
    ```

#### 10. **Total number of orders placed by a customer**
    ```mongodb
    db.order.aggregate([
    {
        $match: {
        cust_email: "rahul.sharma@gmail.com"
        }
    },
    {
        $group: {
        _id: {
            customer_email: "$cust_email"
        },
        no_of_orders_placed: { $sum: 1 }
        }
    },
    {
        $project: {
        _id: 0,
        cust_email: "$_id.customer_email", // Replace with your actual email address
        count: "$no_of_orders_placed"
        }
    }
    ]);
    ```

#### 11. **Total historical spend with the business year to date**
    ```mongodb
    db.order.aggregate([
    {
        $match: {
        $expr: { $eq: [{ $year: { $toDate: "$Order_Placed_timestamp" } }, { $year: new Date() }] }
        }
    },
    {
        $lookup: {
        from: "order_details",
        localField: "order_id",
        foreignField: "order_id",
        as: "order_details"
        }
    },
    {
        $unwind: "$order_details"
    },
    {
        $lookup: {
        from: "item",
        localField: "order_details.item_id",
        foreignField: "item_id",
        as: "item"
        }
    },
    {
        $unwind: "$item"
    },
    {
        $lookup: {
        from: "customer",
        localField: "cust_email",
        foreignField: "cust_email",
        as: "customer"
        }
    },
    {
        $unwind: "$customer"
    },
    {
        $group: {
        _id: {
            cust_email: "$cust_email",
            customer_name: { $concat: ["$customer.First_name", " ", "$customer.Last_name"] }
        },
        total_spend: { $sum: { $multiply: ["$item.price", "$order_details.item_qty"] } }
        }
    },
    {
        $sort: { "_id.cust_email": 1 }
    },
    {
        $project: {
        _id: 0,
        Customer_Name: "$_id.customer_name",
        cust_email: "$_id.cust_email",
        total_spend: 1
        }
    }
    ]);
    ```

#### 12. **Customer expenditure trend chart**
    ```mongodb
    db.order.aggregate([
    {
        $lookup: {
        from: "order_details",
        localField: "order_id",
        foreignField: "order_id",
        as: "order_details"
        }
    },
    {
        $unwind: "$order_details"
    },
    {
        $lookup: {
        from: "item",
        localField: "order_details.item_id",
        foreignField: "item_id",
        as: "item"
        }
    },
    {
        $unwind: "$item"
    },
    {
        $lookup: {
        from: "customer",
        localField: "cust_email",
        foreignField: "cust_email",
        as: "customer"
        }
    },
    {
        $unwind: "$customer"
    },
    {
        $group: {
        _id: {
            customer_name: { $concat: ["$customer.First_name", " ", "$customer.Last_name"] },
            month: { $dateToString: { format: "%Y-%m", date: { $toDate: "$Order_Placed_timestamp" } } }
        },
        expenditure: { $sum: { $multiply: ["$order_details.item_qty", "$item.price"] } }
        }
    },
    {
        $sort: { "_id.customer_name": 1, "_id.month": 1 }
    },
    {
        $project: {
        _id: 0,
        customer_name: "$_id.customer_name",
        month: "$_id.month",
        expenditure: 1
        }
    }
    ]);
    ```

#### 13. **Size of the database**
    ```mongodb
    var stats = db.stats();
    var sizeInMB = stats.dataSize / (1024 * 1024);
    print(sizeInMB + " MB");
    ```

#### 14. **In any of above queries can you change your database design an reduce the cost of the query? (Database Optimization)**
    ```mongodb
    // Create indexes if not already present
    db.order.createIndex({ "cust_email": 1 });
    db.order_details.createIndex({ "order_id": 1 });
    db.item.createIndex({ "item_id": 1 });

    // Execute optimized query
    db.order.aggregate([
    {
        $match: {
        "cust_email": "rahul.sharma@gmail.com"
        }
    },
    {
        $lookup: {
        from: "order_details",
        localField: "order_id",
        foreignField: "order_id",
        as: "order_details"
        }
    },
    {
        $unwind: "$order_details"
    },
    {
        $lookup: {
        from: "item",
        localField: "order_details.item_id",
        foreignField: "item_id",
        as: "item"
        }
    },
    {
        $unwind: "$item"
    },
    {
        $group: {
        _id: "$_id",
        expense: { $sum: { $multiply: ["$item.price", "$order_details.item_qty"] } }
        }
    },
    {
        $sort: { "expense": -1 }
    },
    {
        $limit: 1
    }
    ]);
    ```
