## MongoDB
### MongoDB Shell
cmd在mongoDB安装目录下的bin目录下执行mongo命令/打开bin目录下mongo.exe

*some useful commands:*
- help: list of available Mongo shell commands
- show dbs: list all the databases in this DB server/instance
- use board: work on a specific database named board
- show collections: list all collections in this database
- db.message.remove(): remove all documents from messages collection
- var a=db.messages.findOne()
- printjson(a)
- a.message="hi"
- db.messages.save(a): save method
- db.messages.find(): a read query
- db.messages.update({name:"xxx"},{$set:{message:"xxx"}}): an update documents query
- db.messages.find({name:"John"}): a read query with a specific confition/query which matches only documents with property name which equals to value John
- db.messages.remove({name:"xxx"}): a remove query with a condition

### MongoDB Native Driver
`npm install mongodb`

**以下为2.x版本代码**
- import the driver library:
    `const MongoClient = require('mongodb').MongoClient`
- connecting to the database
    ```
    const url = 'mongodb://localhost:xxxx/xxx'
    MongoClient.connect(url,(err, db) => {
        if(err) return process.exit(1)
        //perform queries
        db.close()
    })```
- create a new document:
    ```
    const insertDocuments = (db, callback) =>{
        const collection = db.collection('xxx')
        collection.insert([{name:'a'}], (error,result) =>{
            if(err) return process.exit(1)
            callback(result)
        })
    }
    ```
- updateing documents:
    ```
    const updateDocument = (db, callback) =>{
        var collection = db.collection('xxx')
        collection.update({id:'xxx'}, {$set: {key1:'a'}}, (error, result) => {
            if(error) return process.exit(1)
            callback(result)
        })
    }
    ```
- removing documents:
    ```
    const removeDocument = (db, callback) =>{
        var collection = db.collection('xxx')
        collection.remove({id:'xxx'}, (error, result) => {
            if(error) return process.exit(1)
            callback(result)
        })
    }
    ```
- finding documents:
    ```
    const findDocument = (db, callback) =>{
        var collection = db.collection('xxx')
        collection.find({}).toArray((error, docs) => {
            if(error) return process.exit(1)
            callback(docs)
        })
    }
    ```

***the methods need to be placed inside of the connect callback to ensure that the proper db reference to database connection exists***
```
MongoClient.connect(url, (error, db) => {
    if(error) return process.exit(1)
    insertDocument(db, ()=> {
        db.close()
    })
})
```

**3.x版本：**

http://mongodb.github.io/node-mongodb-native/3.1/api/
```
const mongodb = require('mongodb')
const url = 'mongodb://localhost:27017'
const dbName = 'xxx'

mongodb.MongoClient.connect(url, (error, client) =>{
    if(error) return process.exit(1)
    const db = client.db(dbName)
    app.get('/xxx', (req,res)=>{
        //查询-find/findOne; 新增-insert/insertOne/insertMany; 更新-update/updateOne/upateMany; 删除-deleteOne/deleteMany
        db.collection('collectionName').find({}).toArray((error, docs)+>{
            if(error) return process.exit(1)
            res.send(docs)
        })
    })
})
```

### MongoDB GUI client MongoUI
cmd任一目录执行mongoui命令

## Mongoose

***Mongoose is object-document mapper***
```
const mongoose = require('mongoose')
mongoose.connect('mongodb://localhost/xxx')

//model
let Book = mongoose.model('Book', {name: String})
//instance
let practicalNodeBook = new Book({name: 'Practical Node.js'})

practicalNodeBook.save((err, results)=>{
    if(err){
        process.exit(1)
    }else{
        process.exit(0)
    }
})
```
### Mongoose schemas:
*is a JSON-ish class that has information about properties/field types of a document, can store information about validation and default values, and whether a particular property is required*

***warning: Mongoose ignores those properties that aren't defined in the model's schema***

### mongoose schema supports these data types:
- String
- Number, long needs to use mongoose-long
- Boolean
- Buffer: a Node.js binary type(images, PDFs, archives, and so on)
- Date
- Array
- Schema.Types.ObjectId: a typical, MongoDB 24-character hex string of a 12-byte binary number
- Schema.Types.Mixed: any type of data
  
***warning: Mongoose does not listen to mixed-type object changes, so call markModified() before saving the object to make sure changes in the mixed-type filed are presistent.***

- custom schema types:
    set, get, default, validate
```
const aSchema = new mongoose.Schema({
    slug:{
        type: String,
        set: function(slug){
            return slug.toLowerCase()
        }
    },
    numberOfLikes: {
        type: Number,
        get: function(value){
        return value.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",")
    },
    posted_at: {
        type: String,
        get: function(value){
            if(!value) return null,
            return value.toUTCString()
        }
    },
    authorId: {
        type: ObjectId,
        default: function(){
            return new mongoose.Types.ObjectId()
        }
    },
    email: {
        type: String,
        unique: true,
        validate: [
            function(email){
                return (email.match(/[a-z0-9!#$%&'*+\/=?^_`{|}-]+(?:\.[a-z0-9!#$%&'*+\/=?^_`{|}-]+)*@(?:<a href="?:[a-z0-9-]*[a-z0-9]" title="" target="_black">a-z0-9</a>?\.)+<a href="?:[a-z0-9-]*[a-z0-9]" title="" target="_blank">a-z0-9</a>?/i) != null)},
                'Invalid emal'
        ]
    }
})
```

chained methods:
- Use Schema.path(name) to get SchemaType
- Use SchemaType.get(fn) to set the getter method
```
aSchema.path('numberOfPosts')
.get(function(value){
    if(value) return value
    return this.posts.length
})
```
path is a fancy name for the nested field name and its parent objects

### Custom Static and Instance Methods:
- instance method:
    ```
    const aSchema = mongoose.Schema({...})
    aSchema.method({
        fnName(params,callback){...}
    })
    let AModel = mongoose.model('AModel',aSchema)
    let aInstance = new AModel({})
    aInstance.fnName(1,()=>{})
    ```
- static method:
    ```
    const aSchema = mongoose.Schema({...})
    aSchema.static({
        fnName(params,callback){...}
    })
    let AModel = mongoose.model('AModel',aSchema)
    Amodel.fnName(1,()=>{})
    ```

*Hooks to keeping code organized:*
```
aSchema.pre('save', function(){...})
aSchema.post('save', function(){...})
```

### Virtual Fields: (don't exist in the database but act just like normal fields in a Mongoose document
call the virtual method to create a virtual  type, apply a getter function with get(fn)
```
aSchema.virtual('keyName')
.get(function(){ return... })
```

***nested documents:***
two ways:
- use Schema.Types.Mixed
    ```
    const userSchema = new mongoose.Schema({
        name: String,
        posts: [mongoose.Schema.Types.Mixed]
    })
    let User = mongoose.model('User', userSchema)
    ```
- create a new schema for the nested document
    ```
    var postSchema = new mongoose.Schema({
        title: String,
        text: String
    })
    var userSchema = new mongoose.Schema({
        name:String,
        posts: [postSchema]
    })
    var User = mongoose.model('User', userSchema)
    ```
to create a new user document or to save a post to an existing user when working with a nested posts document, treat the posts property as an ***array*** and use the ***push*** method from the JavaScript/Node.js API, or use the MongoDB ***$push*** operand

*relationships and Joins with Population:*
```
const userSchema = Schema({
    _id: Number,
    name: String,
    posts: [{type: Schema.Types.ObjectId, ref:'Post'}]
})
const postSchema = Schema({
    _creator: {type: Number, ref:'User'},
    title: String,
    text: String
})

let Post = mongoose.model('Post', postSchema)
let User = mongoose.model('User', userSchema)

User.findOne({name:/azat/i})
.populate('posts')
.exec(function(err, user){...})
```
