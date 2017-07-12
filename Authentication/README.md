# 身份验证{#Authentication}

![Logout button](https://marmelab.com/admin-on-rest/img/login.gif)

Admin-on-rest可以通过您选择的身份验证策略来保护您的admin app。 由于有许多不同的可能策略（Basic Auth，JWT，OAuth等），admin-on-rest只需提供钩子来执行您自己的验证代码。

默认情况下，一个admin-on-rest app不需要身份验证。如果REST API返回401（未经授权）或403（禁止）响应，则用户将被重定向到`/login`路由。你无事可做 - 它已经建好了。

### 配置认证客户端{#Configuring-the-Auth-Client}

默认情况下，`/login`路由呈现一个名为`Login`的特殊组件，它显示一个登录表单，要求输入用户名和密码。

![Default Login Form](https://marmelab.com/admin-on-rest/img/login-form.png)

这种表单在提交时取决于`<Admin>`组件的`authClient`属性。 此函数接收认证请求`(type, params)`，并返回一个Promise。 `Login`调用`authClient`使用`AUTH_LOGIN`类型，`{ login, password }`作为参数的。 这是认证用户的理想场所，并存储他们的凭据。

例如，要通过HTTPS查询身份验证路由并将凭据（令牌）存储在本地存储中，请配置`authClient`，如下所示：

```jsx
// in src/authClient.js
import { AUTH_LOGIN } from 'admin-on-rest';

export default (type, params) => {
    if (type === AUTH_LOGIN) {
        const { username, password } = params;
        const request = new Request('https://mydomain.com/authenticate', {
            method: 'POST',
            body: JSON.stringify({ username, password }),
            headers: new Headers({ 'Content-Type': 'application/json' }),
        })
        return fetch(request)
            .then(response => {
                if (response.status < 200 || response.status >= 300) {
                    throw new Error(response.statusText);
                }
                return response.json();
            })
            .then(({ token }) => {
                localStorage.setItem('token', token);
            });
    }
    return Promise.resolve();
}
```

**提示**：在`localStorage`中存储凭证是一个好主意，以避免在打开新的浏览器标签时重新连接。 但这使您的应用程序[open to XSS attacks](http://www.redotheweb.com/2015/11/09/api-security.html)，所以你最好减少安全性，并也在服务器端添加一个`httpOnly`的cookie。

然后，将此客户端传递给`<Admin>`组件：

```jsx
// in src/App.js
import authClient from './authClient';

const App = () => (
    <Admin authClient={authClient}>
        ...
    </Admin>
);
```

在收到403回复后，admin app会显示登录页面。当用户提交登录表单时，`authClient`现在被调用。 一旦承诺解决，登录表单重定向到上一页，或者如果用户刚刚到达，则重定向到管理员索引。

### 发送凭证到REST API{#SendingCredentials-to-the-REST-API}

要在调用REST API路由时使用凭据，您必须进行调整，这次是`restClient`。 如[REST client documentation](RestClients.html#adding-custom-headers)中所述，`simpleRestClient`和`jsonServerRestClient`将`httpClient`作为第二个参数。这就是您可以更改请求标题，Cookie等的地方。

例如，要将登录期间获取的令牌作为`Authorization`标头传递，请按如下所示配置REST客户端：

```jsx
import { simpleRestClient, fetchUtils, Admin, Resource } from 'admin-on-rest';
const httpClient = (url, options = {}) => {
    if (!options.headers) {
        options.headers = new Headers({ Accept: 'application/json' });
    }
    const token = localStorage.getItem('token');
    options.headers.set('Authorization', `Bearer ${token}`);
    return fetchUtils.fetchJson(url, options);
}
const restClient = simpleRestClient('http://localhost:3000', httpClient);

const App = () => (
    <Admin restClient={restClient} authClient={authClient}>
        ...
    </Admin>
);
```

如果您有一个自定义REST客户端，请不要忘记自己添加凭据。

### 添加注销按钮{#Adding-a-Logout-Button}

如果您向`<Admin>`提供`authClient`属性，则admin-on-rest会在侧栏中显示注销按钮。当用户单击注销按钮时，这将使用`AUTH_LOGOUT`类型调用`authClient`，并从redux存储中删除可能敏感的数据。 解决后，用户将被重定向到登录页面。

例如，要在注销时从本地存储中删除令牌：

```jsx
// in src/authClient.js
import { AUTH_LOGIN, AUTH_LOGOUT } from 'admin-on-rest';

export default (type, params) => {
    if (type === AUTH_LOGIN) {
        // ...
    }
    if (type === AUTH_LOGOUT) {
        localStorage.removeItem('token');
        return Promise.resolve();
    }
    return Promise.resolve();
};
```

![Logout button](https://marmelab.com/admin-on-rest/img/logout.gif)

`authClient`也是通知认证API的好地方，用户凭据在注销后不再有效。

### 在API上捕获身份验证错误{#CatchingAuthenticationErrorsOnTheAPI}

即使用户可能在客户端进行身份验证，服务器端其凭据可能不再是有效的（例如，如果令牌仅在几周内有效）。在这种情况下，API通常会回答所有REST请求具有错误代码401或403 - 但是*你的* API呢？

幸运的是，每当API返回一个错误时，`authClient`用`AUTH_ERROR`类型调用。 再次决定哪些HTTP状态代码应该让用户继续（通过返回已解决的promise）或登录（通过返回被拒绝的promise）。

例如，401和403代码都要将用户重定向到登录页面：

```jsx
// in src/authClient.js
import { AUTH_LOGIN, AUTH_LOGOUT, AUTH_ERROR } from 'admin-on-rest';

export default (type, params) => {
    if (type === AUTH_LOGIN) {
        // ...
    }
    if (type === AUTH_LOGOUT) {
        // ...
    }
    if (type === AUTH_ERROR) {
        const { status } = params;
        if (status === 401 || status === 403) {
            localStorage.removeItem('token');
            return Promise.reject();
        }
        return Promise.resolve();
    }
    return Promise.resolve();
};
```

### 检查导航期间的凭据{#CheckingCredentialsDuringNavigation}

当REST响应使用401状态代码时，重定向到登录页面通常是不够的，因为admin-on-rest保留数据在客户端，并且可能在联系服务器时显示过时的数据 - 即使凭据不再有效。

幸运的是，每当用户导航时，admin-on-rest都使用`AUTH_CHECK`类型调用`authClient`，因此它是检查凭据的理想场所。

例如，要检查本地存储中的令牌是否存在：

```jsx
// in src/authClient.js
import { AUTH_LOGIN, AUTH_LOGOUT, AUTH_ERROR, AUTH_CHECK } from 'admin-on-rest';

export default (type, params) => {
    if (type === AUTH_LOGIN) {
        // ...
    }
    if (type === AUTH_LOGOUT) {
        // ...
    }
    if (type === AUTH_ERROR) {
        // ...
    }
    if (type === AUTH_CHECK) {
        return localStorage.getItem('token') ? Promise.resolve() : Promise.reject();
    }
    return Promise.reject('Unkown method');
};
```

如果promise被拒绝，默认情况下admin-on-rest重定向到`/login`页面。您可以通过将具有`redirectTo`属性的参数传递给被拒绝的promise来覆盖用户重定向的位置：

```jsx
// in src/authClient.js
import { AUTH_LOGIN, AUTH_LOGOUT, AUTH_ERROR, AUTH_CHECK } from 'admin-on-rest';

export default (type, params) => {
    if (type === AUTH_LOGIN) {
        // ...
    }
    if (type === AUTH_LOGOUT) {
        // ...
    }
    if (type === AUTH_ERROR) {
        // ...
    }
    if (type === AUTH_CHECK) {
        return localStorage.getItem('token') ? Promise.resolve() : Promise.reject({ redirectTo: '/no-access' });
    }
    return Promise.reject('Unkown method');
};
```

**提示**：对于`AUTH_CHECK`调用，`params`参数包含`resource`名称，因此可以为不同的资源实现不同的检查：

```jsx
// in src/authClient.js
import { AUTH_LOGIN, AUTH_LOGOUT, AUTH_ERROR, AUTH_CHECK } from 'admin-on-rest';

export default (type, params) => {
    if (type === AUTH_LOGIN) {
        // ...
    }
    if (type === AUTH_LOGOUT) {
        // ...
    }
    if (type === AUTH_ERROR) {
        // ...
    }
    if (type === AUTH_CHECK) {
        const { resource } = params;
        if (resource === 'posts') {
            // check credentials for the posts resource
        }
        if (resource === 'comments') {
            // check credentials for the comments resource
        }
    }
    return Promise.reject('Unkown method');
};
```

**提示**：`authClient`只能用`AUTH_LOGIN`，`AUTH_LOGOUT`，`AUTH_ERROR`或`AUTH_CHECK`调用; 这就是为什么最终返回一个拒绝的promise。

### 自定义登录和注销组件{#CustomizingTheLogin-and-Logout Components}

如果身份验证依赖于用户名和密码，则使用`authClient`和`checkCredentials`就可以实现全功能授权系统。

但是如果您想使用电子邮件而不是用户名呢？ 如果要使用带有第三方身份验证服务的单点登录（SSO），该怎么办？如果要使用双重身份认证怎么办？

对于所有这些情况，您需要实现自己的`LoginPage`组件，该组件将显示在`/ login`路由下，而不是默认的用户名/密码表单以及您自己的`LogoutButton`组件，这将显示在侧边栏。将这两个组件都传递给`<Admin>`组件：

**提示**：在您的自定义`Login`和`Logout`组件中使用`userLogin`和`userLogout`操作。

```jsx
// in src/MyLoginPage.js
import React, { Component } from 'react';
import { connect } from 'react-redux';
import { userLogin } from '../../actions/authActions';

class MyLoginPage extends Component {
    submit = (e) => {
        e.preventDefault();
        // gather your data/credentials here
        const credentials = { };

        // Dispatch the userLogin action (injected by connect)
        this.props.userLogin(credentials);
    }

    render() {
        return (
            <form onSubmit={this.submit}>
            ...
            </form>
        );
    }
};

export default connect(undefined, { userLogin })(MyLoginPage);

// in src/MyLogoutButton.js
import { connect } from 'react-redux';
import { userLogout } from '../../actions/authActions';

const MyLogoutButton = ({ userLogout }) => (
    <button onClick={userLogout}>Logout</button>
);

export default connect(undefined, { userLogout })(MyLogoutButton);

// in src/App.js
import MyLoginPage from './MyLoginPage';
import MyLogoutButton from './MyLogoutButton';

const App = () => (
    <Admin loginPage={MyLoginPage} logoutButton={MyLogoutButton} authClient={authClient}>
    ...
    </Admin>
);
```

### 限制对自定义页面的访问{#RestrictingAccessToACustomPage}

如果您添加[custom pages](./Actions.html)，如果[create an admin app from scratch](./CustomApp.html)，则可能需要手动安全地访问页面。 这是`<Restricted>`组件的目的，您可以将其用作自己的组件的装饰器。

{% raw %}
```jsx
// in src/MyPage.js
import { withRouter } from 'react-router-dom';
import { Restricted } from 'admin-on-rest';

const MyPage = ({ location }) =>
    <Restricted authParams={{ foo: 'bar' }} location={location} />
        <div>
            ...
        </div>
    </Restricted>
}

export default withRouter(MyPage);
```
{% endraw %}


`<Restricted>`组件用`AUTH_CHECK`和`authParams`调用`authClient`函数。 如果响应是一个fulfilled promise，则子组件被呈现。 如果回应是被拒绝的promise，`<Restricted>`重定向到登录表单。成功登录后，用户被重定向到初始位置（这就是为什么必须从路由器获取位置）。