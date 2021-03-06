#Twitter
We're going to build a simple Twitter app that allows users
to sign up, login, logout, post tweets, have profile pages
and view one page with everyone's tweets.

Refer to the commit history in this repo to see how it was
built step-by-step!

https://github.com/WDI-SEA/simple-twitter-solution/commits/master

## Preparing the Project
Create a directory to contain the project and initialize a
git repo.

```
mkdir twitter
cd twitter
git init
```

Create a database, run init sequelize and install some
standard dependencies for the app. 

```
npm init
npm install --save express ejs express-ejs-layouts body-parser pg-hstore sequelize bcrypt express-session
createdb twitter
sequelize init
```

Edit the config files generated by sequelize. Remove the username
and password properties. Change the name of the database to `twitter`.
Change the dialiect to `postgres`.

Configure the `.gitignore` file to ignore the `node_modules` directory:

```
echo 'node_modules' >> .gitignore
```

Create a simple `index.js` server to make sure everything is set up
good so far.

index.js:

```
var express = require('express');
var app = express();

app.get('/', function(req, res) {
  res.send('welcome to twitter');
});

var port = 3000;
app.listen(port, function() {
  console.log("You're listening to the smooth sounds of port " + port);
});  

```

Navigate to http://localhost:3000 to verify that everything is working
correctly. If everything is good then make your first commit with git.

```
git status
git add -A
git commit -m "simple server set up"
```

## Creating A Layout 

Create a directory to contain our views and another directory to contain
static files, like CSS and JavaScript. Prepare these files:

```
mkdir views
touch views/layout.ejs
touch views/nav.ejs
touch views/index.ejs

mkdir static
touch static/style.css
touch static/main.js
```

Set the view engine of our app to ejs, configure the app middleware to
use `express-ejs-layouts`, and to serve the static directory properly:

```
var express = require('express');
var ejsLayouts = require('express-ejs-layouts');

var app = express();
app.set('view engine', 'ejs');
app.use(ejsLayouts);
app.use(express.static(__dirname + '/static'));
```

Copy and paste these basic contents into each file:

views/layout.ejs:
```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Twitter</title>
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css" integrity="sha512-dTfge/zgoMYpP7QbHy4gWMEGsbsdZeCXz7irItjcC3sPUFtf0kuFbDz/ixG7ArTxmDjLXDmezHubeNikyKGVyQ==" crossorigin="anonymous">
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/font-awesome/4.5.0/css/font-awesome.min.css">
    <link rel="stylesheet" href="/style.css">
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/2.1.4/jquery.min.js"></script>
  </head>
  <body>
    <div class="container">
      <% include nav %>
      <%- body %>
    </div>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/js/bootstrap.min.js" integrity="sha512-K1qjQ+NcF2TYO/eI3M6v8EiNYZfA95pQumfvcVrTHtwQVDG+aHRqLi/ETn2uB+1JqwYqVG3LIvdm9lj6imS/pQ==" crossorigin="anonymous"></script>
    <script src="main.js"></script>
  </body>
</html>  

```

views/nav.ejs
```
<nav class="navbar navbar-default" role="navigation">
	<div class="container-fluid">
		<!-- Brand and toggle get grouped for better mobile display -->
		<div class="navbar-header">
			<button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-ex1-collapse">
				<span class="sr-only">Toggle navigation</span>
				<span class="icon-bar"></span>
				<span class="icon-bar"></span>
				<span class="icon-bar"></span>
			</button>
			<a class="navbar-brand" href="#">Twitter</a>
		</div>

		<!-- Collect the nav links, forms, and other content for toggling -->
		<div class="collapse navbar-collapse navbar-ex1-collapse">
			<ul class="nav navbar-nav">
				<li><a href="/">Home</a></li>
			</ul>
			<ul class="nav navbar-nav navbar-right">
				<form method="GET" action="#" class="navbar-form navbar-left" role="search">
					<div class="form-group">
						<input name="q" type="text" class="form-control" placeholder="Search">
					</div>
					<button type="submit" class="btn btn-default">Submit</button>
				</form>
			</ul>
		</div><!-- /.navbar-collapse -->
	</div>
</nav>
```

views/index.ejs
```
<h1>Welcome to Twitter</h1>
<div class="row">
  <div class="col-xs-12">
    Sign up for an account, log in, or simply browse the firehose.
    And remember, 1 + 1 = <%= 1 + 1 %>
  </div>
</div>
```

static/style.css
```
body {
  background-color: lightblue;
  height: 100%;
}

.container {
  background-color: white;
  min-height: 100%;
  padding-top: 1em;
  padding-bottom: 1em;
}
```

static/main.js
```
console.log('twitter');
```

Run the app and make sure everything is working.
- check CSS: is the background of the page lightblue?
- check JS: is 'twitter' logged to the developer console?
- check ejs: does the index page use ejs to properly compute 1 + 1?
- check nav: does the nav bar appear at the top of the page?

If everything is working then it's time to make our second commit!
Run `git status` to see what file have changed.

```
git status
git add -A
git commit -m "building basic page structure"

```

## Adding Users
Before we start creating posts, let's create a way for users to sign
up for the site and log in to their accounts.

First, let's add username, password, login inputs to our nav bar on the
righthand side. Replace the search bar that's already there.

inside views/nav.ejs:
```
  ...

  <ul class="nav navbar-nav navbar-right">
    <form method="POST" action="/auth/login" class="navbar-form navbar-left" role="search">
      <div class="form-group">
        <input name="username" type="text" class="form-control" placeholder="username">
        <input name="password" type="password" class="form-control" placeholder="password">
      </div>
      <button type="submit" class="btn btn-default">Log In</button>
    </form>
  </ul> 

  ...
```

Second, let's create a sign up link off the home page. Replace the words "sign up"
in index.ejs with a link:

```
<a href="/auth/signup">Sign up</a>
```

Create a template for a sign up page. Be sure to set the form's attributes
`method` and `action` or else the form won't know where to submit it's
information! Use an input with type='submit' to create a button that
will submit the form.

views/signup.ejs:
```
<h1>Sign Up For A New Account</h1>
<form method="PUT" action="/auth/signup">
  <input type="text" name="username">
  <input type="password" name="password">
  <input type="submit" value="Sign Up">
</form>

```

Add a routes to index.js to serve the sign up form when users GET the page,
and to process names and values of the form when data is PUT to the server.

index.js:
```
app.get('/auth/signup', function(req, res) {
  res.render('signup');
});

app.put('/auth/signup', function(req, res) {
  // just return all of the form data to the client for now.
  res.send(req.body);
});
```

