# 主题化{#Theming}

无论您需要为单个组件调整CSS规则，还是更改整个应用程序中的标签颜色，您将被覆盖！

### 重写组件样式{#Overriding-A-Component-Style}

大多数admin-on-rest组件支持两种样式属性来设置内联样式：

*`style`：一个样式对象，用于定制组件容器的外观（例如数据表格中的`<td>`）。大多数时候，这就是你想要放置自定义样式的地方。

*`elStyle`：一个样式对象，用于定制组件元素本身的外观，通常是一个material ui组件。根据[their styling documentation](http://www.material-ui.com/#/customization/styles)，当您想要微调material ui组件的显示时，请使用此属性。

这些属性接受一个样式对象：

{% raw %}
```jsx
import { EmailField } from 'admin-on-rest/mui';

<EmailField source="email" style={{ backgroundColor: 'lightgrey' }} elStyle={{ textDecoration: 'none' }} />
// renders in the datagrid as
<td style="background-color:lightgrey">
    <a style="text-decoration:none" href="mailto:foo@example.com">
        foo@example.com
    </a>
</td>
```
{% endraw %}

一些组件支持额外的属性来设计自己的元素。例如，当使用`<Datagrid>`时，可以指定`<Field>`如何使用`headerStyle`渲染头部。以下是如何使列右对齐：

{% raw %}
```jsx
export const ProductList = (props) => (
    <List {...props}>
        <Datagrid>
            <TextField source="sku" />
            <TextField
                source="price"
                style={{ textAlign: 'right' }}
                headerStyle={{ textAlign: 'right' }}
            />
            <EditButton />
        </Datagrid>
    </List>
);
```
{% endraw %}

请参阅每个组件文档以获取支持的样式属性列表。

如果您需要更多地控制HTML代码，您还可以创建自己的[Field](./Fields.html#writing-your-own-field-component)和[Input](./Inputs.html#writing-your-own-input-component)组件。

### 条件格式{#Conditional-Formatting}

有时您希望格式取决于值。Admin-on-rest不提供任何特殊的方法来实现，因为已经有了所有必要的-特别是高阶组件（HOC）。

例如，如果要高亮显示`<TextField>`为红色，如果值高于100，只需将该字段包含到HOC中：

{% raw %}
```jsx
const colored = WrappedComponent => props => props.record[props.source] > 100 ?
    <span style={{ color: 'red' }}><WrappedComponent {...props} /></span> :
    <WrappedComponent {...props} />;

const ColoredTextField = colored(TextField);

export const PostList = (props) => (
    <List {...props}>
        <Datagrid>
            <TextField source="id" />
            ...
            <ColoredTextField source="nb_views" />
            <EditButton />
        </Datagrid>
    </List>
);
```
{% endraw %}

如果您想了解更多关于高阶组件的信息，请查看此教程： [Higher Order Components: A React Application Design Pattern](https://www.sitepoint.com/react-higher-order-components/)

### 响应效用{#Responsive-Utility}

要在移动设备，平板电脑和桌面设备上提供优化的体验，您通常需要根据屏幕尺寸显示不同的组件。 这就是`<Responsive>`组件的目的，它提供了响应式网页设计的声明式方法。

它期望的元素属性名为`small`，`medium`和`large`。它显示与屏幕大小匹配的元素（断点为768和992像素）：

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

**提示**：如果只提供`small`和`medium`，那么`medium`元素也可以用在大屏幕上。 当您省略`small`或`medium`时，存在同样的智能默认值。

**提示**：您还可以使用[material-ui's `withWidth()` higher order component](https://github.com/callemall/material-ui/blob/master/src/utils/withWidth.js)在你自己的组件中注入`with`属性。

### 使用预定义的主题{#Using-a-Predefined-Theme}

Material UI也支持[complete theming](http://www.material-ui.com/#/customization/themes)开箱即用。 Material UI载有两个基础主题：明和暗。 默认情况下，Admin-on-rest使用[明]主题。 要使用黑色的，将其传递给`<Admin>`组件，在`theme`属性（以及`getMuiTheme()`）中。

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

### 编写自定义主题{#Writing-a-Custom-Theme}

如果您需要更多精细调整，则需要根据[Material UI themes documentation](http://www.material-ui.com/#/customization/themes)编写自己的`light`对象。Material UI将会合并自定义主题对象与`light`主题。

```jsx
import {
  cyan500, cyan700,
  pinkA200,
  grey100, grey300, grey400, grey500,
  white, darkBlack, fullBlack,
} from 'material-ui/styles/colors';
import { fade } from 'material-ui/utils/colorManipulator';
import spacing from 'material-ui/styles/spacing';

const myTheme = {
    spacing: spacing,
    fontFamily: 'Roboto, sans-serif',
    palette: {
      primary1Color: cyan500,
      primary2Color: cyan700,
      primary3Color: grey400,
      accent1Color: pinkA200,
      accent2Color: grey100,
      accent3Color: grey500,
      textColor: darkBlack,
      alternateTextColor: white,
      canvasColor: white,
      borderColor: grey300,
      disabledColor: fade(darkBlack, 0.3),
      pickerHeaderColor: cyan500,
      clockCircleColor: fade(darkBlack, 0.07),
      shadowColor: fullBlack,
    },
};
```

`muiTheme`对象包含以下键：

* `spacing`可以用来改变组件的间距。
* `fontFamily` 可用于更改默认字体系列。
* `palette` 可以用来改变组件的颜色。
* `zIndex` 可以用来改变每个组件的级别。
* `isRtl` 可用于启用从右到左的模式。
* 每个组件还有一个键，以便您可以单独自定义它们：
  * `appBar`
  * `avatar`
  * ...

**提示**：参阅[Material UI custom colors documentation](http://www.material-ui.com/#/customization/colors)以查看可用于自定义主题的预定义颜色。

一旦你的主题被定义，将它传递给`<Admin>`组件，在`theme`属性中（与`getMuiTheme（）`一起）。

```jsx
const App = () => (
    <Admin theme={getMuiTheme(myTheme)} restClient={simpleRestClient('http://path.to.my.api')}>
        // ...
    </Admin>
);
```

###使用自定义布局{#Using-a-Custom-Layout}

您可以使用自己的组件作为管理布局，而不是默认布局。只需使用`Admin`组件的`appLayout`属性：

```jsx
// in src/App.js
import MyLayout from './MyLayout';

const App = () => (
    <Admin appLayout={MyLayout} restClient={simpleRestClient('http://path.to.my.api')}>
        // ...
    </Admin>
);
```

使用[default layout](https://github.com/marmelab/admin-on-rest/blob/master/src/mui/layout/Layout.js)作为自定义布局的起点。这是一个简化版本（没有响应支持）：

```jsx
// in src/MyLayout.js
import React, { Component } from 'react';
import PropTypes from 'prop-types';
import { connect } from 'react-redux';
import MuiThemeProvider from 'material-ui/styles/MuiThemeProvider';
import CircularProgress from 'material-ui/CircularProgress';
import injectTapEventPlugin from 'react-tap-event-plugin';
import {
    AdminRoutes,
    AppBar,
    Sidebar,
    Notification,
    setSidebarVisibility as setSidebarVisibilityAction
} from 'admin-on-rest';

injectTapEventPlugin();

const styles = {
    wrapper: {
        // Avoid IE bug with Flexbox, see #467
        display: 'flex',
        flexDirection: 'column',
    },
    main: {
        display: 'flex',
        flexDirection: 'column',
        minHeight: '100vh',
    },
    body: {
        backgroundColor: '#edecec',
        display: 'flex',
        flex: 1,
        overflowY: 'hidden',
        overflowX: 'scroll',
    },
    content: {
        flex: 1,
        padding: '2em',
    },
    loader: {
        position: 'absolute',
        top: 0,
        right: 0,
        margin: 16,
        zIndex: 1200,
    },
};

class MyLayout extends Component {
    componentWillMount() {
        this.props.setSidebarVisibility(true);
    }

    render() {
        const {
            authClient,
            customRoutes,
            dashboard,
            isLoading,
            menu,
            resources,
            title,
            width,
        } = this.props;
        return (
            <MuiThemeProvider>
                <div style={styles.wrapper}>
                    <div style={styles.main}>
                        <AppBar title={title} />
                        <div className="body" style={styles.body}>
                            <div style={styles.content}>
                                <AdminRoutes
                                    customRoutes={customRoutes}
                                    resources={resources}
                                    authClient={authClient}
                                    dashboard={dashboard}
                                />
                            </div>
                            <Sidebar>
                                {menu}
                            </Sidebar>
                        </div>
                        <Notification />
                        {isLoading && <CircularProgress
                            color="#fff"
                            size={width === 1 ? 20 : 30}
                            thickness={2}
                            style={styles.loader}
                        />}
                    </div>
                </div>
            </MuiThemeProvider>
        );
    }
}

MyLayout.propTypes = {
    authClient: PropTypes.func,
    customRoutes: PropTypes.array,
    dashboard: PropTypes.oneOfType([PropTypes.func, PropTypes.string]),
    isLoading: PropTypes.bool.isRequired,
    menu: PropTypes.element,
    resources: PropTypes.array,
    setSidebarVisibility: PropTypes.func.isRequired,
    title: PropTypes.string.isRequired,
    width: PropTypes.number,
};

function mapStateToProps(state) {
    return {
        isLoading: state.admin.loading > 0,
    };
}

export default connect(mapStateToProps, {
    setSidebarVisibility: setSidebarVisibilityAction,
})(MyLayout);
```