---
title: 2019学习计划
date: 2018-12-25 10:54:21
tags: 权限, antd Pro
---
因为后端使用 shiro 控制权限， 用户登录的时候返回的权限信息类似于

```
['test/view', 'test/add', 'test/edit'];
```
仔细看了下 ant design pro 的权限控制组件后发现，可以很好地结合使用

<!-- more -->

## 1. mock 登录返回数据
```
if (password === '123' && userName === 'wxc') {
  res.send({
    status: 'ok',
    type,
    currentAuthority: [
      'test/view', // 页面浏览权限
      'test/add', // 页面添加按钮权限
      'test/edit', // 编辑按钮权限
    ],
  });
  return;
}
```
## 2.修改 antd pro 的权限组件

因为框架已经自带了权限组件，但是这里的权限组件是通过'admin'，'user'这种字符串判断，而我们是登录后返回一个数组，所以要稍微修改下里面的权限判断逻辑

```
// src/components/Authorized/CheckPermissions.js

/**
 * 通用权限检查方法
 * Common check permissions method
 * @param { 权限判定 Permission judgment type string |array | Promise | Function } authority
 * @param { 你的权限 Your permission description  type:string} currentAuthorityString
 * @param { 通过的组件 Passing components } target
 * @param { 未通过的组件 no pass components } Exception
 */
const checkPermissions = (authority, currentAuthorityString, target, Exception) => {
  // 没有判定权限.默认查看所有,开发的时候可以使用
  // Retirement authority, return target

  // 当前获取的权限处理成数组
  const currentAuthority = currentAuthorityString.split(',');

  if (!authority) {
    return target;
  }
  // 数组处理
  if (Array.isArray(authority)) {
    let flag = false;
    authority.some(au => {
      if (currentAuthority.indexOf(au) !== -1) {
        flag = true;
        return true;
      } else {
        return false;
      }
    });
    if (flag) {
      return target;
    }
    return Exception;
  }

  // string 处理
  if (typeof authority === 'string') {
    if (currentAuthority.indexOf(authority) !== -1) {
      return target;
    }
    return Exception;
  }

  // promise和function
  ...
  throw new Error('unsupported parameters');
};
```
修改完毕之后就可以正常使用权限组件了

## 3.记录用户权限
登录获取到接口返回的权限信息后，通过工具类中封装好的setAuthority方法在 localStorage 中保存权限信息

```
import { setAuthority } from '../utils/authority';
...
	setAuthority(payload.currentAuthority);
...
```



效果如下：

保存权限信息完毕之后，还需要通过reloadAuthorized方法来刷新权限组件

```javascript
import { reloadAuthorized } from '../utils/Authorized';
...
	reloadAuthorized();
...
```

下面就可以正常使用 ant design pro 的权限控制功能了

## 4.权限控制方式
####控制菜单显示
这种方式下，路由导向也会做相应的控制，推荐使用这种方式

```javascript
// src/common/menu.js
{
   name: '权限测试页面',
   icon: 'user',
   path: 'test',
   authority: 'test/view',
 },
控制路由导向
// src/common/router.js
'/test': {
  component: dynamicWrapper(app, [], () => import('../routes/Test')),
  authority: 'test/view',
},
或

import Authorized from '../../utils/Authorized';

const { Secured } = Authorized;

@Secured('test/view')
export default class Test extends Component {
  ...
}
页面内组件控制
import Authorized from '../../utils/Authorized';

export default class Test extends Component {
	...
  render() {
    const noMatch = <div>没有权限</div>;
    return (

      <div>
        权限测试页面
        <Authorized authority="test/add" noMatch={noMatch}>
          <button>添加</button>
        </Authorized>
      </div>

​    );
  }
}
```





效果如下：
