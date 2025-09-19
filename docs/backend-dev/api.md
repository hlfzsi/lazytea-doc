# 处理器API

处理器API是后端的重要组成部分，它支持了 请求/响应 模式的运行。

应当支持自定义拓展。

所有返回都应当以Json发送。

## 核心处理器API

核心处理器API 是后端必须实现的API，是二级组件赖以运行的API。

### API列表

#### 1. `read_files`

* **名称**: `read_files`
* **作用**: 读取一个或多个指定路径的文件内容，并将其作为Base64编码的字符串返回。
* **传入参数**:
  * `paths` (数组, 字符串): 必须。需要读取的文件的绝对或相对路径列表。
  * `keys` (数组, 字符串): 可选。与`paths`数组一一对应的键名列表。如果提供，返回结果中的键将使用此列表中的名称；否则，将直接使用文件路径作为键。如果提供，其长度必须与`paths`的长度相同。
* **输出**:
  * **成功**: 一个JSON对象，其中键是`keys`参数中指定的名称（或文件路径），值是对应文件内容的Base64编码字符串。

```json
        {
          "key1": "base64_encoded_content_1",
          "key2": "base64_encoded_content_2"
        }
```

* **失败**: 如果`keys`和`paths`长度不匹配，返回一个包含错误信息的JSON对象。

```json
        {
          "error": "keys and paths must have the same length"
        }
```

---

#### 2. `get_plugins`

* **名称**: `get_plugins`
* **作用**: 获取所有已加载插件的详细信息列表。
* **传入参数**: 无。
* **输出**:
  * 一个JSON对象，键是插件的唯一名称，值是包含插件详细信息的对象。每个插件信息对象包含以下字段：
    * `name` (字符串): 插件的名称。
    * `module` (字符串): 插件的模块名。
    * `meta` (字典): 插件的元数据，包含：
      * `name` (字符串/null): 用户友好的插件显示名称。
      * `description` (字符串): 插件的功能描述。
      * `homepage` (字符串/null): 插件的主页URL。
      * `config_exists` (布尔值): 插件是否存在配置项。
      * `author` (字符串): 插件作者。
      * `version` (字符串): 插件版本。
      * `ui_support`(布尔值)：插件是否支持自定义UI。
      * `pip_name`(字符串)：插件包名。
      * `icon_abspath`(字符串)：插件图标的绝对路径。
      * `html_exists`(布尔值): 插件是否包含HTML界面。
      * 以及其他在插件元数据中定义的额外信息。

---

#### 3. `get_plugin_config`

* **名称**: `get_plugin_config`
* **作用**: 获取指定插件的配置信息，包括其配置项的JSON Schema定义和当前的配置值。
* **传入参数**:
  * `name` (字符串): 必须。目标插件的名称。
* **输出**:
  * **成功**: 一个JSON对象，包含：
    * `schema` (对象): 描述该插件配置项结构的JSON Schema。
    * `data` (字符串): 一个JSON字符串，表示当前插件配置的各项值。
  * **失败**: 如果插件未找到或插件没有配置项，返回一个包含错误信息的JSON对象。

```json
        {
          "error": "Plugin not found"
        }
```

或

```json
        {
          "error": "Plugin config not found"
        }
```

---

#### 4. `save_env`

* **名称**: `save_env`
* **作用**: 保存指定插件的配置。该操作会将新的配置写入到环境配置文件中。
* **传入参数**:
  * `module_name` (字符串): 必须。目标插件的模块名。
  * `data` (对象): 必须。一个JSON对象，其键值对应插件的配置项及其新值。
* **输出**:
  * **成功**: 返回布尔值`true`。
  * **失败**: 如果插件或其配置未找到，或传入的`data`与插件配置模型不匹配，返回一个包含错误信息的JSON对象。

```json
        {
          "error": "Plugin config unmatched \n <traceback>"
        }
```

---

#### 5. `get_matchers`

* **名称**: `get_matchers`
* **作用**: 获取所有插件的匹配器（Matcher）信息，包括它们的触发规则和权限设置。
* **传入参数**: 无。
* **输出**:
  * 一个复杂的JSON对象 (`MatcherRuleModel`)，描述了每个机器人实例下，每个插件的所有匹配器的详细规则和权限配置。其结构大致如下：

```json
{
          "bots": {
            "bot_id_1": {
              "plugins": {
                "plugin_name_1": {
                  "matchers": [
                    {
                      "rule": { /* 规则详情 */ },
                      "permission": { /* 权限详情 */ },
                      "is_on": true
                    }
                  ]
                }
              }
            }
          }
}
```

一个具体的输出示例如下：

一个ID为 `onebot_12345` 的机器人，它加载了两个插件：“天气查询”和“管理员工具”。

* **天气查询插件**: 包含一个匹配器，通过命令 `天气` 或 `weather` 触发。该功能默认开启，且没有设置特殊的权限（即对所有人开放）。
* **管理员工具插件**: 包含两个匹配器。
  1. 第一个匹配器用于“禁言”功能，通过 `禁言` 或 `ban` 开头的消息触发，并且需要@机器人。这个功能设置了权限：只有用户ID为 `10001` 的用户（超级管理员）可以使用。
  2. 第二个匹配器用于“状态检查”，通过正则表达式 `^status$` (即精确匹配"status")触发。这个功能默认是开启的 (`"is_on": true`)，但有一个黑名单，禁止了群 `54321` 使用。

```json
{
  "bots": {
    "onebot_12345": {
      "plugins": {
        "天气查询": {
          "matchers": [
            {
              "rule": {
                "commands": [
                  ["天气", "weather"]
                ],
                "shell_commands": [],
                "regex_patterns": [],
                "keywords": [],
                "startswith": [],
                "endswith": [],
                "fullmatch": [],
                "alconna_commands": [],
                "event_types": [],
                "to_me": false
              },
              "permission": {
                "white_list": {
                  "user": [],
                  "group": []
                },
                "ban_list": {
                  "user": [],
                  "group": []
                }
              },
              "is_on": true
            }
          ]
        },
        "管理员工具": {
          "matchers": [
            {
              "rule": {
                "commands": [],
                "shell_commands": [],
                "regex_patterns": [],
                "keywords": [],
                "startswith": [
                  "禁言",
                  "ban"
                ],
                "endswith": [],
                "fullmatch": [],
                "alconna_commands": [],
                "event_types": [],
                "to_me": true
              },
              "permission": {
                "white_list": {
                  "user": [
                    "10001"
                  ],
                  "group": []
                },
                "ban_list": {
                  "user": [],
                  "group": []
                }
              },
              "is_on": false
            },
            {
              "rule": {
                "commands": [],
                "shell_commands": [],
                "regex_patterns": [
                  "^status$"
                ],
                "keywords": [],
                "startswith": [],
                "endswith": [],
                "fullmatch": [],
                "alconna_commands": [],
                "event_types": [],
                "to_me": false
              },
              "permission": {
                "white_list": {
                  "user": [],
                  "group": []
                },
                "ban_list": {
                  "user": [],
                  "group": [
                    "54321"
                  ]
                }
              },
              "is_on": true
            }
          ]
        }
      }
    }
  }
}
```

---

#### 6. `sync_matchers`

* **名称**: `sync_matchers`
* **作用**: 将前端发送的新的匹配器配置同步并保存到后端。
* **传入参数**:
  * `new_roster` (字符串): 必须。一个JSON字符串，代表了全新的`MatcherRuleModel`对象。
* **输出**: 无明确返回值。操作成功则没有返回，失败会由WebSocket框架处理。

---

#### 7. `update_plugin`

* **名称**: `update_plugin`
* **作用**: 更新指定的插件。该过程会调用pip(可根据实际情况更换)来安装或升级插件包。
* **传入参数**:
  * `plugin_name` (字符串): 必须。需要更新的插件的包名（pip name）。
* **输出**:
  * **成功**: 返回一个字符串，包含更新成功的提示信息和pip的输出。
  * **失败**: 返回一个包含错误信息的JSON对象，例如环境缺少pip、更新失败等。

```json
        {
          "error": "插件 <plugin_name> 更新失败！\n<stderr_output>"
        }
```

---

#### 8. `ui_load`

* **名称**: `ui_load`
* **作用**: 加载指定插件的UI相关模块（通常是`__call__`模块，根据实际情况更换），用于界面功能的动态加载。
* **传入参数**:
  * `plugins_to_load` (数组, 字符串): 必须。需要加载UI的插件名称列表。
* **输出**: 无明确返回值。

---

#### 9. `bot_switch`

* **名称**: `bot_switch`
* **作用**: 切换机器人的在线状态，并广播该状态变更事件。
* **传入参数**:
  * `bot_id` (字符串): 必须。机器人的唯一标识符。
  * `platform` (字符串): 必须。机器人所属的平台。
  * `is_online_now` (布尔值): 必须。`true`表示机器人要上线，`false`表示要下线。
* **输出**: 无明确返回值。

---

#### 10. `get_plugin_custom_html`

* **名称**: `get_plugin_custom_html`
* **作用**: 获取指定插件的自定义UI界面（HTML）。
* **传入参数**:
  * `plugin_name` (字符串): 必须。目标插件的名称。
* **输出**:
  * **成功**: 一个JSON对象，其结构遵循`PluginHTML`模型，包含：
    * `html` (字符串): 插件主页面的HTML内容。
    * `is_rendered` (布尔值): 页面是否已渲染。
    * `context` (对象): 渲染上下文，可JSON化的数据。
    * `includes` (对象): 单独的CSS/JS文件，键为名称，值为文件内容。
  * **失败**: 如果插件没有对应的处理方法或返回类型无效，返回一个包含错误信息的JSON对象。

```json
        {
          "error": "Method not found"
        }
```

### 特殊行为

一级组件应当允许插件注册自定义API，并在某些情况下自动回调。

比如，在`save_env`API中，我们可以校验完成后调用插件自定义的配置热重载API以提高自由度。
