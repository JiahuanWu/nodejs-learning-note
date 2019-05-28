## Express.js
### express project structure:
- app.js: main file, house the embedded server and application logic
- /public: contains static files to be served by the embedded server
- /routes: houses custom routing for the REST API servers(not needed for a web app)
- /routes/users.js: endpoint/routes for resources such as users
- /views: contains templates that can be processed by a template engine(not needed for REST APIs)
- /package.json: project manifest
- /www: boot up script folder

*(a web app is a traditional web application with 100% server-side rendering while a REST API is a data only server)*

### Middleware
a series of precessing units connected together, where the output of one unit  is the input for the next one
```
(req,res,next)=>{
    //run some code
    next(output) //Error or real output
}
```
request -> middleware1 -> middleware2 -> ...middlewareN -> route -> response

to apply a middleware: `app.use(middleware)`

*middlewares are executed in the **order** specified*

middleware has twos types:
1. npm modules, eg: app.use(bodyParser.json())
2. custom middleware, eg: app.use((req,res,next)=>{next())

example:
parsing body: body-parse
```
const bodyParser = require('body-parser')
//AJAX/XHR from single-page applications and other JSON Rest clients
app.use(bodyParser.json())
//HTML web forms with action attribute
app.use(bodyParser.urlencoded({extended:false}))
```
****extended:false*** uses the **querystring** library while ***extended:true*** uses the **qs** library. **qs** allows for rich objects and arrays to be encoded into the URL-encoded format.

### Request and Response
*request*:
- **request.params**: URL parameters
- **request.query**: query string parameters
- **request.route**: current route as a string
- **request.cookie**: cookies, requires cookieParser
- **request.signedCookies**: signed cookies, requires cookie-parser
- **request.body**: body/payload, requires body-parser
- **request.headers**: headers

*response*:(additional methods to the core http's statusCode(), writeHead(), end() and write())
- **reponse.redirect(url)**: redirect request
- **response.send(data)**: send response
- **response.json(data)**: send JSON and force proper headers
- **response.sendfile(path, options, callback)**: render a template
- **response.locals**: pass data to template

### URL parameters, query strings and input validation
- accessing URL parameters:
    ```
    app.get('user/:id/transactions/:transactionId', (req,res)=>{
        const userId = req.params.id,
            transactionId = req.params.transactionId
    })
    ```
- accessing query string data
    `req.query.keyName`
- input validation
  + manual validation can be done in each route which accepts data
  + use express-validator