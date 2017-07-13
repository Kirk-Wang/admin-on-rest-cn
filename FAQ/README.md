# FAQ{#FAQ}

- [我的资源可以有自定义标识符/主键吗？](#can-i-have-custom-identifiers-primary-keys-for-my-resources)
- [如何根据用户权限自定义UI？](#how-can-i-customize-the-ui-depending-on-the-user-permissions)
- [如何根据其输入值自定义表单？](#how-can-i-customize-forms-depending-on-its-inputs-values)

### 我的资源可以有自定义标识符/主键吗?{#can-i-have-custom-identifiers-primary-keys-for-my-resources}

Admin-on-rest需要每个资源都有一个`id`字段来标识它。如果您的REST API为主键使用不同的名称，则必须将该名称映射到自定义[restClient](https://marmelab.com/admin-on-rest/RestClients.html)中的“id”。 例如，要使用名为`_id`的字段作为标识符：

```js
const convertHTTPResponseToREST = (response, type, resource, params) => {
    const { headers, json } = response;
    switch (type) {
    case GET_LIST:
        return {
            data: json.map(resource => { ...resource, id: resource._id } ),
            total: parseInt(headers.get('content-range').split('/').pop(), 10),
        };
    case UPDATE:
    case DELETE:
    case GET_ONE:
        return { ...json, id: json._id }; 
    case CREATE:
        return { ...params.data, id: json._id };
    default:
        return json;
    }
};
```

### 如何根据用户权限自定义UI？{#how-can-i-customize-the-ui-depending-on-the-user-permissions}

一些相当常见的用例可能取决于用户权限：

- 特定视图
- 视图（字段，输入）的部分对于特定用户而言是不同的
- 隐藏或显示菜单项

对于所有这些情况，您可以使用[aor-permissions](https://github.com/marmelab/aor-permissions)插件。

### 如何根据其输入值自定义表单？{how-can-i-customize-forms-depending-on-its-inputs-values}

一些用例：

- 如果另一个输入有一个值，则显示/隐藏某些输入
- 如果另一个输入具有特定值，则显示/隐藏某些输入
- 如果当前表单值与特定约束匹配，则显示/隐藏某些输入

对于所有这些情况，您可以使用[aor-dependent-input](https://github.com/marmelab/aor-dependent-input)插件。