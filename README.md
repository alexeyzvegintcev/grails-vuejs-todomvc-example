<img align="right" src="https://cloud.githubusercontent.com/assets/187726/25786548/f88e486c-335c-11e7-8af0-8a7c470e4112.png" width="350">

<!-- MarkdownTOC autolink="true" bracket="round" -->

- [Overview](#overview)
- [Phase 1 - Stock Vuejs front end](#phase-1---stock-vuejs-front-end)
- [Phase 2 - TodoMVC files](#phase-2---todomvc-files)
- [Phase 3 - Stock Grails Rest Application](#phase-3---stock-grails-rest-application)
- [Phase 4 - Grails Todo Rest Setup](#phase-4---grails-todo-rest-setup)
- [Phase 5 - Vue TodoMVC modifications for rest model](#phase-5---vue-todomvc-modifications-for-rest-model)
- [Phase 6 - Use v-model axios rest wrapper. Add error checking](#phase-6---use-v-model-axios-rest-wrapper-add-error-checking)

<!-- /MarkdownTOC -->

## Overview

Takes the Vue.js TodoMVC example and modifies it to use a [Grails](https://grails.org/) app to store the data.
TodoMVC examples from the poc site http://todomvc.com/
We use this one as a basis for the code https://vuejs.org/v2/examples/todomvc.html

## Phase 1 - Stock Vuejs front end
https://github.com/basejump/grails-vue-todomvc/tree/Phase1-vue-init-webpack

Install the [vue-cli](https://github.com/vuejs/vue-cli) and create a template

``` bash
$ vue init webpack vue-app

  This will install Vue 2.x version of the template.

? Project name vue-app
? Project description Vue.js todoMVC front end for Grails
? Author basejump <xxx@9ci.com>
? Vue build standalone
? Install vue-router? Yes
? Use ESLint to lint your code? Yes
? Pick an ESLint preset Standard
? Setup unit tests with Karma + Mocha? Yes
? Setup e2e tests with Nightwatch? Yes

   vue-cli · Generated "vue-app".

   To get started:

     cd vue-app
     npm install
     npm run dev

   Documentation can be found at https://vuejs-templates.github.io/webpack
```

now npm run dev should work and show the todo with local browser storage


## Phase 2 - TodoMVC files
[see github tag phase2-todomvc-example-copy](https://github.com/basejump/grails-vue-todomvc/tree/Phase2-todomvc-example-copy)

Copy code from https://vuejs.org/v2/examples/todomvc.html according to these [commits](https://github.com/basejump/grails-vue-todomvc/commit/e44c9aaa33b2d639a034ce95688e3235607b857c). 

`npm run dev` and check it out

## Phase 3 - Stock Grails Rest Application
[github tag Phase3-Stock-Grails-Rest-App](https://github.com/basejump/grails-vue-todomvc/tree/Phase3-Stock-Grails-Rest-App)


``` bash
$ grails create-app -profile rest-api -features hibernate5 grails-server

| Grails Version: 3.2.9
```

## Phase 4 - Grails Todo Rest Setup
see [this commit](https://github.com/basejump/grails-vue-todomvc/commit/2c252c176a8ffbe411883a661eb9d504a83a5ca1) for a walk through of the changes made

create the Todo domain

``` bash
$ grails create-domain-class Todo
```

``` groovy
import grails.rest.Resource

@Resource(uri = '/todo', namespace = '/api', formats = ["json"])
class Todo {
	
	String title
	Boolean completed = false
	Boolean archived = false

    static constraints = {
    	title nullable: false
    	completed nullable: false
    	archived nullable: false
    }
}
```

turn on cors in the application.yml

```
grails:
    cors:
        enabled: true
```

add some rows for sanity check
``` groovy
class BootStrap {
    def init = { servletContext ->
        new Todo(title:"Buy beer",completed:true).save(failOnError:true,flush: true)
        new Todo(title:"Drink beer").save(failOnError:true,flush: true)
        new Todo(title:"Create Vue/Grails TodoMVC example").save(failOnError:true,flush: true)
    }
...
```

`grails run-app` and sanity check it with curl

``` bash
$ curl -i -X GET -H "Content-Type: application/json"  localhost:8080/todo
HTTP/1.1 200
X-Application-Context: application:development
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Sun, 07 May 2017 06:01:57 GMT

[{"id":1,"archived":false,"completed":false,"title":"Buy beer"},{"id":2,"archived":false,"completed":false,"title":"Drink beer"}]
```

## Phase 5 - Vue TodoMVC modifications for rest model
[see this commit for changes](https://github.com/basejump/grails-vue-todomvc/commit/37da7efa416993a2b942ac7746778473e24d9ddf) 

1. `npm install axios --save` modify package.json per example and run npm install

2. modify vue-app/config/index.js to change port to 8090 so it doesn't conflict with grails

3. create new src/todoRestApi.js for communication with grails. See file.

4. modify src/main.js to use the new todoModel.js instead of the local storage `todoStorage` from original example

5. Make minor tweak to the complete check box to use even for save

5. `grails run-app` under the grail-server dir. In another shell window cd to vue-app and run 'npm run dev'

## Phase 6 - Use v-model axios rest wrapper. Add error checking
<img align="right" src="https://cloud.githubusercontent.com/assets/187726/25831513/519da1be-342a-11e7-8292-d723f152c0af.png" width="300">

[see this commit for changes](https://github.com/basejump/grails-vuejs-todomvc-example/commit/b657f26b2eb6a580171bb634795916990a5f1862) 

[vmodel](https://github.com/laoshu133/v-model) is a light weight wrapper around [axios](https://github.com/mzabriskie/axios) that allows you to use it more like ngResource and will even look somewhat similiar to Gorm/Grails activerecord way of working with items. While it says its for Vue, vue is not required and [axios](https://github.com/mzabriskie/axios) is all thats needed and since its [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) based API it will uses [bluebird](http://bluebirdjs.com/docs/getting-started.html) as well.


1. `npm install v-model --save` or modify package.json per example and run npm install

2. added todoModel.js to replace the axios based todoRestApi from phase 5 

3. modified main.js to use todoModel and the [vmodel](https://github.com/laoshu133/v-model) Model based methods

4. Add some error checking with `catch` and a div in index.html to diplay errors

5. Modify the Todo domain in grails so we can simulate an error by creaating an item with 'xxx'

6. `grails run-app` under the grail-server dir. In another shell window cd to vue-app and run 'npm run dev'

Try creating a todo with _xxx_ as the title or modifying an existing one. 







