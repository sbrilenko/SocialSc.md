## brief info

* there is no $rootScope in angular2. Need to replace it with special service or multiple services...
* refactor app module by module. It would be great to start with `theApp.config` module since it uses in a lot of places. the module itself
looks prety nice and there is nothing to refactor (except to remove some commented code). But need to replace the way how
this module is using in other modules and other parts of the code
* change build process. We can keep cordova cli for to build the app, but before cordova-cli we need to
have one more build step. We can start with applying ES6 code with `export` and `require`/`import` keywords and configure
our build system to compile this code into `web` directory in some special way (maybe even in one JS file, like `js-merged.js`).
In future we can extend this build system with TypeScript
* UI parts should be incapsulated in angular Components (directives in case of angular1) with separated html,css and js code.
* In angular1 directives you should make sure that "scope" is isolated and do not depends on anything around directive
* try to minimaze `$scope` usage. You can use `controllerAs` instead and use `this.<property>` inside a controller
* do not use `compile` in directives
* do not use `replace:true` in directives (in angular2 it is always `false`)

##Let's start with refactoring by modules

### theApp.config

This module looks fine but:

* need to incapsulate `SPLASHES`, `SUB_DOMAIN`, `MICRO_SITE` and other variables that are copy/pasted from locale to locale. As an example we can move
them to separated property. Actually locales are different only for labels. So idialy, in locales, we have to keep only labels.

* in `app.js` we place `APP_CONFIG` in the `rootScope`. We should avoid this. Let's create separated service (`Config`) for our config.
* we should use `Config` service instead of $rootScope and `APP_CONFIG` constant in any js file.
Since we usualy do `$watch` for changes in any Service from now we have to use events. `Config` setters should notify everyone who
subscribed to the event. See [This Doc](https://developer.mozilla.org/en-US/docs/Web/Guide/Events/Creating_and_triggering_events) for
info about how to create and trigger events. Try to minimaze using of `$watch`. So you should add setters for `Config` variables.
* `$rootScope` contains other variables that should be moved to `Config` or to another service. We have to avoid $rootScope usage.
* So, basicaly we have to find all the `$rootScope` and `APP_CONFIG` (which is in `$rootScope`) usages and replace them with our services.
Since `APP_CONFIG` uses in html files also, please assign `Config` service to the any property of your controller and use service directly in
your html files. As workaround to minimizing the repeating you can create controller that will incapsulate most usable services and then
use this controller via `controllerAs`. So you can access any service just including this new controller.

### theApp.home

* nothing to do here except of remove `$rootScope`. It is unused in this module.
* please notice that `home.html` uses `$root.go()` function to navigate to another page. Please replce it with anything else
(like our controller that incapsulate must usable services, or just include `RouteHelper` service (that you have to create) to you home module).

### theApp.tour

* remove `$rootScope` from the controller
* use `this.<property>` instead of `$scope.<property>`. Add `controllerAs` in the route config and use `controllerAs` value as accesser
to controller vairables in html file.
* html file uses `$root.go` function. Pleaes update it.

### theApp.faq

* repalce `$rootScope` from the controller with `Config`. Please notice in this case `$rootScope` is using in `if` condition, that's why
we have to replace it instead of delete
* html file uses `$root`. Update it.
* `linkHanlder` function from controller is actually using only in html file of `faq` module **BUT** almost same functionality
is using in other modules. See **(<file>:<line>)** `challange.js:133`, `signin.js` and `terms.js` files. So it would be good
to move this function to any service (or create new one).

### theApp.challenges

* replace `$rootScope` from the controller with `Config`.
* `$scope.go()` and `$scope.back()` already should be moved to the separated service. Please use it instead of $scope functions. Also notice
that `go` and `back` functions also set a parameter inside and call `$rootScope` implementations of same functions. You can just add
additional parameter to these functions that already implemented in a service.
* do not use `$scope`. Use `this` instead and `controllerAs` in route config.
* in case of some events like `$viewContentLoaded` you can add it to the grand-parent controller.
* `init` function can be used from similar function from a service. If you want to set default values for variables just declare them
on top of controller code.
* html file uses `$root`. Please replace it.

### theApp.challenge

* don't use `$rootScope` and `$scope`.
* use `controllerAs` in route config
* Do the same steps for `go`, `back` and `$viewContentLoaded` as described in `theApp.challenges` block.

### theApp.leaderboard

* don't use `$rootScope` and `$scope`.
* use `controllerAs` in route config
* Do the same steps for `go`, `back` and `$viewContentLoaded` as described in `theApp.challenges` block.
* Don't use `$root` in html file.

### theApp.team

* don't use `$rootScope` and `$scope`.
* use `controllerAs` in route config
* Do the same steps for `go`, `back` and `$viewContentLoaded` as described in `theApp.challenges` block.
* Don't use `$root` in html file.
* for any `$on` for scope use events in service setters


### theApp.stream

* don't use `$rootScope` and `$scope`.
* use `controllerAs` in route config
* Do the same steps for `go`, `back` and `$viewContentLoaded` as described in `theApp.challenges` block.
* don't use `$root` in html file.

### theApp.map

* it's ok to use jQuery.
* don't use `$rootScope` and `$scope`.
* use `controllerAs` in route config
* Do the same steps for `go`, `back` and `$viewContentLoaded` as described in `theApp.challenges` block.
* Don't use `$root` in html file.

### theApp.terms

* see section `theApp.faq` about `linkHanlder`.
* don't use `$rootScope` and `$scope`.
* use `controllerAs` in route config
* Don't use `$root` in html file.

### theApp.games

Nothing special to do here. Just follow suggestions above for other modules

### theApp.game

Nothing special to do here. Just follow suggestions above for other modules

### theApp.reward

Nothing special to do here. Just follow suggestions above for other modules

### theApp.player

Nothing special to do here. Just follow suggestions above for other modules

### theApp.menu

Nothing special to do here. Just follow suggestions above for other modules

### theApp.signin

Nothing special to do here. Just follow suggestions above for other modules

### theApp.twitter

Nothing special to do here. Just follow suggestions above for other modules

### theApp.facebook

Nothing special to do here. Just follow suggestions above for other modules

### theApp.beacons

Nothing special to do here. Just follow suggestions above for other modules

### theApp.sticker

Nothing special to do here. Just follow suggestions above for other modules

## Global

* almost all controllers have `init()` function. You can call it in controllers but implement this funciton in a service. Since this function
is the same from controller to controller.
* move SweetAlerts (`swal`) to separated service. Each method of service should be separated notification. Of course you can configure it with
method parameters. But do not use it as it is in for example `team.js` file and `renameTeam` function of the `$scope`.
* avoid `$.ajax` usages directly in controller
* in most html files you have img which is `back` button. Replace it with directive
* pull to refresh block is using in `challenges.html`, `leaderboard.html` and `stream.html`. Replace it with directive.
