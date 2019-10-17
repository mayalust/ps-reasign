### 前端交接文档整理
##### 主要目录介绍
- app-oc - 主要是运营中心的内容
- app-sc - 主要是服务中心的内容
- app-freeboard - 主要是仪表板的内容
- app-free-style - 主要是仪表板预览的内容
- js - 主要是仪表板的通用配置js文件
- less - 主要是项目公用的less源文件，之后后通过gulp转化为css
##### app-oc
- index.html 入口的html文件
```html
<html>
<html>
  <head>
    …
    <!-- 所有静态css文件的加载都放在这里 -->
    <!-- 通常js文件的加载不放到这里，而是通过requirejs进行加载-->
  </head>
  <body>
    <!-- 这里是全局控制器的控制器对应的是 js/controllers/appnav-ctrl.js -->:
    <div class="wrapper" ng-controller="AppNavCtrl">
      ...
      <!-- angularRouter路由控制器渲染模版的位置 -->
      <div class="content-wrapper" ng-view></div>
      <!-- uiRouter路由控制器渲染路由模版的位置 -->
      <div class="content-wrapper" ui-view></div>
    </div> 
    <script src="../node_modules/ps-require/index.js" data-main="js/main">
    </script>
    <script src="../node_modules/requirejs/require.js" data-main="js/main">
    </script>
  </body>
</html>
```
##### appnav-ctrl.js, appnav-baogang-ctrl.js 全局控制器
``` 
appnav-ctrl.js是app-oc对应的全局控制器，appnav-baogang-ctrl是app-sc对应的全局控制器，两者的区别是缓存方式不同，服务中心采取了异步缓存的方式，后面以appnav-baogang-ctrl来做范例。
```

##### appnav-baogang-ctrl.js 内容详解
###### 1，初始化baseConfig
```javascript
Info.get("../localdb/info.json",
function(info) {
  ...
  //获取基本配置信息放到scope.baseConfig变量中
});
```
###### 2,校验登录信息，初始化菜单内容（运营中心，服务中心）
```javascript
$scope.$on("loginStatusChanged", function (evt, d) {
  ...
  //登录之后将用户的信息资料压入$scope.userInfo对象备用；
  
  userLoginUIService.initMens($scope, $location);
  //通过用户登录信息中的权限信息构建运营中心菜单（详情请看userLoginUIService.js文件内容详解);

  $rootScope.cacheAllData = new CacheAllData();
  //开启前端数据缓存
  $rootScope.cacheAllData.init(function() {
    console.log("全部数据缓存完成 - 用时 : " + (new Date() - time) / 1000 + "s");
  });
})
```
###### 3,cacheAllData内容详解
```javascript
function CacheAllData() {
  this.events = {}; //事件观察对象
  this.promises = {}; //结果预期对象
}
CacheAllData.prototype.on = function (eventname, handler) {
  this.promises[eventname]().then(handler);
  // 注册事件的回掉函数
}
CacheAllData.prototype.init = function (callback) {
  // 缓存真正开始时候执行的方法
  this.cacheOneByOne[...];
  // 逐一缓存各部分内容
}
CacheAllData.prototype.cacheOneByOne = function cacheOneByOne(queue, callback) {
  // 产生执行队列逐一缓存内容
}
CacheAllData.prototype.getAllDevice = function getAllDevice(callback) {
  //缓存所有设备
}
CacheAllData.prototype.getAllDicts = function getAllDicts(callback) {
  // 缓存所有的字典表
}
CacheAllData.prototype.getAllUserInfo = function getAllUserInfo(callback){
  // 缓存用户信息
}
CacheAllData.prototype.getAllDataTypes = function getAllDataTypes(callback){
  // 缓存全部数据类型信息完成
}
CacheAllData.prototype.getAllUnits = function getAllUnits(callback) {
  // 缓存全部单位字典表完成
}
CacheAllData.prototype.getAllAttrs = function getAllAttrs(callback) {
  // 缓存全部设备属性完成
}
CacheAllData.prototype.getAllDirectives = function getAllDirectives(callback){
  // 缓存全部指令完成
}
CacheAllData.prototype.getDefaultKpis = function getDefaultKpis(callback){
  // 缓存全部默认指标类型完成
}
CacheAllData.prototype.getAllModelDeviceKpiDef = function getAllModelDeviceKpiDef(callback){
  // 缓存全部模型、设备、指标完成
}
CacheAllData.prototype.domainTreeQuery = function domainTreeQuery(callback) {
  // 缓存管理域名（兼容性保留）
}
CacheAllData.prototype.getEnterpriseConfigs = function getEnterpriseConfigs(callback) {
  // 缓存基础配置信息完成
}
```
   ##### user-login-ui-service.js 内容详解
   ```
user-login-ui-service.js 产生实例化的类userLoginUIService,主要作用为：1）校验登录信息，2）获得登录用户角色信息，3）获得和初始化角色菜单内容
```
###### 1,校验登录信息，获取登录用户信息
```javascript
factory.getCurrentUser = function(callback) {
  //校验登录信息
  serviceProxy.get(loginService, 'getCurrentUser', null, function (result) {
    //getCurrentUser实际调用的API
    factory.user.usercurrentRoleID = factory.user.currentRoleID = getCurrentRoleId(roleId, factory.user.roleID);
    //获取当前用户的角色ID
    psTreeData.getCurrentUser(factory.user);
    //psTreeData获取当前调用信息的接口开始渲染设备树(psTreeData后面会有介绍)
  }
}
```
###### 2,初始化菜单功能

```javascript
factory.initMenus = function ($scope, $location) {
  //初始化菜单目录的信息
  $scope.userInfo.functionList.forEach(function (menu) {
    if ($scope.baseConfig && $scope.baseConfig[menu.functionCode]) {
      menu.name = $scope.baseConfig[menu.functionCode].label;
      menu.label = $scope.baseConfig[menu.functionCode].sublabel;
    }
    //检查baseConfig内容，如果有对应的label和sublabel值更新当前的菜单变量
    function menuhandle(menuInfo) {
      //递归根据menuInfo的functionCode生成$scope.menuitems的map对象。
    }
    factory.rootHandler($scope, $location);
    //检测当前用户是否有权限，并且应该跳转到哪个对应的路由：w
  }
}
factory.rootHandler = function ($scope, $location) {
  //根据菜单权限确定是否是授权用户，是否具有跳转到该菜单的权限
}
```

##### ps-tree-data.js 树形结构渲染服务

```javascript

...
structrureDone = resourceUIService.deviceLoader().then(devices => {
  //获取全部设备列表
  return incloseDeviceLoader().then(inclosed => {
    //查看总包设备
    return getResByModelIds([300, 301, 302]).then(d => {
      //获取管理域名，设备，项目的详细信息对象
      ...
      deviceMap = getDeviceMap(remapedDevice);
      //根据树结构的6维码，9维码等构建一个map映射对象
      ...
      //构建树结构对象
      return success(root);
      // 返回构建好的树结构对象
    })
  })
});
...
class treeData {
  constructor(id){
    // 初始化一颗树或树的节点，初始化信息内容可仅包含树的id信息。
  }
  showTree(){
    // 此树或者树的节点是否会在服务中心的设备树中展示，返回值为BOOLEAN
  }
  find(callback){
    // 递归搜索一棵树的所有叶子节点，直到找到符合条件的节点
  }
  getPathLocation(){
    // 获取当前节点所对应的跳转地址信息
  }
  getMeasureLocate(){
    // 获取当前节点包含的测点信息（如果当前节点不是设备则返回null）
  }
  getPathAndShowTreeLocation(){
    // 获取当前节点对应的跳转信息和树结构的显示隐藏信息
  }
  getChildren(callback, showDetail) {
    // 获取当前节点符合条件的所有叶子节点
  }
  getParent() {
    // 获取当前节点的父节点
  }
  getBrothers(){
    // 获取节点的所有兄弟节点信息（不包含节点本身）
  }
  getParents() {
    // 获取
  }
  getAlertState() {
    // 获取节点的告警状态
  }
  getState(){
    // 更新当前节点的信息
  }
  getRoot(){
    // 获取根节点的信息
  }
  $on(name, callback) {
    // 事件触发
  }
  $emit(name, data) {
    // 事件发布
  }
}
...
extend(treeDataService, {
  getValidId(id, type) {
    //获取指定id对应节点，下面第一个设备，如节点本身为设备则返回节点本身
  } ,
  getFirstDevice(){
    // 获取能搜索到的第一个设备
  },
  getAlertState(){ 
    // 获取所有节点的告警状态
  },
  getSensors(){
    // 获取当前设备的所有测点
  },
  getGeneral(){
    // 获取当前节点的测点 + 数据项
  },
  getRoot() {
    return structrureDone;
  }
});
return treeDataService;
```

##### ps-ui-router-handler.js 路由跳转控制模块

```javascript
extend(factory, {
  getDefaultParamByName(routername){
    //通过一个路由的名称查询它对应的默认参数
  }
  getCurrentRouterType(){
    // 获取当前路由对象
  }
  getCurrentTab(){
    // 获取当前tab对象信息
  }
  getCurrentMainTab(){
    // 获取当前根层级信息
  }
  getSubTabs() {
    // 获取二级菜单目录
  }
  getMinorTabs(){
    // 获取三级菜单目录
  }
  moveToNodeByCondition(callback){
    // 根据表达式移动到对应的路由
  }
  moveToNodeByIndex(url, arr, ext) {
    // 根据所提供的索引移动到对应的路由
  }
  getRoleValues(){
    // 获取角色的values对象信息
  }
})


```

##### **common-method-service2.service** 通用服务 

```javascript

class commonMethodService {
  constructor(){}
  getAlertWord(val){
    //根据节点告警状态值选择显示对应的中文
  }
  getTaskJob(appsource) {
    // 根据任务来源选择任务分类
  }
  getExplainerFromDiction(name){
    // 根据字典含义取值
  }
  getAppSource(variables){
    // 获取任务来源
  }
  $findValue(data, attr) {
    // 遍历一个对象找个是否存在某个值，值是多少
  }
  treeNavigateTo(id, deviceOnly, node) {
    // 根据设备id信息，确定移动到的目标路由参数
  }
  filterValues(data, attr) {
    // 遍历一个对象找个是否存在某个值，过滤并得到这些值
  }
  validate(obj, scope) {
    /**验证scope下面对象的每个值都不为undefined**/
  }
  refresh(type, params) {
    /**刷新当前页面的方法**/
  }
  modal({ title, directive, scope, top, width, params }) {
     /** 弹出模态框的方法 **/
  }
  navigateToTracker(ext, ticketNo) {
    /** 跳转到某个页面的方法 **/
  }
  getRoot(){
     /** 获取树的根节点 **/
  }
  getCurrentParents(){
    /** 获取当前节点的所有父节点 **/
  }
  findFlowByTicketNo(ticketNo) {
    /** 通过工单号获取流程图的内容 **/
  }
  getKpiValue(nodeId, kpis) {
    /** 根据ci,kpi查询节点值 **/
  }
  getDataItemValuelist(params) {
    // 获取节点的kpivalue数据
  }
}


```

##### 项目启动方法和本地调试：

###### 新项目安装

```linux
$ sudo npm i
$ npm run pack baogang/config
$ npm run pack core/output
```

###### 项目本地调试

```linux
$ npm run server
```

###### 本地前端打包

```linux
$ npm run gulp release
```

###### 从一个环境拷贝仪表板视图

```linux
$ npm run import [viewId] [环境号]
```

###### 从191环境下载仪表板视图12345678到本地

```linux
$ npm run import 12345678 191
```

##### smart-angular 组件的使用与维护
