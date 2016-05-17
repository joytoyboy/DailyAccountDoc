# APP开发实战第四天
  前两天在实现服务器端接口功能，APP端未进行开发，今天开始继续APP端的开发。

## 目标
 - 实现记账页、添加消费类型页

## 涉及到的知识点

 - 时间选择控件的使用。
 - 下拉选择控件的使用。
 - 照相机/图库的访问。
 - 图片上传。
 - 回退时，回退后的页面刷新。

## 记账页
 
 创建一笔记账，可以是支出、出入记录等。

### 功能分析
 - 日期默认为当前日期
 - 点击日期，弹出日期选择器，进行日期修改
 - 类型和支出方式都为下拉框
 - 当账目类型变化时，分类也相应的进行变化。如选择了“收入”，则分类显示“工资”等分类；选择了“支出”，则显示“衣”等消费分类。
 - 点击分类右侧的“添加”按钮，进入创建分类页。
 - 点击底部“添加”按钮，添加一笔收入。
 - 添加成功后，并不返回前一页，而是清空表单，可以继续添加。
 - 回退时，流水页数据刷新。

### 编码
 1. 在home-flow.html页的标题栏添加“添加”按钮：
    ```
    <ion-view view-title="流水">
        <ion-nav-buttons side="right">
          <button class="button button-positive right"  ui-sref="addAccount">
            添加
          </button>
        </ion-nav-buttons>
    ...
    ```、
 1. 增加addAccount页：
    ```
    .state('addAccount', {
      url: '/addAccount.html',
      templateUrl: './page/addaccount.html',
      controller: 'AddAccountCtrl'
    })
    ```
    在这里直接引入了`AddAccountCtrl`这个controller，这样，就无需要html文件中再引入了。

 1. 在AppConfig中增加账目类型访问函数。
    ```
    // 获取账目类型
    getAccountType: function () {
      return [
        { id: 1,
          name: '支出'
        },
        { id: 2,
          name: '收入'
        },
        { id: 3,
          name: '取现'
        },
        { id: 4,
          name: '存现'
        },
        { id: 5,
          name: '借款'
        },
        { id: 6,
          name: '还借款'
        },
        { id: 7,
          name: '放款'
        },
        { id: 8,
          name: '放款收回'
        }
        ]
    }

    // 获取支付类型
    ,getMoneyMode: function () {
      return [
        { id: 1,
          name: '现金'
        }
        ,{ id: 2,
          name: '借记卡'
        }
        ,{ id: 3,
          name: '信用卡'
        }
        ,{ id: 4,
          name: '其它(如支付宝、油卡等）'
        }
      ]
    }
    ```
    在app.js的AppCtrl中，获取此值。
    ```
    $rootScope.accountType = AppConfig.getAccountType();
    $rootScope.moneyMode = AppConfig.getMoneyMode();
    ```

 1. 新建ctrl_addaccount.js文件，创建AddAccountCtrl控制器。
    初始化时，创建账目模型：
    ```
    $scope.reset = function (form) {
      $scope.accountData = {
        type: $rootScope.accountType[1],
        time: new Date(),
        desc: '',
        moneyMode: $rootScope.moneyMode[0],
        use: '',
        amount: ''
      };
      resetForm(form);
    };
    $scope.reset(null);
    ```
    将其引入到app.js和index.html中。
 1. 新建addaccount.html文件：
    ```
    <ion-view view-title="添加账目">
      <ion-content>
        <form role="form" name="addForm" ng-submit="addAccount(addForm)" novalidate >
          <div class="list">
            <label class="item item-input">
              <span class="input-label">日期</span>
              <input type="date" placeholder="输入日期" name="time" required ng-model="accountData.time">
            </label>

            <label class="item item-input item-select">
              <div class="input-label">
                账目类型
              </div>
              <select class="form-control" name="type" ng-model="accountData.type" ng-options="x.name for x in accountType track by x.id">
              </select>
            </label>

            <label class="item item-input">
              <span class="input-label">描述</span>
              <input type="text" placeholder="输入描述" name="desc" required ng-model="accountData.desc">
            </label>
            <div class="error app-form-error"
                 ng-show="addForm.desc.$dirty && addForm.desc.$invalid">
              <small class="error" ng-show="addForm.desc.$error.required">
                请输入描述
              </small>
            </div>

            <div class="item item-input">
              <label class="item-input-wrapper item-select app-item-input-wrapper">
                <span class="input-label">分类</span>
                <select name="use" ng-model="accountData.use" ng-options="x.name for x in useArray track by x.id">
                </select>
              </label>
              <button class="button button-outline button-small">添加</button>
            </div>

            <label class="item item-input">
              <span class="input-label">金额</span>
              <input type="number" placeholder="输入金额" name="amount" required check-money ng-model="accountData.amount">
            </label>
            <div class="error app-form-error"
                 ng-show="addForm.amount.$dirty && addForm.amount.$invalid">
              <small class="error" ng-show="addForm.amount.$error.required">
                金额不能为空。
              </small>
              <small class="error" ng-show="addForm.amount.$error.checkMoney">
                {{moneyPrompt}}
              </small>
            </div>

            <label class="item item-input item-select">
              <div class="input-label">
                支付方式
              </div>
              <select class="form-control" name="moneyMode" ng-model="accountData.moneyMode" ng-options="x.name for x in moneyMode track by x.id">
              </select>
            </label>
          </div>

          <div class="padding">
            <!--禁用： ng-disabled="addForm.$invalid"-->
            <button type="submit" class="button button-block button-positive" >添加</button>
          </div>
        </form>

      </ion-content>
    </ion-view>
    ```

    `addAccount`函数定义如下：

    ```
    // 添加处理。
    $scope.addAccount = function (form) {
      if (!form.$valid) {
        $rootScope.showAlert('请输入正确的参数。');
        return;
      }

      var accountData = angular.copy($scope.accountData);
      if (accountData.type == null){
        $rootScope.showAlert('请选择正确的账目类型。');
        return;
      }

      if (accountData.moneyMode == null){
        $rootScope.showAlert('请选择正确的支付方式。');
        return;
      }

      accountData.type = accountData.type.id;
      accountData.moneyMode = accountData.moneyMode.id;
      accountData.time = timeStamp(accountData.time);
      console.log(accountData);

      $scope.config = {headers: $rootScope.httpHeaders};
      $scope.url = AppConst.URL + AppConst.ADD_ACCOUNT;
      $http.post($scope.url, accountData, $scope.config)
        .success(function (response) {
          if (response.code != AppConst.CODE_OK){
            $rootScope.showAlert(response.message);
          }
          else{
            $scope.reset(form);
          }
        })
        .error(function (data) {
          //错误代码
          $rootScope.showAlert(data);
        });
    }

    ```
 1. 其中使用了自定义的验证指令：moneyMode，用于验证金额的正确性：

    在app.js中定义指令。
    ```
    // 自定义金额检测指令。
    .directive('checkMoney', [function () {
      return {
        require: "ngModel",
        link: verifyMoney
      };
    }])

    ```

    verifyMoney函数定义如下：
    ```
    function verifyMoney(scope, element, attr, ngModel) {
      scope.moneyPrompt = "金额必须大于等于0，且最多两位小数";
      if (ngModel) {
        // 前面1-9位数，小数点后最多两位
        var regex = /^([1-9][\d]{0,7}|0)(\.[\d]{1,2})?$/;
      }
      var validator = function (value) {
        var validity = ngModel.$isEmpty(value) || regex.test(value);
        ngModel.$setValidity("checkMoney", validity);
        return validity ? value : undefined;
      };
      ngModel.$formatters.push(validator);
      ngModel.$parsers.push(validator);
    }
    ```
 1. 接下来还要从服务器获取分类数组，对于每种账目类型，都有不同的分类。

    从服务器获取来的分类，保存到本地存储。
