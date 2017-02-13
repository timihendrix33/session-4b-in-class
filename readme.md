#Session 4b

##CRUD (continued)

cd into the working directory and npm install all dependencies.

Review the connection settings.

Log into mLab.com and find your database and database user.

Add a variable with the db username and password in the connection URL 

`const mongoUrl = 'mongodb://dannyboynyc:dd2345@ds139969.mlab.com:39969/bcl'` 

(Replace it with your own.)

```
MongoClient.connect(mongoUrl, (err, database) => {...}
```

Run `nodemon app.js`

####Showing entries to users

To show the entries stored in MongoLab:

1. Get entries from MongoLab

2. Use Javascript to display the entries

We can get the entries from MongoLab by using the find method that’s available in the [collection method](https://docs.mongodb.com/manual/reference/method/js-collection/).

```js
app.get('/', (req, res) => {
  var cursor = db.collection('entries').find()
  console.log(cursor)
  res.sendFile(__dirname + '/index.html')
})
```

The find method returns a cursor (a complex Mongo Object that probably doesn’t make much sense).

This cursor object contains all entries from our database. It also contains a [bunch of other properties and methods](https://docs.mongodb.com/manual/reference/method/js-cursor/) that allow us to work with data easily. One method is the toArray method.

The `toArray` method takes callback function that allows us to perform actions on the entries we retrieved from MongoLab. Try doing a console.log() for the results and see what we get:


```js
app.get('/', (req, res) => {
  db.collection('entries').find().toArray((err, results) => {
    console.log(results)
    res.sendFile(__dirname + '/index.html')
  })
})
```

The array of entries should appear in the terminal. 

####Generate HTML to contain the entries

The standard method for outputting content fetched from a database is to use a templating engine. Some popular template engines include Jade/pug, Moustache, Embedded JavaScript (ejs) and Nunjucks. React and Angular offer their own templating engines - but to conclude this exercise we will use Embedded JavaScript (ejs). It’s the easiest to implement.

We use ejs by first installing it and then setting the view engine in Express to ejs.

`$ npm install ejs --save`

and in app.js:

`app.set('view engine', 'ejs')`

Let’s first create an index.ejs file within a views folder so we can start populating data.

```
mkdir views
touch views/index.ejs
```

Now, copy the contents of index.html into index.ejs and add.

```
<% for(var i=0; i<entries.length; i++) { %>
<div class="entry">
  <h2><%= entries[i].label %></h2>
  <p><%= entries[i].content %></p>
</div>
<% } %>
```

In EJS, you can write JavaScript within <% and %> tags. You can also output JavaScript as strings with the <%= and %> tags.

```
.entry {
	background: #eee;
  padding: 0.5rem;
}
```

We’re basically going to loop through the entries array and create strings with `entries[i].label` and `entries[i].content`.

The complete index.ejs file so far should be:

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Test</title>
  <style>
    input, textarea {
      display: block;
      margin: 1rem;
      width: 70%;
    }
    .entry {
      background: #eee;
    }
  </style>
</head>
<body>
  <% for(var i=0; i<entries.length; i++) { %>
  <div class="entry">
    <h2><%= entries[i].label %></h2>
    <p><%= entries[i].content %></p>
  </div>
  <% } %>

  <form action="/entries" method="POST">
    <input type="text" placeholder="label" name="label">
    <input type="text" placeholder="header" name="header">
    <textarea type="text" placeholder="content" name="content"></textarea>
    <button type="submit">Submit</button>
  </form>

</body>
</html>
```

Finally, we have to render this index.ejs file when handling the GET request. Here, we’re setting the results (an array) as the entries array we used in index.ejs above.


```js
app.get('/', (req, res) => {
  db.collection('entries').find().toArray((err, result) => {
    if (err) return console.log(err)
    // renders index.ejs
    res.render('index.ejs', {entries: result})
  })
})
```

Now, refresh your browser and you should be able to see all entries.

##Integration with the old site

We need to 

1. edit our npm scripts to integrate nodemon

2. move the old index.html into index.ejs 

3. re-enable app.use static. 

Edit package.json to proxy the browser sync to our express port number and add nodemon to our list of currently running scripts.

```
 "scripts": {
    "watch-node-sass": "node-sass --watch scss/styles.scss --output public/css/  --source-map true",
    "start": "browser-sync start --proxy 'localhost:9000' --browser \"google chrome\"  --files 'public'",
    "boom!": "concurrently \"nodemon app.js\" \"npm run start\" \"npm run watch-node-sass\" "
  },
```

Migrate index.html into index.ejs something like the below:

```
<!DOCTYPE html>
<html lang="en">
<head>
	<head>
		<meta charset="UTF-8">
		<title>EJS Barclays Live</title>
		<link rel="stylesheet" href="/css/styles.css">
		<meta name="viewport" content="width=device-width, initial-scale=1.0">

		<style>
			input, textarea {
				display: block;
				margin: 1rem;
				width: 70%;
			}
			.entry {
				background: #eee;
			}
		</style>
	</head>
	<body>
		<header>
			<h1>Not on the app store!</h1>
		</header>

		<nav id="main">
			<a class="logo" href="#0"><img src="/img/logo.svg" /></a>
			<ul id="nav-links"></ul>
		</nav>

		<div class="site-wrap">
			<% for(var i=0; i<entries.length; i++) { %>
			<div class="entry">
				<h2><%= entries[i].label %></h2>
				<p><%= entries[i].content %></p>
			</div>
			<% } %>

			<form action="/entries" method="POST">
				<input type="text" placeholder="label" name="label">
				<input type="text" placeholder="header" name="header">
				<textarea type="text" placeholder="content" name="content"></textarea>
				<button type="submit">Submit</button>
			</form>
		</div>
		<script src="/js/navitems.js"></script>
		<script src="/js/main.js"></script>
	</body>
</html>
```

Change the name of index.html to something else.

Enable use.static in app.js, rename /public/index.html (otherwise express will serve it instead of index.ejs), halt nodemon, and run:

`npm run boom!`

Get one entry using parameters:

```
app.get('/:name?', (req, res) => {
  let name = req.params.name
  db.collection('entries').find({
    "label": name
  }).toArray((err, result) => {
    res.render('index.ejs', {entries: result})
  })
})
```

Try: `http://localhost:3000/watchlist`

Edit main.js to remove onload and hashchange events and main.js to remove the hashes.


##Angular as a Templating Engine

Set up `angular` folder with npm install and boom!

`<script src="https://code.angularjs.org/1.5.8/angular.js"></script>`

`<html lang="en"  ng-app="myApp">`

In navItems:

`var app = angular.module('myApp', []);`

`<body ng-controller="NavController">`

```
app.controller("NavController", function( $scope ) {
  $scope.navItems = [
```

Comment out the navItems build script in main.js

```
const markup =
//   `<ul>
//     ${navItems.map(listItem => `<li><a href="${listItem.link}">${listItem.label}</a></li>`).join('')}
//   </ul>`;
// navLinks.innerHTML = markup;
```

Use angular to build it out again:

```
<nav id="main">
  <a class="logo" href="#0"><img src="/img/logo.svg" /></a>
  <ul id="nav-links">
    <li ng-repeat="navItem in navItems">
      <a href={{navItem.link}}>{{navItem.label}}</a>
    </li>
  </ul>
</nav>
```

Build out the content:

```
<div ng-repeat="navItem in navItems">
  <h2>{{ navItem.label }}</h2>
  <h3>{{ navItem.header }}</h3>
  <div ng-bind-html="navItem.content"></div>
</div>
```

Load sanitize:

`<script src="https://code.angularjs.org/1.5.8/angular-sanitize.min.js"></script>`

Use injection to make it available:

`var app = angular.module('myApp', ['ngSanitize']);`

###Details, details

Simple Angular directives:

1. ng-app − This directive starts an AngularJS Application.
2. ng-init − This directive initializes application data.
3. ng-model − This directive defines the model that is variable to be used in AngularJS.
4. ng-repeat − This directive repeats html elements for each item in a collection.

```
<div class="site-wrap"  ng-init="messageText = 'Hello World!'">

<input ng-model="messageText" size="30"/><br/>
Everybody shout "{{ messageText | uppercase }}"
```

Alernates 1 - using an object

`ng-init="greeting = { greeter: 'Daniel' , message: 'Hello World' }"`

```
<input type="text" ng-model="greeting.greeter" size="30"/>
<input type="text" ng-model="greeting.message" size="30"/>
{{greeting.greeter }} says "{{ greeting.message }}"
```

Alternates 2 - using ng-repeat with an array

```
<div class="site-wrap" ng-init="portfolios = ['Call of Booty', 'The Sack of the Innocents', 'Pipe and First Mate']" >

<ul>
  <li ng-repeat="portfolio in portfolios">
    {{ portfolio }}
  </li>
</ul>
```

Alernate 3 - filtering and ordering on an array of objects

```
<div class="site-wrap" ng-init="portfolios = [
{ name: 'Call of Booty', date: '2013-09-01' },
{ name: 'The Sack of the Innocents', date: '2014-04-15' },
{ name: 'Pipe and First Mate', date: '2012-10-01' } ]">

<p>Filter list: <input ng-model="searchFor" size="30"/></p>

<ul>
  <li ng-repeat="portfolio in portfolios | filter:searchFor | orderBy:'date' ">
  {{ portfolio.name  }}</li>
</ul>
```

```

```





###Notes

https://github.com/expressjs/body-parser#bodyparserurlencodedoptions

http://mongoosejs.com/docs/

