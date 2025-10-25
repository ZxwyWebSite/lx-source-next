## LX-Source/接口定义 `v251018`

本程序 API 接口按版本控制：

&emsp;目前 `v0` 版本已冻结，即不会新增方法，但仍可能会按需添加或修改字段，保证参数兼容性。

&emsp;`v1` 版本仍在开发中，`/test/` 路径下为测试接口，不保证稳定性，其余接口可保证基本稳定。

&emsp;接口遵循 RESTful 风格，如无特殊要求，均为 `GET` 请求，并以 json 格式响应。

调用方式：

`http(s)://{addr}/api/{ver}/{*path}`

**addr**: 接口地址

**ver**: 接口版本 `v0` / `v1`

**path**: 接口路径

响应结构（全局统一）：

```json
{
  "code": "int",
  "msg": "string",
  "data": "any",
  "ext": "?any"
}
```

<!-- ```go
type Resp struct {
    Code int    `json:"code"`
    Msg  string `json:"msg"`
    Data any    `json:"data"`
    Ext  any    `json:"ext"`
}
``` -->

**code**: 状态码

目前仍未统一，同时包含两种规范 ~~混用~~，原 [LX-Source](https://github.com/ZxwyWebSite/lx-source/blob/9155f341051d2570e06bf9acaa38476705a3bc4d/src/middleware/resp/resp.go#L22) / [Python 版](https://github.com/MeoProject/lx-music-api-server/tree/v2.0.0?tab=readme-ov-file#%E8%BF%94%E5%9B%9E%E7%A0%81%E8%AF%B4%E6%98%8E) 的 enum 风格 与 [ikun 版](https://github.com/MeoProject/lx-music-api-server/tree/v3.0.2) 的 http 风格

**msg**: 错误提示

**data**: 相应数据

**ext**: 额外信息（？）

---

### V0

&emsp;与 [旧版](https://github.com/ZxwyWebSite/lx-source?tab=readme-ov-file#%E5%8A%9F%E8%83%BD-%E6%9C%AA%E5%85%A8%E9%83%A8%E5%AE%9E%E7%8E%B0) ~~完全~~兼容的接口，[脚本](https://github.com/ZxwyWebSite/lx-source/blob/main/src/server/public/lx-custom-source.js)填写 `apiaddr='http(s)://{addr}/api/v0/'` 即可直接使用。

**格式**：

`http(s)://{addr}/api/v0/{method}/{source}/{id}/{?quality}`

**source**：来源，平台二字码

`wy` / `mg` / `kw` / `kg`

**id**：资源 ID

_不用多说了吧_

**quality**：质量

没做要求的留空，留空时要保留尾部斜杠！

**查询参数：**

**`jmp`**：链接跳转

302 重定向到解析结果链接，若发生错误以 json 形式返回。

支持 `url` / `mv` / `pic` 方式

+`lrc`: 返回 正文+翻译?+罗马? (`v1.1.9+`)

**`key`**：接口密钥

适用于无法设置请求头的情况。

---

**method**：方法

#### ✦ `url` / `link`：获取播放链接

**quality**: `128k` / `192k` / `320k` / `flac` / `flac24bit/fl24/hires` / `dolby/atmos`(`VIP`) / `sky`(`SVIP`) / `master`

```json
{
  "data": "string",
  "ext": "ILinkInfoBase"{
    "qual": "string",
    "expi": "int64"
  }
}
```

**`data`**: 播放链接，当出错时为错误音频

**`ext`**:

&emsp;**`qual`**: 实际音质，由于回落策略可能与请求音质不符

&emsp;**`expi`**: 链接过期绝对时间，Unix 秒级时间戳

#### ✦ `lrc` / `lyric`：获取歌词

```json
{
  "data": "*ILyricInfo"{
    "lyric": "string",
    "tlyric": "?string",
    "rlyric": "?string",
    "lxlyric": "?string",
  }
}
```

**`data`**:

&emsp;**`lyric`**: 普通歌词

&emsp;**`tlyric`**: 翻译歌词，可能为空

&emsp;**`rlyric`**: 罗马音歌词，可能为空

&emsp;**`lxlyric`**: [LX 逐字歌词](https://lxmusic.toside.cn/desktop/custom-source#:~:text=%27...%27%2C%20//-,lx%20%E9%80%90%E5%AD%97%E6%AD%8C%E8%AF%8D,-%EF%BC%8C%E6%B2%A1%E6%9C%89%E5%8F%AF%E4%B8%BA)，可能为空

#### ✦ `mov` / `mv`：获取视频链接

**`quality`**: `240p` / `360p` / `480p` / `720p` / `1080p`

```json
{
  "data": "*ILinkInfo"{
    "qual": "string",
    "expi": "int64",
    "link": "string"
  }
}
```

**`data`**:

&emsp;**`qual`**: 实际画质，由于回落策略可能与请求画质不符

&emsp;**`expi`**: 链接过期绝对时间，Unix 秒级时间戳

&emsp;**`link`**: 播放链接

#### ✦ `mid`：获取音乐信息

```json
{
  "data": "*IMusicInfo"{
    "id": "string",
    "name": "string",
    "alia": "[]string",
    "ar": "[]IArtistItem"[{
      "id": "string",
      "name": "string",
      "alia": "[]string",
    }],
    "al": "IAlbumItem"{
      "id": "string",
      "name": "string",
      "alia": "[]string",
      "covr": "string"
    },
    "dt": "uint32",
    "mv": "string",
    "fee": "uint32",
    "file": "[]IFileInfo"[{
      "qual": "string",
      "size": "string",
      "sizs": "?string",
      "hash": "?string"
    }]
  }
}
```

**`data`**:

&emsp;**`id`**: 音乐 **ID**，可能与传入参数不同

&emsp;**`name`**: 音乐名称

&emsp;**`alia`**: 音乐别名

&emsp;**`ar`**: 歌手信息

&emsp;&emsp;**`id`**: 歌手 ID

&emsp;&emsp;**`name`**: 歌手名称

&emsp;&emsp;**`alia`**: 歌手别名

&emsp;**`al`**: 专辑信息

&emsp;&emsp;**`id`**: 专辑 ID

&emsp;&emsp;**`name`**: 专辑名称

&emsp;&emsp;**`alia`**: 专辑别名

&emsp;&emsp;**`covr`**: 专辑封面，此处为默认（最大）大小，建议使用 `pic` 接口获取缩略图

&emsp;**`dt`**: 歌曲时长（秒）

&emsp;**`mv`**: 视频 ID，为空代表没有视频，部分平台与音乐 ID 不同，建议统一使用此处结果

&emsp;**`fee`**: [付费类型](https://developer.music.163.com/st/developer/document?docId=8e96f389dfc74fcb97af35d1597be77e#:~:text=0-,songFee,-%E5%80%BC)（`v1.1.8+`）

&emsp;&emsp;`0`: 免费：免费歌曲，可直接获取所有音质

&emsp;&emsp;`1`: 会员：普通用户无法免费收听；会员可收听所有音质

&emsp;&emsp;`4`: 数字专辑：所有用户只能在商城购买数字专辑后，才能收听

&emsp;&emsp;`8`: 试听：普通用户可免费收听 128k 音质（`WY` 源支持到 320k）；会员可收听所有音质

&emsp;**`file`**: 文件信息

&emsp;&emsp;**`qual`**: 文件质量

&emsp;&emsp;**`size`**: 文件大小

&emsp;&emsp;**`sizs`**: 已格式化的文件大小，当此项存在时 `size` 为无法获取

&emsp;&emsp;**`hash`**: `KG` 源特定音质哈希值

#### ✦ `vid`：获取视频信息

```jsonc
{
  "data": "*IMovieInfo"{
    "id": "string",
    "name": "string",
    "desc": "string",
    "covr": "string",
    "file": "[]IFileInfo"[{
      //...
    }]
  }
}
```

**`data`**:

&emsp;**`id`**: 视频 ID

&emsp;**`name`**: 视频名称

&emsp;**`desc`**: 视频简介，可能为空

&emsp;**`covr`**: 视频封面

&emsp;**`file`**: 文件信息，同上

#### ✦ `pic`：获取专辑封面（`v1.1.7+`）

**`quality`**: **`s`**`(mall)`[`256`] / **`m`**`(iddle)`[`512`] / **`l`**`(arge)`[`1024`] / `""` 留空原图 / `2048` 自定义（部分支持）

```json
{
  "data": "string"
}
```

**`data`**: 图片链接，由于回落策略实际大小可能与请求不符（`MG`）

---

**脚本下载**：`/api/v0/lx-script.js`

或 `{addr}/api/v0/script` (`v1.1.7+`)

**查询参数：**

**`raw`**：下载模式

浏览器默认会以预览模式打开，此参数设置标头让浏览器下载文件。

**`key`**：接口密钥

开启接口密钥功能后，用于注入用户密钥。

~~（时间问题，没有任何更新，直接把旧版 lx-script 搬上去的）~~

---

### V1

&emsp;更多功能的接口，原打算直接搞在线音乐平台，但近期内是不可能了。

**前缀**：`http(s)://{addr}/api/v1/`

#### ✦ **`search`**：搜索

**传参格式**：

`{pt}/{t}/{q}/{pg}/{sz}`

`{pt}/{t}/{q}?pg={}&sz={}`

`?pt={}&t={}&q={}&pg={}&sz={}`

_目前为手动添加路由，之后可能自动适配参数？_

**查询参数**：

**`pt`**：`plat` 平台二字码，目前仅支持 `wy`

**`t`**：`type` 查询类型，目前支持 `so`：`song` 单曲，`ly`：`lyric` 歌词

**`q`**：`query` 查询内容

**`pg`**：`page` 分页号，从 1 开始

**`sz`**：`size` 返回数量，默认 10

**返回数据**：

```jsonc
{
  "data": "*search_res"{
    "total": "int",
    "items": "any([]*itype.IMusicInfo)"[{
      //...
    }]
  }
}
```

**`data`**:

&emsp;**`total`**: 结果**总数**

&emsp;**`items`**: 当页结果

参考实现：`{addr}/search/search.html`

---

✦ **测试接口 `test/`**：可能不稳定，慎用！

#### ✧ `sse`：流式日志

**前提条件**：配置文件开启 `[Apis].WebLog`

注：返回格式为 `EventStream`，**没有鉴权**

**`type`**:

&emsp;**`log`**: 日志对象

&emsp;&emsp;**`data`**: `['t0','v0','t1','v1']`

&emsp;&emsp;例：`["time","2025-10-18 21:11:03","info","Info ","subs","[Serv]","text","[ 200 ]     20.2699ms |   192.168.10.13 | GET      \"/favicon.ico\""]`

&emsp;**`txt`**: 纯文本

参考实现：`{addr}/test/sse/`

#### ✧ `match`：`WY` 听歌识曲

**查询参数**：

**`duration`**: 录制时长

**`audioFP`**: 音频指纹

参考实现：`{addr}/match/match.html`
