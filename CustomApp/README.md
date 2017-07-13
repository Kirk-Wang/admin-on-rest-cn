# 在其它React app中包含admin-on-rest{#Including-admin-on-rest-on-another-React-app}

`<Admin>`标签是一个很好的快捷方式，在几分钟内就可以启动和运行admin-on-rest。但是，在许多情况下，您需要将admin嵌入到另一个应用程序中，或者深入地定制admin。幸运的是，您可以在任何React应用程序中执行`<Admin>`所做的所有工作。

请注意，您需要进一步了解[redux](http://redux.js.org/)，[react-router](https://github.com/reactjs/react-router)和[redux-saga](https://github.com/yelouafi/redux-saga)。

**提示**：在进入自定义应用程序路线之前，请探索[the `<Admin>` component](./Admin.html)的所有选项。 它们允许您添加自定义路由，自定义reducers，自定义sagas和自定义布局。

以下是使用3个资源引导一个准系统admin-on-rest应用程序的主要代码：`posts`，`comments`和'users`：

```jsx
// in src/App.js
import React from 'react';
import PropTypes from 'prop-types';
import { render } from 'react-dom';

// redux, react-router, redux-form, saga, and material-ui
// form the 'kernel' on which admin-on-rest runs
import { combineReducers, createStore, compose, applyMiddleware } from 'redux';
import { Provider } from 'react-redux';
import createHistory from 'history/createHashHistory';
import { Switch, Route } from 'react-router-dom';
import { ConnectedRouter, routerReducer, routerMiddleware } from 'react-router-redux';
import { reducer as formReducer } from 'redux-form';
import createSagaMiddleware from 'redux-saga';
import MuiThemeProvider from 'material-ui/styles/MuiThemeProvider';
import AppBar from 'material-ui/AppBar';

// prebuilt admin-on-rest features
import {
    adminReducer,
    localeReducer,
    crudSaga,
    simpleRestClient,
    Delete,
    TranslationProvider,
} from 'admin-on-rest';

// your app components
import Dashboard from './Dashboard';
import { PostList, PostCreate, PostEdit, PostShow } from './Post';
import { CommentList, CommentEdit, CommentCreate } from './Comment';
import { UserList, UserEdit, UserCreate } from './User';
// your app labels
import messages from './i18n';

// create a Redux app
const reducer = combineReducers({
    admin: adminReducer([{ name: 'posts' }, { name: 'comments' }, { name: 'users' }]),
    locale: localeReducer(),
    form: formReducer,
    routing: routerReducer,
});
const sagaMiddleware = createSagaMiddleware();
const history = createHistory();
const store = createStore(reducer, undefined, compose(
    applyMiddleware(sagaMiddleware, routerMiddleware(history)),
    window.devToolsExtension ? window.devToolsExtension() : f => f,
));
const restClient = simpleRestClient('http://path.to.my.api/');
sagaMiddleware.run(crudSaga(restClient));

// bootstrap redux and the routes
const App = () => (
    <Provider store={store}>
        <TranslationProvider messages={messages}>
            <ConnectedRouter history={history}>
                <MuiThemeProvider>
                    <AppBar title="My Admin" />
                    <Switch>
                        <Route exact path="/" component={Dashboard} />
                        <Route exact path="/posts" hasCreate render={(routeProps) => <PostList resource="posts" {...routeProps} />} />
                        <Route exact path="/posts/create" render={(routeProps) => <PostCreate resource="posts" {...routeProps} />} />
                        <Route exact path="/posts/:id" hasShow hasDelete render={(routeProps) => <PostEdit resource="posts" {...routeProps} />} />
                        <Route exact path="/posts/:id/show" hasEdit render={(routeProps) => <PostShow resource="posts" {...routeProps} />} />
                        <Route exact path="/posts/:id/delete" render={(routeProps) => <Delete resource="posts" {...routeProps} />} />
                        <Route exact path="/comments" hasCreate render={(routeProps) => <CommentList resource="comments" {...routeProps} />} />
                        <Route exact path="/comments/create" render={(routeProps) => <CommentCreate resource="comments" {...routeProps} />} />
                        <Route exact path="/comments/:id" hasDelete render={(routeProps) => <CommentEdit resource="comments" {...routeProps} />} />
                        <Route exact path="/comments/:id/delete" render={(routeProps) => <Delete resource="comments" {...routeProps} />} />
                        <Route exact path="/users" hasCreate render={(routeProps) => <UsersList resource="users" {...routeProps} />} />
                        <Route exact path="/users/create" render={(routeProps) => <UsersCreate resource="users" {...routeProps} />} />
                        <Route exact path="/users/:id" hasDelete render={(routeProps) => <UsersEdit resource="users" {...routeProps} />} />
                        <Route exact path="/users/:id/delete" render={(routeProps) => <Delete resource="users" {...routeProps} />} />
                    </Switch>
                </MuiThemeProvider>
            </ConnectedRouter>
        </TranslationProvider>
    </Provider>
);
```

此应用程序没有侧边栏，没有主题，没有[auth control](./Authentication.html#restricting-access-to-a-custom-page) - 由你自己添加。从那时起，你可以自定义几乎任何你想要的。