# 编写动作{#Writing-Actions}

管理界面通常必须提供自定义动作，超出了简单的CRUD。例如，在一个管理员的评论中，一个“批准”按钮（在一次单击时，允许更新`is_approved`属性并保存更新的记录） - 是必须的。

如何使用admin-on-rest添加这样的自定义操作？ 答案是双重的，admin-on-rest如何使用Redux和redux-saga，学习并正确的做到这一点会给你更好的理解。

### 简单的方式{#The-Simple-Way}

这是一个完美的“批准”按钮的实现：

```jsx
// in src/comments/ApproveButton.js
import React, { Component } from 'react';
import PropTypes from 'prop-types';
import { connect } from 'react-redux';
import FlatButton from 'material-ui/FlatButton';
import { showNotification as showNotificationAction } from 'admin-on-rest';
import { push as pushAction } from 'react-router-redux';

class ApproveButton extends Component {
    handleClick = () => {
        const { push, record, showNotification } = this.props;
        const updatedRecord = { ...record, is_approved: true };
        fetch(`/comments/${record.id}`, { method: 'PUT', body: updatedRecord })
            .then(() => {
                showNotification('Comment approved');
                push('/comments');
            })
            .catch((e) => {
                console.error(e);
                showNotification('Error: comment not approved', 'warning')
            });
    }

    render() {
        return <FlatButton label="Approve" onClick={this.handleClick} />;
    }
}

ApproveButton.propTypes = {
    push: PropTypes.func,
    record: PropTypes.object,
    showNotification: PropTypes.func,
};

export default connect(null, {
    showNotification: showNotificationAction,
    push: pushAction,
})(ApproveButton);
```

`handleClick`函数通过`fetch`使用一个`PUT`请求REST API，然后显示一个通知（使用`showNotification`）并重定向到评论列表页面（使用`push`）;

`showNotification`和`push`是*动作创建者*。 这是一个Redux术语，用于返回一个简单动作对象的函数。当在第二个参数中给出动作创建者的对象时，`connect()`将[decorate each action creator](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options)使用Redux的`dispatch`方法，所以在`handleClick`函数中，对`showNotification()`的调用其实就是调用`dispatch(showNotification())`。

这个`ApproveButton`可以立即使用，例如在评论列表中，`Datagrid`自动地将`record`注入到它的子节点：

```jsx
// in src/comments/index.js
import ApproveButton from './ApproveButton';

export const CommentList = (props) =>
    <List {...props}>
        <Datagrid>
            <DateField source="created_at" />
            <TextField source="author.name" />
            <TextField source="body" />
            <BooleanField source="is_approved" />
            <ApproveButton />
        </Datagrid>
    </List>;
```

或者，在`<Edit>`页面中，作为[custom action](./CreateEdit.html#actions)：

{% raw %}
```jsx
// in src/comments/CommentEditActions.js
import React from 'react';
import { CardActions } from 'material-ui/Card';
import { ListButton, DeleteButton } from 'admin-on-rest';
import ApproveButton from './ApproveButton';

const CommentEditActions = ({ basePath, data }) => (
    <CardActions style={{ float: 'right' }}>
        <ApproveButton record={data} />
        <ListButton basePath={basePath} />
        <DeleteButton basePath={basePath} record={data} />
    </CardActions>
);

export default CommentEditActions;

// in src/comments/index.js
import CommentEditActions from './CommentEditActions';

export const CommentEdit = (props) =>
    <Edit {...props} actions={<CommentEditActions />}>
        ...
    </List>;
```
{% endraw %}

### 使用REST客户端而不是Fetch{#Using-The-REST-Client-Instead-of-Fetch}

以前的代码使用`fetch()`，这意味着它必须生成原始的HTTP请求。 REST逻辑通常需要一些HTTP管道来处理查询参数，编码，headers，body格式化等。事实证明，你可能已经有一个从REST请求映射到HTTP请求的功能：[REST Client](./RestClients.html)。所以使用这个功能而不是`fetch`是一个好主意 - 只要你已经导出它：

```jsx
// in src/restClient.js
import { simpleRestClient } from 'admin-on-rest';
export default simpleRestClient('http://Mydomain.com/api/');

// in src/comments/ApproveButton.js
import { UPDATE } from 'admin-on-rest';
import restClient from '../restClient';

class ApproveButton extends Component {
    handleClick = () => {
        const { push, record, showNotification } = this.props;
        const updatedRecord = { ...record, is_approved: true };
        restClient(UPDATE, 'comments', { id: record.id, data: updatedRecord })
            .then(() => {
                showNotification('Comment approved');
                push('/comments');
            })
            .catch((e) => {
                console.error(e);
                showNotification('Error: comment not approved', 'warning')
            });
    }

    render() {
        return <FlatButton label="Approve" onClick={this.handleClick} />;
    }
}
```

这下你会懂了：不再`fetch`。就如`fetch`，`restClient`返回一个`Promise`。它的签名是：

```jsx
/**
 * Execute the REST request and return a promise for a REST response
 *
 * @example
 * restClient(GET_ONE, 'posts', { id: 123 })
 *  => new Promise(resolve => resolve({ id: 123, title: "hello, world" }))
 *
 * @param {string} type Request type, e.g GET_LIST
 * @param {string} resource Resource name, e.g. "posts"
 * @param {Object} payload Request parameters. Depends on the action type
 * @returns {Promise} the Promise for a REST response
 */
const restClient = (type, resource, params) => new Promise();
```

关于各种请求类型（`GET_LIST`，`GET_ONE`，`UPDATE`等）的语法，请参阅[REST Client documentation](./RestClients.html#request-format)了解更多详细信息。

###使用自定义动作创建者{#Using-a-Custom-Action-Creator}

在组件内部获取数据很容易。但是，如果您是Redux用户，则可能需要以更为惯用的方式执行此操作-通过分发动作。首先，创建自己的动作创建者来替换对`restClient`的调用：

```jsx
// in src/comment/commentActions.js
import { UPDATE } from 'admin-on-rest';
export const COMMENT_APPROVE = 'COMMENT_APPROVE';
export const commentApprove = (id, data, basePath) => ({
    type: COMMENT_APPROVE,
    payload: { id, data: { ...data, is_approved: true } },
    meta: { resource: 'comments', fetch: UPDATE, cancelPrevious: false },
});
```

这个动作创建者利用了admin-on-rest的内置提取器，它使用`fetch`meta监听动作。在分发时，此操作将触发调用`restClient(UPDATE, 'comments')`，分发一个`COMMENT_APPROVE_LOADING`动作，然后在收到响应后，分发`COMMENT_APPROVE_SUCCESS`或`COMMENT_APPROVE_FAILURE`。

要在组件中使用新的动作创建者，`connect`它：

```jsx
// in src/comments/ApproveButton.js
import React, { Component } from 'react';
import PropTypes from 'prop-types';
import { connect } from 'react-redux';
import FlatButton from 'material-ui/FlatButton';
import { commentApprove as commentApproveAction } from './commentActions';

class ApproveButton extends Component {
    handleClick = () => {
        const { commentApprove, record } = this.props;
        commentApprove(record.id, record);
        // how about push and showNotification?
    }

    render() {
        return <FlatButton label="Approve" onClick={this.handleClick} />;
    }
}

ApproveButton.propTypes = {
    commentApprove: PropTypes.func,
    record: PropTypes.object,
};

export default connect(null, {
    commentApprove: commentApproveAction,
})(ApproveButton);
```

这工作正常：当用户按下“批准”按钮时，API接收`UPDATE`调用，并批准该评论。但是不可能再在`handleClick`中调用`push`或`showNotification`。 这是因为`commentApprove()`会立即返回，无论API调用是否成功。 你如何在只有当操作成功时运行一个函数？

### 使用自定义Saga处理副作用{#Handling-Side-Effects-With-a-Custom-Saga}

`fetch`，`showNotification`和`push`称为*side effects*。 这是一个函数式编程术语，它描述的功能不仅仅是根据输入返回一个值。 Admin-on-rest促进了编程风格，其中side effects与其他代码脱钩，这有利于使其可测试。

在admin-on-rest中，副作用由Sagas处理。[Redux-saga](https://redux-saga.github.io/redux-saga/)是为Redux构建的side effect库，其中side effects由generator函数定义。 这可能听起来很复杂，但并不是这样的：这是需要处理`COMMENT_APPROVE`动作的副作用所需的生成器函数。

```jsx
// in src/comments/commentSaga.js
import { put, takeEvery, all } from 'redux-saga/effects';
import { push } from 'react-router-redux';
import { showNotification } from 'admin-on-rest';

function* commentApproveSuccess() {
    yield put(showNotification('Comment approved'));
    yield put(push('/comments'));
}

function* commentApproveFailure({ error }) {
    yield put(showNotification('Error: comment not approved', 'warning'));
    console.error(error);
}

export default function* commentSaga() {
    yield all([
        takeEvery('COMMENT_APPROVE_SUCCESS', commentApproveSuccess),
        takeEvery('COMMENT_APPROVE_FAILURE', commentApproveFailure),
    ]);
}
```

我们来解释一下，从最后的`commentSaga` generator函数开始。[generator function](http://exploringjs.com/es6/ch_generators.html)（由函数名称中的`*`表示）在通过`yield`调用的语句上暂停，直到yielded语句返回。 `yield []`[并行](https://redux-saga.github.io/redux-saga/docs/advanced/RunningTasksInParallel.html)生成两个命令。`yield takeEvery（[ACTION_NAME]，callback）`执行提供的回调[every time the related action is called](https://redux-saga.github.io/redux-saga/docs/basics/UsingSagaHelpers.html)。 总而言之，当`commentApprove()`发起的拉取成功时，这将执行`commentApproveSuccess`，否则使用`commentApproveFailure`。

对于`commentApproveSuccess`和`commentApproveFailure`，他们只是分发（`put()`）副作用 - 与初始版本相同的副作用。

要使用这个saga，将它传递到`<Admin>`组件的`customSagas`道具中

```jsx
// in src/App.js
import React from 'react';
import { Admin, Resource } from 'admin-on-rest';

import { CommentList } from './comments';
import commentSaga from './comments/commentSaga';

const App = () => (
    <Admin customSagas={[ commentSaga ]} restClient={jsonServerRestClient('http://jsonplaceholder.typicode.com')}>
        <Resource name="comments" list={CommentList} />
    </Admin>
);

export default App;
```

使用此代码，批准审核将显示正确的通知，并重定向到评论列表。而side effects也是[testable](https://redux-saga.github.io/redux-saga/docs/introduction/BeginnerTutorial.html#making-our-code-testable)。

### Bonus: 乐观渲染{#Optimistic-Rendering}

在此示例中，单击“批准”按钮后，用户将重定向到评论列表。Admin-on-rest然后获取`/comments`资源以从服务器获取更新的评论列表。但是，admin-on-rest不等待对此调用的响应来显示评论列表。实际上，它有一个内部实例池（在`state.admin [resource]`），在导航期间保留，并在API调用结束之前使用它来渲染屏幕 - 它被称为*乐观渲染*。

由于自定义`COMMENT_APPROVE`动作包含`fetch：UPDATE`meta，所以admin-on-rest会使用响应自动更新其实例池。这意味着初始渲染（`GET /comments`响应到达之前）将显示已批准的评论！

The fact that admin-on-rest updates the instance pool if you use custom actions with the `fetch` meta should be another motivation to avoid using raw `fetch`.


### 使用自定义Reducer{#Using-a-Custom-Reducer}

除了触发REST调用之外，您可能希望将自己的操作的效果存储在应用程序状态。 例如，如果要显示一个显示比特币当前汇率的小部件，可能需要执行以下操作：

```jsx
// in src/bitcoinRateReceived.js
export const BITCOIN_RATE_RECEIVED = 'BITCOIN_RATE_RECEIVED';
export const bitcoinRateReceived = (rate) => ({
    type: BITCOIN_RATE_RECEIVED,
    payload: { rate },
});
```

可以通过以下组件在挂载时触发此动作：

```jsx
// in src/BitCoinRate.js
import React, { Component } from 'react';
import PropTypes from 'prop-types';
import { connect } from 'react-redux';
import { bitcoinRateReceived as bitcoinRateReceivedAction } from './bitcoinRateReceived';

class BitCoinRate extends Component {
    componentWillMount() {
        fetch('https://blockchain.info/fr/ticker')
            .then(response => response.json())
            .then(rates => rates.USD['15m'])
            .then(bitcoinRateReceived) // dispatch action when the response is received
    }

    render() {
        const { rate } = this.props;
        return <div>Current bitcoin value: {rate}$</div>
    }
}

BitCoinRate.propTypes = {
    bitcoinRateReceived: PropTypes.func,
    rate: PropTypes.number,
};

const mapStateToProps = state => ({ rate: state.bitcoinRate });

export default connect(mapStateToProps, {
    bitcoinRateReceived: bitcoinRateReceivedAction,
})(BitCoinRate);
```

为了将汇率传递到`bitcoinRateReceived()`进入Redux store里，你需要一个reducer：

```jsx
// in src/rateReducer.js
import { BITCOIN_RATE_RECEIVED } from './bitcoinRateReceived';

export const (previousState = 0, { type, payload }) => {
    if (type === BITCOIN_RATE_RECEIVED) {
        return payload.rate;
    }
    return previousState;
}
```

现在的问题是：你如何把这个reducer放在`<Admin>`应用程序中？ 简单：使用`customReducers`属性：

{% raw %}
```jsx
// in src/App.js
import React from 'react';
import { Admin } from 'admin-on-rest';

import rate from './rateReducer';

const App = () => (
    <Admin customReducers={{ rate }} restClient={jsonServerRestClient('http://jsonplaceholder.typicode.com')}>
        ...
    </Admin>
);

export default App;
```
{% endraw %}

**提示**：您可以通过将数据存储在组件状态来避免将数据存储在Redux状态。 处理也比较复杂，而且性能也更差。 仅在真正需要时才使用全局状态。

### 结论{#Conclusion}

您应该为自己的动作按钮选择哪种风格？

第一个版本（带`fetch`）是非常好的，如果您不是单元测试您的组件，或者从纯功能中解除副作用，那么您可以坚持使用它。

另一方面，如果要促进可重用性，分离关注点，遵守admin-on-rest的编码标准，并且如果您知道足够的Redux和Saga，请使用最终版本。
