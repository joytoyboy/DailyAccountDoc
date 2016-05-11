# APP开发实战第一天

## 目标
 - 实现Splash页、Wizard页、登录页。

## 涉及到的知识点

 - 如何创建一个页面。这里的页面的概念对应手机上的一屏，如启动页算一个页面，带Tab栏的页面，每个Tab页也算做一个页面。
 - 如何使用定时设置。如splash页启动3秒后自动跳转到下一页。
 - 页面之间如何跳转。
 - 如何制作图像划屏效果。像wizard页面，几个图片划动翻换图片。
 - 如何向网络提交和请求数据。
 - 如何访问本地数据。如将app配置保存到手机中，从手机中读取配置数据。

## Splash页
 
 ionic自带了splash页，但启动效果非常差，有时不显示，即使显示，也总是黑屏，体验非常不好，这里不去查其中的原因，我们使用自己的页面代替它。
 我们Splash页比较简单，由上下两个图片组成，上面是logo图片、下面是标语图片，效果如下:
  ![](./img/splash.png)

  - 整体大小: 800 x 1280
  - 背景色: #ff6000
  - 文字色: #fff

### 功能分析

  这是APP的启动页，此页的功能如下：

  - 显示3秒后自动淡出，显示下一页。
  - 读取app配置，查看是否为第一次启动，如果是，则下一页为wizard页；否则，进入登录页（在没有home页前先这样做，之后再进行修改）。
  - 如果为第一次启动，将第一次启动信息写入本地，这样下次就不用进入wizard页了。

### 编码

  1. 在www目录下创建一个page文件夹，用于存放我们的页面。
  1. 在page页中创建一个splash.html文件。

    ```
    <!--启动页， 不显示标题栏，控制器为SplashCtrl，稍后定义-->
    <ion-view hide-nav-bar="true" ng-controller="SplashCtrl">
      <div class="app-splash-logo">
        <div class="flexbox"><div>
          <image src="img/splash/logo.png" />
        </div></div>
      </div>
      <div class="app-splash-slogan">
        <image src="img/splash/slogan.png" />
      </div>
    </ion-view>
    ```

    其样式的实现参见style.css。

  1. 将splash.html文件加到index.html中。
    
    打开app.js，对.config(function...配置如下：

    ```
    .config(function ($stateProvider, $urlRouterProvider) {
    $stateProvider
      // splash页面
      .state("splash", {
        templateUrl: "./page/splash.html"
      })
      // wizard页面（稍后定义）
      .state("wizard", {
        templateUrl: "./page/wizard.html"
      })
      // login页面（稍后定义）
      .state("login", {
        templateUrl: "./page/login.html"
      })
      ;

  }).controller("AppCtrl", function ($scope, $state, $ionicNavBarDelegate) {
    // 缺省进入splash页面。
    $state.go('splash');
  }

  ```

  1. 创建一个factory用于存取app本地数据。

    在js文件夹下，创建app_config.js文件，内容如下：  

  ```
  angular.module('app.appConfig', [])
  .factory('AppConfig', function(){
    return {
      // 获取App配置数据
      getConfig: function() {
        var appConfig = window.localStorage['appConfig'];
        if(appConfig) {
          return angular.fromJson(appConfig);
        }
        return return {
          firstRun: true
        };
      },
      // 保存App配置数据
      saveConfig: function(appConfig) {
        window.localStorage['appConfig'] = angular.toJson(appConfig);
      },
    }
  });

  ```   

  在app.js中，引入此factory：

  ```
  // angular.module('starter', ['ionic'])
  // 修改后：
  angular.module('starter', ['ionic','app.appConfig'])

  ```

  在index.html中，新增：

  ```
  /** 原有 */
  <script src="js/app.js"></script>
  /** 新增 */
  <script src="js/app_config.js"></script>

  ```

  1. 在page中增加一个wizard.html和login.html文件，内容暂写"wizard"和"login"即可。

  1. 创建Splash的Controller  
    在js中创建ctrl_splash.js文件，内容如下：
    ```
  angular.module('app.splashCtrl', [])
    .controller('SplashCtrl', ["$scope", "AppConfig", "$timeout", "$state", 
                      function($scope, AppConfig, $timeout, $state) {
      $scope.appConfig = AppConfig.getConfig();

      $scope.getFirstRun = function () {
        if (null == $scope.appConfig){
          return true;
        }

        return $scope.appConfig.firstRun;
      };

      $scope.isFirstRun = $scope.getFirstRun();

      // 将firstRun改为false，保存到本地。
      $scope.appConfig.firstRun = false;
      AppConfig.saveConfig($scope.appConfig);

      // 3秒钟后跳转到wizard页
      $timeout(function(){
        // 如果第一次运行跳转至wizard页，否则跳转至login页。
        $state.go($scope.isFirstRun ? 'wizard' : 'login');
      }, 3000);
    }
    ])
  ;

    ```

    参照上面引入app_config.js的方法，在app.js和index.html中引入。

    然后在splash.html中引入：
    ```
    <ion-view hide-nav-bar="true" ng-controller="SplashCtrl">

    ```

    至此，splash页加处理完毕。现在可以运行看一下效果。
    第一次运行：   
    ![](./img/splash01.gif)

    开始的白屏暂且不管，以后进行优化处理。

    再次运行：   
    ![](./img/splash02.gif)

    - 待解决问题：
     1. 出现splash前白屏且有标题栏
     1. splash与下一页切换不是淡入淡出
     1. 按下返回键splash会重新出现

     下面先解决按下返回键splash页会重新出现的问题。

  1. 禁止下一页后点击回退按钮返回此页  
   需要在下一页来处理。比如进入wizard页，则需要对Wizard页进行如下配置：
   
   ```
    .state("wizard", {
      url: "/wizard.html",
      templateUrl: "./page/wizard.html"
      // 这里定义进入此页后是否将前面的页面清掉，清掉后按回退键就不能进入前一页了。clearHistory为自定义参数。
      params: {'clearHistory': false}
    })

   ```

   进入此页时，需要如下处理：
   ```
   // 之前
   // $state.go('wizard');
   // 现在
   $state.go('wizard', {
     clearHistory: true
   });

   ```

   然后在Wizard的Controller中：  

   ```
  .controller('WizardCtrl', ["$scope", "$state",'$stateParams','$ionicHistory',
    function($scope, $state,$stateParams,$ionicHistory) {
    ...
    if ($stateParams != null && $stateParams.clearHistory){
      $ionicHistory.clearHistory();
    }
  ```
  这样，在真机运行时，按回退键就不会返回上一页了。在浏览器预览中点击回退按钮仍然会回退，不用管。



## Wizard页
 
 Wizard页比较简单，实现一个三屏的滑动图片，最后一个图片带有一个"点击进入"按钮，点击后，进入login页。
 图片滑动使用ion-slides来实现。

### 功能分析

  - 左滑显示下一张图片
  - 最后一张图片显示“点击进入”按钮。
  - 点击“点击进入”按钮，进入login页。

### 编码

 1. 修改wizard.html中的代码：

 ```
<ion-view  ng-controller="WizardCtrl" hide-nav-bar="true">
  <ion-slides options="slideOptions" slider="slideData.slider">
    <ion-slide-page>
      <div class="box app-wizardimg">
        <img src="img/wizard/wizard1.png">
      </div>
    </ion-slide-page>
    <ion-slide-page>
      <div class="box app-wizardimg">
        <img src="img/wizard/wizard2.png">
      </div>
    </ion-slide-page>
    <ion-slide-page>
      <div class="box app-wizardimg">
        <img src="img/wizard/wizard3.png">
      </div>
      <div class="app-centerdiv app-bottom30">
        <a class="button button-positive app-btnGoHome" ui-sref="login">立即进入</a>
      </div>
    </ion-slide-page>
  </ion-slides>
</ion-view>

 ```
 1. 创建ctrl_wizard.js

 ```
angular.module('app.wizardCtrl', [])
  .controller('WizardCtrl', ["$scope", "$state", function($scope, $state) {
    $scope.slideOptions = {
      loop: false,
      speed: 500
    }

    $scope.slideData = {};
    $scope.$watch('slideData.slider', function(nv, ov) {
      $scope.slider = $scope.slideData.slider;
    });
  }
  ])
;

 ``` 

 将ctrl_wizard.js引入到app.js和index.html中。


## 登录页面

接下来处理登录页面。通过输入用户名和密码来登录。
 - 用户名：2-20位字符，允许汉字，名称唯一。
 - 密码：6至20位字母和数字组合，区分大小写。

### 功能描述

 - 点击“登录”按钮，先进行本地输入检查，当不正确时，在input下方显示错误提示。
  - 用户名：不能为空；长度在2-20位字符之间。
  - 密码：不能为空；长度在6-20位之间；只为能数字、小写英文字母、大写英文字母。
 - 本地验证通过后，请求http，如果用户名密码正确，跳转至home页；否则，弹出错误提示框。

 - 点击“注册”按钮，跳转至注册页。
 
### 技术点分析
 - 表单验证
 - 提示对话框使用
 - http post请求及结果分析

### 编码

 1. 修改login.html页，内容如下：