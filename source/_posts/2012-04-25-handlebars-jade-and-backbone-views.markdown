---
layout: post
title: "Handlebars, Jade and Backbone views"
date: 2012-04-25 16:47
comments: true
categories: [backbone, handlebars, jade, coffeescript, cake]
published: false
---

How to precompile Handlebars templates written in Jade for use in a Backbone application<!-- more -->

## Handlebars intro

If you're using Backbone, you're probably using a templating engine to render your views. My engine of choice at the moment is [Handlebars](http://handlebarsjs.com/) because it combines a simple, familiar syntax with the ability to precompile templates. Precompiling our templates allows for a two-fold speed increase, since we only have to include the smaller handlebars.runtime.min.js file (~2KB) instead of the whole library (~20KB) and our templates are no longer compiled on the fly in our Backbone view. However, if you want to precompile your templates, you're going to need some kind of build process. In steps [Cake](http://coffeescript.org/#cake)...


## Coffee and Cake

Cake allows you to write custom build tasks in coffeescript, so you can create a series of tasks for your common build tasks - e.g. compiling coffeescript to javascript and, of course, compiling your Handlebars templates! Here's a simple cake task that compiles all the templates in a directory to a single js file:

``` coffeescript Precompiling Handlebars templates with Cake
task 'handlebars:compile', 'compile Handlebars templates', ->
	exec 'handlebars src/html/* -f www/js/templates.js', (err, stdout, stderr) ->
		err && throw err
		console.log 'Handlebars compiled!'
```

Now all we have to do is include templates.js in our app, and the compiled template will be available at Handlebars.templates[template_name]. So, a very simple Backbone view might appear as follows:

``` coffeescript Minimum-viable Backbone view
class ViewName extends Backbone.View
	initialize : =>
		@render()
	render : =>
		@el.innerHTML = Handlebars.templates['template.html'] @model.toJSON()
```

This is great - we've got an easy, and more importantly, super-fast way of rendering our Backboen views. But what if we could make it even easier?

## Never write HTML again

Handlebars is cool, but used in it's most simplistic way, it still means we have to write HTML, which, let's face it, isn't much fun. A typical Handlebars template might look like:

``` html Handlebars in HTML
<div id="our_div">
	<p>{{handlebars_data}}</p>
</div>
```

It's still a mess of angualr brackets and opening / closing tags. [Jade](https://github.com/visionmedia/jade) is a neat little language that compiles to HTML, thus rescuing us from all this. Consider this simple form:

``` jade
form#some_id(action='')
	p
		label.inline(for='the_input') Label for input
		input(type='text', name='the_input', placeholder='Enter value here')
	p: input(type='submit', value='Submit')
```

... which compiles to:

``` html
<form id="some_id" action="">
	<p>
		<label class="inline" for="the_input">Label for input</label>
		<input type="text" name="the_input" placeholder="Enter value here" />
	</p>
	<p><input type="submit" value="Submit" /></p>
</form>
```

Even in this small example, there's noticably less work involved in typing out the Jade version. In a large project, this really adds up, so if you're not using Jade, you should be.

