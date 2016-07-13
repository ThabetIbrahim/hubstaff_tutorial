# Hubstaff Integration With PHP-LIBRARY

This tutorial will go over how to integrate the Hubstaff PHP API into your own PHP application. 
The Hubstaff API allows you to easily link a User to their Hubstaff account and retrieve useful information such as custom team reports, 
project and activity details, screenshots, and much more. 
The [Github repository](https://github.com/Smulligan85/Hubstaff-Gem-Tutorial) for this tutorial includes two branches. 
The master branch is the starter application that this tutorial will walkthrough and the final-tut branch is the complete application. 
The starter applicaiton is a basic PHP app that includes a User model and some basic authentication. 
This tutorial will go over first linking a User to their Hubstaff account and then go over how to retrieve data. 
We will be retrieving data via two endpoints provided by the Hubstaff API, custom team reports and screenshots. 
Before we begin you will need to set up a [Hubstaff account](https://hubstaff.com/). 
I also recommend creating some data so that your application will be able to view data, specifically 
create a organization, project, notes, and a few screenshots. After you have created some data we need to go to the 
[Hubstaff developer page](https://developer.hubstaff.com/) to create our application and receive our App Token. 
Click My Apps and create a new application. Once we have our App Token we’re ready to dive in.

Clone the master branch down and open the application in your editor of choice. 
First we will edit the hubstaff Config.php file and add the App Token we generated from the Hubstaff developer page.

```PHP
App_Token=“<App Token Hubstaff Provided>”
```
After editing our config.php file we'll be initializing hubstaff api into our project by calling the following

```php
include("hubstaff/hubstaff.php");
$hubstaff = new hubstaff();
```


Next we'll be generating our App authentication token using your hubstaff account email address and password.
If we take a look into pages/dashboard.php file we can see the connection form

```html
<div class = "hubstaff-form">
	<form method = "post" action = "http://<?php echo $_SERVER[HTTP_HOST].$_SERVER[REQUEST_URI]; ?>" >
	  <input type = "text" name = "email" value = "" placeholder="Add your Hubstaff account email address" >
	  <input type = "text" name = "password" value = "" placeholder="Add your Hubstaff account password" >
	  <input type = "submit" value = "Connect">
	</form>
</div>
```
Form submission will call the following php code to generate the authentication token 
```php
$email = $_POST['email'];
$password = $_POST['password'];
$data = $hubstaff->auth($_POST['email'],$_POST['password']);
if(isset($data['auth_token']))
{
	$_SESSION['Auth-Token'] = $data['auth_token'];
	echo "<div class = 'info'>Your auth token is: ".$data['auth_token']."</div>";
}else
{
	echo "<div class = 'info'>error: ".$data['error']."</div>";
}
```

Now we'll move our generated authentication token it into our config.php file.

Once we got all that done we can start requesting account related data like reports, users, organizations, notes and others from hubstaff.

Now let's start with fetching the team reports in a specific period of time.

First we need to spicify all the parameters we're going to use for that operation.
```php
$params = array();
$params['start_date'] = "start_date";
$params['end_date']   = "end_date";
$params["options"][]  = "organizations";
$params["options"][]  = "projects";
$params["options"][]  = "users";
$params["options"][]  = "show_tasks";
$params["options"][]  = "show_notes";
$params["options"][]  = "show_activity";
$params["options"][]  = "include_archived";
		
$value_type = array();
$value_type['organizations'] = 'input';
$value_type['projects'] = 'input';
$value_type['users'] = 'input';
$value_type['start_date'] = 'date';
$value_type['end_date'] = 'date';
$value_type['show_tasks'] = 'select';
$value_type['show_notes'] = 'select';
$value_type['show_activity'] = 'select';
$value_type['include_archived'] = 'select';
```
we're going to have to required parameters "start_date" and "end_date" of type date ("YYYY-MM-DD").

Next we gonna fill our form using the following code.
```php
foreach($params as $index => $param)
{

	if($index == "options")
	{
		foreach($params[$index] as $option_param)
		{
				if($value_type[$option_param] == "input")
				{
					echo '<div class = "input-container" ><span class = "title">'.$option_param.'</span><input type = "text" name = "options['.$option_param.']" ></div>';
				}else if($value_type[$option_param] == "datetime")
				{
					echo '<div class = "input-container" ><span class = "title">'.$option_param.'</span><input type = "text" name = "options['.$option_param.']" class="form-control time" ></div>';
				}else
				{
					echo '<div class = "input-container" ><span class = "title">'.$option_param.'</span><select name = "options['.$option_param.']" ><option>0</option><option>1</option></select></div>';
				}
			}
	}
	else {
		if($value_type[$param] == "input")
		{
			echo '<div class = "input-container" ><span class = "title">'.$param.'</span><input type = "text" name = "'.$param.'" ></div>';
		}else if($value_type[$param] == "datetime")
		{
			echo '<div class = "input-container" ><span class = "title">'.$param.'</span><input type = "text" name = "'.$param.'" class="form-control time" ></div>';
		}else
		{
			echo '<div class = "input-container" ><span class = "title">'.$param.'</span><select name = "'.$param.'" ><option>0</option><option>1</option></select></div>';
		}
	}
}
```


