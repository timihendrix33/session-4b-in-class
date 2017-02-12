#Session 4

coming soon

```
.panels {
  min-height:100vh;
  overflow: hidden;
  display: flex; // //
}
```

Each takes an equal width:

```
.panel {
  flex: 1;
```

```
.panel * { 
  border: 1px solid red;
```

```
.panel {
  flex: 1;
  justify-content: center;
  align-items: center;
  display: flex;
  flex-direction: column;
```

```
.panel * {
  border: 1px solid red;
  flex: 1 0 auto;
```

```
.panel * {
  border: 1px solid red;
  flex: 1 0 auto;
  display: flex;
  justify-content: center;
  align-items: center;
```

```
.panel :first-child {
  transform: translateY(-100%);
}
```

```
.panel :last-child {
  transform: translateY(100%);
}
```

```
.panel.open-active :first-child {
  transform: translateY(0);
}
```

```
$0.classList.add('open-active')
```

Open panels take 5 times as much space:

```
.panel.open {
  flex: 5;
```

```
$0.classList.add('open')
```

```
const panels = document.querySelectorAll('.panel');

function toggleOpen(){
  this.classList.toggle('open')
}
    
panels.forEach( (panel) => panel.addEventListener('click', toggleOpen))
```

```
setTimeout(function(){
      
},0.7)
```

```
function openActive(e){
  console.log(e.propertyName)
  // this.classList.toggle('open-active')
}

panels.forEach( (panel) => panel.addEventListener('transitionend', openActive))
```

```
function openActive(e){
  console.log(e.propertyName)
  if (e.propertyName === 'flex-grow')
  // this.classList.toggle('open-active')
}
```


```
    function openActive(e){
      if(e.propertyName.includes('flex')){
        this.classList.toggle('open-active')
      }
    }
```



##CRUD (continued)

cd into the working directory and npm install all dependencies.

Review the connection settings.

Log into mLab.com and find your database and database user.

Replace the db username and password in the connection URL `mongodb://dannyboynyc:dd2345@ds139969.mlab.com:39969/bcl` with your own.

```
MongoClient.connect('mongodb://******', (err, database) => {
  if (err) return console.log(err)
  db = database
  app.listen(port, () => {
    console.log(`Listening on port ${port}!`)
  })
})
```

Run `nodemon app.js`

####Showing entries to users

To show the entries stored in MongoLab to our users:

1. Get entries from MongoLab

2. Use Javascript to display the entries

We can get the entries from MongoLab by using the find method that’s available in the collection method.

```js
app.get('/', (req, res) => {
  var cursor = db.collection('entries').find()
  console.log(cursor)
  res.sendFile(__dirname + '/index.html')
})
```

The find method returns a cursor (a complex Mongo Object that probably doesn’t make sense).

This cursor object contains all entries from our database. It also contains a bunch of other properties and methods that allow us to work with data easily. One method is the toArray method.

The toArray method takes callback function that allows us to do stuff with entries we retrieved from MongoLab. Let’s try doing a console.log() for the results and see what we get:


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

The standard method for outputting content fetched from a database is to use a templating engine. Some popular template engines include Jade/pug, Moustache, Embedded JavaScript (ejs) and Nunjucks. React and Angular offer their own templating engines but to conclude this exersize we will use Embedded JavaScript (ejs). It’s the easiest to implement.

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
<div>
  <% for(var i=0; i<entries.length; i++) { %>
    <h2><%= entries[i].label %></h2>
    <p><%= entries[i].content %></p>
  <% } %>
</div>
```

In EJS, you can write JavaScript within <% and %> tags. You can also output JavaScript as strings with the <%= and %> tags.

```
<% for(var i=0; i<entries.length; i++) { %>
<div class="entry">
	<h2><%= entries[i].label %></h2>
	<p><%= entries[i].content %></p>
</div>
<% } %>
```

```
.entry {
	background: #eee;
}
```

We’re basically going to loop through the entries array and create strings with `entries[i].label` and `entries[i].content`.

The complete index.ejs file so far should be:

```
  <!DOCTYPE html>
  <html lang="en">
  <head>
    <meta charset="UTF-8">
    <title>MY APP</title>
    <style>
      input, textarea {
        display: block;
        margin: 1rem;
        width: 70%;
      }
    </style>
  </head>
  <body>
    <p>Testing 1 2 3</p>

    <form action="/entries" method="POST">
      <input type="text" placeholder="label" name="label">
      <input type="text" placeholder="header" name="header">
      <textarea type="text" placeholder="content" name="content"></textarea>
      <button type="submit">Submit</button>
    </form>

	<% for(var i=0; i<entries.length; i++) { %>
	<div class="entry">
		<h2><%= entries[i].label %></h2>
		<p><%= entries[i].content %></p>
	</div>
	<% } %>

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

Enable use.static in app.js

`npm run boom!`

Get one entry:

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

Edit main.js to remove onload and hashchange events. 

Edit navItems.js to remove the hashes.

<% include partials/header %>
<% include partials/footer %>




###Notes

https://github.com/expressjs/body-parser#bodyparserurlencodedoptions

http://mongoosejs.com/docs/

