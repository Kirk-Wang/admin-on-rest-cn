# List视图

列表视图显示从REST API提取的记录列表。此视图的入口点是`<List>`组件，它负责获取数据。然后，它将数据传递到迭代器视图-通常是`<Datagrid>`，然后将每个记录属性的渲染委托给[`<Field>`](./Fields.html)组件。


![The List View](https://marmelab.com/admin-on-rest/img/list-view.png)

### `<List>` 组件{#List}

`<List>`组件渲染列表布局（标题、按钮、筛选器、分页），并从 REST API 中提取记录列表。然后将记录列表的呈现委托给其子组件。通常，它是一个`<Datagrid>`，负责显示一个具有每个帖子一行的表格。

**提示**：在Redux条款中，`<List>`是一个connected组件，并且`<Datagrid>`是一个dumb组件。

以下是通过`<List>`组件接受的所有属性：

* [`title`](#page-title)
* [`actions`](#actions)
* [`filters`](#filters) (a React element used to display the filter form)
* [`perPage`](#records-per-page)
* [`sort`](#default-sort-field)
* [`filter`](#permanent-filter) (the permanent filter used in the REST request)
* [`pagination`](#pagination)

下面是显示帖子列表所需的最少代码：

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

这足以显示帖子列表：

![Simple posts list](https://marmelab.com/admin-on-rest/img/list-view.png)

### title（页面标题）{#page-title}

一个列表视图的默认标题是"[resource] list"（例如："Posts list"）。使用`title`属性来自定义列表视图标题：

```jsx
// in src/posts.js
export const PostList = (props) => (
    <List {...props} title="List of posts">
        ...
    </List>
);
```

标题可以是任何一个字符串，或者你自己的元素。

### actions（动作）{#actions}

你可以通过你自己的元素使用`actions`属性替换掉默认的action列表：

```jsx
import { CardActions } from 'material-ui/Card';
import FlatButton from 'material-ui/FlatButton';
import NavigationRefresh from 'material-ui/svg-icons/navigation/refresh';
import CreateButton from '../button/CreateButton';

const cardActionStyle = {
    zIndex: 2,
    display: 'inline-block',
    float: 'right',
};

const PostActions = ({ resource, filters, displayedFilters, filterValues, basePath, showFilter, refresh }) => (
    <CardActions style={cardActionStyle}>
        {filters && React.cloneElement(filters, { resource, showFilter, displayedFilters, filterValues, context: 'button' }) }
        <CreateButton basePath={basePath} />
        <FlatButton primary label="refresh" onClick={refresh} icon={<NavigationRefresh />} />
        {/* Add your custom actions */}
        <FlatButton primary label="Custom Action" onClick={customAction} />
    </CardActions>
);

export const PostList = (props) => (
    <List {...props} actions={<PostActions />}>
        ...
    </List>
);
```

### filters（过滤器）{#filters}

你可以使用`filters`属性添加一个过滤器元素到这个列表：

```jsx
const PostFilter = (props) => (
    <Filter {...props}>
        <TextInput label="Search" source="q" alwaysOn />
        <TextInput label="Title" source="title" defaultValue="Hello, World!" />
    </Filter>
);

export const PostList = (props) => (
    <List {...props} filters={<PostFilter />}>
        ...
    </List>
);
```

过滤器组件必须是一个具有`<Input>`子级的`<Filter>`组件。

**提示**：`<Filter>`是一个特殊的组件，它以两种方式呈现：

- 作为一个过滤器按钮（去添加新过滤器）
- 作为过滤器表单（去输入过滤器值）

它通过检查它的`context`属性这样做的。

**提示**：不要混淆这个 `filters` 属性，期望一个React 元素，具有 `filter` 属性，它期望一个对象来定义永久过滤器 (见下文)。

### perPage（每页记录数）{#records-per-page}

默认情况下，列表为10个一组的结果标记页码。你可以通过指定`perPage`属性覆写这个设置：

```jsx
// in src/posts.js
export const PostList = (props) => (
    <List {...props} perPage={25}>
        ...
    </List>
);
```

### sort（默认排序字段）{#default-sort-field}

传递一个对象字面量作为`sort`属性来确定用于排序的`field` 和 `order`：

{% raw %}
```jsx
// in src/posts.js
export const PostList = (props) => (
    <List {...props} sort={{ field: 'published_at', order: 'DESC' }}>
        ...
    </List>
);
```
{% endraw %}

`sort`定义了*default*排序顺序；这个列表通过在列标题上单击保持可排序。

### 禁用排序{#DisablingSorting}

通过传递一个设置为`false`的`sortable`属性，它可能会为一个指定的字段去禁用排序：

{% raw %}
```jsx
// in src/posts.js
import React from 'react';
import { List, Datagrid, TextField } from 'admin-on-rest/lib/mui';

export const PostList = (props) => (
    <List {...props}>
        <Datagrid>
            <TextField source="id" sortable={false} />
            <TextField source="title" />
            <TextField source="body" />
        </Datagrid>
    </List>
);
```
{% endraw %}

### filter（永久过滤器）{#permanent-filter}

您可以选择始终过滤列表，而不允许用户禁用此过滤器，例如只显示已发布的帖子。在`filter`属性中写被传递到REST客户端的过滤器：

{% raw %}
```jsx
// in src/posts.js
export const PostList = (props) => (
    <List {...props} filter={{ is_published: true }}>
        ...
    </List>
);
```
{% endraw %}

发送到REST客户端的实际过滤器参数是*user*过滤器（通过`filters`组件表单设置的）和*permanent*过滤器组合的结果。用户不能通过`filter`的方式来重写设置的永久筛选器。

### pagination（分页）{#pagination}

你可以通过你自己分页元素来替换默认分页元素，使用`pagination`属性。分页元素接收当前页、每页的记录数、记录总数以及更改页面的`setPage()`函数。

因此，如果您想要通过一个"<previous - next>"分页来替换默认的分页。创建如下所示的页状组件：

```jsx
import FlatButton from 'material-ui/FlatButton';
import ChevronLeft from 'material-ui/svg-icons/navigation/chevron-left';
import ChevronRight from 'material-ui/svg-icons/navigation/chevron-right';
import { Toolbar, ToolbarGroup } from 'material-ui/Toolbar';

const PostPagination = ({ page, perPage, total, setPage }) => {
    const nbPages = Math.ceil(total / perPage) || 1;
    return (
        nbPages > 1 &&
            <Toolbar>
                <ToolbarGroup>
                {page > 1 &&
                    <FlatButton primary key="prev" label="Prev" icon={<ChevronLeft />} onClick={() => setPage(page - 1)} />
                }
                {page !== nbPages &&
                    <FlatButton primary key="next" label="Next" icon={<ChevronRight />} onClick={() => setPage(page + 1)} labelPosition="before" />
                }
                </ToolbarGroup>
            </Toolbar>
    );
}

export const PostList = (props) => (
    <List {...props} pagination={<PostPagination />}>
        ...
    </List>
);
```

### `<Datagrid>`组件{#Datagrid}

datagrid组件渲染纪录列表作为一个表格。它通常被用作[`<List>`](#the-list-component)和 [`<ReferenceManyField>`](./Fields.html#referencemanyfield)组件的子级。

以下是通过这个组件接受的所有属性：

* [`styles`](#custom-grid-style)
* [`rowStyle`](#row-style-function)
* [`options`](#options)
* [`headerOptions`](#options)
* [`bodyOptions`](#options)
* [`rowOptions`](#options)

它渲染的列数与接收的`<Field>`子级相同。

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
            <EditButton />
        </Datagrid>
    </List>
);
```

datagrid是一个*迭代器*组件：它接收一个ids数组，和一个数据store，并且应该遍历ids以显示每个记录。迭代器组件的另一个例子是[`<SingleFieldList>`](#the-singlefieldlist-component)。

### 自定义表格样式{#CustomGridStyle}

你可以通过一个`styles`对象作为属性自定义datagrid样式。该对象应具有以下属性：

```jsx
const datagridStyles = {
    table: { },
    tbody: { },
    tr: { },
    header: {
        th: { },
        'th:first-child': { }, // special style for the first header column
    },
    cell: {
        td: { },
        'td:first-child': { }, // special style for the first column
    },
};

export const PostList = (props) => (
    <List {...props}>
        <Datagrid styles={datagridStyles}>
            ...
        </Datagrid>
    </List>
);
```

**提示**：如果你想要为每列独立地重写`header`和`cell`，在`<Field>`组件中使用`headerStyle`和`style`属性：

{% raw %}
```jsx
export const PostList = (props) => (
    <List {...props}>
        <Datagrid>
            <TextField source="id" />
            <TextField source="title" />
            <TextField
                source="views"
                style={{ textAlign: 'right' }}
                headerStyle={{ textAlign: 'right' }}
            />
        </Datagrid>
    </List>
);
```
{% endraw %}

**提示**：如果您想更进一步，并按单元格应用一个自定义样式单元格，请查看[主题章节的条件格式部分](./Theming.html#conditional-formatting.)

### 行样式函数{#RowStyleFunction}

你可以基于记录自定义datagrid行样式（适用于`<tr>`元素），感谢`rowStyle`属性，它期望是一个函数。

例如，如果记录的一个值 - 如其视图的数量 - 通过某个阈值，则允许对整个行应用自定义背景。

```jsx
const postRowStyle = (record, index) => ({
    backgroundColor: record.nb_views >= 500 ? '#efe' : 'white',
});
export const PostList = (props) => (
    <List {...props}>
        <Datagrid rowStyle={postRowStyle}>
            ...
        </Datagrid>
    </List>
);
```

### `options`、`headerOptions`、`bodyOptions`和`rowOptions`{#options-headerOptions-bodyOptions-rowOptions}

Admin-on-rest依赖[material-ui的`<Table>`组件](http://www.material-ui.com/#/components/table)用于呈现datagrid。`options`、 `headerOptions`、`bodyOptions`和`rowOptions`属性允许你去重写`<Table>`、`<TableHeader>`、`<TableBody>`和`<TableRow>`的属性。

例如，若要获得表上的一个固定的标头，重写`<Table>`属性用`options`：

{% raw %}
```jsx
export const PostList = (props) => (
    <List {...props}>
        <Datagrid options={{ fixedHeader: true, height: 400 }}>
            ...
        </Datagrid>
    </List>
);
```
{% endraw %}

若要启用条纹的行和行悬停，重写`<TableBody>`属性用`bodyOptions`：

{% raw %}
```jsx
export const PostList = (props) => (
    <List {...props}>
        <Datagrid bodyOptions={{ stripedRows: true, showRowHover: true }}>
            ...
        </Datagrid>
    </List>
);
```
{% endraw %}

有关你可以通过这些选项覆盖所有可用的属性列表，请参阅[material-ui`<Table>`组件文档](http://www.material-ui.com/#/components/table)。

### `<SimpleList>`组件{#SimpleList}

对于移动设备，`<Datagrid>`通常是不可用的-仅仅没有足够的空间来显示多列。在这种情况下，约定是使用简单列表，它具有每行只有一列。`<SimpleList>`组件为这个目的服务，利用[material-ui的`<List>`和`<ListItem>`组件](http://www.material-ui.com/#/components/list)。你可以使用它作为`<List>`或者`<ReferenceManyField>`子级：

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

`<SimpleList>`遍历列表数据。对于每个记录，它执行primaryText、secondaryText、 leftAvatar、leftIcon、rightAvatar和rightIcon的属性函数，并传递结果作为相应的<ListItem>属性。

**提示**：在小屏幕上使用一个`<SimpleList>`并在较大的屏幕上使用一个`<Datagrid>`，使用`<Responsive>`组件：

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
                    ...
                </Datagrid>
            }
        />
    </List>
);
```

### `<SingleFieldList>`组件{#SingleFieldList}

当你想要只显示记录列表的一个属性，而不是使用`<Datagrid>`，请使用`<SingleFieldList>`。它期望单个`<Field>`作为子级。它特别适用于`<ReferenceManyField>`组件：


```jsx
// Display all the books by the current author
<ReferenceManyField reference="books" target="author_id">
    <SingleFieldList>
        <ChipField source="title" />
    </SingleFieldList>
</ReferenceManyField>
```

![ReferenceManyFieldSingleFieldList](https://marmelab.com/admin-on-rest/img/reference-many-field-single-field-list.png)

### 使用一个自定义迭代器{#Using-a-Custom-Iterator}

一个`<List>`能委托到任何迭代器组件 - `<Datagrid>`仅仅是一个例子。一个迭代器组件必须接受至少两个属性：

- `ids`是当前显示在列表中的id的数组。
- `data`是一个用于此resource的所有取来数据的对象，通过id索引。

例如，如果您希望显示卡片列表而不是datagrid，该怎么办？

![Custom iterator](https://marmelab.com/admin-on-rest/img/custom-iterator.png)

很简单： 创建您自己的迭代器组件，如下所示：

{% raw %}
```jsx
// in src/comments.js
const cardStyle = {
    width: 300,
    minHeight: 300,
    margin: '0.5em',
    display: 'inline-block',
    verticalAlign: 'top'
};
const CommentGrid = ({ ids, data, basePath }) => (
    <div style={{ margin: '1em' }}>
    {ids.map(id =>
        <Card key={id} style={cardStyle}>
            <CardHeader
                title={<TextField record={data[id]} source="author.name" />}
                subtitle={<DateField record={data[id]} source="created_at" />}
                avatar={<Avatar icon={<PersonIcon />} />}
            />
            <CardText>
                <TextField record={data[id]} source="body" />
            </CardText>
            <CardText>
                about&nbsp;
                <ReferenceField label="Post" resource="comments" record={data[id]} source="post_id" reference="posts" basePath={basePath}>
                    <TextField source="title" />
                </ReferenceField>
            </CardText>
            <CardActions style={{ textAlign: 'right' }}>
                <EditButton resource="posts" basePath={basePath} record={data[id]} />
            </CardActions>
        </Card>
    )}
    </div>
);
CommentGrid.defaultProps = {
    data: {},
    ids: [],
};

export const CommentList = (props) => (
    <List title="All comments" {...props}>
        <CommentGrid />
    </List>
);
```
{% endraw %}

如你所见，没有任何东西阻止你使用`<Field>`组件在自己的组件之内..。另外, 请注意, 组件构建链接需要`basePath`组件, 它也会被注入。
