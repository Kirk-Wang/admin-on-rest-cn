# Show视图

Show视图以只读方式显示从API获取的记录。它将记录的实际呈现方式委托给一个布局组件——通常是`<SimpleShowLayout>`。这个布局组件使用它的子元素（[`<Fields>`](./Fields.html) 组件）来呈现每个记录字段。

![post show view](https://marmelab.com/admin-on-rest/img/show-view.png)

### `<Show>`组件{#show}

`<Show>`组件呈现页面标题和操作，并从REST API获取记录。 它不负责渲染实际的记录 - 这是它的子组件（通常是`<SimpleShowLayout>`）的工作，它们传递`record`作为属性。

以下是通过`<Show>`组件接受的所有属性：

* [`title`](#page-title)
* [`actions`](#actions)

下面是去显示一个视图来显示一个帖子所必须的最少代码：

{% raw %}
```jsx
// in src/App.js
import React from 'react';
import { jsonServerRestClient, Admin, Resource } from 'admin-on-rest';

import { PostCreate, PostEdit, PostShow } from './posts';

const App = () => (
    <Admin restClient={jsonServerRestClient('http://jsonplaceholder.typicode.com')}>
        <Resource name="posts" show={PostShow} create={PostCreate} edit={PostEdit} />
    </Admin>
);

export default App;

// in src/posts.js
import React from 'react';
import { Show, SimpleShowLayout, TextField, DateField, EditButton, RichTextField } from 'admin-on-rest';

export const PostShow = (props) => (
    <Show {...props}>
        <SimpleShowLayout>
            <TextField source="title" />
            <TextField source="teaser" />
            <RichTextField source="body" />
            <DateField label="Publication date" source="created_at" />
        </SimpleShowLayout>
    </Show>
);
```
{% endraw %}

这就足以显示这个帖子显示视图

![post show view](https://marmelab.com/admin-on-rest/img/post-show.png)

###  title（页面标题）{#page-title}

默认情况下，Show视图的显示标题是"[resource_name] #[record_id]"。

您可以通过指定一个自定义`title`属性来自定义此标题：

```jsx
export const PostShow = (props) => (
    <Show title="Post view" {...props}>
        ...
    </Show>
);
```

更有趣的是，你可以通过传递一个组件作为`title`。Admin-on-rest克隆此组件，并在<ShowView>中，注入当前`record`。这允许根据当前记录自定义标题：

```jsx
const PostTitle = ({ record }) => {
    return <span>Post {record ? `"${record.title}"` : ''}</span>;
};
export const PostShow = (props) => (
    <Show title={<PostTitle />} {...props}>
        ...
    </Show>
);
```

### actions（动作）{#actions}

你可以通过你自己的元素使用`actions`属性来替换默认动作列表：

```jsx
import { CardActions } from 'material-ui/Card';
import FlatButton from 'material-ui/FlatButton';
import NavigationRefresh from 'material-ui/svg-icons/navigation/refresh';
import { ListButton, EditButton, DeleteButton } from 'admin-on-rest';

const cardActionStyle = {
    zIndex: 2,
    display: 'inline-block',
    float: 'right',
};

const PostShowActions = ({ basePath, data, refresh }) => (
    <CardActions style={cardActionStyle}>
        <EditButton basePath={basePath} record={data} />
        <ListButton basePath={basePath} />
        <DeleteButton basePath={basePath} record={data} />
        <FlatButton primary label="Refresh" onClick={refresh} icon={<NavigationRefresh />} />
        {/* Add your custom actions */}
        <FlatButton primary label="Custom Action" onClick={customAction} />
    </CardActions>
);

export const PostShow = (props) => (
    <Show actions={<PostShowActions />} {...props}>
        ...
    </Show>
);
```

## `<SimpleShowLayout>`组件{#SimpleShowLayout}

`<SimpleShowLayout>`组件从它的父组件接收`record`作为属性。它负责渲染实际的视图。`<SimpleShowLayout>`逐行呈现其子组件（在`<div>`组件内）。

`<SimpleShowLayout>`逐行呈现其子组件（在`<div>`组件内）。

```jsx
export const PostShow = (props) => (
    <Show {...props}>
        <SimpleShowLayout>
            <TextField source="title" />
            <RichTextField source="body" />
            <NumberField source="nb_views" />
        </SimpleShowLayout>
    </Show>
);
```

可以通过指定`style` prop来覆盖它的样式，例如：

```jsx
const styles = {
    container: {
        display: 'flex',
    },
    item: {
        marginRight: '1rem',
    },
};

export const PostShow = (props) => (
    <Show {...props}>
        <SimpleShowLayout style={styles.container}>
            <TextField source="title" style={styles.item} />
            <RichTextField source="body" style={styles.item} />
            <NumberField source="nb_views" style={styles.item} />
        </SimpleShowLayout>
    </Show>
);
```