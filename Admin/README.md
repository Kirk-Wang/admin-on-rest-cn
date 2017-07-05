# `<Admin>`组件

`<Admin>`组件可创建一个具有它自己的状态、路由和控制器逻辑的应用程序。`<Admin>`仅需要一个`restClient`和至少一个子 `<Resource>`来保证工作：

```jsx
// in src/App.js
import React from 'react';

import { simpleRestClient, Admin, Resource } from 'admin-on-rest';

import { PostList } from './posts';

const App = () => (
    <Admin restClient={simpleRestClient('http://path.to.my.api')}>
        <Resource name="posts" list={PostList} />
    </Admin>
);

export default App;
```

以下是通过该组件接受的所有属性：

* [`restClient`](#restclient)
* [`title`](#title)
* [`dashboard`](#dashboard)
* [`menu`](#menu)
* [`theme`](#theme)
* [`appLayout`](#applayout)
* [`customReducers`](#customreducers)
* [`customSagas`](#customsagas)
* [`customRoutes`](#customroutes)
* [`authClient`](#authclient)
* [`loginPage`](#loginpage)
* [`logoutButton`](#logoutbutton)
* [`locale`](#internationalization)
* [`messages`](#internationalization)
* [`initialState`](#initialstate)

### `restClient`（rest客户端）{#restClient}

唯一必需的属性，它必须是一个返回一个promise的函数，具有下列签名：

```jsx
/**
 * Execute the REST request and return a promise for a REST response
 *
 * @example
 * restClient(GET_ONE, 'posts', { id: 123 })
 *  => new Promise(resolve => resolve({ id: 123, title: "hello, world" }))
 *
 * @param {string} type Request type, e.g GET_LIST
 * @param {string} resource Resource name, e.g. "posts"
 * @param {Object} payload Request parameters. Depends on the action type
 * @returns {Promise} the Promise for a REST response
 */
const restClient = (type, resource, params) => new Promise();
```

`restClient`也是添加自定义http 标头、身份验证等的理想场所。[rest客户端章节](http://admin-on-rest.com/2017/06/18/rest-clients/)文档列出了可用的REST客户端，以及如何构建自己的REST客户端。

### `title`（标题）{#title}

默认情况下，一个admin app的头部使用'Admin on REST'作为主app标题。它可能是你会想要自定义的第一件事。这个`title`属性正是为这个目的服务的。

```jsx
const App = () => (
    <Admin title="My Custom Admin" restClient={simpleRestClient('http://path.to.my.api')}>
        // ...
    </Admin>
);
```

### `dashboard`（仪表盘）{#dashboard}

默认情况下，一个admin app的主页是第一个子`<Resource>`的`list`。但你也可以改为指定一个自定义组件。使用Material UI的`<Card>`组件和admin-on-rest的`<ViewTitle>`组件适合于在常规设计中：

{% raw %}
```jsx
// in src/Dashboard.js
import React from 'react';
import { Card, CardText } from 'material-ui/Card';
import { ViewTitle } from 'admin-on-rest/lib/mui';

export default () => (
    <Card>
        <ViewTitle title="Dashboard" />
        <CardText>Lorem ipsum sic dolor amet...</CardText>
    </Card>
);
```
{% endraw %}

```jsx
// in src/App.js
import Dashboard from './Dashboard';

const App = () => (
    <Admin dashboard={Dashboard} restClient={simpleRestClient('http://path.to.my.api')}>
        // ...
    </Admin>
);
```

![Custom home page](http://static.marmelab.com/admin-on-rest/dashboard.png)

### `menu`（菜单）{#menu}

Admin-on-rest使用`<Resource>`组件的列表作为`<Admin>`子级来为每个具有`list`组件的resource构建一个菜单。

如果你想要添加或删除菜单项，例如要链接到非资源页面，你可以创建你自己的菜单组件：

```jsx
// in src/Menu.js
import React from 'react';
import { MenuItemLink } from 'admin-on-rest';

export default ({ resources, onMenuTap, logout }) => (
    <div>
        <MenuItemLink to="/posts" primaryText="Posts" onTouchTap={onMenuTap} />
        <MenuItemLink to="/comments" primaryText="Comments" onTouchTap={onMenuTap} />
        <MenuItemLink to="/custom-route" primaryText="Miscellaneous" onTouchTap={onMenuTap} />
        {logout}
    </div>
);
```

**提示**：注意`MenuItemLink`组件。 必须使用它来避免移动视图中的不必要的副作用。

然后, 传递它作为 `menu`属性到`<Admin>`组件：

```jsx
// in src/App.js
import Menu from './Menu';

const App = () => (
    <Admin menu={Menu} restClient={simpleRestClient('http://path.to.my.api')}>
        // ...
    </Admin>
);
```

**提示**: 如果你要使用身份验证，别忘了在你的自定义菜单组件中去渲染`logout`属性。此外，`onMenuTap`函数作为属性传递被用于来关闭移动设备上的侧栏。

### `theme`（主题）{#theme}

Material UI支持[theming](http://www.material-ui.com/#/customization/themes)。这使您可以通过重写字体、颜色和间距来自定义admin的外观。你可以通过使用`theme`属性来提供一个自定义的material ui：

```jsx
import darkBaseTheme from 'material-ui/styles/baseThemes/darkBaseTheme';
import getMuiTheme from 'material-ui/styles/getMuiTheme';

const App = () => (
    <Admin theme={getMuiTheme(darkBaseTheme)} restClient={simpleRestClient('http://path.to.my.api')}>
        // ...
    </Admin>
);
```

![Dark theme](https://marmelab.com/admin-on-rest/img/dark-theme.png)

有关预定义主题和自定义主题的详细信息，请参阅[Material UI Customization documentation](http://www.material-ui.com/#/customization/themes)。

### `appLayout`（app布局）{#appLayout}

如果你想要深入定制应用程序头部、菜单或通知，最好的方法是提供自定义布局组件。它必须包含一个`{children}`占位符，其中admin-on-rest将呈现资源在那里。如果你使用material UI的fields和 inputs,它*必须*包含一个`<MuiThemeProvider>`元素。最后，当app在后台读取数据时如果你想要在app头部显示spinner。这个布局应该连接到redux store。

使用[默认布局](https://github.com/marmelab/admin-on-rest/blob/master/src/mui/layout/Layout.js)作为起点,并查看[主题文档](./Theming.html#using-a-custom-layout)中的列子。

```jsx
// in src/App.js
import MyLayout from './MyLayout';

const App = () => (
    <Admin appLayout={MyLayout} restClient={simpleRestClient('http://path.to.my.api')}>
        // ...
    </Admin>
);
```

### `customReducers`（自定义Reducer）{#customReducers}

`<Admin>`app使用[Redux](http://redux.js.org/)来管理状态。 这个状态有下列键：

```jsx
{
    admin: { /*...*/ }, // used by admin-on-rest
    form: { /*...*/ }, // used by redux-form
    routing: { /*...*/ }, // used by react-router-redux
}
```

如果你的组件分发自定义动作：你或许需要注册你自己的reducer来更新与这些action一起的state。让我们假设你想要保持bitcoin汇率在state中的`bitcoinRate`键中。你可能有一个类似如下的reducer：

```jsx
// in src/bitcoinRateReducer.js
export default (previousState = 0, { type, payload }) => {
    if (type === 'BITCOIN_RATE_RECEIVED') {
        return payload.rate;
    }
    return previousState;
}
```

去在`<Admin>` app注册这个reducer，仅仅在`customReducers`属性中传递它：

{% raw %}
```jsx
// in src/App.js
import React from 'react';
import { Admin } from 'admin-on-rest';

import bitcoinRateReducer from './bitcoinRateReducer';

const App = () => (
    <Admin customReducers={{ bitcoinRate: bitcoinRateReducer }} restClient={jsonServerRestClient('http://jsonplaceholder.typicode.com')}>
        ...
    </Admin>
);

export default App;
```
{% endraw %}

现在的state将如下：

```jsx
{
    admin: { /*...*/ }, // used by admin-on-rest
    form: { /*...*/ }, // used by redux-form
    routing: { /*...*/ }, // used by react-router-redux
    bitcoinRate: 123, // managed by rateReducer
}
```

### `customSagas`（自定义Saga）{#customSagas}

`<Admin>`app使用[redux-saga](https://github.com/redux-saga/redux-saga)来处理副作用。

如果你的组件分发自定义动作，你或许需要去注册你自己的side effects作为sagas。让我们假设你想要每当`BITCOIN_RATE_RECEIVED`动作被分发时显示一个通知。你可能有一个saga类似如下：

```jsx
// in src/bitcoinSaga.js
import { put, takeEvery } from 'redux-saga/effects';
import { showNotification } from 'admin-on-rest';

export default function* bitcoinSaga() {
    yield takeEvery('BITCOIN_RATE_RECEIVED', function* () {
        yield put(showNotification('Bitcoin rate updated'));
    })
}
```

去在`<Admin>`app注册这个saga，仅仅在`customSagas`属性中传递它：

```jsx
// in src/App.js
import React from 'react';
import { Admin } from 'admin-on-rest';

import bitcoinSaga from './bitcoinSaga';

const App = () => (
    <Admin customSagas={[ bitcoinSaga ]} restClient={jsonServerRestClient('http://jsonplaceholder.typicode.com')}>
        ...
    </Admin>
);

export default App;
```

### `customRoutes`（自定义Route）{#customRoutes}

去注册你自己的路由，创建一个模块返回一个[react-router](https://github.com/ReactTraining/react-router) `<Route>` 组件列表：

```jsx
// in src/customRoutes.js
import React from 'react';
import { Route } from 'react-router-dom';
import Foo from './Foo';
import Bar from './Bar';

export default [
    <Route exact path="/foo" component={Foo} />,
    <Route exact path="/bar" component={Bar} />,
];
```

然后，传递这个数组作为在`<Admin>`组件中的`customRoutes`属性：

```jsx
// in src/App.js
import React from 'react';
import { Admin } from 'admin-on-rest';

import customRoutes from './customRoutes';

const App = () => (
    <Admin customRoutes={customRoutes} restClient={jsonServerRestClient('http://jsonplaceholder.typicode.com')}>
        ...
    </Admin>
);

export default App;
```

现在，当用户浏览到 `/foo`或`/bar`时，你定义的组件将出现在屏幕的主要部分。

**提示**：你自己决定去创建一个[自定义菜单](#applayout)入口或者自定义按钮，来引导你的自定义页面。

**提示**：你的自定义页面优先于admin-on-rest的自己的路由。这意味着`customRoutes`允许你覆盖任何你想要的路由！

**提示**：要像其他admin-on-rest页面一样，您的自定义页面应具有以下结构：

```jsx
// in src/Foo.js
import React from 'react';
import { Card } from 'material-ui/Card';
import { ViewTitle } from 'admin-on-rest';

const Foo = () => (
    <Card>
        <ViewTitle title="My Page" />
        ...
    </Card>
));

export default Foo;
```

### `authClient`（授权客户端）{#authClient}

`authClient`属性期望是一个返回一个promise的函数，来控制应用程序的认证策略：

```jsx
import { AUTH_LOGIN, AUTH_LOGOUT, AUTH_ERROR, AUTH_CHECK } from 'admin-on-rest';

const authClient(type, params) {
    // type can be any of AUTH_LOGIN, AUTH_LOGOUT, AUTH_ERROR, and AUTH_CHECK
    // ...
    return Promise.resolve();
};

const App = () => (
    <Admin authClient={authClient} restClient={jsonServerRestClient('http://jsonplaceholder.typicode.com')}>
        ...
    </Admin>
);
```

[身份验证文档](./Authentication.html)解释了如何详细地实现这些功能。

### `loginPage`（登陆页）{#loginPage}

如果您要自定义登录页，或切换到其他身份验证策略而不是用户名/密码表单，请将您自己的组件作为`loginPage`属性传递。每当调用`/login`路由时，Admin-on-rest将显示此组件。

```jsx
import MyLoginPage from './MyLoginPage';

const App = () => (
    <Admin loginPage={MyLoginPage}>
        ...
    </Admin>
);
```

有关更多解释，请参见[身份验证文档](./Authentication.html#customizing-the-login-and-logout-components) 。

### `logoutButton`（注销按钮）{#logoutButton}

如果你自定义`loginPage`，则可能还需要重写 `logoutButton`，因为它们共享身份验证策略。

```jsx
import MyLoginPage from './MyLoginPage';
import MyLogoutButton from './MyLogoutButton';

const App = () => (
    <Admin loginPage={MyLoginPage} logoutButton={MyLogoutButton}>
        ...
    </Admin>
);
```

### `initialState`（初始状态）{#initialState}

`initialState`属性让你传递预加载状态到Redux。更多细节请参阅[Redux 文档](http://redux.js.org/docs/api/createStore.html#createstorereducer-preloadedstate-enhancer)。

### 国际化{#Internationalization}

`locale`和`messages`属性让你翻译这个GUI。[翻译文档](./Translation.html)详细说明了此过程。

## 在Admin和Resource组件外使用admin-on-rest{#Using-admin-on-rest-without-Admin-and-Resource}

使用<Admin>`和`<Resource>`完全是可选的。如果你觉得喜欢自己引导redux app，它是完全有可能的。前往[包含在另一个app](./CustomApp.html)给了一个详细的指引。