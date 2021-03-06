# TDD-Express-Routes
On going tutorial I am writting on test driven development with express (will add more when I have time).

## Create a basic express App
This tutorial assumes you have a basic knowledge of Javascript, Node and Express. 

Create two files under the TDD_SAMPLE_APP app folder you just created:  
*package.json*
*app.js*

  add the following code to *package.json*
  ```json
  {
    "name": "tdd_sample_app",
    "version": "1.0.0",
    "scripts": {
        "start": "node app.js",
        "test": "mocha ./tests --recursive"
    },
    "author": "your name",
    "contributors": [],
    "dependencies": {
        "body-parser": "^1.4.3",
        "chai": "^4.2.0",
        "express": "^4.16.0",
        "mocha": "^5.2.0",
        "morgan": "^1.1.1",
        "npm": "^5.7.1",
        "request": "^2.79.0"
    },
    "devDependencies": {
        "supertest": "^3.4.2"
    }
}

  ```
  
  add the following code to *app.js*
  
  ```js
  

  // set up ========================
    var express  = require('express');
    var app = express();                              
    var morgan = require('morgan');             
    var bodyParser = require('body-parser');   
    
        
    const Request = require('request');

    app.use(morgan('dev'));                                         
    app.use(bodyParser.json());                                     

   
    var server = app.listen(process.env.PORT || 8080, function () {
    var port = server.address().port;
    console.log("App now running on port", port);
    });

  
app.get('/', function(req, res) { //route all other  requests here
         
          res.status(200);
          res.send("Hello World");
                               
});


module.exports = app;

  ```
  
  Run the following command
```bash
npm i
```

  
  Let's pause for a second. We've added a few key dependecies to our package.json that we will use for testing.
  *chai, mocha,* and *supertest*
  
  ### Supertest
  We will use supertest to test our routes
  
  ### Mocha 
  Mocha will run our tests 
  
  ### Chai 
  Chai will handle our assertions which can be written in several ways depending on comfort (we'll cover a few)
  
 Back to business. 
 
 Now create a dir called *tests* and under that dir create file called *app.tests.js*
 
 ![alt text](https://res.cloudinary.com/veedbeta/image/upload/v1549398138/image_3_tiiwpb.png)
 
 
  Run the following command
```bash
npm test
```
Since we have not written any tests we should see
````bash
0 passing
````
Let's write our first test:

Add the following code to *app.tests.js*

````javascript

const expect = require('chai').expect;
const assert = require('chai').assert;
const should = require('chai').should;

const request = require('supertest');

const app = require('../app');

describe('Unit testing /math/add ROUTE', function() {

    it('should return 422 status if numbers are missing from query params', function() {
      return request(app)
        .get('/math/add')
        .then(function(response){
            assert.equal(response.status, 422);
        });
    });
   
});

`````
 Run the following command
```bash
npm test
```
We should see that 1 test has failed 

```` bash
0 passing (924ms)
  1 failing

  1) Unit testing /math/add ROUTE
       should return 422 status if numbers are missing from query params:

      AssertionError: expected 404 to equal 422
      + expected - actual

      -404
      +422

````
Lets break down our first test. In the first few lines we import the neccessary modules. Pretty straight foward. Describe.() then allows us to group and label our tests. In this case we are grouping all of our tests related to the route *'math/add'*.

Our first test calls our *'math/add'* route and asserts that the response status code should be 422 because we did not pass the correct query parameters 

When this test is actually run it fails because the route does not yet exist therefore express returns a 404 response code instead. 

In Test Driven Development this is expected. We first write a unit test against code that does not yet exist; expecting this test to fail. We then write just enough code to pass that unit test.

Add the following code to existing code in *app.js*

````javascript
  app.get('/math/add', function(req, res) { //route all other  requests here

      if("numbers" in req.query){

      }else{
          res.status(422);
          res.send("Missing parameters");
      }

  });
  ````
  We have now created the route we are testing. We then check to see if *numbers* is present in the query paramaters of the request and otherwise respond with a 422 status code. This should be enough to pass our first test.  

 Run the following command
```bash
npm test
```

We should now see that our test has passed.

Let's created another unit test. 
Update *app.tests.js*:

````js
const expect = require('chai').expect;
const assert = require('chai').assert;
const should = require('chai').should;

const request = require('supertest');

const app = require('../app');

describe('Unit testing /math/add ROUTE', function() {

    it('should return 422 status if numbers are missing from query params', function() {
      return request(app)
        .get('/math/add')
        .then(function(response){
            assert.equal(response.status, 422);
        });
    });

    it('should return 422 status if numbers param is not an array', function() {
        return request(app)
            .get('/math/add')
            .query({ numbers: 5})
            .then(function(response){
                assert.equal(response.status, 422);
            });
    });

});



````

Our 2nd unit test is calling our route this time with the *numbers* param present in the request but as a singular number. We want this param to be an array of numbers.

We know this test will fail. Let's update our route to pass this test.

````js
  app.get('/math/add', function(req, res) { //route all other  requests here

      if("numbers" in req.query){

          if(req.query.numbers instanceof Array){
              
          }else{
              res.status(422);
              res.send("Invalid parameters");
          }

      }else{
          res.status(422);
          res.send("Missing parameters");
      }

  });
````

If we run the following command both our tests should now pass!
```bash
npm test
```

Let's another unit test:

````js
it('should return 500 status if numbers array contains non numeric value', function() {
        return request(app)
            .get('/math/add')
            .query({ numbers: [2, 3, 4, "5"]})
            .then(function(response){
                assert.equal(response.status, 500);
            });
    });
````

In our 3rd test we are calling our route with the correct numbers param as an array but this time with one array element as a string. Our route should respond with a 500 error code.

Let's update our route to pass this test:

```js
app.get('/math/add', function(req, res) { //route all other  requests here

      if("numbers" in req.query){

          if(req.query.numbers instanceof Array){

              for(var i = 0; i < req.query.numbers.length;i++){

                  if(isNaN(req.query.numbers[i])){

                      res.status(500);
                      res.send("Non numerical value found");
                      return;

                  }

              }

          }else{
              res.status(422);
              res.send("Invalid parameters");
          }

      }else{
          res.status(422);
          res.send("Missing parameters");
      }

  });
````

Our route now loops through each element in the numbers array and checks to ensure that it is a number otherwise it send a 500 response code. 

Let's add another test:

````js
 it('should return 200 status and some sort of numerical answer if a valid array of numbers is passed', function() {
      return request(app)
        .get('/math/add')
        .query({ numbers: [2, 3, 4, 5]})
        .then(function(response){
            assert.equal(response.status, 200);
            assert.property(response.body, 'answer');
            assert.typeOf(response.body.answer, 'number');
        });
    });
````


This test calls our route with the correct parameters. We then assert that the since valid query params were passed with the request, the response body should contain an answer and it should be of type *number*. 

Let's update our route to pass this test:

````js
 app.get('/math/add', function(req, res) { //route all other  requests here

      if("numbers" in req.query){

          if(req.query.numbers instanceof Array){

              for(var i = 0; i < req.query.numbers.length;i++){

                  if(isNaN(req.query.numbers[i])){

                      res.status(500);
                      res.send("Non numerical value found");
                      return;

                  }

              }

              res.status(200);
              res.send({answer:0});


          }else{
              res.status(422);
              res.send("Invalid parameters");
          }

      }else{
          res.status(422);
          res.send("Missing parameters");
      }

  });
````

Let's add one more test:
````js
    it('should add a list of numbers correctly and return the answer', function() {
        return request(app)
            .get('/math/add')
            .query({ numbers: [2,2,2]})
            .then(function(response){
                assert.equal(response.status, 200);
                assert.property(response.body, 'answer');
                assert.typeOf(response.body.answer, 'number');
                assert.equal(response.body.answer, 6);
            });
    });
 ````
   
   


    
    
    
  

