# NodeJS & MySQL Tutorial as Part of SQL Series with Colt Steele
___

___

Example in a file called filename.js

	for(var i = 0; i < 500; i++){
		  console.log("HELLO WORLD!");
	}

Run file

	node filename.js
	
Now to connect node to msql:

	npm install --save mysql
	// Use --save when using package locks
	
Note that specific instructions for using mysql module can be found in the [github repo for the mysql module](https://github.com/mysqljs/mysql).
	
Call an instance of the database connection:

	var mysql = require('mysql');

	var connection = mysql.createConnection({
		  host     : 'localhost',
		  user     : 'root',     // your root username
		  database : 'join_us'   // the name of your db
	});

Now make a simple query:

	var q = 'SELECT CURTIME() as time, CURDATE() as date, NOW() as now';
	connection.query(q, function (error, results, fields) { 
	// q is defined with the var above as an SQL query
	// out put from db is error, results, and fields
	// if error contains anything process stops
	// results[0] calls first item in output 'results' array 
	// w/ time, date, or now as labels
		  if (error) throw error; 
		  console.log(results[0].time);
		  console.log(results[0].date);
		  console.log(results[0].now);
	});
	
Another example pulling data from a table:
	
	var q = 'SELECT COUNT(*) AS total FROM users ';
	connection.query(q, function (error, results, fields) {
		  if (error) throw error;
		  console.log(results[0].total);
	});
	
Inserting hard coded data:

	var q = 'INSERT INTO users (email) VALUES ("rusty_the_dog@gmail.com")';

	connection.query(q, function (error, results, fields) {
		  if (error) throw error;
		  console.log(results);
	});
	
Now insertion through function (using faker module here):

	var person = {
			email: faker.internet.email(),
			created_at: faker.date.past()
	};

	var end_result = connection.query('INSERT INTO users SET ?', person, function(err, result) {
	// Notice 'person' is a SET of key/value pairs
		  if (err) throw err;
		  console.log(result);
	});
	
Here's an example of using faker module to insert lots of fake data into a db:

	var mysql = require('mysql');
	var faker = require('faker');
 
	var connection = mysql.createConnection({
	  host     : 'localhost',
	  user     : 'root',
	  database : 'join_us'
	});
 
	var data = [];
	for(var i = 0; i < 500; i++){
		data.push([
			faker.internet.email(),
			faker.date.past()
		]);
	}
 
	var q = 'INSERT INTO users (email, created_at) VALUES ?';

	connection.query(q, [data], function(err, result) {
	  console.log(err);
	  console.log(result);
	});

	connection.end();
	
### Solutions To 500 Users Exercises (Direct from video)

-- Challenge 1 - Formatting Earliest Date User Joined

	SELECT 
		DATE_FORMAT(MIN(created_at), "%M %D %Y") as earliest_date 
	FROM users;
	
-- Challenge 2 - Find Email of Earliest User

	SELECT * 
	FROM users 
	WHERE created_at = (
		SELECT Min(created_at) 
		FROM users);
		
-- Challenge 3 - Users According to the Month Joined

	SELECT Monthname(created_at) AS month, 
		   Count(*) AS count 
	FROM users 
	GROUP BY month 
	ORDER BY count DESC; 
	
-- Challenge 4 - Count Number of Yahoo Users

	SELECT Count(*) AS yahoo_users 
	FROM users 
	WHERE email LIKE '%@yahoo.com'; 
	
-- Challenge 5 - Count Total Number of Users Per Email Vendor

	SELECT CASE 
			 WHEN email LIKE '%@gmail.com' THEN 'gmail' 
			 WHEN email LIKE '%@yahoo.com' THEN 'yahoo' 
			 WHEN email LIKE '%@hotmail.com' THEN 'hotmail' 
			 ELSE 'other' 
		   end AS provider, 
		   Count(*) AS total_users 
	FROM users 
	GROUP BY provider 
	ORDER BY total_users DESC; 

### Course Work for Express Web App

Includes zipped code: [git repo](https://github.com/filchyboy/join_us_work)

#### Basic Web App

The following goes in the app.js file within the abode code base.

	var express = require('express');
	var app = express();
	// I'm calling this the `HEAD` for later notations

	// The following are routes
	
	app.get("/", function(req, res){
		var joke = "What do you call a dog that does magic tricks? A labracadabrador.";
		res.send(joke);
	});
	
	app.get("/random_num", function(req, res){
		 var num = Math.floor((Math.random() * 10) + 1);
		 res.send("Your lucky number is " + num);
	});
	
	app.get("/count", function(req, res){
		var q = 'SELECT COUNT(*) as count FROM users';
	 	connection.query(q, function (error, results) {
	 		if (error) throw error;
	 		var msg = "We have " + results[0].count + " users";
	 		res.send(msg);
		});
	});
	
Server:
 
	app.listen(8080, function () {
	 	console.log('App listening on port 8080!');
	});
	
Here's a basic `home.ejs` file needed for templating:

	<h1>JOIN US</h1>

	<p class="lead">Enter your email to join <strong><%= count %></strong> 
	others on our waitlist. We are 100% not a cult. </p>

	<form method="POST" action='/register'>
	 <input type="text" class="form" name="email" placeholder="Enter Your Email">
	 <button>Join Now</button>
	</form>
		
To connect this template you would use `render` as in this:
	
	app.get("/", function(req, res){
		res.render("home");
	});
	
But to make the rendering engine work you must call this instance like this:

	app.set("view engine", "ejs");
	// This goes in the `HEAD` section
	// Where ejs is name of the templating framework
	// There are others that all work similarly
	// And "view engine" calls the template home.ejs
	// through "home" in res.render("home");
	
Here's the `POST` route:

	app.post('/register', function(req,res){
		 var person = {email: req.body.email};
		 connection.query('INSERT INTO users SET ?', person, function(err, result) {
		 console.log(err);
		 console.log(result);
		 res.redirect("/");
		 });
	});
	
Use of body-parser npm module allows for forms postings to be retrieved

	app.use(bodyParser.urlencoded({ extended : true}));
	// This goes in the `HEAD` section, it allows the `app`
	// to use the body-parser module. In this instance we would
	// place this code immediately before or after `app.set`
	
	var bodyParser = require("body-parser");
	// This also goes in the `HEAD` section connection to the
	// list of other "required" `var` assignments.
	
In conjunction with the `render` function we want to be able to access `/public` files throughout the web application. Another bit of code in the `app.use` portion of the `HEAD` will make this possible:

	app.use(express.static(__dirname + "/public"));
	// Again this goes in the `HEAD`	
	



## Links
[[Colt Steele  Instruction - 202008311320]]
[[MYSQL Notes - 202008101447]]
[[NodeJS - 202009010844]]

## References
[github repo for mysql module](https://github.com/mysqljs/mysql)
