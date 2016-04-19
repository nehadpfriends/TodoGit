// server.js

var express = require('express');
var mongoose = require('mongoose');
var bodyParser = require('body-parser');
var morgan = require('morgan');
var methodOverride = require('method-override');

mongoose.connect("mongodb://root:abc123@ds023490.mlab.com:23490/webappstory", function(err){
	if(err){
		console.log(err);
	} else{
		console.log("connected to database");
	}
});


	var app = express();

	app.use(express.static(__dirname + '/public'));                 // set the static files location /public/img will be /img for users
    app.use(morgan('dev'));                                         // log every request to the console
    app.use(bodyParser.urlencoded({'extended':'true'}));            // parse application/x-www-form-urlencoded
    app.use(bodyParser.json());                                     // parse application/json
    app.use(bodyParser.json({ type: 'application/vnd.api+json' })); // parse application/vnd.api+json as json
    app.use(methodOverride());

   var Todo = mongoose.model('Todo', {
        text : String,
        id:Number
    });

app.get('*', function(req, res){
	res.sendFile('public/index.html');
});

  // api ---------------------------------------------------------------------
    // get all todos
    app.get('/api/todos', function(req, res) {

        // use mongoose to get all todos in the database
        Todo.find(function(err, todos) {

            // if there is an error retrieving, send the error. nothing after res.send(err) will execute
            if (err)
                res.send(err)

            res.json(todos); // return all todos in JSON format
        });
    });

    // create todo and send back all todos after creation
    app.post('/api/todos', function(req, res) {

        // create a todo, information comes from AJAX request from Angular
        Todo.create({
            text : req.body.text,
            done : false
        }, function(err, todo) {
            if (err)
                res.send(err);

            // get and return all the todos after you create another
            Todo.find(function(err, todos) {
                if (err)
                    res.send(err)
                res.json(todos);
            });
        });

    });

    // delete a todo
    app.delete('/api/todos/:todo_id', function(req, res) {
        Todo.remove({
            _id : req.params.todo_id
        }, function(err, todo) {
            if (err)
                res.send(err);

            // get and return all the todos after you create another
            Todo.find(function(err, todos) {
                if (err)
                    res.send(err)
                res.json(todos);
            });
        });
    });

    app.put('/api/todos/:todo_id', function (req, res) {
    	console.log(req.params.todo_id);
     Todo.findById(req.params.todo_id, function(err, todo){
     	console.log(err);
     	if(err){
     		res.send("Not Found");
     	}
     	console.log(req.body.text);
    	var text = req.body.text;
    	console.log(todo);
    	todo.text=text;
    	todo.save(function(err){
    		if(!err){
    			res.json(todo);
    		} 
    		res.send(err);
    	});
    });



});

/*app.put('/api/todos/:todo_id/:todo_text', function(req, res){

	Todo.create({
            text : req.body.text,
            _id : req.params.todo_id
       
        }, function(err, todo) {
            if (err)
                res.send(err);
	
	Todo.update({todo_id: _id}, {text: text}, function(text){
	Todo.find(function(err, todos) {
                if (err)
                    res.send(err)
                res.json(todos);
            });
});
});
	});*/



app.listen(4450);
    console.log("App listening on port 4440");


