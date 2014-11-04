* Change index.html to reference main.min.css instead of main.css
* Import facebook big and small logo
* Remove source map for gulp style
* Fix title app

### index.html
```html
<head>
    <meta charset="UTF-8">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no, target-densitydpi=device-dpi">
    <title>Facebook</title>
    <link rel="stylesheet" href="styles/main.css">
</head>
```

### Scaffold module ngFacebook and common
```
yo angular-famous-ionic:module ngFacebook
yo angular-famous-ionic:module common
```

### Scaffold service facebook
```
yo angular-famous-ionic:service ngFacebook facebook
```

### Scaffold login, newsfeed and requests controller
```
yo angular-famous-ionic:controller common login
yo angular-famous-ionic:controller common newsfeed

```

### create the views
```
cd client/scripts/common/views
touch login.html tabs.html requests.html messages.html notifications.html more.html newsfeed.html
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
        // Because the OAuth login spans multiple processes, we need to keep the success/error handlers as variables
        // inside the module instead of keeping them local within the login function.
        var deferredLogin;
        // Indicates if the app is running inside Cordova
        var runningInCordova;
        // Used in the exit event handler to identify if the login has already been processed elsewhere (in the oauthCallback function)
        var loginProcessed;
        document.addEventListener('deviceready', function() {
            runningInCordova = true;
        }, false);

        /**
         * Initialize the ngFacebook module. You must use this function and initialize the module with an appId before you can
         * use any other function.
         * @param appId - The id of the Facebook app
         * @param redirectURL - The OAuth redirect URL. Optional. If not provided, we use sensible defaults.
         * @param store - The store used to save the Facebook token. Optional. If not provided, we use sessionStorage.
         */
        var init = function(appId, redirectURL, store) {
            fbAppId = appId;
            if (redirectURL) {
                oauthRedirectURL = redirectURL;
            }
            if (store) {
                tokenStore = store;
            }
        };

        /**
         * Login to Facebook using OAuth. The app will redirect to the facebook page and back the application page.
         * @param fbScope - The set of Facebook permissions requested
         */
        var login = function(fbScope) {
            if (!fbAppId) {
                throw new Error('Facebook App Id is not set');
            }
            fbScope = fbScope || '';
            logout();
            var oauthRedirectURL = document.location.origin;
            window.location = FB_LOGIN_URL + '?client_id=' + fbAppId + '&redirect_uri=' + oauthRedirectURL + '&response_type=token&display=popup&scope=' + fbScope;
        };

        /**
         * Application-level logout: we simply discard the token.
         */
        var logout = function() {
            tokenStore['fbtoken'] = undefined;
        };

        /**
         * Helper function to de-authorize the app
         * @param success
         * @param error
         * @returns {*}
         */
        var revokePermissions = function() {
            return api({
                    method: 'DELETE',
                    path: '/me/permissions'
                })
                .success(function() {
                    console.log('Permissions revoked');
                });
        };

        /**
         * Checks the response url callback from facebook and store the access_token
         */
        var oauthCallback = function(url) {
            // Parse the OAuth data received from Facebook
            var queryString;
            var obj;
            loginProcessed = true;
            if (url.indexOf('access_token=') >= 0) {
                queryString = url.substr(url.indexOf('#') + 2);
                obj = parseQueryString(queryString);
                tokenStore['fbtoken'] = obj['access_token'];
                return true;
            } else if (url.indexOf('error=') >= 0) {
                queryString = url.substring(url.indexOf('?') + 2, url.indexOf('#'));
                obj = parseQueryString(queryString);
                return false;
            } else {
                return false;
            }
        };

        /**
         * Lets you make any Facebook Graph API request.
         * @param obj - Request configuration object. Can include:
         *  method:  HTTP method: GET, POST, etc. Optional - Default is 'GET'
         *  path:    path in the Facebook graph: /me, /me.friends, etc. - Required
         *  params:  queryString parameters as a map - Optional
         */
        var api = function(obj) {

            var method = obj.method || 'GET',
                params = obj.params || {};

            params['access_token'] = tokenStore['fbtoken'];

            return $http({
                    method: method,
                    url: 'https://graph.facebook.com' + obj.path,
                    params: params
                })
                .error(function(data, status, headers, config) {
                    if (data.error && data.error.type === 'OAuthException') {
                        $rootScope.$emit('OAuthException');
                    }
                });
        };

        /**
         * Helper function for a POST call into the Graph API
         * @param path
         * @param params
         * @returns {*}
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
         * @param path
         * @param params
         * @returns {*}
         */
        var get = function(path, params) {
            return api({
                method: 'GET',
                path: path,
                params: params
            });
        };

        /**
         * @private
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




### main.scss
```css
$fb_blue:               #3b5998 !default;
$fb_medium_blue:        #6d84b4 !default;
$fb_lighter_blue:       #afbdd4 !default;
$fb_lightest_blue:      #d8dfea !default;

ion-nav-view {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: transparent;
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

.padding {
    color:gray;
}
```

### main.js
* add ionic as a dependency
* add run

```js
.run(['$ionicPlatform', 'facebook',
        function($ionicPlatform, facebook) {
             facebook.init('323666037785577');
            $ionicPlatform.ready(function() {
                // Hide the accessory bar by default (remove this to show the accessory bar above the keyboard
                // for form inputs)
                if(window.cordova && window.cordova.plugins.Keyboard) {
                    window.cordova.plugins.Keyboard.hideKeyboardAccessoryBar(true);
                }
                if(window.StatusBar) {
                    // org.apache.cordova.statusbar required
                    window.StatusBar.styleDefault();
                }
            });

        }
    ]);

```

### common/index.js
change the routes
```js
$urlRouterProvider.otherwise('/login');
            $stateProvider
                .state('login', {
                    url: '/login',
                    template: require('./views/login.html'),
                    controller: fullaname + '.login as vm'
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
                            controller: fullaname + '.newsfeed as vm'
                        }
                    }
                })
                .state('tabs.requests', {
                    url: '/requests',
                    views: {
                        'tab-requests': {
                            template: require('./views/requests.html'),
                            controller: fullaname + '.newsfeed as vm'
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
```

and add a run fn
```js
 app.run(function($rootScope, $window, $state, facebook) {
        if (facebook.oauthCallback($window.location.href)) {
            $state.go('tabs.newsfeed');
        }
    });
```

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
            <button class="button button-block button-positive fb-button-login">Login</button>
        </div>
    </ion-content>
</ion-view>
```

### tabs.html
```html
<ion-tabs class="tabs-icon-top tabs-light fbnews fade-in-not-out">

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

### newsfeed.js
```js
'use strict';
var controllername = 'newsfeed';

module.exports = function(app) {
    /*jshint validthis: true */

    var deps = ['$rootScope', '$ionicLoading', '$state', facebook'];

    function controller($rootScope, $ionicLoading, $state, facebook) {
        var vm = this;
        //$ionicBackdrop.retain();
        $ionicLoading.show({
            template: 'Loading...'
        });
        if ($rootScope.items) {
            vm.items = $rootScope.items;
            $ionicLoading.hide();
            //$ionicBackdrop.release();
        } else {
            vm.items = [];
            var fields = {
                'fields':'full_picture,likes,description,comments,from,story,message,name,created_time'
            };
            facebook.get('/me/home', fields ).then(function(res) {
                vm.items = res.data.data;
                $rootScope.items = res.data.data;
                $ionicLoading.hide();
                //$ionicBackdrop.release();
            });
        }

        vm.logout = function() {
            facebook.logout();
            $state.go('login');
        }
    }

    controller.$inject = deps;
    app.controller(app.name + '.' + controllername, controller);
};

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

        <!-- CARD GOES HERE -->

    </ion-content>
</ion-view>

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

### requests.html
```html
<ion-view>
    <ion-header-bar class="bar bar-positive bar-fb-blue">
        <h1 class="title">Friends Requests</h1>
    </ion-header-bar>
    <ion-content>

        <ion-list ng-repeat="item in vm.items">
            <ion-item class="item-thumbnail-left">
                <img ng-src="https://graph.facebook.com/{{ item.from.id }}/picture?width=120&height=120"/>
                <h2>{{item.from.name}}</h2>
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
