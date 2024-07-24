# MEAN Stack Deployment to Ubuntu in AWS
<<<<<<< HEAD


=======
>>>>>>> b4a0810fb973e144a3afe16d5636a73e7a44f8b1

## Install Node.Js

1. Update ubuntu
~~~
sudo apt update
~~~
2. Upgrade ubuntu
~~~
sudo apt upgrade
~~~
3. Add certificates
~~~
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates

curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
~~~
4. Install NodeJS
~~~
sudo apt install -y nodejs
~~~

## Install MongoDB

~~~
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
~~~
~~~
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
~~~
1. Install MongoDB

~~~
sudo apt install -y mongodb
~~~
2. Start The server
~~~
sudo service mongodb start
~~~
3. Verify that the service is up and running
~~~
sudo systemctl status mongodb
~~~
4. Install [npm](https://www.npmjs.com) - Node package manager.
~~~
sudo apt install -y npm
~~~
5. Install 'body-parser package

> We need 'body-parser' package to help us process JSON files passed in requests to the server.
~~~
sudo npm install body-parser
~~~
6. Create a folder named 'Books'
~~~
mkdir Books && cd Books
~~~
7. In the Books directory, Initialize npm project
~~~
npm init
~~~
8. Add a file to it named server.js
~~~
vi server.js
~~~
9. Copy and paste the web server code below into the server.js file.
~~~
var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});
~~~
## Install Express and set up routes to the server

1. We will use express mongoose

~~~
sudo npm install express mongoose
~~~
2. In the 'Books' folder, create a folder named apps
~~~
mkdir apps && cd apps
~~~
3. Create a file named routes.js
~~~
vi routes.js
~~~
4. Copy and paste the code below into routes.js
~~~
var Book = require('./models/book');
module.exports = function(app) {
  app.get('/book', function(req, res) {
    Book.find({}, function(err, result) {
      if ( err ) throw err;
      res.json(result);
    });
  }); 
  app.post('/book', function(req, res) {
    var book = new Book( {
      name:req.body.name,
      isbn:req.body.isbn,
      author:req.body.author,
      pages:req.body.pages
    });
    book.save(function(err, result) {
      if ( err ) throw err;
      res.json( {
        message:"Successfully added book",
        book:result
      });
    });
  });
  app.delete("/book/:isbn", function(req, res) {
    Book.findOneAndRemove(req.query, function(err, result) {
      if ( err ) throw err;
      res.json( {
        message: "Successfully deleted the book",
        book: result
      });
    });
  });
  var path = require('path');
  app.get('*', function(req, res) {
    res.sendfile(path.join(__dirname + '/public', 'index.html'));
  });
};
~~~
5. In the 'apps' folder, create a folder named models
~~~
mkdir models && cd models
~~~
6. Create a file named book.js
~~~
vi book.js
~~~
7. Copy and paste the code below into 'book.js'
~~~
var mongoose = require('mongoose');
var dbHost = 'mongodb://localhost:27017/test';
mongoose.connect(dbHost);
mongoose.connection;
mongoose.set('debug', true);
var bookSchema = mongoose.Schema( {
  name: String,
  isbn: {type: String, index: true},
  author: String,
  pages: Number
});
var Book = mongoose.model('Book', bookSchema);
module.exports = mongoose.model('Book', bookSchema);
~~~
##  Access the routes with AngularJS
1. Change the directory back to 'Books'
~~~
cd ../..
~~~
2. Create a folder named public
~~~
mkdir public && cd public
~~~
3. Add a file named script.js
~~~
vi script.js
~~~
4. Copy and paste the Code below (controller configuration defined) into the script.js file.
~~~
var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope, $http) {
  $http( {
    method: 'GET',
    url: '/book'
  }).then(function successCallback(response) {
    $scope.books = response.data;
  }, function errorCallback(response) {
    console.log('Error: ' + response);
  });
  $scope.del_book = function(book) {
    $http( {
      method: 'DELETE',
      url: '/book/:isbn',
      params: {'isbn': book.isbn}
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
  $scope.add_book = function() {
    var body = '{ "name": "' + $scope.Name + 
    '", "isbn": "' + $scope.Isbn +
    '", "author": "' + $scope.Author + 
    '", "pages": "' + $scope.Pages + '" }';
    $http({
      method: 'POST',
      url: '/book',
      data: body
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
});
~~~
5. In 'public' folder, create a file named index.html
~~~
vi index.html
~~~
6. Cpoy and paste the code below into index.html file.
~~~
<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div>
      <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
      </table>
      <button ng-click="add_book()">Add</button>
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>

        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>

          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>
~~~
7. Change the directory back up to 'Books'
~~~
cd ..
~~~
8. Start the server by running this command:
~~~
node server.js
~~~
![alt text](<Screenshot 2024-05-30 151749.png>)
9. The server is now up and running, we can connect it via port 3300. You can launch a separate Putty or SSH console to test what curl command returns locally.
~~~
curl -s http://localhost:3300
~~~
![alt text](<open port 3300.png>)
10. Add port 3300 under port 80 in the security group in AWS as done in the previous project.
11. run {http://16.16.241.119:3300/}
![alt text](<Screenshot 2024-05-30 170144.png>)
