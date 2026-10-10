# Request Contents

- **Request Query Parameters**

    Get data from client via URL Query Parameters (query string), this can be used in any endpoint, and its data is accessible inside the ```request.query``` object of fastify routes.

    ```js
    app.get('/query',(req,res)=>{
      console.log(req.query);
      res.send(req.query);
    });
    ```

    Using this route, we will be able to pass query parameters in the URL to the fastify ```/query``` endpoint declared above; for example from the browser we input:

    ```url
    localhost:8080/query?name=bob&age=18
    ```

    Parts of the query parameter in the URL.

    - ```?``` starts the query in the URL.
    - ```=``` creates a key-value pair ```KEY=VALUE```.
    - ```&``` to add another key-value pair, used to supply multiple key-value pairs in one query URL.

    Then we get the query key-value pairs on the backend (note that all values are strings, unless a schema converts them):

    ```json
    {
      "name" : "bob",
      "age" : "18"
    }
    ```

- **Request Parameter**

    Get data from client via URL path parameters (route parameters), this can only be used on specifically defined endpoints, and its data is accessible inside the ```request.params``` object of fastify routes.

    ```js
    app.get('/parameter/:startday-:endday/:time',(req,res)=>{
      console.log(req.params);
      res.send(req.params);
    });
    ```

    Using the route above, we will be able to pass parameters in the URL to the fastify end point ```/parameter```; for example from the browser we input:

    ```url
    localhost:8080/parameter/mon-fri/8am
    ```

    Here we assign values to the parameters, the ```startday``` was given the value ```mon```, ```endday``` was given the value ```fri```, and ```time``` given the value ```8am```.

    Using the ```req.params``` object we will be able to access the key-value pairs at the backend:

    ```json
    {
      "startday" : "mon",
      "endday" : "fri",
      "time" : "8am"
    }
    ```

    **Request Body**

    The ```req.body``` will contain the parsed request payload of HTTP methods like ```POST```, ```PUT``` and ```PATCH```, e.g. JSON sent by frontend ```fetch()``` calls or HTML form submissions (Fastify parses JSON and plain text out of the box; form submissions need the `@fastify/formbody` plugin).