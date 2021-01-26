---
layout: post
title: "ReactNative问题集(JS篇)"
subtitle: "ReactNative日常问题集结"
date: 2021-01-26
author: "WenDan"
header-img: "img/post-bg.jpg"
tags:
  - 前端开发
  - React Native
---

# ReactNative 遇到的问题（js篇）

## 1、react-native can't setState unmount component

### 场景一：ClassComponent --- setState

一般出现在数据请求返回过慢，并且页面已经销毁，这时候再触发setState就会报错。

#### 解决方式一：

增加一个字段来判断当前组件是否unmount，在setState之前判断字段，再操作

```javascript
componentWillUnmount() {
  this._isMounted = false;
}

fetchData = () => {
  fetch(api).then(res => {
    if (this._isMounted) {
      this.setState({
        xxx: xxx,
      });
    }
  });
}
```

#### 解决方式二：

上述方法需要每个组件都加上这么一段重复的代码。因此想到使用高阶组件解决

```javascript
function withIsMounted(Comp) {
  return class extends Comp {
    componentDidMount() {
      this._isMounted = true;
    }
    componentWillUnmount() {
      this._isMounted = false;
    }
  }
}

withIsMounted(YourComp);

class YourComp extends Component {
  fetchData = () => {
    fetch(api).then(res => {
      if (this._isMounted) {
        this.setState({
          xxx: xxx,
        });
      }
    });
  }
}
```

但是这个办法还是需要每个组件都加一个高阶函数，还是需要针对组件单独开发部分代码。

#### 考虑拉到顶层：设置myReact

```javascript
<!--myReact.js-->
import * as React from 'react';
export default React;

function safe(setState, ctx) {
    return (...args) => {
        if(ctx._isMounted) {
            setState.bind(ctx)(...args);
        }
    };
}
function did(didMount, ctx) {
    ctx._isMounted = true;
    didMount && didMount.call(ctx);
}
function willU(willUnmount, ctx) {
    ctx._isMounted = false;
    willUnmount && willUnmount.call(ctx);
}
function Wrap(Comp) {
    return class extends Comp {
        constructor(props) {
            super(props);
            this.setState = safe(this.setState, this);
            this.componentDidMount = did(this.componentDidMount, this);
            this.componentWillUnmount = willU(this.componentWillUnmount, this);
        }
    }
}

export const Component = Wrap(React.Component);
export const PureComponent = Wrap(React.PureComponent);
```

#### 使用:
1. 直接使用

```javascript
<!--老版本-->
import React from 'react';
<!--新版本-->
import React from './Components/myReact';
```

2. 为了方便全局使用，设置.babelrc文件

```javascript
<!--.babelrc-->
"plugins": [
    ["module-resolver", {
        "alias": {
            "myReact": "./Components/myReact"
        },
        "extensions": [".js"]
    }]
]

<!--使用-->
import React from 'myReact';
```

### 场景二：FuncComponent --- hooks

#### 解决：增加ref，标记当前isMounted状态

```javascript
const isMounted = useRef<boolean|null>(null);
useEffect(() => {
isMounted.current = true;
return () => {
    isMounted.current = null;
};
}, []);

fetchdata = () => {
    fetch(api).then(res => {
        if(isMounted.current) {
            setXXX(res);
        }
    });
}
```

![参照官方手册](/img/in-post/react-native/16075735488820.jpg)