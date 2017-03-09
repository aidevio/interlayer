# interlayer
[![npm version](https://img.shields.io/npm/v/interlayer.svg?style=flat-square)](https://www.npmjs.com/package/interlayer)
[![npm downloads](https://img.shields.io/npm/dm/interlayer.svg?style=flat-square)](https://www.npmjs.com/package/interlayer)
[![github license](https://img.shields.io/github/license/donkilluminatti/interlayer.svg)](https://github.com/DonKilluminatti/interlayer/blob/master/LICENSE)
[![Build Status](https://travis-ci.org/aidevio/interlayer.svg?branch=master)](https://travis-ci.org/aidevio/interlayer)
[![Code Climate](https://codeclimate.com/github/aidevio/interlayer/badges/gpa.svg)](https://codeclimate.com/github/aidevio/interlayer)

Server current in alpha. You can offer or report new issue here: [New issue](https://github.com/DonKilluminatti/interlayer/issues/new)*

!!! MAJOR UPDATE 0.3.0: I'm was break old initialization, rewrite please it in your projects.

## Features
* Serve your files as is
* auto-reload server on file changes(new files not handle reload)
* cluster your server is need
* postgres\mysql\redis DAL's for save data
* simple localization

## Install
```js
npm install --save interlayer
```

## How to use
```js
let config = {
    port: 80,
    serve: ['files']
};
require('interlayer')(config);
```
Tree example:
* /node_modules/
* package.json
* /files/
    * index.html
    * style.css
    * script.js
    * /images/
        * logo.jpeg
* index.js

## Possible configuration params:
`Config` object properties

`port`: Port number of web server. (Default: 8080)
`startPath` / `rootPath`: Root server path. (Default: ./)
`logPath`: Path where will be created `logs.log` file. (Default: ./)
`timeout`: Timeout number in seconds at the expiration of user will see `{error: 'TIMEOUT'}` **Note, execution of the method is not interrupted**
`numOfServers` / `clusters`: Number of phisical processors | number of parallel servers for load balancing. If number more than 1, uses node cluster.
`useWatcher`: Boolean value determine is server will restart automatically when files in the folder with modules was changed.
`useDals`: An object of dal modules which need to include. Last version supports redis, mysql, postgress. For built-in redis [see here](https://github.com/NodeRedis/node_redis#options-object-properties), mysql [see here](https://github.com/mysqljs/mysql#connection-options), postgres [see here](https://github.com/brianc/node-postgres/wiki/Client#parameters)
`serve`: An array folders to serve. Priority over the last folder.
`modules: An array folders with modules. Priority over the last folder. [How to create](#create-module)
`dals`: An array of folders with your dal modules. Priority over the last folder. [How to create](#create-dal)
`middleware`: An array of folders with middlewares. Priority over the last folder. [How to create](#create-middleware)
`middlewareOrder`: An array with ordered names of middlewares
`middlewareTimeout`: Timeout number in seconds at the expiration of user will see `{error: 'TIMEOUT'}` **Note, execution of the middlewares is not interrupted**
`views`: An array of folders with files, which you can be uses as templates, or returned through the api. Priority over the last folder.
`i18n`: An array of folders with localization files. Priority over the last folder. [How to create](#localization)
`defaultHeaders`: An object with default headers, which have to be added at every response.

## Experimental properties
`disableNagleAlgoritm`: Boolean flag to disable Nagle algoritm for all connections. [Read more](https://en.wikipedia.org/wiki/Nagle%27s_algorithm)
`debug`: Allow to display `log.d` in console and adding to log file


## Config.modules option
Example of modules/myModule.js

```js
// you may need to log something, see above methodLog methods
let log = global.logger.create('moduleID');

// define meta information for method, without it method will be unvisible
exports._myMethod = {
    toJson: true
};
exports.myMethod = (request, cb) => {
    log.i('I am log without requestId')
    cb(null, {ok: true});
};
```

### Features
`global.logger.create('moduleID')` params:
```js
let methodLog = global.logger.create('moduleID')
methodLog.i() // Usual log - displayed in green
methodLog.w() // Warn - displayed in yellow
methodLog.e() // Error - displayed in red
methodLog.c() // Critical error - displayed in white
```

Also you can set default module meta information, which define module methods metas
```js
exports.__meta = {
    contentType: 'json'
};
```

These metas can be difined in module meta or in method meta:
* `contentType = 'json';` || `toJson = true` -
* `timeout = 60;` - timeout in seconds before response on hung request will be `{error: 'TIMEOUT'}`
* `addToRoot = true;` - if you specify this option then the method will be located at ~~myModule~~/`myMethod` without specifying the module name
* `skipRequestLog = true;` - if you specify this option it will disable save and display in console log information about calling this method
* `prerun = (request, moduleMeta, cb) => {}` - prerun function, like main method, takes request, and cb, but also takes module meta at the second parametr; May be usefull for preparing request.

Also you can add initialization for module where simpleContext is `Object({DAL: {}})`
```js
exports.__init = (simpleContext) => {
    // do something, example some work with using simpleContext.redis.blpop
};
```

Lest consider request propetries and methods
*exports.myMethod = (**request**, cb)*
###### request properties
* `request.ip` - Client ip adress
* `request.method` - Uppercased type of request - POST|GET|...
* `request.isPost` - true|false
* `request.params` - An object of parsed GET params
* `request.post` - An object of parsed POST params
* `request.cookies` - An object parsed cookies
* `request.headers` - An object of request headers
* `request.config` - An object of configuration which you specified at start of server
* `request.DAL` - An object with DALs, which you specified in `config.useDals`

###### request methods
* `request.modifyLog(log)` - *>=0.2.10* modify log instanse by adding to top of logged arguments by default
* `request.getView('file.html', cb)` - *>=0.1.7* return in `cb` file(from one of folders specified in `config.view`) content `cb(null, content)` or error `cb(error)`
* `request.getViewSync('file')` - *>=0.1.7* sync version of getView. return file(from one of folders specified in `config.view`) content or *null* if file not found
* `request.addCookies(key, value)` - set cookies to response (alias: addCookie,setCookie,setCookies - *>=0.3.4*)
* `request.rmCookies(key)` - delete cookies of expire cookies in responce (alias: rmCookie,delCookie,delCookies - *>=0.3.4*)
* `request.l18n(key, def)` - return localized string(folder with localization must be defined in `config.i18n = []`). In key not found, returns `def`

###### Manual responses
* `request.getResponse()` - this method return response instance
* `request.error(error)` - return an error in the response, where *error* is text of Error instance
* `request.end(text, code, headers, type)` - instead of calling callback you can return custom response where:
 * *text* is responce
 * *code* is HTTP status code
 * *headers* is object with headers
 * *type* only makes sense in the value `bin` - for binary response

And finally consider method callback
*exports.myMethod = (request, **cb**)*
```js
cb(err, text, code, headers, type)
//- err - may be error instance or string, number, array, object
//- text - responce, may be string, number, array, object, buffer
//- code - is HTTP status code
//- headers - manual headers for response
//- type - only makes sense in the value `bin` - for responce binary data
```

###### Use dals:
```js
    request.DAL.redis.get('somekey', (err, data) => {
        if(err){
            log.e('redis.get somekey', err);
            return cb(err);
        }
        ...
    });
```


## Create dal
Example of dals/nameofdal.js
Then you can add `nameofdal` to `config.useDals` array (ex: `config.useDals = ['nameofdal'];`)
```js
// init is not required
exports.init = (config) => {
};

// but methods is required
exports.methods = {
    get: () => {},
    set: () => {}
}
```


## Create middleware
Example of middleware/session.js
```js

exports.triggers = {
    'meta.checkSessions': 'checkSession', // you can specified string - name of function exported in module
    'request.params.test': exports.test // also you can specified function or method of object
};

// but methods is required
// request context and callback described in module
exports.checkSession = (request, moduleMeta, cb) => {

};

exports.test = (request, moduleMeta, cb) => {

};
```


## Localization
Example of i18n/en.js
**Note! You have to use double quotes, instead single quotes, cause it's json file**
```json
{
    "title_error_404": "Nothing found, 404, Bill Gates site"
}
```
