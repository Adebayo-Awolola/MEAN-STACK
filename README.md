# MEAN-STACK
Step 1: Install NodeJs
Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine. Node.js is used in this tutorial to set up the Express routes and AngularJS controllers.
Start Your Instance.

![image](https://github.com/user-attachments/assets/2c377a98-8347-4868-8a84-4866a34fceac)

Update ubuntu
sudo apt update
Upgrade ubuntu
sudo apt upgrade

![image](https://github.com/user-attachments/assets/89f9d294-02fc-4623-af35-dd5eb406a929)

Add certificates
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates

curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -

![image](https://github.com/user-attachments/assets/0276c6dd-bf6d-4e54-99b5-0c16316882ed)


Install NodeJS
sudo apt install -y nodejs

![image](https://github.com/user-attachments/assets/279f0a35-2a8d-410d-ae19-19426c6d6410)


Step 2: Install MongoDB
MongoDB stores data in flexible, JSON-like documents. Fields in a database can vary from document to document and data structure can be changed over time. For our example application, we are adding book records to MongoDB that contain book name, isbn number, author, and number of pages.
curl -fsSL https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
![image](https://github.com/user-attachments/assets/19722c71-2780-4b84-b110-0adb13687567)

echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
This single line tells APT everything it needs to know about what the source is and where to find it:
deb: This means that the source entry references a regular Debian architecture. In other cases, this part of the line might read deb-src, which means the source entry represents a Debian distribution’s source code.
[ arch=amd64,arm64 ]: This specifies which architectures the APT data should be downloaded to. In this case, it specifies the amd64 and arm64 architectures.
https://repo.mongodb.org/apt/ubuntu: This is a URI representing the location where the APT data can be found. In this case, the URI points to the HTTPS address where the official MongoDB repository is located.
focal/mongodb-org/4.4: Ubuntu repositories can contain several different releases. This specifies that you only want version 4.4 of the mongodb-org package available for the focal release of Ubuntu (“Focal Fossa” being the code name of Ubuntu 20.04).
multiverse: This part points APT to one of the four main Ubuntu repositories. In this case, it’s pointing to the multiverse repository.
After running this command, update your server’s local package index so APT knows where to find the mongodb-org package:
sudo apt update

Install MongoDB
sudo apt-get install -y mongodb-org
Start The server
sudo systemctl start mongod.service
Verify that the service is up and running
sudo systemctl status MongoDB

![image](https://github.com/user-attachments/assets/253bb271-1884-4f44-aa15-27fded56d78d)


Install [npm](https://www.npmjs.com) - Node package manager.
sudo apt install -y npm
Install 'body-parser package
We need 'body-parser' package to help us process JSON files passed in requests to the server.
sudo npm install body-parser
![image](https://github.com/user-attachments/assets/51e9328f-0b35-43c9-b843-69f72452fffe)


Create a folder named 'Books'
mkdir Books && cd Books
In the Books directory, Initialize npm project
npm init
Press enter to accept default for all steps.
![image](https://github.com/user-attachments/assets/bea5ff98-762a-4f20-99bb-c9a6b9040021)


Add a file to it named server.js
vi server.js
Copy and paste the web server code below into the server.js file.
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const path = require('path');

const app = express();
const PORT = process.env.PORT || 3300;

// MongoDB connection
mongoose.connect('mongodb://localhost:27017/test', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
})
.then(() => console.log('MongoDB connected'))
.catch(err => console.error('MongoDB connection error:', err));

app.use(express.static(path.join(__dirname, 'public')));
app.use(bodyParser.json());

require('./apps/routes')(app);

app.listen(PORT, () => {
  console.log(`Server up: http://localhost:${PORT}`);
});

![image](https://github.com/user-attachments/assets/d36e813b-7ad0-4749-b64e-6c82b0a34cbe)

Step 3: Install Express and set up routes to the server
Express is a minimal and flexible Node.js web application framework that provides features for web and mobile applications. We will use Express in to pass book information to and from our MongoDB database.
We also will use Mongoose package which provides a straight-forward, schema-based solution to model your application data. We will use Mongoose to establish a schema for the database to store data of our book register.
sudo npm install express mongoose
In 'Books' folder, create a folder named apps
mkdir apps && cd apps
Create a file named routes.js
vi routes.js
![image](https://github.com/user-attachments/assets/1834a41b-15e9-4655-a82d-ce9e9f26ee32)

Copy and paste the code below into routes.js
const Book = require('./models/book');
const path = require('path');

module.exports = function(app) {
  app.get('/book', async (req, res) => {
    try {
      const books = await Book.find();
      res.json(books);
    } catch (err) {
      res.status(500).json({ message: 'Error fetching books', error: err.message });
    }
  });

  app.post('/book', async (req, res) => {
    try {
      const book = new Book({
        name: req.body.name,
        isbn: req.body.isbn,
        author: req.body.author,
        pages: req.body.pages
      });
      const savedBook = await book.save();
      res.status(201).json({
        message: 'Successfully added book',
        book: savedBook
      });
    } catch (err) {
      res.status(400).json({ message: 'Error adding book', error: err.message });
    }
  });

  app.delete('/book/:isbn', async (req, res) => {
    try {
      const result = await Book.findOneAndDelete({ isbn: req.params.isbn });
      if (!result) {
        return res.status(404).json({ message: 'Book not found' });
      }
      res.json({
        message: 'Successfully deleted the book',
        book: result
      });
    } catch (err) {
      res.status(500).json({ message: 'Error deleting book', error: err.message });
    }
  });

  app.get('*', (req, res) => {
    res.sendFile(path.join(__dirname, '../public', 'index.html'));
  });
};
In the 'apps' folder, create a folder named models
mkdir models && cd models
Create a file named book.js
vi book.js
Copy and paste the code below into 'book.js'
const mongoose = require('mongoose');

const bookSchema = new mongoose.Schema({
  name: { type: String, required: true },
  isbn: { type: String, required: true, unique: true, index: true },
  author: { type: String, required: true },
  pages: { type: Number, required: true, min: 1 }
}, {
  timestamps: true
});

module.exports = mongoose.model('Book', bookSchema);

![image](https://github.com/user-attachments/assets/3db2ee96-8786-4961-99c9-e2dce046f9f7)

Step 4 - Access the routes with AngularJS
AngularJS provides a web framework for creating dynamic views in your web applications. In this tutorial, we use AngularJS to connect our web page with Express and perform actions on our book register.
Change the directory back to 'Books'
cd ../..


Create a folder named public
mkdir public && cd public


Add a file named script.js
vi script.js


Copy and paste the Code below (controller configuration defined) into the script.js file.
angular.module('myApp', [])
  .controller('myCtrl', function($scope, $http) {
    function fetchBooks() {
      $http.get('/book')
        .then(response => {
          $scope.books = response.data;
        })
        .catch(error => {
          console.error('Error fetching books:', error);
        });
    }

    fetchBooks();

    $scope.del_book = function(book) {
      $http.delete(`/book/${book.isbn}`)
        .then(() => {
          fetchBooks();
        })
        .catch(error => {
          console.error('Error deleting book:', error);
        });
    };

    $scope.add_book = function() {
      const newBook = {
        name: $scope.Name,
        isbn: $scope.Isbn,
        author: $scope.Author,
        pages: $scope.Pages
      };

      $http.post('/book', newBook)
        .then(() => {
          fetchBooks();
          // Clear form fields
          $scope.Name = $scope.Isbn = $scope.Author = $scope.Pages = '';
        })
        .catch(error => {
          console.error('Error adding book:', error);
        });
    };
  });


In 'public' folder, create a file named index.html
vi index.html


Copy and paste the code below into index.html file.
<!DOCTYPE html>
<html ng-app="myApp" ng-controller="myCtrl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Book Management</title>
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.8.2/angular.min.js"></script>
  <script src="script.js"></script>
  <style>
    body { font-family: Arial, sans-serif; margin: 20px; }
    table { border-collapse: collapse; width: 100%; }
    th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
    th { background-color: #f2f2f2; }
    input[type="text"], input[type="number"] { width: 100%; padding: 5px; }
    button { margin-top: 10px; padding: 5px 10px; }
  </style>
</head>
<body>
  <h1>Book Management</h1>
  
  <h2>Add New Book</h2>
  <form ng-submit="add_book()">
    <table>
      <tr>
        <td>Name:</td>
        <td><input type="text" ng-model="Name" required></td>
      </tr>
      <tr>
        <td>ISBN:</td>
        <td><input type="text" ng-model="Isbn" required></td>
      </tr>
      <tr>
        <td>Author:</td>
        <td><input type="text" ng-model="Author" required></td>
      </tr>
      <tr>
        <td>Pages:</td>
        <td><input type="number" ng-model="Pages" required></td>
      </tr>
    </table>
    <button type="submit">Add Book</button>
  </form>

  <h2>Book List</h2>
  <table>
    <thead>
      <tr>
        <th>Name</th>
        <th>ISBN</th>
        <th>Author</th>
        <th>Pages</th>
        <th>Action</th>
      </tr>
    </thead>
    <tbody>
      <tr ng-repeat="book in books">
        <td>{{book.name}}</td>
        <td>{{book.isbn}}</td>
        <td>{{book.author}}</td>
        <td>{{book.pages}}</td>
        <td><button ng-click="del_book(book)">Delete</button></td>
      </tr>
    </tbody>
  </table>
</body>
</html>

![image](https://github.com/user-attachments/assets/326ed9a2-5828-4f55-bba7-a4c2bee88e7c)


Change the directory back up to 'Books'
cd ..
Start the server by running this command:
node server.js

The server is now up and running, we can connect it via port 3300. You can launch a separate Putty or SSH console to test what curl command returns locally.
curl -s http://localhost:3300
It shall return an HTML page, it is hardly readable in the CLI, but we can also try and access it from the Internet.
For thiS, you must open TCP port 3300 in your AWS Web Console for your EC2 Instance.

![image](https://github.com/user-attachments/assets/990c44d5-013f-423b-8429-264d78f396a3)

Now you can access our Book Register web application from the Internet with a browser using a Public IP address or Public DNS name.
Quick reminder on how to get your server's Public IP and public DNS name:
You can find it in your AWS web console in EC2 details
Run curl -s http://169.254.169.254/latest/meta-data/public-ipv4 for Public IP address or curl -s http://169.254.169.254/latest/meta-data/public-hostname for Public DNS name.
This is how your Web Book Register Application will look like in browser:

![image](https://github.com/user-attachments/assets/0315fb3e-6984-4651-afeb-5f7d256b9314)





