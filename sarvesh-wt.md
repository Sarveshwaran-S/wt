# Sarvesh WT – Code Notes

Collection of Web Technology programs: AngularJS frontends, Express/MongoDB backends, and XML/XSL.

## 1. Student Management (Frontend only – AngularJS)

```html
<!DOCTYPE html>
<html ng-app="myApp">
<head>
    <title>Student Management</title>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.8.2/angular.min.js"></script>
</head>

<body ng-controller="myCtrl">

    <h2>Student Management System</h2>

    <input type="number" ng-model="student.id" placeholder="Student ID">
    <input type="text" ng-model="student.name" placeholder="Student Name">
    <input type="text" ng-model="student.dept" placeholder="Department">

    <button ng-click="add()">Add</button>
    <button ng-click="update()">Update</button>

    <hr>

    <table border="1" cellpadding="10">
        <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Department</th>
            <th>Actions</th>
        </tr>

        <tr ng-repeat="s in students">
            <td>{{s.id}}</td>
            <td>{{s.name}}</td>
            <td>{{s.dept}}</td>
            <td>
                <button ng-click="edit($index)">Edit</button>
                <button ng-click="remove($index)">Delete</button>
            </td>
        </tr>
    </table>

    <script>
        var app = angular.module("myApp", []);

        // SERVICE
        app.service("StudentService", function() {
            this.students = [];

            // CREATE
            this.add = function(s) {
                this.students.push({
                    id: s.id,
                    name: s.name,
                    dept: s.dept
                });
            };

            // READ
            this.getAll = function() {
                return this.students;
            };

            // UPDATE
            this.update = function(i, s) {
                this.students[i].id = s.id;
                this.students[i].name = s.name;
                this.students[i].dept = s.dept;
            };

            // DELETE
            this.remove = function(i) {
                this.students.splice(i, 1);
            };
        });

        // CONTROLLER
        app.controller("myCtrl", function($scope, StudentService) {
            $scope.students = StudentService.getAll();
            $scope.student = {};
            $scope.index = -1;

            $scope.add = function() {
                if ($scope.student.id == null ||
                    !$scope.student.name ||
                    !$scope.student.dept) {
                    alert("Enter all details");
                    return;
                }

                StudentService.add($scope.student);
                $scope.student = {};
            };

            $scope.edit = function(i) {
                $scope.index = i;

                $scope.student = {
                    id: $scope.students[i].id,
                    name: $scope.students[i].name,
                    dept: $scope.students[i].dept
                };
            };

            $scope.update = function() {
                if ($scope.index == -1) {
                    alert("Select a student to edit");
                    return;
                }

                StudentService.update($scope.index, $scope.student);
                $scope.student = {};
                $scope.index = -1;
            };

            $scope.remove = function(i) {
                StudentService.remove(i);
                $scope.student = {};
                $scope.index = -1;
            };
        });
    </script>

</body>
</html>
```

## 2. MongoDB Frontend – Supermarket Products (AngularJS)

```html
<!DOCTYPE html>
<html ng-app="myApp">
<head>
    <title>Supermarket Product Details</title>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.8.2/angular.min.js"></script>
</head>

<body ng-controller="ProductController">

    <h2>Supermarket Product Details</h2>

    ID:
    <input type="number" ng-model="id"><br><br>

    Name:
    <input type="text" ng-model="name"><br><br>

    Price:
    <input type="number" ng-model="price"><br><br>

    Quantity:
    <input type="number" ng-model="quantity"><br><br>

    <button ng-click="addProduct()" ng-show="!editMode">Add</button>
    <button ng-click="updateProduct()" ng-show="editMode">Update</button>
    <button ng-click="clearForm()" ng-show="editMode">Cancel</button>

    <p>{{message}}</p>

    <h3>Product List</h3>

    <button ng-click="loadProducts()">Refresh List</button>

    <table border="1" cellpadding="10">
        <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Price</th>
            <th>Quantity</th>
            <th>Actions</th>
        </tr>

        <tr ng-repeat="p in products">
            <td>{{p.id}}</td>
            <td>{{p.name}}</td>
            <td>{{p.price}}</td>
            <td>{{p.quantity}}</td>
            <td>
                <button ng-click="editProduct($index)">Edit</button>
                <button ng-click="deleteProduct($index)">Delete</button>
            </td>
        </tr>
    </table>

    <script>
        var app = angular.module("myApp", []);

        // SERVICE
        app.service("ProductService", function($http) {

            var api = "http://localhost:3000/products";

            this.getProducts = function() {
                return $http.get(api);
            };

            this.addProduct = function(product) {
                return $http.post(api, product);
            };

            this.updateProduct = function(id, product) {
                return $http.put(api + "/" + id, product);
            };

            this.deleteProduct = function(id) {
                return $http.delete(api + "/" + id);
            };
        });

        // CONTROLLER
        app.controller("ProductController",
            function($scope, ProductService) {

            $scope.products = [];
            $scope.editMode = false;
            $scope.editId = null;
            $scope.message = "";

            // READ
            $scope.loadProducts = function() {
                ProductService.getProducts().then(function(response) {
                    $scope.products = response.data;
                }).catch(function(error) {
                    $scope.message = "Load error: " +
                        (error.data && error.data.error
                            ? error.data.error
                            : error.statusText || "Check backend connection");
                });
            };

            // CREATE
            $scope.addProduct = function() {
                if ($scope.id == null || $scope.name == null ||
                    $scope.name.trim() === "" || $scope.price == null ||
                    $scope.quantity == null) {
                    $scope.message = "Please enter all product details.";
                    return;
                }

                var product = {
                    id: Number($scope.id),
                    name: $scope.name.trim(),
                    price: Number($scope.price),
                    quantity: Number($scope.quantity)
                };

                ProductService.addProduct(product).then(function() {
                    $scope.message = "Product added successfully!";
                    $scope.clearForm();
                    $scope.loadProducts();
                }).catch(function(error) {
                    $scope.message = "Add error: " +
                        (error.data && error.data.error
                            ? error.data.error
                            : error.statusText || "Check backend connection");
                });
            };

            // EDIT
            $scope.editProduct = function(index) {
                var p = $scope.products[index];

                $scope.id = p.id;
                $scope.name = p.name;
                $scope.price = p.price;
                $scope.quantity = p.quantity;

                $scope.editId = p._id;
                $scope.editMode = true;
                $scope.message = "";
            };

            // UPDATE
            $scope.updateProduct = function() {
                if ($scope.id == null || !$scope.name ||
                    $scope.name.trim() === "" || $scope.price == null ||
                    $scope.quantity == null) {
                    $scope.message = "Please enter all product details.";
                    return;
                }

                var product = {
                    id: Number($scope.id),
                    name: $scope.name.trim(),
                    price: Number($scope.price),
                    quantity: Number($scope.quantity)
                };

                ProductService.updateProduct($scope.editId, product)
                    .then(function() {
                        $scope.message = "Product updated successfully!";
                        $scope.clearForm();
                        $scope.loadProducts();
                    }).catch(function(error) {
                        $scope.message = "Update error: " +
                            (error.data && error.data.error
                                ? error.data.error
                                : error.statusText || "Check backend connection");
                    });
            };

            // DELETE
            $scope.deleteProduct = function(index) {
                var p = $scope.products[index];

                if (confirm("Are you sure you want to delete this product?")) {
                    ProductService.deleteProduct(p._id)
                        .then(function() {
                            $scope.message = "Product deleted successfully!";
                            $scope.loadProducts();
                        }).catch(function(error) {
                            $scope.message = "Delete error: " +
                                (error.data && error.data.error
                                    ? error.data.error
                                    : error.statusText || "Check backend connection");
                        });
                }
            };

            // CLEAR INPUTS
            $scope.clearForm = function() {
                $scope.id = null;
                $scope.name = "";
                $scope.price = null;
                $scope.quantity = null;

                $scope.editMode = false;
                $scope.editId = null;
            };

            // LOAD PRODUCTS ON PAGE OPEN
            $scope.loadProducts();
        });
    </script>

</body>
</html>
```

## 3. MongoDB Backend (Express + Mongoose)

```javascript
const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");
require("dotenv").config();

const app = express();

app.use(cors());
app.use(express.json());

// Product schema
const productSchema = new mongoose.Schema({
    id: {
        type: Number,
        required: true
    },
    name: {
        type: String,
        required: true
    },
    price: {
        type: Number,
        required: true
    },
    quantity: {
        type: Number,
        required: true
    }
});

const Product = mongoose.model("Product", productSchema);

// READ: Get all products
app.get("/products", async (req, res) => {
    try {
        const products = await Product.find();
        res.json(products);
    } catch (err) {
        console.error("READ ERROR:", err.message);
        res.status(500).json({ error: err.message });
    }
});

// CREATE: Add a product
app.post("/products", async (req, res) => {
    try {
        console.log("Received product:", req.body);

        const product = await Product.create(req.body);

        res.status(201).json(product);
    } catch (err) {
        console.error("ADD PRODUCT ERROR:", err.message);
        res.status(400).json({ error: err.message });
    }
});

// UPDATE: Update using MongoDB document ID
app.put("/products/:id", async (req, res) => {
    try {
        const product = await Product.findByIdAndUpdate(
            req.params.id,
            req.body,
            { new: true, runValidators: true }
        );

        if (!product) {
            return res.status(404).json({ error: "Product not found" });
        }

        res.json(product);
    } catch (err) {
        console.error("UPDATE ERROR:", err.message);
        res.status(400).json({ error: err.message });
    }
});

// DELETE: Delete using MongoDB document ID
app.delete("/products/:id", async (req, res) => {
    try {
        const product = await Product.findByIdAndDelete(req.params.id);

        if (!product) {
            return res.status(404).json({ error: "Product not found" });
        }

        res.json({ message: "Product deleted successfully" });
    } catch (err) {
        console.error("DELETE ERROR:", err.message);
        res.status(400).json({ error: err.message });
    }
});

// Start server after MongoDB connects
async function startServer() {
    try {
        await mongoose.connect(process.env.MONGO_URI);
        console.log("MongoDB connected successfully");

        const port = process.env.PORT || 3000;

        app.listen(port, () => {
            console.log("Backend running at http://localhost:" + port);
        });
    } catch (err) {
        console.error("MongoDB connection failed:", err.message);
    }
}

startServer();
```

## 4. Alternative Backend – Students (Express + Mongoose)

```javascript
const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");
require("dotenv").config();

const app = express();

app.use(cors());
app.use(express.json());

// Connect to MongoDB
mongoose.connect(process.env.MONGO_URI)
    .then(() => console.log("MongoDB connected"))
    .catch(err => console.log("MongoDB error:", err.message));

// Student schema
const Student = mongoose.model("Student", new mongoose.Schema({
    rollNo: { type: String, required: true },
    name: { type: String, required: true },
    department: String,
    email: String
}));

// View all students
app.get("/students", async (req, res) => {
    try {
        res.json(await Student.find());
    } catch (err) {
        res.status(500).json({ error: err.message });
    }
});

// Add a student
app.post("/students", async (req, res) => {
    try {
        const student = await Student.create(req.body);
        res.status(201).json(student);
    } catch (err) {
        res.status(400).json({ error: err.message });
    }
});

// Update a student
app.put("/students/:id", async (req, res) => {
    try {
        const student = await Student.findByIdAndUpdate(
            req.params.id,
            req.body,
            { new: true, runValidators: true }
        );

        if (!student) {
            return res.status(404).json({ message: "Student not found" });
        }

        res.json(student);
    } catch (err) {
        res.status(400).json({ error: err.message });
    }
});

// Delete a student
app.delete("/students/:id", async (req, res) => {
    try {
        const student = await Student.findByIdAndDelete(req.params.id);

        if (!student) {
            return res.status(404).json({ message: "Student not found" });
        }

        res.json({ message: "Student deleted" });
    } catch (err) {
        res.status(400).json({ error: err.message });
    }
});

// Start server
app.listen(process.env.PORT || 3000, () => {
    console.log("Server running on port 3000");
});
```

## 5. Simple Backend using Postman (In-memory)

```javascript
const express = require("express");
const cors = require("cors");

const app = express();

app.use(cors());
app.use(express.json());

// Temporary student storage
let students = [];
let nextId = 1;

// View all students
app.get("/students", (req, res) => {
    res.json(students);
});

// Add a student
app.post("/students", (req, res) => {
    const student = {
        id: nextId++,
        rollNo: req.body.rollNo,
        name: req.body.name,
        department: req.body.department,
        email: req.body.email
    };

    students.push(student);
    res.status(201).json(student);
});

// View one student
app.get("/students/:id", (req, res) => {
    const student = students.find(
        s => s.id === Number(req.params.id)
    );

    if (!student) {
        return res.status(404).json({
            message: "Student not found"
        });
    }

    res.json(student);
});

// Update a student
app.put("/students/:id", (req, res) => {
    const student = students.find(
        s => s.id === Number(req.params.id)
    );

    if (!student) {
        return res.status(404).json({
            message: "Student not found"
        });
    }

    student.rollNo = req.body.rollNo;
    student.name = req.body.name;
    student.department = req.body.department;
    student.email = req.body.email;

    res.json(student);
});

// Delete a student
app.delete("/students/:id", (req, res) => {
    const index = students.findIndex(
        s => s.id === Number(req.params.id)
    );

    if (index === -1) {
        return res.status(404).json({
            message: "Student not found"
        });
    }

    students.splice(index, 1);

    res.json({ message: "Student deleted" });
});

// Start server
app.listen(3000, () => {
    console.log("Server running on port 3000");
});
```

## 6. students.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="students.xsl"?>

<students>
    <student>
        <name>Arun</name>
        <age>20</age>
        <department>IT</department>
    </student>
    <student>
        <name>Priya</name>
        <age>21</age>
        <department>CSE</department>
    </student>
    <student>
        <name>Rahul</name>
        <age>19</age>
        <department>ECE</department>
    </student>
</students>
```

## 7. students.xsl

```xml
<?xml version="1.0" encoding="UTF-8"?>

<xsl:stylesheet version="1.0"
    xmlns:xsl="http://www.w3.org/1999/XSL/Transform">

    <xsl:template match="/">
        <html>
            <head>
                <title>Student Details</title>
            </head>
            <body>
                <h2>Student Details</h2>

                <table border="1" cellpadding="10">
                    <tr>
                        <th>Name</th>
                        <th>Age</th>
                        <th>Department</th>
                    </tr>

                    <xsl:for-each select="students/student">
                        <tr>
                            <td>
                                <xsl:value-of select="name"/>
                            </td>
                            <td>
                                <xsl:value-of select="age"/>
                            </td>
                            <td>
                                <xsl:value-of select="department"/>
                            </td>
                        </tr>
                    </xsl:for-each>

                </table>
            </body>
        </html>
    </xsl:template>

</xsl:stylesheet>
```


// test.py selenium


from selenium import webdriver
from selenium.webdriver.common.by import By
import time

driver = webdriver.Chrome()

driver.get("http://127.0.0.1:5500/login.html")


driver.find_element(By.ID, "username").send_keys("admin")

driver.find_element(By.ID, "password").send_keys("12345")

driver.find_element(By.ID, "loginBtn").click()

time.sleep(1)

result = driver.find_element(By.ID, "message").text
print(result)


assert result == "Login Successful"

print("Test Passed!")

driver.quit()


// login.html

```html
<html>
<body>
    <h2>Login Page</h2>

    <input type="text" id="username" placeholder="Username"><br><br>

    <input type="password" id="password" placeholder="Password"><br><br>

    <button id="loginBtn" onclick="login()">Login</button>

    <p id="message"></p>

    <script>
        function login() {
            var u = document.getElementById("username").value;
            var p = document.getElementById("password").value;

            if (u == "admin" && p == "12345") {
                document.getElementById("message").innerHTML = "Login Successful";
            } else {
                document.getElementById("message").innerHTML = "Login Failed";
            }
        }
    </script>
</body>
</html>

```


//normal postman


const express = require('express')
const app = express()

app.use(express.json())

let student = [
    {id: 1, name: "vignesh", mark: 90},
    {id: 2, name: "sarvesh", mark: 100}
];

app.get('/', (req, res) => {
    res.send("Welcome to student details");
});

app.get('/students', (req, res) => {
    res.send(student);
});

app.post('/students', (req, res) => {
    student.push(req.body);
    res.send("student added");
});

app.put('/student/:id', (req, res) => {
    let s = student.find(x => x.id == req.params.id);
    if (s) Object.assign(s, req.body);
    res.send("student updated");
});

app.delete('/student/:id', (req, res) => {
    student = student.filter(x => x.id != req.params.id);
    res.send("Student deleted");
});

app.listen(8080, () => {
    console.log("Server running on port 8080");
});
