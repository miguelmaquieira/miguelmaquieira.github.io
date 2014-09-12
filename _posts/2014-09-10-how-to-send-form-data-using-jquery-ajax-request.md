---
layout: post
title: How to sumbit form using jQuery and ajax request
tags: [jquery,ajax,mockjax,bootstrap]
---

## The problem

Passive data collection from your users is every day more and more common in modern applications, but you still require to get user information proactively. Normally, sending the whole form data as a key-value map use to be enough, but sometimes you need to build more complex data structures before submiting the form. And here is when **JSON** came to your resque. Furthermore, multidevice applications use a SOA arquitecture approach where JSON has became an standard to share data between the UI and the business layers. Then, we show 4 different ways to prepare data for your server request using the jQuery serialization feature.

## Technologies
+ [Bootstrap 3](http://getbootstrap.com/)
+ [jQuery](http://jquery.com/)
+ [jQuery-mockjax](https://github.com/appendto/jquery-mockjax)


## Solution

### Building the UI

Guess that we want to send user registration data to backend service. The form data structure is the following:

<table style="position: relative; width: 75%; margin: 0 auto; font-size: 16px;">
	<tr>
		<td colspan="2">
			User:
		</td>
	</tr>
	<tr>
		<td width="50%" style="border: none;">
			<ul>
				<li>First name</li>
				<li>Last name</li>
				<li>Address</li>
				<ul>
					<li>Street</li>
					<li>City</li>
					<li>Zip Code</li>
				</ul>
			</ul>
		</td>
		<td style="width: 50%; height: 100%; border: none; position: absolute">
			<ul>
				<li>Email</li>
				<ul>
					<li>[0]</li>
					<li>[1]</li>
					<li>...</li>
				</ul>
			</ul>
		</td>
	<tr>
</table>
<br>

We use **bootstrap** to build our sig up form.

<img src="/assets/img/2014-09-10_form_bootstrap3.png"/>

<a href="https://github.com/miguelmaquieira/blog-examples/blob/master/how-to-send-form-as-json-using-ajax-request/index.html" target="_blank">Click here to see the entire html code</a>

#### Creating **email** array fields dynamically

1 - The entry form data must allow to add multiple emails per user. So, we create the field for the first email, and we'll also let the user add new fields if necessary.

```html
<div class="control-group" id="emails">
	<div class="controls">
		<div class="entry input-group form-group">
			<span class="input-group-addon">@</span>
			<input class="form-control" name="emails[0]" placeholder="add email" type="text">
			<span class="input-group-btn">
				<button class="btn btn-success btn-add" type="button">
					<span class="glyphicon glyphicon-plus"></span>
				</button>
			</span>
		</div>
	</div>
</div>
<small>Press <span class="glyphicon glyphicon-plus gs"></span> to add another email :)</small>
```

2 - Javascript code to add a new email entry field:

``` javascript
	
$(document).on('click', '.btn-add', function(event) {
	event.preventDefault();
	var controlForm = $('.controls');
	var currentEntry = $(this).parents('.entry:first');
	var newEntry = $(currentEntry.clone()).appendTo(controlForm);
	newEntry.find('input').val('');
	controlForm.find('.entry:not(:last) .btn-add')
        .removeClass('btn-add').addClass('btn-remove')
        .removeClass('btn-success').addClass('btn-danger')
        .html('<span class="glyphicon glyphicon-minus"></span>');
        
	var inputs = $('.controls .form-control');
	$.each(inputs, function(index, item) {
  		item.name = 'emails[' + index + ']';
	});
});
```
3 - And javascript code to remove an entry field:

``` javascript

$(document).on('click', '.btn-remove', function(event) {
	event.preventDefault();
	$(this).parents('.entry:first').remove();
	var inputs = $('.controls .form-control');
	$.each(inputs, function(index, item) {
		item.name = 'emails[' + index + ']';
	});
}); 
```
### Mocking the response from server

You have to include **jquery-mockjax** library to fake responses from the server (you can read more about this library here: [how to mock ajax responses]({% post_url 2014-09-12-how-to-mockup-ajax-responses %}))

``` html
<form id="ajaxForm" class="form-vertical" role="form" autocomplete="off" method="POST" action="/ajaxRequest/">

```

``` javascript
$.mockjax({
	url: '/ajaxRequest/plainjson/',
    type: 'POST',
    contentType: 'text/json',
    responseTime: 0,
    response: function(settings) {
      var data = settings.data;
      this.responseText = data;
    }
});  
```

### Form data JSON serialization

Here we show 4 different ways to serialize form data for submission. We go from the most simple type provided by jquery till a custom processing to build a readable json structure (object oriented)

1 - [jquery.serialize()](http://api.jquery.com/serialize). This method creates a text string in standard URL-encoded notacion.

Main code:

``` javascript

var form = $('#ajaxForm');
var data = $(form).serialize();

```
Result:

``` html

firstname = firsname_value & lastname = lastname_value & address[street] = street_value & address[city] = city_value & address[zip] = zip_value & emails[0] = email0_value & emails[1] = email1_value

```

2 - [jquery.serializeArray()](http://api.jquery.com/serializeArray). Creates a JS array of objects ready to be encoded as a JSON string. 

Main code:

``` javascript

var form = $('#ajaxForm');
var arrayData = $(form).serializeArray();
var data = JSON.stringify(arrayData);

```
Result:

``` json

[{"name":"firstname", "value":"firstname_value"},{"name":"lastname", "value":"lastname_value"}, {"name":"address[street]","value":"street_value"},{"name":"address[city]","value":"city_value"}, {"name":"address[zip]","value":"zip_value"},{"name":"emails[0]","value":"email0_value"}, {"name":"emails[1]","value":"email1_value"}]

```

3 - Prettify jqyery.serializeArray() function. 

Main code:

``` javascript

var form = $('#ajaxForm');
var jsonData = {};
$.each($(form).serializeArray(), function() {
	jsonData[this.name] = this.value;
});
var data = JSON.stringify(jsonData);

```
Result:

``` json

[{"firstname":"firstname_value","lastname":"lastname_value","address[street]":"street","address[city]":"street_value","address[zip]":"zip_value","emails[0]":"email0_value","emails[1]":"email1_value"}]

````

4 - Custom serialization taking care about arrays fields and group fields.

Code:

<script src="https://gist.github.com/miguelmaquieira/4419d5a67e825c6c01cc.js"></script>

Result:

``` json

{"firstname":"firstname_value","lastname":"lastname_value","address":{"street":"street_value","city":"city_value","zip":"zip_value"},"emails":["email0_value", "email1_value"]}

````

### Resources:

<ul>
	<li>
		Test this code <a href="http://embed.plnkr.co/laHpfo/preview" target="_blank">here</a>
	</li>
	<li>
		Download source <a href="https://github.com/miguelmaquieira/blog-examples/tree/master/how-to-send-form-as-json-using-ajax-request" target="_blank">here</a>
	</li>
</ul>