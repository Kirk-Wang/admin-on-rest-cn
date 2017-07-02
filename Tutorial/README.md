# Admin-on-REST 教程

这个15分钟的教程将介绍如何根据现有的REST API创建一个新的管理应用程序。

![admin-on-rest blog demo](http://static.marmelab.com/admin-on-rest.gif)

## 安装{#Installation}

Admin-on-REST使用React。我们将用Facebook的[react-create-app](https://github.com/facebookincubator/create-react-app)去创建一个空的React app， 并且安装 `admin-on-rest` 包：

```sh
npm install -g create-react-app
create-react-app test-admin
cd test-admin/
yarn add admin-on-rest
yarn start
```

您应该是在3000端口上启动并运行着一个空的React应用程序。

## 与API进行关联{#MakingContactWithTheAPI}

我们将使用[JSONPlaceholder](http://jsonplaceholder.typicode.com/)，一个用于测试和原型设计的假REST API，作为管理员的数据源。

```
curl http://jsonplaceholder.typicode.com/posts/12
```

```json
{
  "id": 12,
  "title": "in quibusdam tempore odit est dolorem",
  "body": "itaque id aut magnam\npraesentium quia et ea odit et ea voluptas et\nsapiente quia nihil amet occaecati quia id voluptatem\nincidunt ea est distinctio odio",
  "userId": 2
}
```

JSONPlaceholder为帖子，评论和用户提供端点。 我们建立的管理员将允许创建，检索，更新和删除（CRUD）这些资源。

通过以下代码替换`src/App.js`：

```jsx
// in src/App.js
import React from 'react';
import { jsonServerRestClient, Admin, Resource } from 'admin-on-rest';

import { PostList } from './posts';

const App = () => (
    <Admin restClient={jsonServerRestClient('http://jsonplaceholder.typicode.com')}>
        <Resource name="posts" list={PostList} />
    </Admin>
);

export default App;
```

`App`组件现在渲染了一个`<Admin>`组件，它是admin-on-rest的主组件。这个组件期望一个REST client作为一个参数 - 一个有转换REST命令到HTTP请求能力的函数。由于REST不是一个标准，你将可能不得不提供一个自定义 client去连接你自己的API。 稍后我们将深入到REST客户端。现在，让我们好好利用的`jsonServerRestClient`，它说的是与JSONPlaceholder相同的REST dialect 。

`<Admin>`组件可以包含一个或多个`<Resource>`组件，每个resource被映射到API中的一个端点。 首先, 我们将显示帖子列表。下面这个`<PostList>`组件👀起来🐘这样子：

```jsx
// in src/posts.js
import React from 'react';
import { List, Datagrid, TextField } from 'admin-on-rest';

export const PostList = (props) => (
    <List {...props}>
        <Datagrid>
            <TextField source="id" />
            <TextField source="title" />
            <TextField source="body" />
        </Datagrid>
    </List>
);
```

这个帖子列表的主组件是一个`<List>`组件，负责从API中抓取信息，显示页面标题，和处理分页。这个list然后委托到一个`<Datagrid>`组件实际显示帖子列表，负责显示一个为每个帖子一行的表。这个datagrid使用它的子组件（这里，列表中的`<TextField>`）来确定要渲染的列。在API响应中每个Field组件映射一个不同的字段，由`source`属性指定。

这足以显示帖子列表：

![Simple posts list](https://marmelab.com/admin-on-rest/img/simple-post-list.png)

这个列表已经可使用：您可以通过单击列标题来重新排序，或者使用底部的分页控件更改页面。

## 字段类型{#FieldTypes}

你刚已👀到`<TextField>`组件，但admin-on-rest提供了许多字段组件去映射各种内容类型。举个例子，[在JSONPlaceholder中的端点`/users` ](http://jsonplaceholder.typicode.com/users)包含了邮箱📮。

```
curl http://jsonplaceholder.typicode.com/users/2
```

```json
{
  "id": 2,
  "name": "Ervin Howell",
  "username": "Antonette",
  "email": "Shanna@melissa.tv",
  "address": {
    "street": "Victor Plains",
    "suite": "Suite 879",
    "city": "Wisokyburgh",
    "zipcode": "90566-7771",
    "geo": {
      "lat": "-43.9509",
      "lng": "-34.4618"
    }
  },
  "phone": "010-692-6593 x09125",
  "website": "anastasia.net",
  "company": {
    "name": "Deckow-Crist",
    "catchPhrase": "Proactive didactic contingency",
    "bs": "synergize scalable supply-chains"
  }
}
```

让我们创建一个新的`UserList`，使用`<EmailField>`映射`email`字段：

```jsx
// in src/users.js
import React from 'react';
import { List, Datagrid, EmailField, TextField } from 'admin-on-rest';

export const UserList = (props) => (
    <List title="All users" {...props}>
        <Datagrid>
            <TextField source="id" />
            <TextField source="name" />
            <TextField source="username" />
            <EmailField source="email" />
        </Datagrid>
    </List>
);
```

你会注意到这个列表覆写了默认`title`。去包含一个新的`users`resource组件在admin app中，并添加它在`src/App.js`中：

```jsx
// in src/App.js
import { PostList } from './posts';
import { UserList } from './users';

const App = () => (
    <Admin restClient={jsonServerRestClient('http://jsonplaceholder.typicode.com')}>
        <Resource name="posts" list={PostList} />
        <Resource name="users" list={UserList} />
    </Admin>
);
```

![Simple user datagrid](https://marmelab.com/admin-on-rest/img/simple-user-list.png)

侧栏现在可以访问第二个资源，Users。这个users列表显示这个email为一个`<a href="mailto:">`标签。

在admin-on-rest中，所有字段都是一个个简单的React组件。在运行时, 他们接收从API上获取的 `record` （例如：`{ "id": 2, "name": "Ervin Howell", "username": "Antonette", "email": "Shanna@melissa.tv", ... }`)，和它们应该显示的`source`字段(例如：'email')。

这意味着编写自定义字段组件非常简单。例如，去创建一个`UrlField`：

```jsx
// in admin-on-rest/src/mui/field/UrlField.js
import React from 'react';
import PropTypes from 'prop-types';

const UrlField = ({ record = {}, source }) =>
    <a href={record[source]}>
        {record[source]}
    </a>;

UrlField.propTypes = {
    record: PropTypes.object,
    source: PropTypes.string.isRequired,
};

export default UrlField;
```

## 关系{#Relationships}

在JSONPlaceholder中, 每个`post`记录包含一个`userId`字段，它指向一个`user`：

```json
{
    "id": 1,
    "userId": 1,
    "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
    "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"
}
```

Admin-on-REST知道如何利用这些外键来获取引用。例如，在这些帖子列表中包含用户名，使用`<ReferenceField>`：

```jsx
// in src/posts.js
import React from 'react';
import { List, Datagrid, TextField, EmailField, ReferenceField } from 'admin-on-rest';

export const PostList = (props) => (
    <List {...props}>
        <Datagrid>
            <TextField source="id" />
            <ReferenceField label="User" source="userId" reference="users">
                <TextField source="name" />
            </ReferenceField>
            <TextField source="title" />
            <TextField source="body" />
        </Datagrid>
    </List>
);
```

当我们显示这些帖子列表时，这个app现在会获取关联的用户纪录，并显示他们的`name`作为一个`<TextField>`。 注意这个`label`属性：你可以用它在字段组件上来自定这个字段标签🏷️。

![reference posts in comment list](https://marmelab.com/admin-on-rest/img/reference-posts.png)

**提示**：Reference组件总是传递它们检索的数据到一个子组件，它是负责显示数据的。

## 创建和编辑{#CreationAndEdition}

一个admin界面有关于显示远程数据的，但也有关于编辑和创建的。Admin-on-REST提供了`<Create>`和`<Edit>`组件来做这件事。添加它们到`posts`脚本：

```jsx
// in src/posts.js
import React from 'react';
import { List, Edit, Create, Datagrid, ReferenceField, TextField, EditButton, DisabledInput, LongTextInput, ReferenceInput, SelectInput, SimpleForm, TextInput } from 'admin-on-rest';

export const PostList = (props) => (
    <List {...props}>
        <Datagrid>
            <TextField source="id" />
            <ReferenceField label="User" source="userId" reference="users">
                <TextField source="name" />
            </ReferenceField>
            <TextField source="title" />
            <TextField source="body" />
            <EditButton />
        </Datagrid>
    </List>
);

const PostTitle = ({ record }) => {
    return <span>Post {record ? `"${record.title}"` : ''}</span>;
};

export const PostEdit = (props) => (
    <Edit title={<PostTitle />} {...props}>
        <SimpleForm>
            <DisabledInput source="id" />
            <ReferenceInput label="User" source="userId" reference="users">
                <SelectInput optionText="name" />
            </ReferenceInput>
            <TextInput source="title" />
            <LongTextInput source="body" />
        </SimpleForm>
    </Edit>
);

export const PostCreate = (props) => (
    <Create {...props}>
        <SimpleForm>
            <ReferenceInput label="User" source="userId" reference="users" allowEmpty>
                <SelectInput optionText="name" />
            </ReferenceInput>
            <TextInput source="title" />
            <LongTextInput source="body" />
        </SimpleForm>
    </Create>
);
```

注意在`<PostList>`字组件中增加的`<EditButton>`字段：这是给予获🉐编辑页的访问。也是如此，这个 `<Edit>`组件用了一个自定义的`<PostTitle>`组件作为标题，它展示了给一个规定的页面自定义标题的方法。

如果你已经理解`<List>`组件，这个`<Edit>`和`<Create>`组件将不足为奇。它们负责获取记录，并显示页面标题。它们传递纪录到这个`<SimpleForm>`组件，它负责表单布局，默认值，和验证。就像`<Datagrid>`，`<SimpleForm>`用它的子组件来确定要显示的表单输入🐘。它期望*input components*作为字组件。`<DisabledInput>`，`<TextInput>`，`<LongTextInput>`和`<ReferenceInput>`都是🐘这样的组件。

至于`<ReferenceInput>`，它采用🐘同的属性作为`<ReferenceField>`（在早先的列表页用过）。 `<ReferenceInput>`用这些属性去获取可能引用关联到当前纪录的API（在这个🍐子中, 可能 `users`关联到 `post`）。它然后传递这些可能的引用到字组件（`<SelectInput>`），它是负责显示它们（在这种情况通过它们的`name`），并且让用户选择一个。在HTML中`<SelectInput>`渲染为一个`<select>`🏷️标签。

**提示**：`<Edit>`和`<Create>`使用相同的`<ReferenceInput>`配置，➗了`allowEmpty`属性，它在`<Create>`中是必需的。

在这个名为posts的resource组件中使用新的`<PostEdit>`和`<PostCreate>`组件，在`<Resource>`组件中仅仅添加它们作为`edit`和`create`的属性：

```jsx
// in src/App.js
import { PostList, PostEdit, PostCreate } from './posts';
import { UserList } from './users';

const App = () => (
    <Admin restClient={jsonServerRestClient('http://jsonplaceholder.typicode.com')}>
        <Resource name="posts" list={PostList} edit={PostEdit} create={PostCreate} />
        // ...
    </Admin>
);
```

Admin-on-rest自动地添加一个"create"按钮在帖子列表顶部来给予访问`<PostCreate>`组件。并且 `<EditButton>`呈现在列表中的每一行来提供访问`<PostEdit>`组件。

![post list with access to edit and create](https://marmelab.com/admin-on-rest/img/editable-post.png)

这个表单在创建和编辑页中已经是可使用了的。在提交上它分别发送`POST`和`PUT`请求到REST API。

![post edition form](https://marmelab.com/admin-on-rest/img/post-edition.png)

**注意**：JSONPlaceholder是一个只读的 api；虽然它似乎接受`POST`和`PUT`请求，但它并没有考虑账户的新建和编辑 - 这就是为什么，在这种特定情况下，你将在创建后看到错误，并且在保存之后将不会看到你的编辑。它只是一个JSONPlaceholder的假象。

## 删除{#Deletion}

在一个删除视图中没有太多去配置。去添加删除能力到`Resource`组件，简单地使用捆绑来自于admin-on-rest中`<Delete>`的组件，并且用`remove`属性®️注册它（'delete'在JavaScript中是一个保留关键字）：

```jsx
// in src/App.js
import { Delete } from 'admin-on-rest';

const App = () => (
    <Admin restClient={jsonServerRestClient('http://jsonplaceholder.typicode.com')}>
        <Resource name="posts" list={PostList} edit={PostEdit} create={PostCreate} remove={Delete} />
        // ...
    </Admin>
);
```

在编辑视图中，一个新的"delete"按钮出现了。并且你也可以使用`<DeleteButton>`作为列表中的一个字段。

![post deletion view](https://marmelab.com/admin-on-rest/img/post-deletion.png)

## 过滤器{#Filters}

让我们回到帖子列表一分钟。它提供排序和分页，但缺少一个功能：搜索内容的能力。

Admin-on-rest可以使用输入组件在列表视图中创建一个多条件的搜索引擎。首先，创建一个`<Filter>`组件，就像编写一个`<SimpleForm>`组件一样，使用输入组件作为子元素。然后，将其添加到列表中使用`filters`属性：

```jsx
// in src/posts.js
import { Filter, ReferenceInput, SelectInput, TextInput } from 'admin-on-rest';

const PostFilter = (props) => (
    <Filter {...props}>
        <TextInput label="Search" source="q" alwaysOn />
        <ReferenceInput label="User" source="userId" reference="users" allowEmpty>
            <SelectInput optionText="name" />
        </ReferenceInput>
    </Filter>
);

export const PostList = (props) => (
    <List {...props} filters={<PostFilter />}>
        // ...
    </List>
);
```

第一个过滤器‘q’利用了JSONPlaceholder提供的全文功能。它是`alwaysOn`，所以它总是出现在屏幕上。第二个筛选器，‘userId’可以通过位于列表顶部的“add filter”按钮来添加。因为它是一个`<ReferenceInput>`，所以它已经填充了可能的用户。它可以由终端用户关闭。

过滤器是“search-as-you-type”，这意味着当用户在筛选表单中输入新值时，列表会立即刷新 (通过API请求)。

![posts search engine](https://marmelab.com/admin-on-rest/img/filters.gif)

## 自定义菜单图标{#CustomizingtheMenuIcons}

侧栏菜单显示出posts和users图标一样。幸运的是，自定义菜单图标只是将`icon`属性传递给每个 `<Resource>`：

```jsx
// in src/App.js
import PostIcon from 'material-ui/svg-icons/action/book';
import UserIcon from 'material-ui/svg-icons/social/group';

const App = () => (
    <Admin restClient={jsonServerRestClient('http://jsonplaceholder.typicode.com')}>
        <Resource name="posts" list={PostList} edit={PostEdit} create={PostCreate} remove={Delete} icon={PostIcon} />
        <Resource name="users" list={UserList} icon={UserIcon} />
    </Admin>
);
```

![custom menu icons](https://marmelab.com/admin-on-rest/img/custom-menu.png)

## 使用自定义主页{#UsingaCustomHomePage}

默认情况下，admin-on-rest显示第一个资源列表页为主页。如果你想要显示一个自定义组件来代替，传递它在`<Admin>`组件的`dashboard`属性中。

{% raw %}
```jsx
// in src/Dashboard.js
import React from 'react';
import { Card, CardHeader, CardText } from 'material-ui/Card';

export default () => (
    <Card style={{ margin: '2em' }}>
        <CardHeader title="Welcome to the administration" />
        <CardText>Lorem ipsum sic dolor amet...</CardText>
    </Card>
);
```
{% endraw %}

```jsx
// in src/App.js
import Dashboard from './Dashboard';

const App = () => (
    <Admin dashboard={Dashboard} restClient={jsonServerRestClient('http://jsonplaceholder.typicode.com')}>
        // ...
    </Admin>
);
```

![Custom home page](https://marmelab.com/admin-on-rest/img/dashboard.png)

## 添加登录页{#AddingaLoginPage}

大多数admin apps都需要身份验证。Admin-on-rest可以在显示页面之前检查用户凭据，并在REST API返回403错误代码时重定向到登录表单。

这些凭据是*什么*，以及*如何*获取它们，都是你必须回答的问题。Admin-on-rest对您的身份验证策略（basic auth，OAuth，custom route，等等）不作任何假设，但是在权限地方给你一个钩子去插入你的逻辑－通过调用一个`authClient`功能。

对于本教程，由于没有我们能使用的公共认证API，让我们使用一个假身份验证提供程序，接受每一个登录请求，并存储 `username`在`localStorage`中。每个页面更改将需要`localStorage`包含一个`username`项。

`authClient`是一个简单函数，他必须返回一个`Promise`：

```jsx
// in src/authClient.js
import { AUTH_LOGIN, AUTH_LOGOUT, AUTH_ERROR, AUTH_CHECK } from 'admin-on-rest';

export default (type, params) => {
    // called when the user attempts to log in
    if (type === AUTH_LOGIN) {
        const { username } = params;
        localStorage.setItem('username', username);
        // accept all username/password combinations
        return Promise.resolve();
    }
    // called when the user clicks on the logout button
    if (type === AUTH_LOGOUT) {
        localStorage.removeItem('username');
        return Promise.resolve();
    }
    // called when the API returns an error
    if (type === AUTH_ERROR) {
        const { status } = params;
        if (status === 401 || status === 403) {
            localStorage.removeItem('username');
            return Promise.reject();
        }
        return Promise.resolve();
    }
    // called when the user navigates to a new location
    if (type === AUTH_CHECK) {
        return localStorage.getItem('username') ? Promise.resolve() : Promise.reject();
    }
    return Promise.reject('Unknown method');
};
```

**提示**：由于`restClient`响应是异步的，因此您可以轻松地在其中获取身份验证服务器。

要启用此身份验证策略，传递这个client作为`<Admin>`组件中的`authClient`属性：

```jsx
// in src/App.js
import Dashboard from './Dashboard';
import authClient from './authClient';

const App = () => (
    <Admin authClient={authClient} restClient={jsonServerRestClient('http://jsonplaceholder.typicode.com')}>
        // ...
    </Admin>
);
```

一旦应用程序重新加载，登录表单背后现在接受每个用户：

![Login form](https://marmelab.com/admin-on-rest/img/dashboard.png)

## 响应式列表{#ResponsiveList}

admin-on-rest布局已是响应式。尝试调整浏览器大小来查看侧栏如何切换到较小屏幕上的抽屉样式。

但是一个响应式布局不足以去做一个响应式的app。datagrid组件在桌面上工作良好，但绝对不能适应移动设备。如果你的管理必须在移动设备上使用，你得为小屏幕提供替代的组件。

首先，您应该知道您不必使用`<Datagrid>`组件作为`<List>`子级。您可以使用您喜欢的任何其他组件。例如，`<SimpleList>`组件：

```jsx
// in src/posts.js
import React from 'react';
import { List, SimpleList } from 'admin-on-rest';

export const PostList = (props) => (
    <List {...props}>
        <SimpleList
            primaryText={record => record.title}
            secondaryText={record => `${record.views} views`}
            tertiaryText={record => new Date(record.published_at).toLocaleDateString()}
        />
    </List>
);
```

`<SimpleList>`组件使用[material-ui的`<List>`和`<ListItem>`组件](http://www.material-ui.com/#/components/list)，并且期望函数作为`primaryText`, `secondaryText`, 和 `tertiaryText` 的属性.

<img src="https://marmelab.com/admin-on-rest/img/mobile-post-list.png" alt="Mobile post list" style="display:block;margin:2em auto;box-shadow:none;filter:drop-shadow(13px 12px 7px rgba(0,0,0,0.5));" />

在移动方面工作正常，但现在桌面用户体验更差。最好的折衷办法是想要小的屏幕上用`<SimpleList>`，其它屏膜上用`<Datagrid>`。这就是`<Responsive>`组件的来源：

```jsx
// in src/posts.js
import React from 'react';
import { List, Responsive, SimpleList, Datagrid, TextField, ReferenceField, EditButton } from 'admin-on-rest';

export const PostList = (props) => (
    <List {...props}>
        <Responsive
            small={
                <SimpleList
                    primaryText={record => record.title}
                    secondaryText={record => `${record.views} views`}
                    tertiaryText={record => new Date(record.published_at).toLocaleDateString()}
                />
            }
            medium={
                <Datagrid>
                    <TextField source="id" />
                    <ReferenceField label="User" source="userId" reference="users">
                        <TextField source="name" />
                    </ReferenceField>
                    <TextField source="title" />
                    <TextField source="body" />
                    <EditButton />
                </Datagrid>
            }
        />
    </List>
);
```

这完全按你期望的方式工作。这里的教训是admin-on-rest为布局注意了响应式的web设计，但它是你的工作－在页面中用`<Responsive>`。

![Responsive List](https://marmelab.com/admin-on-rest/img/responsive-list.gif)

## 使用其它rest Dialect{#UsingAnotherRESTDialect}

这是本教程房间里的大象。在现实世界的项目中，你的API的REST dialect与JSONPLaceholder dialect不匹配。写一个REST客户端可能是你要做的第一件事，你将得使admin-on-rest依赖在你的API上，这可能需要几个小时的额外工作。

Admin-on-rest委托每个REST调用到一个REST客户端函数。这个函数必须简单地返回一个promise结果。这样给予了极大地自由来映射任何API dialect、添加身份验证标头、使用多个域中的端点等。

例如，让我们猜想你不得不用这个my.api.url API，它期望有如下参数：

| Action              | Expected REST request |
|---------------------|---------------------- |
| Get list            | `GET http://my.api.url/posts?sort=['title','ASC']&range=[0, 24]&filter={title:'bar'}` |
| Get one record      | `GET http://my.api.url/posts/123` |
| Get several records | `GET http://my.api.url/posts?filter={ids:[123,456,789]}` |
| Update a record     | `PUT http://my.api.url/posts/123` |
| Create a record     | `POST http://my.api.url/posts/123` |
| Delete a record     | `DELETE http://my.api.url/posts/123` |

Admin-on-rest为这个列表的每个动作定义了自定义动词。就像HTTP动词（`GET`，`POST`，等。），REST动词限定一个请求到REST服务器。Admin-on-rest动词被叫做`GET_LIST`，`GET_ONE`，`GET_MANY`，`CREATE`，`UPDATE`和 `DELETE`。REST客户端将得映射这些动词的每一个对应一个（或多个）HTTP 请求。

My.api.url API 客户端代码如下所示：

```jsx
// in src/restClient
import {
    GET_LIST,
    GET_ONE,
    GET_MANY,
    GET_MANY_REFERENCE,
    CREATE,
    UPDATE,
    DELETE,
    fetchUtils,
} from 'admin-on-rest';

const API_URL = 'my.api.url';

/**
 * @param {String} type One of the constants appearing at the top if this file, e.g. 'UPDATE'
 * @param {String} resource Name of the resource to fetch, e.g. 'posts'
 * @param {Object} params The REST request params, depending on the type
 * @returns {Object} { url, options } The HTTP request parameters
 */
const convertRESTRequestToHTTP = (type, resource, params) => {
    let url = '';
    const { queryParameters } = fetchUtils;
    const options = {};
    switch (type) {
    case GET_LIST: {
        const { page, perPage } = params.pagination;
        const { field, order } = params.sort;
        const query = {
            sort: JSON.stringify([field, order]),
            range: JSON.stringify([(page - 1) * perPage, page * perPage - 1]),
            filter: JSON.stringify(params.filter),
        };
        url = `${API_URL}/${resource}?${queryParameters(query)}`;
        break;
    }
    case GET_ONE:
        url = `${API_URL}/${resource}/${params.id}`;
        break;
    case GET_MANY: {
        const query = {
            filter: JSON.stringify({ id: params.ids }),
        };
        url = `${API_URL}/${resource}?${queryParameters(query)}`;
        break;
    }
    case GET_MANY_REFERENCE: {
        const { page, perPage } = params.pagination;
        const { field, order } = params.sort;
        const query = {
            sort: JSON.stringify([field, order]),
            range: JSON.stringify([(page - 1) * perPage, (page * perPage) - 1]),
            filter: JSON.stringify({ ...params.filter, [params.target]: params.id }),
        };
        url = `${API_URL}/${resource}?${queryParameters(query)}`;
        break;
    }
    case UPDATE:
        url = `${API_URL}/${resource}/${params.id}`;
        options.method = 'PUT';
        options.body = JSON.stringify(params.data);
        break;
    case CREATE:
        url = `${API_URL}/${resource}`;
        options.method = 'POST';
        options.body = JSON.stringify(params.data);
        break;
    case DELETE:
        url = `${API_URL}/${resource}/${params.id}`;
        options.method = 'DELETE';
        break;
    default:
        throw new Error(`Unsupported fetch action type ${type}`);
    }
    return { url, options };
};

/**
 * @param {Object} response HTTP response from fetch()
 * @param {String} type One of the constants appearing at the top if this file, e.g. 'UPDATE'
 * @param {String} resource Name of the resource to fetch, e.g. 'posts'
 * @param {Object} params The REST request params, depending on the type
 * @returns {Object} REST response
 */
const convertHTTPResponseToREST = (response, type, resource, params) => {
    const { headers, json } = response;
    switch (type) {
    case GET_LIST:
        return {
            data: json.map(x => x),
            total: parseInt(headers.get('content-range').split('/').pop(), 10),
        };
    case CREATE:
        return { data: { ...params.data, id: json.id } };
    default:
        return { data: json };
    }
};

/**
 * @param {string} type Request type, e.g GET_LIST
 * @param {string} resource Resource name, e.g. "posts"
 * @param {Object} payload Request parameters. Depends on the request type
 * @returns {Promise} the Promise for a REST response
 */
export default (type, resource, params) => {
    const { fetchJson } = fetchUtils;
    const { url, options } = convertRESTRequestToHTTP(type, resource, params);
    return fetchJson(url, options)
        .then(response => convertHTTPResponseToREST(response, type, resource, params));
};
```

使用这个客户端代替之前的`jsonServerRestClient`仅仅是切换一个函数的问题：

```jsx
// in src/app.js
import myApiRestClient from './restClient';

const App = () => (
    <Admin restClient={myApiRestClient} dashboard={Dashboard}>
        // ...
    </Admin>
);
```

## 结论{#Conclusion}

Admin-on-rest是以考虑定制为基础构建的。你可以将任何admin-on-rest组件替换为你自己的组件，例如显示自定义列表布局或为一个给定的resource不同的编辑表单。

现在你已经完成了这个教程，继续阅读[admin-on-rest documentation](https://kirk-wang.gitbooks.io/admin-on-rest/)，和阅读[Material UI components documentation](http://www.material-ui.com/#/)。