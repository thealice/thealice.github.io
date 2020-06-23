---
layout: post
title:      "Lessons learned building my first Javascript App"
date:       2020-06-14 04:38:39 -0400
permalink:  lessons_learned_from_building_my_first_javascript_app
---


## Technical Requirements

Our fourth portfolio project had the following minimum requirements:

1. The application must be an HTML, CSS, and JavaScript frontend with a Rails API backend. All interactions between the client and the server must be handled asynchronously (AJAX) and use JSON as the communication format.

2. The JavaScript application must use Object Oriented JavaScript (classes) to encapsulate related data and behavior.

3. The domain model served by the Rails backend must include a resource with at least one has-many relationship. For example, if you were building an Instagram clone, you might display a list of photos with associated comments.

4. The backend and frontend must collaborate to demonstrate Client-Server Communication. Your application should have at least 3 AJAX calls, covering at least 2 of Create, Read, Update, and Delete (CRUD). Your client-side JavaScript code must use `fetch` with the appropriate HTTP verb, and your Rails API should use RESTful conventions.

My friends and I like to play a pictionary-like game when we're together and since we've been sheltering-in-place I decided to try and code one to play via video chat. 

The Rails API backend is where any data for the project is presisted to the database. In my case, I had an API for what I was calling "Themes" (the category a game and a prompt fit under) and "Prompts" (the word(s) a player has to draw). On my local machine these could be found at `http://localhost:3000/api/v1/themes` and `http://localhost:3000/api/v1/prompts`. When the frontend needs access to data stored in the backend database, for instance, when we begin a game and the game needs to know all the different prompt cards that make up a deck, we can use Asynchronous JavaScript and XML (AJAX) to make requests to the server without reloading the page. We can retrieve that data using the `fetch()` method, which returns a data object in response with information about each Prompt. THEN we process that response with a `then()` method to get the actual content of the prompts in JSON (or text) format. Finally, we use another `then()` method to actually do something with those Prompts. You can see how this works in the code below:

![fetch example code](https://i.imgur.com/ux0SWiZ.png)

Much of my project ended up being about manipulating the DOM (adding and removing game setup forms and instructional text) to keep this a single page app. But the part I was most excited to build was the drawing portion. Going into the project I had no idea how this would be done. After a little bit of research I found there is an HTML5 element called [canvas](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API) that makes all of that possible with surprisingly little code. According to the [MDN documentation](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API) "The actual drawing is done using the `CanvasRenderingContext2D` interface."


