# admin-on-rest 中文文档

admin-on-rest是一个用于构建在REST服务之上的浏览器中运行的管理应用程序前端框架，它使用了ES6，[React](https://facebook.github.io/react/)和[Material Design](https://material.io/)。 它被 [marmelab](https://marmelab.com/) 开源和维护。 

[Documentation](https://marmelab.com/admin-on-rest/) - [Source](https://github.com/marmelab/admin-on-rest)

[👀演示](https://marmelab.com/admin-on-rest-demo/) - [👃文档](https://kirk-wang.gitbooks.io/admin-on-rest/) - [发行版本](https://github.com/marmelab/admin-on-rest/releases) - [服务支持](http://stackoverflow.com/questions/tagged/admin-on-rest)- [社区](https://www.admin-on-rest.com/)

[![admin-on-rest-视频展示（请自带梯子）](https://marmelab.com/admin-on-rest/img/admin-on-rest-demo-still.png)](https://vimeo.com/205118063)

## 产品功能{#Features}

* 适应任何REST后端
* 完善的文档
* 乐观的UI呈现（在服务器返回之前乐观的UI呈现）
* 关系（多对一，一对多）
* 国际化（i18n）
* 条件格式化
* 主题化（可更换的主题）
* 支持任何身份验证提供程序（REST API，OAuth，Basic Auth，...）
* 功能全面的数据表格（排序，分页，过滤器）
* Filter-as-you-type
* 支持任何表单布局（简单的，选项卡的，等等）
* 数据验证
* 自定义操作
* 用于各种数据类型的大型组件库: boolean，number，rich text，等等。
* WYSIWYG（所见即所得）编辑
* 自定义仪表板，菜单，布局
* 超级容易扩展和覆盖（它仅仅只是React组件）
* 高度可定制的界面
* 可以连接到多个后端
* 使用了React生态系统中最好的库（Redux，redux-form，redux-saga，material-ui，recompose）
* 可以被包含在另一个React应用中
* 受流行库 [ng-admin](https://github.com/marmelab/ng-admin) 的启发 (ng-admin 也是被marmelab开源和维护)

## 安装{#Installation}

Admin-on-rest 可以从npm获得. 你可以安装它（和它所必需的依赖）
使用：

```sh
npm install --save-dev admin-on-rest
```

## 用法{#Usage}

阅读大约15分钟的 [教程](./Tutorial.html) 介绍。之后，前往[文档](./index.html)，或者检出一个栗子🌰用法的[演示源码](https://github.com/marmelab/admin-on-rest-demo)。

## 秒👀看个栗子🌰{#At-a-Glance}

```jsx
// in app.js
import React from 'react';
import { render } from 'react-dom';
import { simpleRestClient, Admin, Resource } from 'admin-on-rest';

import { PostList, PostEdit, PostCreate, PostIcon } from './posts';

render(
    <Admin restClient={simpleRestClient('http://localhost:3000')}>
        <Resource name="posts" list={PostList} edit={PostEdit} create={PostCreate} icon={PostIcon}/>
    </Admin>,
    document.getElementById('root')
);
```

`<Resource>` 组件是一个配置组件，它允许你为每个管理视图定义子组件： `list`，`edit`，and `create`。 
这些组件使用来自于admin-on-rest中的 Material UI 和 定制组件：

{% raw %}
```jsx
// in posts.js
import React from 'react';
import { List, Datagrid, Edit, Create, SimpleForm, DateField, TextField, EditButton, DisabledInput, TextInput, LongTextInput, DateInput } from 'admin-on-rest';
export PostIcon from 'material-ui/svg-icons/action/book';

export const PostList = (props) => (
    <List {...props}>
        <Datagrid>
            <TextField source="id" />
            <TextField source="title" />
            <DateField source="published_at" />
            <TextField source="average_note" />
            <TextField source="views" />
            <EditButton basePath="/posts" />
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
            <TextInput source="title" />
            <TextInput source="teaser" options={{ multiLine: true }} />
            <LongTextInput source="body" />
            <DateInput label="Publication date" source="published_at" />
            <TextInput source="average_note" />
            <DisabledInput label="Nb views" source="views" />
        </SimpleForm>
    </Edit>
);

export const PostCreate = (props) => (
    <Create title="Create a Post" {...props}>
        <SimpleForm>
            <TextInput source="title" />
            <TextInput source="teaser" options={{ multiLine: true }} />
            <LongTextInput source="body" />
            <TextInput label="Publication date" source="published_at" />
            <TextInput source="average_note" />
        </SimpleForm>
    </Create>
);
```
{% endraw %}

## 它是否适用于我的REST API？{#Does-It-Work-With-My-REST-API}

Yes.

Admin-on-rest使用一种称为*REST client*概念的适配器方法。它目前的rest客户端能被用于blueprint为你设计的API，或者你可以写你自己的REST客户端去查询一个现存的API。写一个自定义的客户端就是几个小时之内的事情。

![REST client architecture](https://marmelab.com/admin-on-rest/img/rest-client.png)

查看详细的[REST客户端文档](https://marmelab.com/admin-on-rest/RestClients.html)。

## 包含🔋电池但可拆装{#Batteries-Included-But-Removable}

Admin-on-rest被设计为一个松散合的React组件库，它建立在[material-ui](http://www.material-ui.com/#/)之上，除了控制器功能实现了Redux方法以外。它也非常容易用你自己的方法去替换admin-on-rest的一部分，例如，去自定义datagrid，GraphQL代替REST，或者bootstrap代替Material Design。

## 贡献{#Contributing}

欢迎所有在[GitHub repository](https://github.com/marmelab/admin-on-rest)上的PR。尝试遵循现有文件的编码风格，并包括单元测试和文档。为全面的代码审查做好准备，并且对合并要有耐心 - 这是一项开源的倡议。

您可以运行这个示例应用程序通过调用：

```sh
make run
```

然后浏览到[http://localhost:4000/](http://localhost:4000/)。

如果你想要贡献这个文档，安装jekyll，然后调用：

```sh
make doc
```

然后浏览到[http://localhost:4000/](http://localhost:4000/)

您可以运行单元测试通过调用：

```sh
make test
```

如果你正使用admin-on-rest作为依赖项，并且如果您想尝试并hack它，下面是建议的过程：

```sh
# in myapp
# install admin-on-rest from GitHub in another directory
$ cd ..
$ git clone git@github.com:marmelab/admin-on-rest.git && cd admin-on-rest && make install
# replace your node_modules/admin-on-rest by a symbolic link to the github checkout
$ cd ../myapp
$ npm link ../admin-on-rest
# go back to the checkout, and replace the version of react by the one in your app
$ cd ../admin-on-rest
$ npm link ../myapp/node_modules/react
$ make watch
# in another terminal, go back to your app, and start it as usual
$ cd ../myapp
$ npm run
```

## 许可{#License}

Admin-on-rest遵循[MIT Licence](https://github.com/marmelab/admin-on-rest/blob/master/LICENSE.md)， 由[marmelab](http://marmelab.com)赞助和支持。

## 捐赠{#Donate}

这个库是免费使用的，即使是商业用途。如果你想要回馈，请交流它，帮助新人，或贡献代码。但最好的回馈是**捐赠给慈善机构**。我们推荐 [Doctors Without Borders](http://www.doctorswithoutborders.org/)。
