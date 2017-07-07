# Create和Edit 视图{#CreateEdit}

“创建”和“编辑”视图都显示一个表单，使用空记录初始化的（对于“创建视图”）或具有一条从API（对于“编辑”视图）提取的记录。 `<Create>`和`<Edit>`组件然后将表单的实际渲染委托给表单组件 - 通常是`<SimpleForm>`。 此表单组件使用其子项（[`<Input>`](./Inputs.html)组件）来渲染每个表单输入。

![post creation form](https://marmelab.com/admin-on-rest/img/create-view.png)

![post edition form](https://marmelab.com/admin-on-rest/img/create-view.png)

### `<Create>`和`<Edit>`组件{#CreateEditComponents}

`<Create>`和`<Edit>`组件渲染页面标题和操作，并从REST API获取记录。 他们不负责渲染实际的表单 - 这是他们的子组件（通常是`<SimpleForm>`）的工作，他们将`record`作为属性传递给它们。

这里是`<Create>`和`<Edit>`组件接受的所有属性：

* [`title`](#page-title)
* [`actions`](#actions)

以下是显示一个表单以创建和编辑评论所需的最少代码：

{% raw %}
```jsx
// in src/App.js
import React from 'react';
import { jsonServerRestClient, Admin, Resource } from 'admin-on-rest';

import { PostCreate, PostEdit } from './posts';

const App = () => (
    <Admin restClient={jsonServerRestClient('http://jsonplaceholder.typicode.com')}>
        <Resource name="posts" create={PostCreate} edit={PostEdit} />
    </Admin>
);

export default App;

// in src/posts.js
import React from 'react';
import { Create, Edit, SimpleForm, DisabledInput, TextInput, DateInput, LongTextInput, ReferenceManyField, Datagrid, TextField, DateField, EditButton } from 'admin-on-rest';
import RichTextInput from 'aor-rich-text-input';

export const PostCreate = (props) => (
    <Create {...props}>
        <SimpleForm>
            <TextInput source="title" />
            <TextInput source="teaser" options={{ multiLine: true }} />
            <RichTextInput source="body" />
            <DateInput label="Publication date" source="published_at" defaultValue={new Date()} />
        </SimpleForm>
    </Create>
);

export const PostEdit = (props) => (
    <Edit title={<PostTitle />} {...props}>
        <SimpleForm>
            <DisabledInput label="Id" source="id" />
            <TextInput source="title" validate={required} />
            <LongTextInput source="teaser" validate={required} />
            <RichTextInput source="body" validate={required} />
            <DateInput label="Publication date" source="published_at" />
            <ReferenceManyField label="Comments" reference="comments" target="post_id">
                <Datagrid>
                    <TextField source="body" />
                    <DateField source="created_at" />
                    <EditButton />
                </Datagrid>
            </ReferenceManyField>
        </SimpleForm>
    </Edit>
);
```
{% endraw %}

这足以显示帖子编辑表单：

![post edition form](https://marmelab.com/admin-on-rest/img/post-edition.png)

**提示**：您可能会发现为`<Create>`和`<Edit>`视图重复相同的输入组件很麻烦。 实际上，这两个视图几乎从来没有完全相同的表单输入。 例如，在以前的代码片段中，`<Edit>`视图显示了对当前帖子的相关评论，这对于新的帖子是没有意义的。 因此，为两个视图拥有两组单独的输入组件是一个有意的选择。 但是，如果您具有相同的输入组件集，则将它们导出为自定义Form组件以避免重复。

### title（页面标题）{#page-title}

默认情况下，“创建”视图的标题为“Create [resource_name]”，“编辑”视图的标题为“Edit [resource_name]＃[record_id]”。

您可以通过指定一个自定义的`title`属性来自定义此标题：

```jsx
export const PostEdit = (props) => (
    <Edit title="Post edition" {...props}>
        ...
    </Edit>
);
```

更有趣的是，您可以将组件作为`title`传递。 Admin-on-rest克隆此组件，并在`<EditView>`中注入当前的`record`。 这允许根据当前记录自定义标题：

```jsx
const PostTitle = ({ record }) => {
    return <span>Post {record ? `"${record.title}"` : ''}</span>;
};
export const PostEdit = (props) => (
    <Edit title={<PostTitle />} {...props}>
        ...
    </Edit>
);
```

### actions（动作）{#actions}

您可以通过在你自己元素上使用`actions`属性替换默认操作列表：

```jsx
import { CardActions } from 'material-ui/Card';
import FlatButton from 'material-ui/FlatButton';
import NavigationRefresh from 'material-ui/svg-icons/navigation/refresh';
import { ListButton, ShowButton, DeleteButton } from 'admin-on-rest';

const cardActionStyle = {
    zIndex: 2,
    display: 'inline-block',
    float: 'right',
};

const PostEditActions = ({ basePath, data, refresh }) => (
    <CardActions style={cardActionStyle}>
        <ShowButton basePath={basePath} record={data} />
        <ListButton basePath={basePath} />
        <DeleteButton basePath={basePath} record={data} />
        <FlatButton primary label="Refresh" onClick={refresh} icon={<NavigationRefresh />} />
        {/* Add your custom actions */}
        <FlatButton primary label="Custom Action" onClick={customAction} />
    </CardActions>
);

export const PostEdit = (props) => (
    <Edit actions={<PostEditActions />} {...props}>
        ...
    </Edit>
);
```

### `<SimpleForm>` 组件{#SimpleForm}

`<SimpleForm>`组件从其父组件接收`record`作为属性。 它负责渲染实际的表单。 它还负责验证表单数据。 最后，它收到一个`handleSubmit`函数作为属性，当用户提交表单时，以更新的记录作为参数调用。

默认情况下`<SimpleForm>`会在用户按下`ENTER`时提交表单，如果您想要改变这种行为，你可以传递`false`给`submitOnEnter`属性。

`<SimpleForm>`逐行渲染它的子组件（在`<div>`组件内）。 它使用`redux-form'。

![post edition form](https://marmelab.com/admin-on-rest/img/post-edition.png)

以下是`<SimpleForm>`组件接受的所有属性：

* [`defautValue`](#default-values)
* [`validation`](#validation)
* [`submitOnEnter`](#submit-on-enter)
* [`redirect`](#redirection-after-submission)
* [`toolbar`](#toolbar)

```jsx
export const PostCreate = (props) => (
    <Create {...props}>
        <SimpleForm>
            <TextInput source="title" />
            <RichTextInput source="body" />
            <NumberInput source="nb_views" />
        </SimpleForm>
    </Create>
);
```

### `<TabbedForm>` 组件{#TabbedForm}

就像`<SimpleForm>`一样，`<TabbedForm>`接收`record`属性，渲染实际的表单，并在提交上处理表单验证。 但是，`<TabbedForm>`组件会按选项卡渲染输入。 这些选项卡通过使用`<FormTab>`组件设置，该组件期望一个`label`和一个`icon`属性。

默认情况下，当用户按ENTER键时`<TabbedForm>`提交表单，如果要更改此行为，你可以为submitOnEnter属性传递false。

![tabbed form](https://marmelab.com/admin-on-rest/img/tabbed-form.gif)

以下是`<TabbedForm>`组件接受的所有属性：

* [`defautValue`](#default-values)
* [`validation`](#validation)
* [`submitOnEnter`](#submit-on-enter)
* [`redirect`](#redirection-after-submission)
* [`toolbar`](#toolbar)

{% raw %}
```jsx
import { TabbedForm, FormTab } from 'admin-on-rest'

export const PostEdit = (props) => (
    <Edit {...props}>
        <TabbedForm>
            <FormTab label="summary">
                <DisabledInput label="Id" source="id" />
                <TextInput source="title" validate={required} />
                <LongTextInput source="teaser" validate={required} />
            </FormTab>
            <FormTab label="body">
                <RichTextInput source="body" validate={required} addLabel={false} />
            </FormTab>
            <FormTab label="Miscellaneous">
                <TextInput label="Password (if protected post)" source="password" type="password" />
                <DateInput label="Publication date" source="published_at" />
                <NumberInput source="average_note" validate={[ number, minValue(0) ]} />
                <BooleanInput label="Allow comments?" source="commentable" defaultValue />
                <DisabledInput label="Nb views" source="views" />
            </FormTab>
            <FormTab label="comments">
                <ReferenceManyField reference="comments" target="post_id" addLabel={false}>
                    <Datagrid>
                        <TextField source="body" />
                        <DateField source="created_at" />
                        <EditButton />
                    </Datagrid>
                </ReferenceManyField>
            </FormTab>
        </TabbedForm>
    </Edit>
);
```
{% endraw %}

## defautValue（默认值）{#default-values}

要定义默认值，您可以添加一个`defaultValue`属性到表单组件（`<SimpleForm>`，`<Tabbedform>`等），或者添加一个defaultValue到每个输入组件。

### 全局默认值{#GlobalDefaultValue}

表单`defaultValue`属性的值可以是一个对象或一个返回对象的函数，为创建的记录指定默认值。 例如：

```jsx
const postDefaultValue = { created_at: new Date(), nb_views: 0 };
export const PostCreate = (props) => (
    <Create {...props}>
        <SimpleForm defaultValue={postDefaultValue}>
            <TextInput source="title" />
            <RichTextInput source="body" />
            <NumberInput source="nb_views" />
        </SimpleForm>
    </Create>
);
```

**提示**：你可以在表单`defaultValue`中包含不被列为输入组件的属性，如上例中的created_at属性。

### 每个字段默认值{#PerFieldDefaultValue}

或者，您可以在`<Input>`组件中直接指定一个`defaultValue`。  Admin-on-rest将会将子默认值与表单默认值（input > form）合并：

```jsx
export const PostCreate = (props) => (
    <Create {...props}>
        <SimpleForm>
            <DisabledInput source="id" defaultValue={() => uuid()}/>
            <TextInput source="title" />
            <RichTextInput source="body" />
            <NumberInput source="nb_views" defaultValue={0} />
        </SimpleForm>
    </Create>
);
```

## 验证{#Validation}

Admin-on-rest依赖于[redux-form](http://redux-form.com/)进行验证。

要验证表单提交的值，您可以向表单组件添加一个`validate`属性，将其添加到单个输入中，甚至可以混合使用两种方法。

### 全局验证{#GlobalValidation}

表单`validate`属性的值必须是将记录作为输入的函数，并返回一个具有由字段索引的错误消息的对象。 例如：

```jsx
const validateUserCreation = (values) => {
    const errors = {};
    if (!values.firstName) {
        errors.firstName = ['The firstName is required'];
    }
    if (!values.age) {
        errors.age = ['The age is required'];
    } else if (values.age < 18) {
        errors.age = ['Must be over 18'];
    }
    return errors
};

export const UserCreate = (props) => (
    <Create {...props}>
        <SimpleForm validate={validateUserCreation}>
            <TextInput label="First Name" source="firstName" />
            <TextInput label="Age" source="age" />
        </SimpleForm>
    </Create>
);
```

**提示**：您传递给`<SimpleForm>`和`<TabbedForm>`的属性最后为`reduxForm()`参数。 这意味着除了`validate`之外，还可以传递`warn`或`asyncValidate`函数。 有关详细信息，请阅读[`reduxForm()` documentation](http://redux-form.com/6.5.0/docs/api/ReduxForm.md/)。

### 每个字段验证：函数验证{#PerFieldValidation-FunctionValidator}

或者，您可以在`<Input>`组件中直接指定一个`validate`属性，使用一个函数或一组函数。 当没有错误或错误字符串时，这些函数应该返回`undefined`。

Admin-on-rest将所有单个功能混合到单个函数，就像上一个功能一样：

```jsx
const required = value => value ? undefined : 'Required';
const maxLength = max => value =>
  value && value.length > max ? `Must be ${max} characters or less` : undefined;
const number = value => value && isNaN(Number(value)) ? 'Must be a number' : undefined;
const minValue = min => value =>
  value && value < min ? `Must be at least ${min}` : undefined;

const ageValidation = (value, values) => {
    if (!value) {
        return 'The age is required';
    }
    if (age < 18) {
        return 'Must be over 18';
    }
    return [];
}

export const UserCreate = (props) => (
    <Create {...props}>
        <SimpleForm>
            <TextInput label="First Name" source="firstName" validate={[ required, maxLength(15) ]} />
            <TextInput label="Age" source="age" validate={[ required, number, minValue(18) ]}/>
        </SimpleForm>
    </Create>
);
```

输入验证函数接收当前字段值和当前记录的所有字段的值。这允许复杂的验证场景（例如，验证两个密码是相同的）。

**提示**：验证器函数接收表单`props`作为第三个参数，包括`translate`函数。这样可以建立国际化验证器：

```jsx
const required = (value, allValues, props) => value ? undefined : props.translate('myroot.validation.required');
```

**提示**：将您的输入组件的属性传递给redux-form`<Field>`组件。所以除了`validate`，你也可以使用`warn`。

**提示**：您可以*同时*使用表单验证和输入验证。

### 内置字段验证器{#Built-inFieldValidators}

Admin-on-rest已经捆绑了一些验证器功能，您只需要和使用作为字段验证器：

* `required`如果该字段是强制性的，
* `minValue(min, message)` 指定整数的最小值，
* `maxValue(max, message)` 指定整数的最大值，
* `minLength(min, message)` 指定字符串的最小长度，
* `maxLength(max, message)` 指定字符串的最大长度，
* `email` 检查输入是否是有效的电子邮件地址，
* `regex(pattern, message)` 验证输入与正则表达式匹配，
* `choices(list, message)` 以验证输入是否在给定列表内，

使用示例：

```jsx
import { required, minLength, maxLength, minValue, maxValue, number, regex, email, choices } from 'admin-on-rest';

export const UserCreate = (props) => (
    <Create {...props}>
        <SimpleForm>
            <TextInput label="First Name" source="firstName" validate={[ required, minLength(2), maxLength(15) ]} />
            <TextInput label="Email" source="email" validate={email} />
            <TextInput label="Age" source="age" validate={[ number, minValue(18) ]}/>
            <TextInput label="Zip Code" source="zip" validate={regex(/^\d{5}$/, 'Must be a valid Zip Code')}/>
            <SelectInput label="Sex" source="sex" choices={[
                { id: 'm', name: 'Male' },
                { id: 'f', name: 'Female' },
            ]} validation={choices(['m', 'f'], 'Must be Male or Female')}/>
        </SimpleForm>
    </Create>
);
```

### 回车提交{#SubmitOnEnter}

默认情况下，在任何表单字段中按`ENTER`提交表单 - 这是大多数情况下的预期行为。 但是，您的某些自定义输入组件（例如，Google Maps小部件）可能会有`ENTER`键的特殊处理程序。在这种情况下，要在enter上禁用自动表单提交，请将表单组件的`submitOnEnter`属性设置为`false`：

```jsx
export const PostEdit = (props) => (
    <Edit {...props}>
        <SimpleForm submitOnEnter={false}>
            ...
        </SimpleForm>
    </Edit>
);
```

### 提交后重定向{#RedirectionAfterSubmission}

默认：

- 在`<Create>`视图中提交表单重定向到`<Edit>`视图
- 在`<Edit>`视图中提交表单重定向到`<List>`视图

您可以通过设置表单组件的`redirect`属性来自定义重定向。可能的值为“edit”，“show”，“list”和`false`以禁用重定向。例如，要在编辑后重定向到`<Show>`视图：

```jsx
export const PostEdit = (props) => (
    <Edit {...props}>
        <SimpleForm redirect="show">
            ...
        </SimpleForm>
    </Edit>
);
```

当用户在其中一个表单字段中按`ENTER`时，这会影响提交按钮和表单提交。

### 工具栏{#Toolbar}

在表单底部，工具栏显示提交按钮。您可以通过设置`toolbar`属性来覆盖此组件，以显示您选择的按钮。

最常见的用例是在`<Create>`视图中显示两个提交按钮：

- 一个是创建并重定向到新资源的“`<Show>`视图
- 一个是在创建后重定向到一个空白的`<Create>`视图（允许批量创建）

![Form toolbar](https://marmelab.com/admin-on-rest/img/form-toolbar.png)

对于该用例，请使用一个具有自定义的`redirect`属性的`<SaveButton>`组件：

```jsx
impot { SaveButton, Toolbar } from 'admin-on-rest';

const PostCreateToolbar = props => <Toolbar {...props} >
    <SaveButton label="post.action.save_and_show" redirect="show" submitOnEnter={true} />
    <SaveButton label="post.action.save_and_add" redirect={false} submitOnEnter={false} raised={false} />
</Toolbar>;

export const PostEdit = (props) => (
    <Edit {...props}>
        <SimpleForm toolbar={<PostCreateToolbar />} redirect="show">
            ...
        </SimpleForm>
    </Edit>
);
```

**提示**：使用admin-on-rest的`<Toolbar>`组件，而不是material-ui的`<Toolbar>`组件。 前者建立在后者之上，并增加了对替代移动布局的支持（因此也是响应）。

**提示**：不要忘记也可以设置Form组件的`redirect`属性来处理通过`ENTER`键的提交。
