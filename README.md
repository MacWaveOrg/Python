# MacWaveOrg/Python

MacWave 自带的 CPython。客户端只读这一个仓库，地址由 `MACWAVE_PYTHON_DOMAIN` 决定，
默认指向本仓库的 raw 地址。

## 为什么要自带解释器

用户机器上的 `python3` 由 Homebrew / pyenv / 系统自带的版本轮流接管，升级一次就可能
换掉解释器，而第三方依赖仍留在旧版本的 `site-packages` 里 —— 3.0.5 之前真实发生过
一次「系统 Python 从 3.14 升到 3.15，所有命令都报模块缺失」。

自带之后，「用哪个解释器」由 MacWave 自己决定，报错也就能稳定复现。
下载失败（网络问题）时回退到系统 `python3`，安装不会因此中断。

## 目录

```
pythondata/
    latest.toml                   现在用哪个版本（唯一入口）
    3.14.9/
        python_url.toml           各架构的上游地址与校验值
        <上游同名文件>             我们自己托管的那份字节
```

## 两条下载路径

| 场景 | 从哪里取 | 校验值 |
| --- | --- | --- |
| 默认 | 本仓库的 `pythondata/<版本>/<文件名>` | `python_url.toml` 里的 `*_sha256` |
| `--install-python-from-source` | `python_url.toml` 里的 `*_url`（上游） | 同上 |

**字节原样搬运，不重打包**，所以校验值与上游一致、两条路径共用一份 sha256。
文件名统一取上游地址的末段，客户端据此判断落盘名。

## 客户端怎么用

1. 读 `pythondata/latest.toml` 拿到版本号；
2. 本机已装 `python/<版本>` 且能跑 → 直接用，一个网络请求都不再发；
3. 否则读 `pythondata/<版本>/python_url.toml`，按当前架构取地址与校验值，下载、校验、解压；
4. 归一化成 `python/<版本>/bin/python3`，并把 `python/current` 指向它。

第 2 步必须先问一次版本号：否则「本机已有 3.14.9」会让升级到「要求 3.15」的版本时
永远发现不了自己落后。

## 发布一个新解释器版本要做什么

1. 上传上游同名文件到 `pythondata/<新版本>/`（`gh release` 或普通 blob 均可，客户端
   只按路径取字节）；
2. 写 `pythondata/<新版本>/python_url.toml`，`*_url` 与 `*_sha256` 必须成对，
   各自取自同一个上游文件；
3. 最后改 `pythondata/latest.toml` —— 它就是「上线开关」。

`*_sha256` 必须是实测值。写错会让客户端在解压前就拒绝这份归档（这是有意的：
宁可装不上，也不要把来路不明的二进制解到用户机器上）。
