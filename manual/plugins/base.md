# dtable

## Init
> 插件开发环境分为两种, 开发环境和集成环境, 由于环境不同, 所以初始化的方式也不同:
> 在开发环境中, 编码人员需要提供插件所需的配置文件, 用于初始化插件, 获取插件需要的数据。
> 在使用环境，用户安装插件后, 插件可以直接读取当前浏览器中的 base 的数据, 用于初始化插件。

### 初始化插件

#### init

开发环境下初始化插件

```javascript
import DTable from 'dtable-sdk';

const dtable = new DTable();
const settings = {
  "server": "https://cloud.seatable.cn",
  "APIToken": "50c17897ae8b1c7c428d459fc2c379a9bc3806cc",
}
await dtable.init(config);
```

#### initBrowser

使用环境下初始化插件

```javascript
import DTable from 'dtable-sdk';

const dtable = new Dtable();
const dtableStore = window.app.dtableStore; // 从集成环境中读取初始化数据
await dtable.initBrowser(dtableStore);
```


### 监听事件变化

#### subscribe

|事件类型|描述 | 用途|
|-|-|-|
|dtable-connect|表示与 server 已经建立链接, 数据加载完成 |  更新 state, 更新 UI 显示|
|local-dtable-changed|表示本地执行了某些操作, 数据发生变化| 更新 state, 更新 UI 显示 |
|remote-dtable-changed|表示本地执行了 server 端发送的某些操作, 数据发生变化| 更新 state, 更新 UI 显示 |

```javascript
import DTable from 'dtable-sdk';

const dtable = new Dtable();
dtable.subscribe('dtable-connect', () => {...});
dtable.subscribe('local-dtable-changed', () => {...});
dtable.subscribe('remote-dtable-changed', () => {...});
```

## 插件开发初始化例子

由于需要兼容两个环境, 所以一般插件开发的初始化操作如下:

```javascript
import Dtable from 'dtable-sdk';
import PropTypes from 'prop-types';

const propsTypes = {
  isDevelopment: PropTypes.bool
};

const settings = {
  "server": "https://cloud.seatable.cn",
  "APIToken": "50c17897ae8b1c7c428d459fc2c379a9bc3806cc",
};

class App extends React.Component {

  constructor(props) {
    super(props);
    this.state = {
      isLoading: true
    };
    this.dtable = new Dtable();
  }

  async componentDidMount() {
    const { isDevelopment } = this.props;
    if (isDevelopment) {
      await dtable.init(settings);
      await this.dtable.syncWithServer();
      this.dtable.subscribe('dtable-connect', this.resetData);
    } else {
      const dtableStore = window.app.dtableStore;
      dtable.initBrowser(dtableStore);
    }

    this.dtable.subscribe('local-dtable-changed', this.resetData);
    this.dtable.subscribe('remote-dtable-changed', this.resetData);
  }

  resetData = () => {
    // ...
    this.setState({isLoading: false});
  }

  render() {
    return (
      ...
    );
  }

}
```
