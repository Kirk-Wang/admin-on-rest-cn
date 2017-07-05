# `<Resource>`组件 

一个`<Resource>`组件将一个API端点映射到一个CRUD界面。下面的管理应用程序为 [`http://jsonplaceholder.typicode.com/posts`](http://jsonplaceholder.typicode.com/posts)和 [`http://jsonplaceholder.typicode.com/users`](http://jsonplaceholder.typicode.com/users)中的JSONPlaceholder API公开的资源提供了一个只读界面：

```jsx
// in src/App.js
import React from 'react';
import { jsonServerRestClient, Admin, Resource } from 'admin-on-rest';

import { PostList } from './posts';
import { UserList } from './posts';

const App = () => (
    <Admin restClient={jsonServerRestClient('http://jsonplaceholder.typicode.com')}>
        <Resource name="posts" list={PostList} />
        <Resource name="users" list={UserList} />
    </Admin>
);
```

`<Resource>`允许你为每个CRUD操作定义一个组件，使用下列属性名称：

* `list`
* `create`
* `edit`
* `show`
* `remove`

这里是一个更完整的admin，具有为所有CRUD操作的组件：

```jsx
import React from 'react';
import { jsonServerRestClient, Admin, Resource } from 'admin-on-rest';

import { PostList, PostCreate, PostEdit, PostShow, PostIcon } from './posts';
import { UserList } from './posts';
import { CommentList, CommentEdit, CommentCreate, CommentIcon } from './comments';

const App = () => (
    <Admin restClient={jsonServerRestClient('http://jsonplaceholder.typicode.com')}>
        <Resource name="posts" list={PostList} create={PostCreate} edit={PostEdit} show={PostShow} remove={Delete} icon={PostIcon} />
        <Resource name="users" list={UserList} />
        <Resource name="comments" list={CommentList} create={CommentCreate} edit={CommentEdit} remove={Delete} icon={CommentIcon} />
        <Resource name="tags" />
    </Admin>
);
```

**注意**：在hood下，`<Resource>`组件使用了react-router去创建各自的路由：

* `/`映射到`list`组件
* `/create`映射到`create`组件
* `/:id`映射到`edit`组件
* `/:id/show`映射到`show`组件
* `/:id/delete`映射到`remove`组件

**注意**: 当你声明一个引用时，您必须添加一个`<Resource>`（通过`<ReferenceField>`、`<ReferenceArrayField>`、`<ReferenceManyField>`、`<ReferenceInput>`、`<ReferenceArrayInput>`或`<ReferenceManyInput>`），因为admin-on-rest使用resource定义的数据存储结构。这就是为什么在上面的例子有一个空的`tag`resource。

`<Resource>`也接受额外的属性：

* [`name`](#name)
* [`icon`](#icon)
* [`options`](#icon)

### `name`（名字）{#name}

Admin-on-rest使用`name`属性既确定API端点（被传递到`restClient`）又为resource构成URL。

```jsx
<Resource name="posts" list={PostList} create={PostCreate} edit={PostEdit} show={PostShow} remove={PostRemove} />
```

对于此资源，admin-on-rest将为数据获取`http://jsonplaceholder.typicode.com/posts`端点。

路由将按如下方式映射组件：

* `/posts/`映射到`PostList`
* `/posts/create`映射到`PostCreate`
* `/posts/:id`映射到`PostEdit`
* `/posts/:id/show`映射到`PostShow`
* `/posts/:id/delete`映射到`PostRemove`

**提示**: 如果您想使用特殊的API端点（例如：＇http://jsonplaceholder.typicode.com/my-custom-posts-endpoint＇）而不更改 admin-on-rest应用程序中的url（所以仍然使用`/posts`），在你自己的[`restClient`](./Admin.html#restclient)中写一个从resource`name`（`posts`）到API端点(`my-custom-posts-endpoint`)的映射。

### `icon`（图标）{#icon}

Admin-on-rest将在菜单中呈现`icon`属性组件：

```jsx
// in src/App.js
import React from 'react';
import PostIcon from 'material-ui/svg-icons/action/book';
import UserIcon from 'material-ui/svg-icons/social/group';
import { jsonServerRestClient, Admin, Resource } from 'admin-on-rest';

import { PostList } from './posts';

const App = () => (
    <Admin restClient={jsonServerRestClient('http://jsonplaceholder.typicode.com')}>
        <Resource name="posts" list={PostList} icon={PostIcon} />
        <Resource name="users" list={UserList} icon={UserIcon} />
    </Admin>
);
```

### options（选项）{#options}

`options.label`允许在菜单中自定义一个给定资源的显示名称。

{% raw %}
```jsx
<Resource name="v2/posts" options={{ label: 'Posts' }} list={PostList} />
```
{% endraw %}