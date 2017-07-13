# 翻译{#Translation}

admin-on-rest界面使用英语作为默认语言。但是它也支持任何其他语言，归功于[polyglot.js]（http://airbnb.io/polyglot.js/）库。

### 改变区域设置{#Changing-Locale}

为了处理翻译，`<Admin>`组件支持：

- 一个`locale` 属性期望是一个字符串（'en'，'fr'等），和
— 一个`messages`属性，期望是一个字典对象。

Admin-on-rest仅仅载有英语区域语言；如果你要使用其他语言环境，则必须安装第三方软件包。 例如，要将界面更改为法语，请安装`aor-language-french`npm软件包，然后配置`<Admin>`组件，如下所示：

```jsx
import React from 'react';
import { Admin, Resource, resolveBrowserLocale } from 'admin-on-rest';
import frenchMessages from 'aor-language-french';

const messages = {
    fr: frenchMessages,
};

const App = () => (
    <Admin ...(your props) locale="fr" messages={messages}>
        ...
    </Admin>
);

export default App;
```

### 可用区域设置{#Available-Locales}

您可以找到以下语言的翻译包：

- English (`en`) is the default
- Chinese (`cn`): [downup2u/aor-language-chinese](https://github.com/downup2u/aor-language-chinese)
- Chinese (Traditional) (`cht`): [leesei/aor-language-chinese-traditional](https://github.com/leesei/aor-language-chinese-traditional)
- Czech (`cs`): [magikMaker/aor-language-czech](https://github.com/magikMaker/aor-language-czech)
- Dutch (`nl`): [pimschaaf/aor-language-dutch](https://github.com/pimschaaf/aor-language-dutch)
- French (`fr`): [marmelab/aor-language-french](https://github.com/marmelab/aor-language-french)
- German (`de`): [der-On/aor-language-german](https://github.com/der-On/aor-language-german)
- Hebrew (`he`): [mstmustisnt/aor-language-hebrew](https://github.com/mstmustisnt/aor-language-hebrew)
- Hungarian (`hu`): [s33m4nn/aor-language-hungarian](https://github.com/s33m4nn/aor-language-hungarian)
- Italian (`it`): [stefsava/aor-language-italian](https://github.com/stefsava/aor-language-italian)
- Japanese (`ja`): [kuma-guy/aor-language-japanese](https://github.com/kuma-guy/aor-language-japanese)
- Polish (`pl`): [KamilDzierbicki/aor-language-polish](https://github.com/KamilDzierbicki/aor-language-polish)
- Portuguese (`pt`): [movibe/aor-language-portugues](https://github.com/movibe/aor-language-portugues)
- Russian (`ru`): [cytomich/aor-language-russian](https://github.com/cytomich/aor-language-russian)
- Spanish (`es`): [blackboxvision/aor-language-spanish](https://github.com/BlackBoxVision/aor-language-spanish)
- Swedish (`sv`): [StefanWallin/aor-language-swedish](https://github.com/StefanWallin/aor-language-swedish)
- Thai (`th`): [liverbool/aor-language-thai](https://github.com/liverbool/aor-language-thai)
- Turkish (`tr`): [ismailbaskin/aor-language-turkish](https://github.com/ismailbaskin/aor-language-turkish)
- Vietnamese (`vi`): [kimkha/aor-language-vietnamese](https://github.com/kimkha/aor-language-vietnamese)

如果你想提交一个新的翻译，请随时提交一个PR来更新[this page](https://github.com/marmelab/admin-on-rest/blob/master/docs/Translation.md)具有指向您翻译软件包的链接。

### 在运行时更改区域设置{#Changing-Locale-At-Runtime}

如果要提供在运行时更改语言环境的功能，则必须提供所有可能翻译的信息：

```jsx
import React from 'react';
import { Admin, Resource, englishMessages } from 'admin-on-rest';
import frenchMessages from 'aor-language-french';

const messages = {
    fr: frenchMessages,
    en: englishMessages,
};

const App = () => (
    <Admin ...(your props) locale="en" messages={messages}>
        ...
    </Admin>
);

export default App;
```

然后，使用`changeLocale`动作创建者分派`CHANGE_LOCALE`动作。例如，以下组件可以切换英文和法文之间的语言：

```jsx
import React, { Component } from 'react';
import { connect } from 'react-redux';
import RaisedButton from 'material-ui/RaisedButton';
import { changeLocale as changeLocaleAction } from 'admin-on-rest';

class LocaleSwitcher extends Component {
    switchToFrench = () => this.changeLocale('fr');
    switchToEnglish = () => this.changeLocale('en');

    render() {
        const { changeLocale } = this.props;
        return (
            <div>
                <div style={styles.label}>Language</div>
                <RaisedButton style={styles.button} label="en" onClick={this.switchToEnglish} />
                <RaisedButton style={styles.button} label="fr" onClick={this.switchToFrench} />
            </div>
        );
    }
}

export default connect(undefined, { changeLocale: changeLocaleAction })(LocaleSwitcher);
```

### 使用浏览器区域设置{#Using-The-Browser-Locale}

Admin-on-rest提供了一个名为`resolveBrowserLocale()`的帮助函数，可帮助您根据用户浏览器中配置的区域设置引入动态区域设置属性。要使用它，只需将函数传递作为`locale`属性。

```jsx
import React from 'react';
import { Admin, Resource, englishMessages, resolveBrowserLocale } from 'admin-on-rest';
import frenchMessages from 'aor-language-french';

const messages = {
    fr: frenchMessages,
    en: englishMessages,
};

const App = () => (
    <Admin ...(your props) locale={resolveBrowserLocale()} messages={messages}>
        ...
    </Admin>
);

export default App;
```

### 翻译信息{#Translation-Messages}

`message`值应该是一个字典，每个语言支持一个条目。对于给定的语言，键标识界面组件，值是翻译的字符串。这个字典是一个简单的JavaScript对象，如下所示：

```jsx
{
    en: {
        aor: {
            action: {
                delete: 'Delete',
                show: 'Show',
                list: 'List',
                save: 'Save',
                create: 'Create',
                edit: 'Edit',
                cancel: 'Cancel',
            },
            ...
        },
    },
    fr: {
        aor: {
            action: {
                delete: 'Supprimer',
                show: 'Afficher',
                list: 'Liste',
                save: 'Enregistrer',
                create: 'Créer',
                edit: 'Éditer',
                cancel: 'Quitter',
            },
            ...
        }
    }
}
```

所有核心翻译都在`aor`命名空间中，以防止与您自己的自定义翻译冲突。运行时使用的root key由`locale`属性的值决定。

可用的默认信息[here](https://github.com/marmelab/admin-on-rest/blob/master/src/i18n/messages.js)。

### 翻译资源和字段名称{#Translating-Resource-and-Field-Names}

默认情况下，Admin-on-rest在界面中的所有位置都使用资源名称（“post”，“comment”等）和字段名称（“title”，“first_name”等）。它简单地“人性化”技术标识符，使其看起来更好（例如“first_name”变为“First name”）。

However, before humanizing names, admin-on-rest checks the `messages` dictionary for a possible translation, with the following keys:
然而，在对名称进行人性化之前，admin-on-rest会使用以下键检查`messages`字典以进行可能的翻译：

- `${locale}.resources.${resourceName}.name`用于资源名称（用于菜单和页面标题）
- `${locale}.resources.${resourceName}.fields.${fieldName}`用于字段名称（用于datagrid header和表单输入标签）

这可以通过使用`resources`键传递`messages`对象来翻译自己的资源和字段名称：

```jsx
{
    en: {
        resources: {
            shoe: {
                name: 'Shoe |||| Shoes',
                fields: {
                    model: 'Model',
                    stock: 'Nb in stock',
                    color: 'Color',
                },
            },
            customer: {
                name: 'Customer |||| Customers',
                fields: {
                    first_name: 'First name',
                    last_name: 'Last name',
                    dob: 'Date of birth',
                }
            }
        }
    },
    ...
}
```

正如你所看到的，[polyglot pluralization]（http://airbnb.io/polyglot.js/#pluralization）在这里使用，但它是可选的。

使用`resources`键是在Field和Input组件中使用`label`属性的替代方案，其优点是支持翻译。

### 混合界面和域翻译{#Mixing-Interface-and-Domain-Translations}

当翻译admin时，界面消息（例如“List”，“Page”等）通常来自第三方软件包，而您的域信息（例如“Shoe”，“Date of birth”等）来自你自己的代码。这意味着您需要将这些消息合并，才能将其传递给“<Admin>”。组合消息的方法是使用ES6解构：

```jsx
// interface translations
import { englishMessages } from 'admin-on-rest';
import frenchMessages from 'aor-language-french';

// domain translations
import * as domainMessages from './i18n';

const messages = {
    fr: { ...frenchMessages, ...domainMessages.fr },
    en: { ...englishMessages, ...domainMessages.en },
};

const App = () => (
    <Admin ...(your props) messages={messages}>
        ...
    </Admin>
);
```

### 翻译自己的组件{#Translating-Your-Own-Components}

翻译系统使用React`context`将翻译向组件树传递下来。要翻译一个句子，请使用上下文中的`translate`函数。 当然，这假设你以前已经将相应的翻译添加到`Admin`组件的`messages`属性中。

```jsx
// in src/MyHelloButton.js
import React, { Component } from 'react';
import PropTypes from 'prop-types';

class MyHelloButton {
    render() {
        const { translate } = this.context;
        return <button>{translate('myroot.hello.world')}</button>;
    }
}
MyHelloButton.contextTypes = {
    translate: PropTypes.function,
};

// in src/App.js
const messages = {
    en: {
        myroot: {
            hello: {
                world: 'Hello, World!',
            },
        },
    },
};
```

然而，使用上下文使组件更难测试。这就是为什么admin-on-rest提供了一个`translate`高阶组件，它只是将`translate`功能从上下文传递给属性：

```jsx
// in src/MyHelloButton.js
import React from 'react';
import { translate } from 'admin-on-rest';

const MyHelloButton = ({ translate }) => (
    <button>{translate('myroot.hello.world')}</button>
);

export default translate(MyHelloButton);
```

**提示**：对于您的信息标识符，请选择与`aor`和`resources`不同的根名称，这些名称是保留的。

**提示**：对于字段和输入标签或页面标题，不要使用`translate`，因为它们已经被翻译：

```jsx
// don't do this
<TextField source="first_name" label={translate('myroot.first_name')} />

// do this instead
<TextField source="first_name" label="myroot.first_name" />

// or even better, use the default translation key
<TextField source="first_name" />
// and translate the `resources.customers.fields.first_name` key
```

### 使用特定Polyglot Features{#Using-Specific-Polyglot-Features}

Polyglot.js是一个神奇的库：除了小巧，完全维护，完全框架不可知之外，它还提供了一些很好的功能，如插值和多元化，您可以在admin-on-rest中使用。

```jsx
const messages = {
    'hello_name': 'Hello, %{name}',
    'count_beer': 'One beer |||| %{smart_count} beers',
}

// interpolation
translate('hello_name', { name: 'John Doe' });
=> 'Hello, John Doe.'

// pluralization
translate('count_beer', { smart_count: 1 });
=> 'One beer'

translate('count_beer', { smart_count: 2 });
=> '2 beers'

// default value
translate('not_yet_translated', { _: 'Default translation' })
=> 'Default translation'
```

要查找更详细的示例，请参阅[http://airbnb.io/polyglot.js/](http://airbnb.io/polyglot.js/)