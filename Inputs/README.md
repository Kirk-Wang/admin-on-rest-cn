# 输入组件{#InputComponents}

一个输入组件显示一个输入或一个下拉列表，一个单选按钮列表等。这些组件允许编辑记录属性，并且在`<Edit>`，`<Create>`和`<Filter>`视图中是常见的。

```jsx
// in src/posts.js
import React from 'react';
import { Edit, DisabledInput, LongTextInput, ReferenceInput, SelectInput, SimpleForm, TextInput } from 'admin-on-rest';

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
```

所有输入组件都接受以下属性：

* `source`：到view/edit的您的实体的属性名称。此属性是必需的。
* `defaultValue`：当属性为`null`或`undefined`时要设置的值。
* `validate`：当前属性的验证规则（请参阅[Validation Documentation](./CreateEdit.html#validation)）
* `label`：用作输入标签的表头。省略时默认为`source`。
* `style`：用于定制字段容器的外观和风格的样式对象（例如表单中的`<div>`）。
* `elStyle`：一个样式对象，用于定制字段元素本身的外观。

其他一些属性逐步实现。`<TextInput />`和`<NumberInput />`输入也接受以下属性：

* `onBlur`：当表单域失去焦点时调用的函数。它希望收到[React SyntheticEvent](https://facebook.github.io/react/docs/events.html)和该字段的当前值的任意一个。
* `onChange`：在更改表单字段时调用的函数。它希望收到[React SyntheticEvent](https://facebook.github.io/react/docs/events.html)和该字段的新值的任意一个。
* `onFocus`：当字段获取焦点时调用的函数。它以`event`作为参数。

```jsx
<TextInput source="zb_title" label="Title" />
```

**提示**：如果编辑一条具有复杂结构的记录，则可以使用路径作为`source`参数。例如，如果API返回以下“book”记录：

```jsx
{
    id: 1234,
    title: 'War and Peace',
    author: {
        firstName: 'Leo',
        lastName: 'Tolstoi'
    }
}
```

然后你可以显示一个文本输入来编辑作者名字，如下所示：

```jsx
<TextInput source="author.firstName" />
```

**提示**：如果您的界面必须支持多种语言，请勿使用`label`，并将本地化标签放在字典中。有关详细信息，请参阅[Translation documentation](./Translation.html#translating-resource-and-field-names)。

### `<AutocompleteInput>`组件{#AutocompleteInput}

要让用户使用自动填充的下拉列表选择列表中的值，请使用`<AutocompleteInput>`。 它使用[Material ui的`<AutoComplete>`组件](http://www.material-ui.com/#/components/auto-complete)和`fuzzySearch`过滤器。设置`choices`属性来确定选项列表（使用`id`，`name`元组）。

```jsx
import { AutocompleteInput } from 'admin-on-rest';

<AutocompleteInput source="category" choices={[
    { id: 'programming', name: 'Programming' },
    { id: 'lifestyle', name: 'Lifestyle' },
    { id: 'photography', name: 'Photography' },
]} />
```

您还可以自定义用于选项名称和值的属性，这要归功于`optionText`和`optionValue`属性：

```jsx
const choices = [
    { _id: 123, full_name: 'Leo Tolstoi', sex: 'M' },
    { _id: 456, full_name: 'Jane Austen', sex: 'F' },
];
<AutocompleteInput source="author_id" choices={choices} optionText="full_name" optionValue="_id" />
```

`optionText`也接受一个函数，所以你可以随意设置选项文本：

```jsx
const choices = [
   { id: 123, first_name: 'Leo', last_name: 'Tolstoi' },
   { id: 456, first_name: 'Jane', last_name: 'Austen' },
];
const optionRenderer = choice => `${choice.first_name} ${choice.last_name}`;
<AutocompleteInput source="author_id" choices={choices} optionText={optionRenderer} />
```

您可以自定义用于过滤结果的`filter`函数。默认情况下，它是`AutoComplete.fuzzyFilter`，但您可以使用[通过`AutoComplete`提供的功能](http://www.material-ui.com/#/components/auto-complete)的任何一个或你自己的一个函数（`(searchText: string, key: string) => boolean`）：

```jsx
import { AutocompleteInput } from 'admin-on-rest';
import AutoComplete from 'material-ui/AutoComplete';

<AutocompleteInput source="category" filter={AutoComplete.caseInsensitiveFilter} choices={choices} />
```

默认情况下会翻译选择项，因此您可以使用翻译标识符作为选择项：

```jsx
const choices = [
   { id: 'M', name: 'myroot.gender.male' },
   { id: 'F', name: 'myroot.gender.female' },
];
```

然而，在某些情况下（例如在`<ReferenceInput>`内），您可能不希望翻译该选项。在这种情况下，请将`translateChoice`属性设置为false。

```jsx
<AutocompleteInput source="gender" choices={choices} translateChoice={false}/>
```

最后，如果你想要覆盖Material UI的`<AutoComplete>`属性的任何一个，请使用`options`属性：

{% raw %}
```jsx
<AutocompleteInput source="category" options={{
    fullWidth: true,
    filter: AutoComplete.fuzzyFilter,
}} />
```
{% endraw %}

有关详细信息，请参阅[Material UI Autocomplete documentation](http://www.material-ui.com/#/components/auto-complete)。

**提示**：如果要使用相关记录的列表填充`choices`属性，则应使用[`<ReferenceInput>`](#referenceinput)装饰`<AutocompleteInput>`，并将`choices`留空：

```jsx
import { AutocompleteInput, ReferenceInput } from 'admin-on-rest'

<ReferenceInput label="Post" source="post_id" reference="posts">
    <AutocompleteInput optionText="title" />
</ReferenceInput>
```

**提示**：`<AutocompleteInput>`是一个无状态组件，所以它只允许*过滤*选择列表，而不是*扩展*。 如果您需要根据`fetch`调用的结果填充选项列表（如果[`<ReferenceInput>`](#referenceinput)不能满足您的需要），你得[编写自己的输入组件](#writing-your-own-input-component)基于material-ui `<AutoComplete>`组件。

**提示**：Admin-on-rest的`<AutocompleteInput>`只有一个大写字母A，而material-ui的`<AutoComplete>`有一个大写字母A和一个大写字母C。不要混淆组件！

### `<BooleanInput>`与`<NullableBooleanInput>`组件{#BooleanInput-NullableBooleanInput}

`<BooleanInput />`是一个切换按钮，允许您把`true`或`false`值归于一个记录字段。

```jsx
import { BooleanInput } from 'admin-on-rest';

<BooleanInput label="Allow comments?" source="commentable" />
```

![BooleanInput](https://marmelab.com/admin-on-rest/img/boolean-input.png)

此输入不处理`null`值。如果您必须处理非设置的布尔值，则需要`<NullableBooleanInput />`组件。

您可以使用`options`属性传递被Material UI`Toggle`组件支持的任何选项。

{% raw %}
```jsx
<BooleanInput source="finished" options={{
    labelPosition: 'right'
}} />
```
{% endraw %}

有关详细信息，请参阅[Material UI Toggle documentation](http://www.material-ui.com/#/components/toggle)。

`<NullableBooleanInput />`渲染为下拉列表，允许在true，false和null值之间进行选择。

```jsx
import { NullableBooleanInput } from 'admin-on-rest';

<NullableBooleanInput label="Allow comments?" source="commentable" />
```

![NullableBooleanInput](https://marmelab.com/admin-on-rest/img/nullable-boolean-input.png)

### `<CheckboxGroupInput>`组件{#CheckboxGroupInput}

如果要让用户通过显示所有值来选择可能值列表中的多个值，则`<CheckboxGroupInput>`是正确的组件。 设置`choices`属性来确定选项（使用`id`，`name`元组）：

```jsx
import { CheckboxGroupInput } from 'admin-on-rest';

<CheckboxGroupInput source="category" choices={[
    { id: 'programming', name: 'Programming' },
    { id: 'lifestyle', name: 'Lifestyle' },
    { id: 'photography', name: 'Photography' },
]} />
```

![CheckboxGroupInput](https://marmelab.com/admin-on-rest/img/checkbox-group-input.png)

您还可以自定义用于选项名称和值的属性，这要归功于`optionText`和`optionValue`属性：

```jsx
const choices = [
    { _id: 123, full_name: 'Leo Tolstoi', sex: 'M' },
    { _id: 456, full_name: 'Jane Austen', sex: 'F' },
];
<CheckboxGroupInput source="author_id" choices={choices} optionText="full_name" optionValue="_id" />
```

`optionText`也接受一个函数，所以你可以随意设置选项文本：

```jsx
const choices = [
   { id: 123, first_name: 'Leo', last_name: 'Tolstoi' },
   { id: 456, first_name: 'Jane', last_name: 'Austen' },
];
const optionRenderer = choice => `${choice.first_name} ${choice.last_name}`;
<CheckboxGroupInput source="author_id" choices={choices} optionText={optionRenderer} />
```

`optionText`也接受一个React元素，它将被克隆，并接收相关联选项作为`record`属性。 您可以在那里使用Field组件。

```jsx
const choices = [
   { id: 123, first_name: 'Leo', last_name: 'Tolstoi' },
   { id: 456, first_name: 'Jane', last_name: 'Austen' },
];
const FullNameField = ({ record }) => <span>{record.first_name} {record.last_name}</span>;
<CheckboxGroupInput source="gender" choices={choices} optionText={<FullNameField />}/>
```

默认情况下会翻译选择项，因此您可以使用翻译标识符作为选择项：

```jsx
const choices = [
    { id: 'programming', name: 'myroot.category.programming' },
    { id: 'lifestyle', name: 'myroot.category.lifestyle' },
    { id: 'photography', name: 'myroot.category.photography' },
];
```

然而，在某些情况下（例如在`<ReferenceInput>`内），您可能不希望翻译该选择项。在这种情况下，请将`translateChoice`属性设置为false。

```jsx
<CheckboxGroupInput source="gender" choices={choices} translateChoice={false}/>
```

最后，如果要覆盖任何Material UI的`<Checkbox>`属性，请使用`options`属性：

{% raw %}
```jsx
<CheckboxGroupInput source="category" options={{
    labelPosition: 'right'
}} />
```
{% endraw %}

有关详细信息，请参阅[Material UI Checkbox documentation](http://www.material-ui.com/#/components/checkbox)。

### `<DateInput>`组件{#DateInput}

理想的编辑日期，`<DateInput>`呈现一个美丽的具有完全本地化支持的[Date Picker](http://www.material-ui.com/#/components/date-picker)。

```jsx
import { DateInput } from 'admin-on-rest';

<DateInput source="published_at" />
```

![DateInput](https://marmelab.com/admin-on-rest/img/date-input.gif)

您可以通过设置`options`属性覆写Material UI的`<DatePicker>`的任何属性。

{% raw %}
```jsx
<DateInput source="published_at" options={{
    mode: 'landscape',
    minDate: new Date(),
    hintText: 'Choisissez une date',
    DateTimeFormat,
    okLabel: 'OK',
    cancelLabel: 'Annuler'
    locale: 'fr'
}} />
```
{% endraw %}

有关详细信息，请参阅[Material UI Datepicker documentation](http://www.material-ui.com/#/components/date-picker)。

### `<DisabledInput>`组件{#DisabledInput}

当您希望在一个`<Edit>`表单中显示一个记录属性时，不要让用户更新它（例如自动递增的主键），请使用`<DisabledInput>`：

```jsx
import { DisabledInput } from 'admin-on-rest';

<DisabledInput source="id" />
```

![DisabledInput](https://marmelab.com/admin-on-rest/img/disabled-input.png)

**提示**：要将不可编辑的字段添加到`<Edit>`视图中，还可以使用admin-on-rest其中一个`Field`组件：

```jsx
// in src/posts.js
import { Edit, LongTextInput, SimpleForm, TextField } from 'admin-on-rest';

export const PostEdit = (props) => (
    <Edit {...props}>
        <SimpleForm>
            <TextField source="title" /> {/* NOT EDITABLE */}
            <LongTextInput source="body" />
        </SimpleForm>
    </Edit>
);
```

**提示**：您甚至可以使用自己的组件，只要它接受`record`属性：

```jsx
// in src/posts.js
import { Edit, LongTextInput, SimpleForm } from 'admin-on-rest';
const titleStyle = { textOverflow: 'ellipsis', overflow: 'hidden', maxWidth: '20em' };
const Title = ({ record }) => <span style={titleStyle}>{record.title}</span>;
Title.defaultProps = {
    addLabel: true,
};

export const PostEdit = (props) => (
    <Edit {...props}>
        <SimpleForm>
            <Title label="Title" />
            <LongTextInput source="body" />
        </SimpleForm>
    </Edit>
);
```

### `<ImageInput>`组件{#ImageInput}

`<ImageInput>`允许使用[react-dropzone](https://github.com/okonet/react-dropzone)上传一些图片。

![ImageInput](https://marmelab.com/admin-on-rest/img/image-input.png)

使用`<ImageInput>`子级启用预览，如下所示：

```jsx
<ImageInput source="pictures" label="Related pictures" accept="image/*">
    <ImageField source="src" title="title" />
</ImageInput>
```

编写用于显示当前值的自定义字段组件很简单：它是标准的[field](./Fields.html#writing_your_own_field_component)。

当接收到**新的**文件时，`ImageInput`会将一个`rawFile`属性添加到作为子级`record`属性传递的对象中。 这个`rawFile`是新添加的文件[File](https://developer.mozilla.org/en-US/docs/Web/API/File)的实例。这可以用于显示关于自定义字段中的大小或mimetype的信息。

除了admin-on-rest那些之外，`ImageInput`组件还接受所有[react-dropzone properties](https://github.com/okonet/react-dropzone#features)。例如，如果您需要一次上传几个图像，只需将`multiple`DropZone属性添加到您的`<ImageInput />`字段。

如果默认的Dropzone标签不符合您的需要，您可以传递一个`placeholder`属性来覆盖它。属性可以是任何React可以渲染的东西（`PropTypes.node`）：

```jsx
<ImageInput source="pictures" label="Related pictures" accept="image/*" placeholder={<p>Drop your file here</p>}>
    <ImageField source="src" title="title" />
</ImageInput>
```

请注意，图像上传返回一个[File](https://developer.mozilla.org/en/docs/Web/API/File)对象。您有责任根据您的API行为来处理它。您可以将其编码为base64，或将其作为多部分表单数据发送。通过扩展REST客户端，查看[this example](./RestClients.html#decorating-your-rest-client-example-of-file-upload)有关base64编码数据。

### `<FileInput>`组件{#FileInput}

`<FileInput>`允许使用[react-dropzone](https://github.com/okonet/react-dropzone)上传一些文件。

![FileInput](https://marmelab.com/admin-on-rest/img/file-input.png)

使用`<FileInput>`子级启用预览（实际上是一个简单的文件名称列表），如下所示：

```jsx
<FileInput source="files" label="Related files" accept="application/pdf">
    <FileField source="src" title="title" />
</FileInput>
```

编写用于显示当前值的自定义字段组件很简单：它是标准的[field](./Fields.html#writing_your_own_field_component)。

当接收到**新的**文件时，`FileInput`将会将一个`rawFile`属性添加到作为子级`record`属性传递的对象中。 这个`rawFile`是新添加的文件的[File](https://developer.mozilla.org/en-US/docs/Web/API/File)实例。 这可以用于显示关于自定义字段中的大小或mimetype的信息。

除了admin-on-rest那些之外，`FileInput`组件还接受所有[react-dropzone properties](https://github.com/okonet/react-dropzone#features)。例如，如果您需要一次上传多个文件，只需将`multiple`DropZone属性添加到您的`<FileInput />`字段。

如果默认的Dropzone标签不符合您的需要，您可以传递一个`placeholder`属性来覆盖它。属性可以是任何React可以渲染的东西（`PropTypes.node`）：

```jsx
<FileInput source="files" label="Related files" accept="application/pdf" placeholder={<p>Drop your file here</p>}>
    <ImageField source="src" title="title" />
</FileInput>
```

请注意，文件上传返回[File](https://developer.mozilla.org/en/docs/Web/API/File)对象。 您有责任根据您的API行为来处理它。您可以将其编码为base64，或将其作为多部分表单数据发送。通过扩展REST客户端，查看[this example](./RestClients.html#decorating-your-rest-client-example-of-file-upload)关于base64编码数据。

### `<LongTextInput>`组件{#LongTextInput}

`<LongTextInput>`是多行文本值的最佳选择。它呈现为一个自动展开的文本区域。

```jsx
import { LongTextInput } from 'admin-on-rest';

<LongTextInput source="teaser" />
```

![LongTextInput](https://marmelab.com/admin-on-rest/img/long-text-input.png)

### `<NumberInput>`组件{#NumberInput}

`<NumberInput>`转换为HTMl`<input type="number">`。由于[known React bug]（https://github.com/facebook/react/issues/1425），因此数字值是必需的，在那种情况下这阻止了使用更通用的[`<TextInput>`](#textinput)。

```jsx
import { NumberInput } from 'admin-on-rest';

<NumberInput source="nb_views" />
```

您可以自定义`step`属性（默认为“any”）：

```jsx
<NumberInput source="nb_views" step={1} />
```

### `<RadioButtonGroupInput>`组件{#RadioButtonGroupInput}

如果要让用户通过显示的所有值（而不是将它们隐藏在下拉列表之后）选择一个在可能值列表中的值，如[`<SelectInput>`](#selectinput)，`<RadioButtonGroupInput>` 是正确的组件。设置`choices`属性来确定选项（使用`id`，`name`元组）：

```jsx
import { RadioButtonGroupInput } from 'admin-on-rest';

<RadioButtonGroupInput source="category" choices={[
    { id: 'programming', name: 'Programming' },
    { id: 'lifestyle', name: 'Lifestyle' },
    { id: 'photography', name: 'Photography' },
]} />
```

![RadioButtonGroupInput](https://marmelab.com/admin-on-rest/img/radio-button-group-input.png)

您还可以自定义用于选项名称和值的属性，这要归功于`optionText`和`optionValue`属性：

```jsx
const choices = [
    { _id: 123, full_name: 'Leo Tolstoi', sex: 'M' },
    { _id: 456, full_name: 'Jane Austen', sex: 'F' },
];
<RadioButtonGroupInput source="author_id" choices={choices} optionText="full_name" optionValue="_id" />
```

`optionText`也接受一个函数，所以你可以随意设置选项文本：

```jsx
const choices = [
   { id: 123, first_name: 'Leo', last_name: 'Tolstoi' },
   { id: 456, first_name: 'Jane', last_name: 'Austen' },
];
const optionRenderer = choice => `${choice.first_name} ${choice.last_name}`;
<RadioButtonGroupInput source="author_id" choices={choices} optionText={optionRenderer} />
```

`optionText`也接受一个React元素，它将被克隆，并接收相关选择作为`record`属性。您可以在那里使用Field组件。

```jsx
const choices = [
   { id: 123, first_name: 'Leo', last_name: 'Tolstoi' },
   { id: 456, first_name: 'Jane', last_name: 'Austen' },
];
const FullNameField = ({ record }) => <span>{record.first_name} {record.last_name}</span>;
<RadioButtonGroupInput source="gender" choices={choices} optionText={<FullNameField />}/>
```

选项被默认翻译，因此您可以使用翻译标识符作为选项：

```jsx
const choices = [
   { id: 'M', name: 'myroot.gender.male' },
   { id: 'F', name: 'myroot.gender.female' },
];
```

然而，在某些情况下（例如在`<ReferenceInput>`内），您可能不希望翻译该选项。在这种情况下，请将`translateChoice`属性设置为false。

```jsx
<RadioButtonGroupInput source="gender" choices={choices} translateChoice={false}/>
```

最后，如果要覆盖Material UI的`<RadioButtonGroup>`任何属性，请使用`options`属性：

{% raw %}
```jsx
<RadioButtonGroupInput source="category" options={{
    labelPosition: 'right'
}} />
```
{% endraw %}

有关详细信息，请参阅[Material UI SelectField documentation](http://www.material-ui.com/#/components/radio-button)。

提示：如果要使用相关记录列表填充`choices`属性，则应使用[`<ReferenceInput>`](#referenceinput)装饰`<RadioButtonGroupInput>`，并将选项留空：

```jsx
import { RadioButtonGroupInput, ReferenceInput } from 'admin-on-rest'

<ReferenceInput label="Author" source="author_id" reference="authors">
    <RadioButtonGroupInput optionText="last_name" />
</ReferenceInput>
```

### `<ReferenceInput>`组件{#ReferenceInput}

对于外键值使用`<ReferenceInput>`，即让用户从另一个REST端点中选择一个值。该组件在引用资源中获取可能的值（使用`GET_LIST`REST方法），然后将渲染委托给一个子组件，它将可能的选择作为`choices`属性传递给该子组件。

这意味着您可以使用`<ReferenceInput>`具有[`<SelectInput>`](#selectinput), [`<AutocompleteInput>`](#autocompleteinput)或[`<RadioButtonGroupInput>`](#radiobuttongroupinput)任何一个，或者甚至使用您选择的组件，只要它支持`choices`属性。

该组件需要一个`source`和`reference`属性。 例如，要使`comment`的`post_id`可编辑：

```jsx
import { ReferenceInput, SelectInput } from 'admin-on-rest'

<ReferenceInput label="Post" source="post_id" reference="posts">
    <SelectInput optionText="title" />
</ReferenceInput>
```

![ReferenceInput](https://marmelab.com/admin-on-rest/img/reference-input.gif)

**注意**：你**必须**为引用资源添加一个`<Resource>` - admin-on-rest需要它来获取引用数据。如果您想在侧边栏菜单中隐藏它，你**可以**省略此引用中的`list`属性。

```jsx
<Admin restClient={myRestClient}>
    <Resource name="comments" list={CommentList} />
    <Resource name="posts" />
</Admin>
```

当允许空值时，设置`allowEmpty`属性。

```jsx
import { ReferenceInput, SelectInput } from 'admin-on-rest'

<ReferenceInput label="Post" source="post_id" reference="posts" allowEmpty>
    <SelectInput optionText="title" />
</ReferenceInput>
```

**提示**：对于`<Filter>`组件的所有Input组件子项，默认设置`allowEmpty`：

```jsx
const CommentFilter = (props) => (
    <Filter {...props}>
        <ReferenceInput label="Post" source="post_id" reference="posts"> // no need for allowEmpty
            <SelectInput optionText="title" />
        </ReferenceInput>
    </Filter>
);
```

您可以调整此组件如何使用`perPage`，`sort`和`filter`属性获取可能的值。

{% raw %}
```jsx
// by default, fetches only the first 25 values. You can extend this limit
// by setting the `perPage` prop.
<ReferenceInput
     source="post_id"
     reference="posts"
     perPage={100}>
    <SelectInput optionText="title" />
</ReferenceInput>

// by default, orders the possible values by id desc. You can change this order
// by setting the `sort` prop (an object with `field` and `order` properties).
<ReferenceInput
     source="post_id"
     reference="posts"
     sort={{ field: 'title', order: 'ASC' }}>
    <SelectInput optionText="title" />
</ReferenceInput>

// you can filter the query used to populate the possible values. Use the
// `filter` prop for that.
<ReferenceInput
     source="post_id"
     reference="posts"
     filter={{ is_published: true }}>
    <SelectInput optionText="title" />
</ReferenceInput>
```
{% endraw %}

封闭的组件可以进一步过滤结果（例如，对于`<AutocompleteInput>`）。ReferenceInput将一个`setFilter`函数作为属性传递给它的子组件。它使用该值为查询创建一个过滤器 - 默认情况下为`{ q: [searchText] }`。 您可以自定义映射
`searchText => searchQuery`通过设置一个自定义的`filterToQuery`函数属性：

```jsx
<ReferenceInput
     source="post_id"
     reference="posts"
     filterToQuery={searchText => ({ title: searchText })}>
    <SelectInput optionText="title" />
</ReferenceInput>
```

### `<ReferenceArrayInput>`组件{#ReferenceArrayInput}

使用`<ReferenceArrayInput>`编辑引用值数组，即让用户从另一个REST端点中选择值列表（通常是外键）。

在引用端点中，`<ReferenceArrayInput>`获取相关资源（使用`CRUD_GET_MANY`REST方法）与获取可能的资源（使用`CRUD_GET_MATCHING` REST方法）一样

例如，如果post对象有很多标签，则post资源可能如下所示：

```js
{
    id: 1234,
    tag_ids: [1, 23, 4]
}
```

那么`<ReferenceArrayInput>`将从这两个调用中获取标签资源的列表：

```
http://myapi.com/tags?id=[1,23,4]
http://myapi.com/tags?page=1&perPage=25
```

一旦接收到重复数据删除的引用资源，该组件将渲染委托给一个子组件，它将可能的选择作为`choices`属性传递给该子组件。

这意味着您可以使用具有[`<SelectArrayInput>`](#selectarrayinput)的`<ReferenceArrayInput>`，或者使用您选择的组件，只要它支持`choices`属性。

该组件需要一个`source`和`reference`属性。 例如，要使`post`的`tag_ids`可编辑：

```js
import { ReferenceArrayInput, SelectArrayInput } from 'admin-on-rest'

<ReferenceArrayInput source="tag_ids" reference="tags">
    <SelectArrayInput optionText="name" />
</ReferenceArrayInput>
```

![SelectArrayInput](https://marmelab.com/admin-on-rest/img/select-array-input.gif)

**注意**：你*必须*为引用资源添加一个`<Resource>` - admin-on-rest需要它来获取引用数据。 如果要在侧边栏菜单中隐藏它，可以在此引用中省略list属性。

```js
<Admin restClient={myRestClient}>
    <Resource name="posts" list={PostList} edit={PostEdit} />
    <Resource name="tags" />
</Admin>
```

当允许空值时，设置`allowEmpty`属性。

```js
import { ReferenceArrayInput, SelectArrayInput } from 'admin-on-rest'

<ReferenceArrayInput source="tag_ids" reference="tags" allowEmpty>
    <SelectArrayInput optionText="name" />
</ReferenceArrayInput>
```

**提示**：对于`<Filter>`组件的所有Input组件子项，默认设置`allowEmpty`：

您可以调整此组件如何使用`perPage`，`sort`和`filter`属性获取可能的值。

{% raw %}
```js
// by default, fetches only the first 25 values. You can extend this limit
// by setting the `perPage` prop.
<ReferenceArrayInput
     source="tag_ids"
     reference="tags"
     perPage={100}>
    <SelectArrayInput optionText="name" />
</ReferenceArrayInput>

// by default, orders the possible values by id desc. You can change this order
// by setting the `sort` prop (an object with `field` and `order` properties).
<ReferenceArrayInput
     source="tag_ids"
     reference="tags"
     sort={{ field: 'title', order: 'ASC' }}>
    <SelectArrayInput optionText="name" />
</ReferenceArrayInput>

// you can filter the query used to populate the possible values. Use the
// `filter` prop for that.
<ReferenceArrayInput
     source="tag_ids"
     reference="tags"
     filter={{ is_published: true }}>
    <SelectArrayInput optionText="name" />
</ReferenceArrayInput>
```
{% endraw %}

封闭的组件可以进一步过滤结果（例如，对于`<SelectArrayInput>`）。`ReferenceArrayInput`将一个`setFilter`函数作为属性传递给它的子组件。它使用该值为查询创建一个过滤器 - 默认情况下为`{ q: [searchText] }`。 您可以自定义映射
`searchText => searchQuery`通过设置一个自定义的`filterToQuery`函数属性：

```js
<ReferenceArrayInput
     source="tag_ids"
     reference="tags"
     filterToQuery={searchText => ({ name: searchText })}>
    <SelectArrayInput optionText="name" />
</ReferenceArrayInput>
```

### `<RichTextInput>`组件{#RichTextInput}

`<RichTextInput>` is the ideal component if you want to allow your users to edit some HTML contents. It
is powered by [Quill](https://quilljs.com/).

**Note**: Due to its size, `<RichTextInput>` is not bundled by default with admin-on-rest. You must install it first, using npm:

```sh
npm install aor-rich-text-input --save
```

Then use it as a normal input component:

```jsx
import RichTextInput from 'aor-rich-text-input';

<RichTextInput source="body" />
```

![RichTextInput](./img/rich-text-input.png)

You can customize the rich text editor toolbar using the `toolbar` attribute, as described on the [Quill official toolbar documentation](https://quilljs.com/docs/modules/toolbar/).

```jsx
<RichTextInput source="body" toolbar={[ ['bold', 'italic', 'underline', 'link'] ]} />
```

## `<SelectInput>`

To let users choose a value in a list using a dropdown, use `<SelectInput>`. It renders using [Material ui's `<SelectField>`](http://www.material-ui.com/#/components/select-field). Set the `choices` attribute to determine the options (with `id`, `name` tuples):

```jsx
import { SelectInput } from 'admin-on-rest';

<SelectInput source="category" choices={[
    { id: 'programming', name: 'Programming' },
    { id: 'lifestyle', name: 'Lifestyle' },
    { id: 'photography', name: 'Photography' },
]} />
```

![SelectInput](./img/select-input.gif)

You can also customize the properties to use for the option name and value, thanks to the `optionText` and `optionValue` attributes:

```jsx
const choices = [
    { _id: 123, full_name: 'Leo Tolstoi', sex: 'M' },
    { _id: 456, full_name: 'Jane Austen', sex: 'F' },
];
<SelectInput source="author_id" choices={choices} optionText="full_name" optionValue="_id" />
```

`optionText` also accepts a function, so you can shape the option text at will:

```jsx
const choices = [
   { id: 123, first_name: 'Leo', last_name: 'Tolstoi' },
   { id: 456, first_name: 'Jane', last_name: 'Austen' },
];
const optionRenderer = choice => `${choice.first_name} ${choice.last_name}`;
<SelectInput source="author_id" choices={choices} optionText={optionRenderer} />
```

`optionText` also accepts a React Element, that will be cloned and receive the related choice as the `record` prop. You can use Field components there.

```jsx
const choices = [
   { id: 123, first_name: 'Leo', last_name: 'Tolstoi' },
   { id: 456, first_name: 'Jane', last_name: 'Austen' },
];
const FullNameField = ({ record }) => <span>{record.first_name} {record.last_name}</span>;
<SelectInput source="gender" choices={choices} optionText={<FullNameField />}/>
```

Enabling the `allowEmpty` props adds an empty choice (with `null` value) on top of the options, and makes the value nullable:

```jsx
<SelectInput source="category" allowEmpty choices={[
    { id: 'programming', name: 'Programming' },
    { id: 'lifestyle', name: 'Lifestyle' },
    { id: 'photography', name: 'Photography' },
]} />
```

The choices are translated by default, so you can use translation identifiers as choices:

```jsx
const choices = [
   { id: 'M', name: 'myroot.gender.male' },
   { id: 'F', name: 'myroot.gender.female' },
];
```

However, in some cases, you may not want the choice to be translated. In that case, set the `translateChoice` prop to false.

```jsx
<SelectInput source="gender" choices={choices} translateChoice={false}/>
```

Note that `translateChoice` is set to false when `<SelectInput>` is a child of `<ReferenceInput>`.

Lastly, use the `options` attribute if you want to override any of Material UI's `<SelectField>` attributes:

{% raw %}
```jsx
<SelectInput source="category" options={{
    maxHeight: 200
}} />
```
{% endraw %}

Refer to [Material UI SelectField documentation](http://www.material-ui.com/#/components/select-field) for more details.

**Tip**: If you want to populate the `choices` attribute with a list of related records, you should decorate `<SelectInput>` with [`<ReferenceInput>`](#referenceinput), and leave the `choices` empty:

```jsx
import { SelectInput, ReferenceInput } from 'admin-on-rest'

<ReferenceInput label="Author" source="author_id" reference="authors">
    <SelectInput optionText="last_name" />
</ReferenceInput>
```

If, instead of showing choices as a dropdown list, you prefer to display them as a list of radio buttons, try the [`<RadioButtonGroupInput>`](#radiobuttongroupinput). And if the list is too big, prefer the [`<AutocompleteInput>`](#autocompleteinput).

## `<SelectArrayInput>`

To let users choose several values in a list using a dropdown, use `<SelectArrayInput>`. It renders using [material-ui-chip-input](https://github.com/TeamWertarbyte/material-ui-chip-input). Set the `choices` attribute to determine the options (with `id`, `name` tuples):

```js
import { SelectArrayInput } from 'admin-on-rest';

<SelectArrayInput label="Tags" source="categories" choices={[
    { id: 'music', name: 'Music' },
    { id: 'photography', name: 'Photo' },
    { id: 'programming', name: 'Code' },
    { id: 'tech', name: 'Technology' },
    { id: 'sport', name: 'Sport' },
]} />
```

![SelectArrayInput](./img/select-array-input.gif)

You can also customize the properties to use for the option name and value,
thanks to the `optionText` and `optionValue` attributes.

```js
const choices = [
   { _id: '1', name: 'Book', plural_name: 'Books' },
   { _id: '2', name: 'Video', plural_name: 'Videos' },
   { _id: '3', name: 'Audio', plural_name: 'Audios' },
];
<SelectArrayInput source="categories" choices={choices} optionText="plural_name" optionValue="_id" />
```

`optionText` also accepts a function, so you can shape the option text at will:

```js
const choices = [
   { id: '1', name: 'Book', quantity: 23 },
   { id: '2', name: 'Video', quantity: 56 },
   { id: '3', name: 'Audio', quantity: 12 },
];
const optionRenderer = choice => `${choice.name} (${choice.quantity})`;
<SelectArrayInput source="categories" choices={choices} optionText={optionRenderer} />
```

The choices are translated by default, so you can use translation identifiers as choices:

```js
const choices = [
   { id: 'books', name: 'myroot.category.books' },
   { id: 'sport', name: 'myroot.category.sport' },
];
```

However, in some cases, you may not want the choice to be translated. In that case, set the `translateChoice` prop to false.

```js
<SelectArrayInput source="gender" choices={choices} translateChoice={false}/>
```

Note that `translateChoice` is set to false when `<SelectArrayInput>` is a child of `<ReferenceArrayInput>`.

Lastly, use the `options` attribute if you want to override any of the `<ChipInput>` attributes:

{% raw %}
```js
<SelectArrayInput source="category" options={{ fullWidth: true }} />
```
{% endraw %}

Refer to [the ChipInput documentation](https://github.com/TeamWertarbyte/material-ui-chip-input) for more details.

**Tip**: If you want to populate the `choices` attribute with a list of related records, you should decorate `<SelectArrayInput>` with [`<ReferenceArrayInput>`](#referencearrayinput), and leave the `choices` empty:

```js
import { SelectArrayInput, ReferenceArrayInput } from 'admin-on-rest'

<ReferenceArrayInput source="tag_ids" reference="tags">
    <SelectArrayInput optionText="name" />
</ReferenceArrayInput>
```

## `<TextInput>`

`<TextInput>` is the most common input. It is used for texts, emails, URL or passwords. In translates to an HTML `<input>` tag.

```jsx
import { TextInput } from 'admin-on-rest';

<TextInput source="title" />
```

![TextInput](./img/text-input.png)

You can choose a specific input type using the `type` attribute, for instance `text` (the default), `email`, `url`, or `password`:

```jsx
<TextInput label="Email Address" source="email" type="email" />
```

**Warning**: Do not use `type="number"`, or you'll receive a string as value (this is a [known React bug](https://github.com/facebook/react/issues/1425)). Instead, use [`<NumberInput>`](#numberinput).

## Transforming Input Value to/from Store

The data format returned by the input component may not be what your store desires. Since Admin-on-rest uses Redux Form, we can use its `parse()` and `format()` functions to transform the input value to and from the store. It's better to understand the [input value's lifecycle](http://redux-form.com/6.5.0/docs/ValueLifecycle.md/) before you start.

Mnemonic for the two functions:
- `parse()`: input -> store
- `format()`: store -> input

Say the user would like to input values of 0-100 to a percentage field but your API (hence store) expects 0-1.0. You can use simple `parse()` and `format()` functions to archive the transform:

```jsx
<NumberInput source="percent" format={v => v*100} parse={v => v/100} label="Formatted number" />
```

`<DateInput>` stores and returns a `Date` object. If you would like to store the ISO date `"YYYY-MM-DD"` in your store:

```jsx
const dateFormatter = v => {
  // v is a string of "YYYY-MM-DD" format
  const match = /(\d{4})-(\d{2})-(\d{2})/.exec(v);
  if (match === null) return;
  const d = new Date(match[1], parseInt(match[2])-1, match[3]);
  if (isNaN(d)) return;
  return d;
};

const dateParser = v => {
  // v is a `Date` object
  if (!(v instanceof Date) || isNaN(v)) return;
  const pad = '00';
  const yy = v.getFullYear().toString();
  const mm = ((v.getMonth() + 1).toString();
  const dd = v.getDate().toString();
  return `${yy}-${(pad + mm).slice(-2)}-${(pad + dd).slice(-2)}`;
};

<DateInput source="isodate" format={dateFormatter} parse={dateParser} label="ISO date" />
```

## Third-Party Components

You can find components for admin-on-rest in third-party repositories.

* [dreinke/aor-color-input](https://github.com/dreinke/aor-color-input): a color input using [React Color](http://casesandberg.github.io/react-color/), a collection of color pickers.
* [LoicMahieu/aor-tinymce-input](https://github.com/LoicMahieu/aor-tinymce-input): a TinyMCE component, useful for editing HTML

## Writing Your Own Input Component

If you need a more specific input type, you can also write it yourself. You'll have to rely on redux-form's [`<Field>`](http://redux-form.com/6.5.0/docs/api/Field.md/) component, so as to handle the value update cycle.

For instance, let's write a component to edit the latitude and longitude of the current record:

```jsx
// in LatLongInput.js
import { Field } from 'redux-form';
const LatLngInput = () => (
    <span>
        <Field name="lat" component="input" type="number" placeholder="latitude" />
        &nbsp;
        <Field name="lng" component="input" type="number" placeholder="longitude" />
    </span>
);
export default LatLngInput;

// in ItemEdit.js
const ItemEdit = (props) => (
    <Edit {...props}>
        <SimpleForm>
            <LatLngInput />
        </SimpleForm>
    </Edit>
);
```

`LatLngInput` takes no props, because the `<Field>` component can access the current record via its context. The `name` prop serves as a selector for the record property to edit. All `Field` props except `name` and `component` are passed to the child component/element (an `<input>` in that example). Executing this component will render roughly the following code:

```html
<span>
    <input type="number" placeholder="longitude" value={record.lat} />
    <input type="number" placeholder="longitude" value={record.lng} />
</span>
```

This component lacks a label. Admin-on-rest provides the `<Labeled>` component for that:

```jsx
// in LatLongInput.js
import { Field } from 'redux-form';
import { Labeled } from 'admin-on-rest';
const LatLngInput = () => (
    <Labeled label="position">
        <span>
            <Field name="lat" component="input" type="number" placeholder="latitude" />
            &nbsp;
            <Field name="lng" component="input" type="number" placeholder="longitude" />
        </span>
    </Labelled>
);
export default LatLngInput;
```

Now the component will render with a label:

```html
<label>Position</label>
<span>
    <input type="number" placeholder="longitude" value={record.lat} />
    <input type="number" placeholder="longitude" value={record.lng} />
</span>
```

Adding a label to an input component is such a common operation that admin-on-rest has the ability to do it automatically: just set the `addLabel` prop, and specify the label in the `label` prop:

```jsx
// in LatLongInput.js
import { Field } from 'redux-form';
const LatLngInput = () => (
    <span>
        <Field name="lat" component="input" type="number" placeholder="latitude" />
        &nbsp;
        <Field name="lng" component="input" type="number" placeholder="longitude" />
    </span>
);
export default LatLngInput;
// in ItemEdit.js
const ItemEdit = (props) => (
    <Edit {...props}>
        <SimpleForm>
            <LatLngInput addLabel label="Position" />
        </SimpleForm>
    </Edit>
);
```

**Tip**: To avoid repeating them each time you use the component, you should define `label` and `addLabel` as `defaultProps`:

```jsx
// in LatLongInput.js
import { Field } from 'redux-form';
const LatLngInput = () => (
    <span>
        <Field name="lat" component="input" type="number" placeholder="latitude" />
        &nbsp;
        <Field name="lng" component="input" type="number" placeholder="longitude" />
    </span>
);
LatLngInput.defaultProps = {
    addLabel: true,
    label: 'Position',
}
export default LatLngInput;
// in ItemEdit.js
const ItemEdit = (props) => (
    <Edit {...props}>
        <SimpleForm>
            <LatLngInput />
        </SimpleForm>
    </Edit>
);
```

**Tip**: The `<Field>` component supports dot notation in the `name` prop, to edit nested props:

```jsx
const LatLongInput = () => (
    <span>
        <Field name="position.lat" component="input" type="number" placeholder="latitude" />
        &nbsp;
        <Field name="position.lng" component="input" type="number" placeholder="longitude" />
    </span>
);
```

Instead of HTML `input` elements, you can use admin-on-rest components in `<Field>`. For instance, `<NumberInput>`:

```jsx
// in LatLongInput.js
import { Field } from 'redux-form';
import { NumberInput } from 'admin-on-rest';
const LatLngInput = () => (
    <span>
        <Field name="lat" component={NumberInput} label="latitude" />
        &nbsp;
        <Field name="lng" component={NumberInput} label="longitude" />
    </span>
);
export default LatLngInput;

// in ItemEdit.js
const ItemEdit = (props) => (
    <Edit {...props}>
        <SimpleForm>
            <DisabledInput source="id" />
            <LatLngInput />
        </SimpleForm>
    </Edit>
);
```

`<NumberInput>` receives the props passed to the `<Field>` component - `label` in the example. `<NumberInput>` is already labelled, so there is no need to also label the `<LanLngInput>` component - that's why `addLabel` isn't set as default prop this time.

**Tip**: If you need to pass a material ui component to `Field`, use a [field renderer function](http://redux-form.com/6.5.0/examples/material-ui/) to map the props:

```jsx
import TextField from 'material-ui/TextField';
const renderTextField = ({ input, label, meta: { touched, error }, ...custom }) => (
    <TextField
        hintText={label}
        floatingLabelText={label}
        errorText={touched && error}
        {...input}
        {...custom}
    />
);
const LatLngInput = () => (
    <span>
        <Field name="lat" component={renderTextField} label="latitude" />
        &nbsp;
        <Field name="lng" component={renderTextField} label="longitude" />
    </span>
);
```

For more details on how to use redux-form's `<Field>` component, please refer to [the redux-form doc](http://redux-form.com/6.5.0/docs/api/Field.md/).

**Tip**: If you only need one `<Field>` component in a custom input, you can let admin-on-rest do the `<Field>` decoration for you by setting the `addField` default prop to `true`:

```jsx
// in PersonEdit.js
import SexInput from './SexInput.js';
const PersonEdit = (props) => (
    <Edit {...props}>
        <SimpleForm>
            <SexInput source="sex" />
        </SimpleForm>
    </Edit>
);

// in SexInput.js
import SelectField from 'material-ui/SelectField';
import MenuItem from 'material-ui/MenuItem';
const SexInput = ({ input, meta: { touched, error } }) => (
    <SelectField
        floatingLabelText="Sex"
        errorText={touched && error}
        {...input}
    >
        <MenuItem value="M" primaryText="Male" />
        <MenuItem value="F" primaryText="Female" />
    </SelectField>
);
SexInput.defaultProps = {
    addField: true, // require a <Field> decoration
}
export default SexInput;

// equivalent of

import SelectField from 'material-ui/SelectField';
import MenuItem from 'material-ui/MenuItem';
import { Field } from 'redux-form';
const renderSexInput = ({ input, meta: { touched, error } }) => (
    <SelectField
        floatingLabelText="Sex"
        errorText={touched && error}
        {...input}
    >
        <MenuItem value="M" primaryText="Male" />
        <MenuItem value="F" primaryText="Female" />
    </SelectField>
);
const SexInput = ({ source }) => <Field name={source} component={renderSexInput} />
export default SexInput;
```

Most admin-on-rest input components use `addField: true` in default props.

**Tip**: `<Field>` injects two props to its child component: `input` and `meta`. To learn more about these props, please refer to [the `<Field>` component documentation](http://redux-form.com/6.5.0/docs/api/Field.md/#props) in the redux-form website.
