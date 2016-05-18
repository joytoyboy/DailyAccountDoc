# APP开发实战第二天

## 目标
 - 完善登录页、实现注册页、首页中的流水Tab页部分。
 - server的搭建、数据请求接口实现。（不在此介绍）

## 涉及到的知识点

 - 如何向网络提交和请求数据。
 - Tab页的使用。
 - 列表框的使用。

## 登录页面  
### 编码  
 继续昨天，完成昨天未完成的部分。
 1. 实现http请求 
   目前为止，表单的本地验证部分处理完毕。现在实现http请求。http请求需要实现服务，其内容不在此部分介绍。
   请求接口参见：[与服务的API接口设计](./app_api.md)
    关于http请求的更多信息参见：https://docs.angularjs.org/api/ng/service/$http#post

    - 获取header数据
    每个请求接口都需要传递header数据，我们在AppConfig中来设置要传递的header数据。

    - 发送请求：
    在$scope.login函数中，增加：

    ```
    $scope.login = function (valid) {
      console.log($scope.loginData);
      if (!valid) {
        $rootScope.showAlert('请输入正确的参数。');
        return;
      }

      $scope.config = {headers: $rootScope.httpHeaders};
      $scope.url = AppConst.URL + AppConst.LOGIN;
      $http.post($scope.url, $scope.loginData, $scope.config)
        .success(function (response) {
          if (response.code != AppConst.CODE_OK){
            $rootScope.showAlert(response.message);
          }
        })
        .error(function (data) {
          //错误代码
          $rootScope.showAlert(data);
        });
    }

    ```

    在浏览器中预览时，会出现跨域请求错误，在app安装到手机上后，此问题不存在。下面说明如何解决这个问题。

 1. 解决跨域请求的问题  

    参考http://ionichina.com/topic/54f051698cbbaa7a56a49f98

    1. ionic.project修改：

       ```
        {
        "name": "DailyAccount",
        "app_id": "",
        "proxies": [
              {
                "path": "/admin/if/",
                "proxyUrl": "http://mmb.qsc365.com/admin/if/"
              }
            ]
        }

       ```
       其中，proxies为增加的内容。

    1. 在app.js中，对`angular.module('starter', ['ionic','app.appConfig','app.splashCtrl','app.wizardCtrl','app.loginCtrl'])`增加：
       ```
       angular.module('starter', ['ionic','app.appConfig','app.splashCtrl','app.wizardCtrl','app.loginCtrl'])
       .constant('ApiEndpoint', {
       url: 'http://localhost:8100/admin/if/'
       })
       ```
       **提示**：当以`ionic serve`在浏览器中启动时，占用的端口号为8100
       
    1. 安装replace模块

       ```
       npm install --save replace
       ```

    1. 打开gulpfile.js文件

       增加：

       ```
          // 原来内容：
          var sh = require('shelljs');
          // 追加内容：
          var replace = require('replace');
          var replaceFiles = ['./www/js/app.js'];

          ...

          // 文件尾部：

          gulp.task('add-proxy', function() {
            return replace({
              regex: "http://mmb.qsc365.com/admin/if",
              replacement: "http://localhost:8100/admin/if",
              paths: replaceFiles,
              recursive: false,
              silent: false,
            });
            });

            gulp.task('remove-proxy', function() {
            return replace({
              regex: "http://localhost:8100/admin/if",
              replacement: "http://mmb.qsc365.com/admin/if",
              paths: replaceFiles,
              recursive: false,
              silent: false,
            });
          });
       ```

 1. 解决后台收不到请求数据的问题
    虽然现在接口能够调用了，但是测试发现，后台并未收到$scope.loginData中的数据。得需要做一些设置。
    - 创建utils文件util_http.js
      ```
      function httpTransform(httpProvider) {
          httpProvider.defaults.headers.post['X-Requested-With'] = 'XMLHttpRequest';
          httpProvider.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded;charset=utf-8';

          /**
           * 重写angular的param方法，使angular使用jquery一样的数据序列化方式  The workhorse; converts an object to x-www-form-urlencoded serialization.
           * @param {Object} obj
           * @return {String}
           */
          var param = function (obj) {
            var query = '', name, value, fullSubName, subName, subValue, innerObj, i;

            for (name in obj) {
              value = obj[name];

              if (value instanceof Array) {
                for (i = 0; i < value.length; ++i) {
                  subValue = value[i];
                  fullSubName = name + '[' + i + ']';
                  innerObj = {};
                  innerObj[fullSubName] = subValue;
                  query += param(innerObj) + '&';
                }
              }
              else if (value instanceof Object) {
                for (subName in value) {
                  subValue = value[subName];
                  fullSubName = name + '[' + subName + ']';
                  innerObj = {};
                  innerObj[fullSubName] = subValue;
                  query += param(innerObj) + '&';
                }
              }
              else if (value !== undefined && value !== null)
                query += encodeURIComponent(name) + '=' + encodeURIComponent(value) + '&';
            }

            return query.length ? query.substr(0, query.length - 1) : query;
          };

          // Override $http service's default transformRequest
          httpProvider.defaults.transformRequest = [function (data) {
            return angular.isObject(data) && String(data) !== '[object File]' ? param(data) : data;
          }];
        }

      ```
    - 在index.html中引入此js文件

    - 在app.js中进行配置
      ```
        .config(function ($stateProvider, $urlRouterProvider,$httpProvider) {
            httpTransform($httpProvider);
        }
      ```

     OK，现在运行`ionic serve`就可以正常请求了，因为现在还没有用户，因此会提示用户不存在，下面实现注册页面，注册页面完成后，就可以创建用户并登录了。


## 注册页
 
### 功能分析

  注册页实现简单的注册功能。

  - 输入用户名、密码、昵称选填。
  - 用户名和密码的验证同login页。昵称为0-20字符。
  - 创建成功后，返回登录页，在登录页点击回退按钮时，不再进行此页。

### 编码

  1. 在page页中创建一个reg.html文件。  
     内容如下：

     ```
     ```

     > 因为checkPwd指令可以与login页共用，因此将其从ctrl_login.js移到app.js中。

  1. 新问题：如何验证两次密码是否相同。

     - 创新新的指令：

     ```
      .directive('checkPwdMatch', [function () {
        return {
          restrict: 'A',
          require: "ngModel",
          link: function (scope, element, attr, ngModel) {
            var pwdValidator = function (value) {
              var otherInput = element.inheritedData("$formController")[attr.checkPwdMatch];
              var validity = !ngModel.$isEmpty(value) && value == otherInput.$viewValue;
              // console.log(otherInput.$viewValue + value + validity);
              ngModel.$setValidity("checkPwdMatch", validity);
              return validity ? value : undefined;
            };
            ngModel.$formatters.push(pwdValidator);
            ngModel.$parsers.push(pwdValidator);
          }
        };
      }])
     ```
     html中如下设置：
     ```
       <label class="item item-input">
          <input type="password" placeholder="输入密码" name="pwd" required ng-minlength="6" ng-maxlength="20" check-pwd ng-model="regData.pwd">
        </label>
        <div class="error app-form-error"
             ng-show="regForm.pwd.$dirty && regForm.pwd.$invalid">
          <small class="error" ng-show="regForm.pwd.$error.required">
            请输入密码
          </small>
          <small class="error" ng-show="regForm.pwd.$error.minlength">
            不能少于6字符
          </small>
          <small class="error" ng-show="regForm.pwd.$error.maxlength">
            不能超过20字符
          </small>
          <small class="error" ng-show="regForm.pwd.$error.checkPwd">
            密码只能为字母和数组的组合
          </small>
        </div>

        <label class="item item-input">
          <input type="password" placeholder="再输一遍密码" name="pwd2" required check-pwd-match="pwd" ng-model="regData.pwd2">
        </label>
        <div class="error app-form-error"
             ng-show="regForm.pwd2.$dirty && regForm.pwd2.$invalid">
          <small class="error" ng-show="regForm.pwd2.$error.required">
            请输入密码
          </small>
          <small class="error" ng-show="regForm.pwd2.$error.checkPwdMatch">
            两次输入的密码不一致
          </small>
        </div>

     ```
   通过对pwd2设置check-pwd-match="pwd", "pwd"为前一个密码input的name。来指定在directive('checkPwdMatch')函数中，通过`var otherInput = element.inheritedData("$formController")[attr.checkPwdMatch];`获取到的input。两者的值进行比较。

   - 待解决问题：
     - 先输入pwd2，再输入pwd，并不会进行两者数据的验证，如何在输入pwd时，pwd2也进行验证？
     - 如何清空表单？
      直接将input绑定的model重置即可。如：$scope.regData.name = $scope.regData.pwd = $scope.regData.pwd2 = $scope.regData.nickname = '';
      （但这样修改后错误验证的提示未消除？）

# APP开发实战第三天

## 目标
   - 首页中的流水Tab页部分。

  由于http请求遇到的坑比较多，未能完成首页的开发。今天完成首页的开发。

## 涉及到的知识点
   - Tab页的使用。
   - 列表框的使用。

## 登录页面  
### 编码  
 继续昨天，完成昨天未完成的部分。用户登录后，会回token，我们需要将用户名和token保存起来，每次请求时使用。下次再登录时，将用户名从本地读取出来，显示在登录页。
 
 1. 保存用户信息
    我们在已有的AppConfig factory中保存用户，增加userName和token字段：
    ```
      return {
        firstRun: true,
        userName: '',
        token: '',
      };

    ```
  
    在LoginCtrl中，引入AppConfig:
      ```
      .controller('LoginCtrl', ['$scope', '$rootScope', '$state','$stateParams','$ionicHistory', '$http', 'AppConfig',
        'AppConst',
        function($scope, $rootScope, $state,$stateParams,$ionicHistory,$http,AppConfig, AppConst) {

      ```
  
    $scope.login函数中，保存用户信息。
      ```
      $http.post($scope.url, $scope.loginData, $scope.config)
                .success(function (response) {
                  if (response.code != AppConst.CODE_OK){
                    $rootScope.showAlert(response.message);
                  }
                  else{
                    // 将用户名保存起来，下次登录显示。
                    if (response.data.name){
                      $rootScope.appConfig.userName = response.data.name;
                      $rootScope.appConfig.token = response.data.token;
                      AppConfig.saveConfig($rootScope.appConfig);
                    }
                  }
                })
      ```

    读取用户信息，设置给页面：
  
      ```
        // 表单数据
        $scope.loginData = {
          name: $rootScope.appConfig.userName, // 用户名
          pwd: ''   // 密码
        };
      ```
  
    在调用时，会出错：
    > Cannot read property 'userName' of undefined
    将读取$rootScope.appConfig的代码从ctrl_splash.js中移到app.js中的AppCtrl中。
      
    ```
    .controller("AppCtrl", ['$scope', '$rootScope', "AppConfig", '$state', '$ionicNavBarDelegate', '$http','$ionicPopup',
    function ($scope, $rootScope, AppConfig, $state, $ionicNavBarDelegate, $http,$ionicPopup) {
    $rootScope.appConfig = AppConfig.getConfig();
    ...
    ```
        
 1. 表单重置时，error信息不会重置。
  在通过将model数据重置，表单中的元素会自动重置，但是表单的error提示不会重置。怎么样才能像页面初次进入时的样子呢？
  网上也有相同的提问：
  http://stackoverflow.com/questions/26015010/angularjs-form-reset-error
  
  ```
    // reset dirty state
    $scope.loginForm.$setPristine();
    // reset to clear validation
    $scope.loginForm.$setValidity();
    $scope.loginForm.$setUntouched();
  ```  
  
  网上查到，在angular中，表单是FormController的一个实例。表单实例可以随意地使用name属性暴露到scope中。但我却找不到$scope.xxxForm的定义。网上也有同样的疑问：
  http://stackoverflow.com/questions/22436501/simple-angularjs-form-is-undefined-in-scope
  从网上没找到比较好的解决方法，我的方法是：在表单提交函数中传入form名：
  
  ```
  $scope.login = function (loginForm){
    loginForm是有效值，可对其进行操作。
  ...}
  
  ```
  

## Home页面
 
 Home页面是一个Tab页，由三部分组成，Header、Content和Tab Bar。Tab Bar有三个Tab图标，分别为流水、统计和我的，点击时进入相面的Tab页。

### 功能分析

  - 默认选中流水Tab页。
  - 进入此页后，清除历史记录，不可再回退。
  - 点击TabItem，可以切换Tab页。

### 编码
 1. 在page文件夹下创建home.html，代码如下：
    ```
    <ion-tabs ng-controller="HomeCtrl" class="tabs-icon-top tabs-color-active-positive">
    
      <ion-tab title="流水" icon="ion-arrow-swap" href="#/home/flow">
        <ion-nav-view name="home-flow"></ion-nav-view>
      </ion-tab>
    
      <ion-tab title="统计" icon="ion-pie-graph" href="#/home/stat">
        <ion-nav-view name="home-stat"></ion-nav-view>
      </ion-tab>
    
      <ion-tab title="我的" icon="ion-person" href="#/home/mine">
        <ion-nav-view name="home-mine"></ion-nav-view>
      </ion-tab>
    </ion-tabs>
    ```
 1. 在page文件夹下，创建home-flow.html、home-stat.html和home-mine.html，
  内容先分别写入"flow"、"stat"、"mine"即可。
  
 1. 在app.js的config中，加入home页面：
    ```
    .state("home", {
      'url' : '/home',
      templateUrl: "./page/home.html",
      abstract: true,
      // 这里定义进入此页后是否将前面的页面清掉，清掉后按回退键就不能进入前一页了。
      params: {'clearHistory': false}
    })
    .state('home.flow', {
      // **坑**：这里不能写成`/home/flow`，因为继承关系，会自动转换为`/home/flow`。
      url: '/flow',
      views: {
        'home-flow': {
          templateUrl: './page/home-flow.html',
          controller: 'HomeFlowCtrl'
        }
      }
    })
    .state('home.stat', {
      url: '/stat',
      views: {
        'home-stat': {
          templateUrl: './page/home-stat.html',
          controller: 'HomeStatCtrl'
        }
      }
    })
    .state('home.mine', {
      url: '/mine',
      views: {
        'home-mine': {
          templateUrl: './page/home-mine.html',
          controller: 'HomeMineCtrl'
        }
      }
    })

    ```
 1. 在js文件夹下创建相应的Controller文件：
 
    ctrl_home.js
  
    ```
    angular.module('app.homeCtrl', [])
      .controller('HomeCtrl', ["$scope", '$rootScope', "AppConfig", "$state",
                           function($scope, $rootScope, AppConfig, $state) {
      }
      ])
    ;
    ```
 
    ctrl_home_flow.js
  
    ```
    angular.module('app.homeFlowCtrl', [])
      .controller('HomeFlowCtrl', ["$scope", '$rootScope', "AppConfig", "$state",
                           function($scope, $rootScope, AppConfig, $state) {
      }
      ])
    ;
    ```
 
    ctrl_home_stat.js
  
    ```
    angular.module('app.homeStatCtrl', [])
      .controller('HomeStatCtrl', ["$scope", '$rootScope', "AppConfig", "$state",
                           function($scope, $rootScope, AppConfig, $state) {
      }
      ])
    ;
    ```
 
    ctrl_home_mine.js
  
    ```
    angular.module('app.homeMineCtrl', [])
      .controller('HomeMineCtrl', ["$scope", '$rootScope', "AppConfig", "$state",
                           function($scope, $rootScope, AppConfig, $state) {
      }
      ])
    ;
    ```
  
 1. 在app.js和index.html中引入这些js文件。

 1. 新建fact_flow.js文件，创建FlowFactory:
  
    ```
    angular.module('app.flow', [])
    .factory('FlowFact', function(){
      return {
        // 从服务器获取流水项。
        requestItems: function(http, from, count) {
          return [
            { img: 'img/ic_out.png',
              title: '泡妞',
              moneyType: '现金',
              price: '400元',
              type: '社交',
            }
            , { img: 'img/ic_in.png',
              title: '发工资',
              moneyType: '银行卡',
              price: '5000元',
              type: '工资',
            }
            , { img: 'img/ic_in.png',
              title: '发工资',
              moneyType: '银行卡',
              price: '5000元',
              type: '工资',
            }
            , { img: 'img/ic_in.png',
              title: '发工资',
              moneyType: '银行卡',
              price: '5000元',
              type: '工资',
            }
            , { img: 'img/ic_in.png',
              title: '发工资',
              moneyType: '银行卡',
              price: '5000元',
              type: '工资',
            }
          ];
        },
      }
    })
    ;

    ```
    这里先写死，以方便看效果，之后实现从http获取数据。
  
 1. 在app.js和index.html中引入此js文件。

 1. 修改home-flow.html
  
    ```
    <ion-view view-title="流水">
        <ion-content class="padding">
          <ion-list >
            <ion-item ng-repeat="item in flowItems"
                      class="item-avatar-left">
              <img ng-src="{{item.img}}" class="app-clear-radius">
              <h2>{{item.title}}</h2>
              <div>
                <span class="app-left app-tag">
                  {{item.moneyType}}
                </span>
                <span class="app-item-price">{{item.price}}</span>
                <span class="app-right app-tag {{flowItemTypeMap[item.type].style}}">{{item.type}}</span>
              </div>
              <ion-option-button class="button-positive"
                                 ng-click="share(item)">
                Share
              </ion-option-button>
              <ion-option-button class="button-info"
                                 ng-click="edit(item)">
                Edit
              </ion-option-button>
              <ion-delete-button class="ion-minus-circled"
                                 ng-click="flowItems.splice($index, 1)">
              </ion-delete-button>
              <ion-reorder-button class="ion-navicon"
                                  on-reorder="reorderItem(item, $fromIndex, $toIndex)">
              </ion-reorder-button>

            </ion-item>
          </ion-list>
        </ion-content>
    </ion-view>

    ```
  
    效果如下：
    ![](./img/home-flow.png)
  
 1. 在标题栏添加“记账”按钮，点击此按钮，可以进入记账页。
