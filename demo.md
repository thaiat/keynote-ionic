## Preparation
* `yo mcfly --mobile`
* codio startup.sh
* Import facebook big and small logo
* Remove source map for gulp style
* Change browsersync port and tunnel
* Change browsersync watch on main.min.css
* Change auto save


### index.html
```html
<head>
    <meta charset="UTF-8">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no, target-densitydpi=device-dpi">
    <title>Facebook</title>
    <link rel="stylesheet" href="styles/main.min.css">
</head>
```

```html
<body class="platform-cordova">
```

## Demo

### Show the project
```
gulp help
gulp lint
gulp karma
````

### Scaffold modules ngFacebook and common
```
yo mcfly:module ngFacebook --skip-route
yo mcfly:module common
```

### Scaffold service facebook
```
yo mcfly:service ngFacebook facebook
```

### Scaffold login, newsfeed and controller
```
yo mcfly:controller common login
yo mcfly:controller common newsfeed

```

### create the views
```
cd client/scripts/common/views
touch login.html tabs.html requests.html messages.html notifications.html more.html newsfeed.html
cd ../../../..
```

Launch
```
gulp karma
gulp browsersync
```

### Populate facebook.js
```js
'use strict';
var servicename = 'facebook';

module.exports = function(app) {

    var dependencies = ['$window', '$rootScope', '$q', '$http'];

    function service($window, $rootScope, $q, $http) {
        var FB_LOGIN_URL = 'https://www.facebook.com/dialog/oauth';
        // By default we store fbtoken in sessionStorage. This can be overriden in init()
        var tokenStore = $window.localStorage;
        var fbAppId;
        var oauthRedirectURL;
        // Indicates if the app is running inside Cordova
        var runningInCordova;

        document.addEventListener('deviceready', function() {
            runningInCordova = true;
        }, false);

        /**
         * Initialize the ngFacebook module. You must use this function and initialize the module with an appId before you can
         * use any other function.
         * @param {String} appId - The id of the Facebook app
         * @param {String} redirectURL - The OAuth redirect URL. Optional. If not provided, we use sensible defaults.
         * @param {Object} store - The store used to save the Facebook token. Optional. If not provided, we use sessionStorage.
         */
        var init = function(appId, redirectURL, store) {
            fbAppId = appId;
            if(redirectURL) {
                oauthRedirectURL = redirectURL;
            }
            if(store) {
                tokenStore = store;
            }
        };

        /**
         * Application-level logout: we simply discard the token.
         */
        var logout = function() {
            tokenStore.fbtoken = undefined;
        };

        /**
         * Login to Facebook using OAuth. The app will redirect to the facebook page and back the application page.
         * @param {String} fbScope - The set of Facebook permissions requested
         */
        var login = function(fbScope) {
            if(!fbAppId) {
                throw new Error('Facebook App Id is not set');
            }
            fbScope = fbScope || ''; //email,user_friends,public_profile,user_about_me,user_actions.news,user_actions.video,user_activities,user_events';
            logout();
            oauthRedirectURL = oauthRedirectURL || document.location.origin;
            if(!runningInCordova) {
                window.location = FB_LOGIN_URL + '?client_id=' + fbAppId + '&redirect_uri=' + oauthRedirectURL + '&response_type=token&display=popup&scope=' + fbScope;
            } else {
                window.location = FB_LOGIN_URL + '?client_id=' + fbAppId + '&redirect_uri=' + oauthRedirectURL + '&response_type=token&display=popup&scope=' + fbScope;
            }

        };

        /**
         * @private
         * @param {String} queryString - The query string
         * @returns {Object} - The parsed query string available as an object
         */
        var parseQueryString = function(queryString) {
            var qs = decodeURIComponent(queryString);
            var obj = {};
            var params = qs.split('&');
            params.forEach(function(param) {
                var splitter = param.split('=');
                obj[splitter[0]] = splitter[1];
            });
            return obj;
        };

        /**
         * Checks the response url callback from facebook and store the access_token
         * @param {String} url - The url we are getting back from facebook
         * @returns {Boolean} - true if successfull, false otherwise
         */
        var oauthCallback = function(url) {
            // Parse the OAuth data received from Facebook
            var queryString;
            var obj;

            if(url.indexOf('access_token=') >= 0) {
                queryString = url.substr(url.indexOf('#') + 2);
                obj = parseQueryString(queryString);
                tokenStore.fbtoken = obj.access_token;
                return true;
            } else if(url.indexOf('error=') >= 0) {
                queryString = url.substring(url.indexOf('?') + 2, url.indexOf('#'));
                obj = parseQueryString(queryString);
                return false;
            } else {
                return false;
            }
        };

        /**
         * Lets you make any Facebook Graph API request.
         * @param {Object} obj - Request configuration object. Can include:
         *  method:  HTTP method: GET, POST, etc. Optional - Default is 'GET'
         *  path:    path in the Facebook graph: /me, /me.friends, etc. - Required
         *  params:  queryString parameters as a map - Optional
         * @returns {Promise} - returns a promise
         */
        var api = function(obj) {

            var method = obj.method || 'GET';
            var params = obj.params || {};

            params.access_token = tokenStore.fbtoken;

            return $http({
                method: method,
                url: 'https://graph.facebook.com' + obj.path,
                params: params
            })
                .error(function(data, status, headers, config) {
                    if(data.error && data.error.type === 'OAuthException') {
                        $rootScope.$emit('OAuthException');
                    }
                });
        };

        /**
         * Helper function to de-authorize the app
         * @returns {Promise} - returns a promise
         */
        var revokePermissions = function() {
            return api({
                method: 'DELETE',
                path: '/me/permissions'
            });
        };

        /**
         * Helper function for a POST call into the Graph API
         * @param {String} path - The path of the http post
         * @param {Object} params - The params
         * @returns {Promise} - returns a promise
         */
        var post = function(path, params) {
            return api({
                method: 'POST',
                path: path,
                params: params
            });
        };

        /**
         * Helper function for a GET call into the Graph API
         * @param {String} path - The path of the http get
         * @param {Object} params - The params
         * @returns {Promise} - returns a promise
         */
        var get = function(path, params) {
            return api({
                method: 'GET',
                path: path,
                params: params
            });
        };

        return {
            init: init,
            login: login,
            logout: logout,
            revokePermissions: revokePermissions,
            api: api,
            post: post,
            get: get,
            oauthCallback: oauthCallback
        };

    }
    service.$inject = dependencies;
    app.factory(servicename, service);
};
```

### Remove routes from ngFacebook

### Add the following code to main.js
```js
'use strict';

var namespace = 'main';

var angular = require('angular');
require('angular-ionic');

var app = angular.module(namespace, [
    'ui.router', 'ionic',
    // inject:modules start
    require('./common')(namespace).name,
    require('./ngFacebook')(namespace).name
    // inject:modules end
]);

app.run(['facebook', '$window', '$state', '$ionicPlatform', function(facebook, $window, $state, $ionicPlatform) {
    facebook.init('323666037785577');

    if(facebook.oauthCallback($window.location.href)) {
        $state.go('tabs.newsfeed');
    }
}]);

module.exports = app;
```


### main.scss
```css
$fb_blue:               #3b5998 !default;
$fb_medium_blue:        #6d84b4 !default;
$fb_lighter_blue:       #afbdd4 !default;
$fb_lightest_blue:      #d8dfea !default;

ion-nav-view {
    background-color: transparent !important;
}

.item-divider {
    color:gray;
}

.fblogin {
    background-color:$fb_blue !important;
}

.fb-button-login {
    background-color:rgb(80,107,161) !important;
    border-color:rgb(80,107,161) !important;
    color:white !important;
    top:100px;
}

.fbnews .tab-item {
  color: gray !important; }

.fbnews .tab-item-active {
  color: rgb(91,147,252) !important; }

.fbsearch input {
  background-color: transparent;
  color: white; }

.fbsearch {
  background-color: #31487C;
}

.bar-fb-blue {
    background-color: #4c6ba4 !important;
}

.small-font {
    font-size:13px !important;
    color:#6A6A74 !important;
}

.gray-background {
    background-color: rgb(210, 213, 218);
}
.padding {
    color:gray;
}
```


### common/index.js
change the routes
```js
            $stateProvider
                .state('login', {
                    url: '/login',
                    template: require('./views/login.html'),
                    controller: fullname + '.login as vm'
                })
                .state('tabs', {
                    url: '/tabs',
                    abstract: true,
                    template: require('./views/tabs.html')
                })
                .state('tabs.newsfeed', {
                    url: '/newsfeed',
                    views: {
                        'tab-newsfeed': {
                            template: require('./views/newsfeed.html'),
                            controller: fullname + '.newsfeed as vm'
                        }
                    }
                })
                .state('tabs.requests', {
                    url: '/requests',
                    views: {
                        'tab-requests': {
                            template: require('./views/requests.html'),
                            controller: fullname + '.newsfeed as vm'
                        }
                    }
                })
                .state('tabs.messages', {
                    url: '/messages',
                    views: {
                        'tab-messages': {
                            template: require('./views/messages.html')
                        }
                    }
                })
                .state('tabs.notifications', {
                    url: '/notifications',
                    views: {
                        'tab-notifications': {
                            template: require('./views/notifications.html')
                        }
                    }
                })
                .state('tabs.more', {
                    url: '/more',
                    views: {
                        'tab-more': {
                            template: require('./views/more.html')
                        }
                    }
                });
    $urlRouterProvider.otherwise('/login');
```

### Restart gulp browsersync

### index.html
change to the ui-view to
```html
 <ion-nav-view></ion-nav-view>
```
and add to body
```
class="plaform-cordova"
```


### login.js
```js
'use strict';
var controllername = 'login';

module.exports = function(app) {
    /*jshint validthis: true */

    var deps = ['facebook'];

    function controller(facebook) {
        var vm = this;
        vm.login = function() {
            facebook.login();
        };
    }

    controller.$inject = deps;
    app.controller(app.name + '.' + controllername, controller);
};
```

### login.html
```html
<ion-view>
    <ion-content class="fblogin">
        <div class="content padding has-header">
            <div class="button-block" style="text-align:center">
                <img src="images/facebook-title.png" width="60%">
            </div>
            <button class="button button-block button-positive fb-button-login" >Login</button>
        </div>
    </ion-content>
</ion-view>
```



### tabs.html
```html
<ion-tabs class="tabs-icon-top tabs-light fbnews xfade-in-not-out">

  <ion-tab title="News Feed" icon="icon ion-document-text"  ui-sref="tabs.newsfeed">
    <ion-nav-view name="tab-newsfeed"></ion-nav-view>
  </ion-tab>

  <ion-tab title="Requests" icon="icon ion-person-stalker" ui-sref="tabs.requests">
    <ion-nav-view name="tab-requests"></ion-nav-view>
  </ion-tab>

  <ion-tab title="Messages" icon="icon ion-chatbox" ui-sref="tabs.messages">
    <ion-nav-view name="tab-messages"></ion-nav-view>
  </ion-tab>

  <ion-tab title="Notifications" icon="icon ion-android-earth" ui-sref="tabs.notifications">
    <ion-nav-view name="tab-notifications"></ion-nav-view>
  </ion-tab>

  <ion-tab title="More" icon="icon ion-navicon" ui-sref="tabs.more">
    <ion-nav-view name="tab-more"></ion-nav-view>
  </ion-tab>

</ion-tabs>
```



### notifications.html
```html
<ion-view>
	<ion-header-bar class="bar-positive bar-fb-blue">
		<h1 class="title">Notifications</h1>
	</ion-header-bar>
	<ion-content></ion-content>
</ion-view>
```

### messages.html
```html
<ion-view>
	<ion-header-bar class="bar-positive bar-fb-blue">
		<h1 class="title">Messages</h1>
	</ion-header-bar>
	<ion-content></ion-content>
</ion-view>
```

Do for all views

Add `ng-click="vm.login()"`
anf check the result

### more.html
```html
<ion-view>
    <ion-header-bar class="bar-positive bar-fb-blue">
        <div class="item-input-wrapper fbsearch"> <i class="icon ion-ios7-search placeholder-icon"></i>
            <input type="search" class="" placeholder="Search">
        </div>
        <button class="button button-clear"> <i class="icon ion-navicon"></i>
        </button>
    </ion-header-bar>

    <ion-content>

        <div class="list">
            <a class="item item-icon-left" href="">
                <i class="icon ion-email energized"></i>
                Avi Haiat
            </a>

            <a class="item item-icon-left item-icon-right" href="">
                <i class="icon ion-chatbubble-working assertive"></i>
                Update Info
                <span class="badge badge-positive">18</span>
            </a>

            <div class="item item-divider">
                APPS
            </div>

            <a class="item item-icon-left" href="">
                <i class="icon ion-mic-a dark"></i>
                Record album
                <span class="item-note">Grammy</span>
            </a>

            <a class="item item-icon-left" href="">
                <i class="icon ion-person-stalker positive"></i>
                Find Friends
                <span class="badge badge-positive">20+</span>
            </a>

        </div>

    </ion-content>
</ion-view>
```

### newsfeed.js
```js
'use strict';
var controllername = 'newsfeed';
var _ = require('lodash');
module.exports = function(app) {
    /*jshint validthis: true */

    var deps = ['$rootScope', '$ionicLoading', '$state', 'facebook'];

    function controller($rootScope, $ionicLoading, $state, facebook) {
        var vm = this;
        //$ionicBackdrop.retain();
        $ionicLoading.show({
            template: '<i class="icon ion-loading-c" style="font-size:40px;"></i>',
            animation: 'fade-in',
            showBackdrop: true
        });
        if ($rootScope.items) {
            vm.items = $rootScope.items;
            vm.friends = $rootScope.friends;
            $ionicLoading.hide();
            //$ionicBackdrop.release();
        } else {
            vm.items = [];
            vm.friends = [];
            var fields = {
                'fields':'full_picture,likes,description,comments,from,story,message,name,created_time'
            };
            facebook.get('/me/home', fields ).then(function(res) {
                vm.items = res.data.data;
                $rootScope.items = res.data.data;

                vm.friends = _.chain(vm.items)
                    .map(function(item) {
                        return item.from;
                    })
                    .uniq(function(item) {
                        return item.id;
                    })
                    .value();
                $rootScope.friends = vm.friends;

                $ionicLoading.hide();
                //$ionicBackdrop.release();
            });
        }

        vm.logout = function() {
            facebook.logout();
            $state.go('login');
        };
    }

    controller.$inject = deps;
    app.controller(app.name + '.' + controllername, controller);
};

```

### requests.html
```html
<ion-view>
    <ion-header-bar class="bar bar-positive bar-fb-blue">
        <h1 class="title">Friends Requests</h1>
    </ion-header-bar>
    <ion-content>

        <ion-list ng-repeat="item in vm.friends">
            <ion-item class="item-thumbnail-left">
                <img ng-src="https://graph.facebook.com/{{ item.id }}/picture?width=120&height=120"/>
                <h2>{{item.name}}</h2>
                <p>1 mutual friend</p>
                <div class="button-bar">
                    <button class="button button-positive button-small" >Confirm</button>
                    <button class="button button-light button-small " style="margin-left:10px">Not Now</button>
                </div>

            </ion-item>
        </ion-list>

    </ion-content>
</ion-view>
```

### newsfeed.html
```html
<ion-view class="gray-background">

    <!--     HEADER (SEARCH) -->
    <ion-header-bar class="bar-positive bar-fb-blue">
        <div class="item-input-wrapper fbsearch">
            <i class="icon ion-ios7-search placeholder-icon"></i>
            <input type="search" placeholder="Search">
        </div>
        <button class="button button-clear" ng-click="vm.logout()">
            <i class="icon ion-navicon"></i>
        </button>
    </ion-header-bar>

    <!--     SUB HEADER  -->
    <ion-header-bar class="bar-subheader bar-stable">
        <div class="button-bar">
            <button class="button button-clear small-font">
                <i class="icon ion-compose"></i>
                Status
            </button>
            <button class="button button-clear small-font">
                <i class="icon ion-camera"></i>
                Photo
            </button>
            <button class="button button-clear small-font">
                <i class="icon ion-location"></i>
                Check in
            </button>
        </div>
    </ion-header-bar>

    <!--     CONTENT -->
    <ion-content>
        <!-- PULL TO REFRESH GOES HERE -->
        <!-- CARD GOES HERE -->

    </ion-content>
</ion-view>

```


card
```html
<div class="list card fbcard" ng-repeat="item in vm.items">

            <div class="item item-avatar">
                <img ng-src="https://graph.facebook.com/{{ item.from.id }}/picture" />
                <h2>{{item.from.name}}</h2>
                <p>{{item.created_time | date:'MMM d, y h:mma'}}</p>
            </div>
            <div class="item item-body">
                <p ng-if="item.story">{{item.story}}</p>
                <p>{{item.message}}</p>
                <p ng-if="item.name">{{item.name}}</p>
                <p ng-if="item.description">{{item.description}}</p>
                <img ng-if="item.full_picture" ng-src="{{ item.full_picture }}" style="width:100%; box-shadow:5px 5px 5px grey" />
            </div>
            <a class="item" ng-if="item.likes || item.comments">

                <h4>
                    <span ng-if="item.likes" class="padding">{{item.likes.data.length}} Likes</span>
                    <span ng-if="item.comments" class="padding">{{item.comments.data.length}} Comments</span>
                </h4>
            </a>

            <div class="item positive" style="padding:0;background-color:#F7F7F7">
                <div class="button-bar bar-positive positive">
                    <button class="button button-light button-clear small-font" >
                        <i class="icon ion-thumbsup"></i>
                        Like
                    </button>
                    <button class="button button-light button-clear small-font">
                        <i class="icon ion-chatbox"></i>
                        Comment
                    </button>
                    <button class="button button-light button-clear small-font">
                        <i class="icon ion-share"></i>
                        Share
                    </button>
                </div>
            </div>
        </div>
```


pull to refresh (under content)

```html
<ion-refresher  on-refresh="vm.doRefresh()">
</ion-refresher>

```
newsfeed.js
```js
'use strict';
var controllername = 'newsfeed';
var _ = require('lodash');
module.exports = function(app) {
    /*jshint validthis: true */

    var deps = ['$q', '$rootScope', '$ionicLoading', '$state', 'facebook'];

    function controller($q, $rootScope, $ionicLoading, $state, facebook) {
        var vm = this;
        var activate = function() {
            var deferred = $q.defer();
            $ionicLoading.show({
                template: '<i class="icon ion-loading-c" style="font-size:40px;"></i>',
                animation: 'fade-in',
                showBackdrop: true
            });
            if($rootScope.items) {
                vm.items = $rootScope.items;
                vm.friends = $rootScope.friends;
                $ionicLoading.hide();
                deferred.resolve(vm.items);
            } else {
                vm.items = [];
                vm.friends = [];
                var fields = {
                    'fields': 'full_picture,likes,description,comments,from,story,message,name,created_time'
                };
                facebook.get('/me/home', fields).then(function(res) {
                    vm.items = res.data.data;
                    $rootScope.items = res.data.data;

                    vm.friends = _.chain(vm.items)
                        .map(function(item) {
                            return item.from;
                        })
                        .uniq(function(item) {
                            return item.id;
                        })
                        .value();
                    $rootScope.friends = vm.friends;

                    $ionicLoading.hide();
                    deferred.resolve(vm.items);
                    //$ionicBackdrop.release();
                });
            }
            return deferred.promise;
        };

        activate();

        vm.logout = function() {
            facebook.logout();
            $state.go('login');
        };

        vm.doRefresh = function() {
            delete $rootScope.items;

            activate().
            finally(function() {
                $rootScope.$broadcast('scroll.refreshComplete');
            });
        };
    }

    controller.$inject = deps;
    app.controller(app.name + '.' + controllername, controller);
};
```
