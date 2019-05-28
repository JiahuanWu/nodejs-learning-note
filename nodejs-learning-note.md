## Launching Node, Globals and Process
### launching Node.js

- REPL (read-eval-print-loop)
  ```
  node
  ```
- Eval option(e)
  ```
  node -e "console.log('hello')"
  ```
- Launching Node code from a file
  ```
  node file.js
  node /xx/file.js
  ```

### Globals
- process
- global - global object, has properties
- module.exports and exports
#### main properties of the *global* object:
- process
- require()
- module *and* module.exports
- console *and* console.log()
- setTimeout() *and* setInterval()
- __dirname(当前执行脚本所在的目录) *and* __filename(当前正在执行脚本所在位置的绝对路径)

## Node Modules with Require and Module
### require
- import core modules/packages
- import npm modules/packages
- import single file in a project
- import single JSON files
- import folders in a project(an alias for importing an index.js in that folder)

### require caching
Node.js will cache imports. So the code outside mudole.exports run just once even though we imported the same module twice.

### Node Patterns for module.exports
- export a function: `module.exports = function(ops){...}`
- export an object: `module.exports = {...}`
- export multiple functions: `module.exports.methodA = function(ops){...}` `exports.methodA = function(ops){...}`
- export multiple objects:`module.exports.objA = {...}` `exports.objA={...}`


*`module.exports.name = ...` or `exports.name = ...` are used for multiple export points in a single file. They are equivalent to using `module.exports = {name:...}*
eg:
```
exports.methodA = function(){...}
exports.methodB = function(){...}
exports.methodAC= function(){...}
```
```
module.exports = {
    methodA(){...}
    methodB(){...}
    methodC(){...}
}
```

## Core Modules
### main core modules:
- fs: work with the file system, files and folders
  - fs.readFile(): reads files asynchronously
  - fs.writeFile(): writes data to files asychronously
- path: parse file system paths across platforms
  - path.join(): create paths that are platform independent(eg: path.join('app','server.js'))
- querystring: parse query string data
- net: work with networking for various protocols
- stream: work with data streams
- events: implement event emitters
- child_process: spawn external processes
- os: access OS-level information including platform, number of CPUs, memory, uptime, etc.
- url: parse URLs
- http: make requests(client) and accept requests(server)
- https: do the same sa http only for HTTPS
- util: various utilities including promisify which turns any standard Node core method into a promise-base API
- assert: perform assertion based testing
- crypto: encrypt and hash information

## Event Emitters
### flow
```
const EventEmitter = require('events')

class Job extends EventEmitter{}
job =  new Job()

job.on('done', (param)=>{...})

job.emit('done', p1)
job.removeAllListeners()
```
### Modular Events:
seperate the event emitter class definition and the event emission into its own module but allow the observers to be defined in the main program. This aloows us to customize the module behavior without changing the module code itself.
eg:
*job.js*
```
const EventEmitter = require('events')
class Job extends EventEmitter{
    constructor(ops){
        super(ops)
        this.on('start',()=>{
            this.process()
        })
    }
    process(){
        setTimeout(()=>{
            this.emit('done',...)
        },700)
    }
}

module.exports = Job
```
*weekly.js*
```
var Job = require('./job,js')
var job = new Job()

job.on('done',(params)=>{

})

job.emit('start')
```

## HTTP
```
const http = require('http')
const url = ""
http.get(url, (response)=>{
    response.on('data', (chunk)=>{...})
    response.on('end', ()=>{...})
}).on('error', (error)=>{...})
```
### HTTP client with core http
1. no-buffer: get a small chunk of the overall response (usually a single line of the overall payload/body) during each **data** event; can process the data right away (preferred for *large data*) or save it all together in a buffer variable for future use once all the data has been received (preferred for JSON)
2. buffer: create a new variable as a *buffer variable* and save the chunks into it.

### HTTP client for JOSN
must get the entire response; use the *buffer variable*, when the response has ended, we parse the JSON(try/catch to handle failtures that may occur due to mal-formated JSON input)

### HTTP client POST request
```
const options = {
    ...
    method: 'POST'
}
http.request(options, (res)=>{...})
```

## HTTP Server with Core Http
```
const http = require('http')
const port = 3000
http.createServer((req,res)=>{
    res.writeHead(200, {...})
    res.end(...)
}).listen(port)
```
- `res.write(...)` adds data to the response
- `res.end(...)` finishes the response
- `res.statusCode = ...` change the status code of the response

user curl to get the response from the server in the terminal:
```
curl http://localhost:3000
```

- request.header: information about incoming requests headers
- request.method: information about the incoming requests methods
- request.url: information about the incoming request URL

processing incoming request body in the server:
```
...
http.createServer((req,res)=>{
    if(req.method == 'POST'){
        let buff = ''
        req.on('data', (chunk)=>{
            buff += chunk
        })
        req.on('end', ()=>{...})
    }else{...}
}).listen(port)
```