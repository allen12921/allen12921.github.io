---
title: "Build chromium for your aarch64 Linux"
categories:
  - 坑
tags:
  - browser
---
# 缘起
最近有研发同学需要在线上服务器中用Puppeteer来进行在浏览器中截图的操作，于是乎需要安装chrome，x86的机器上yum一键搞定，就在临门一脚准备上线时，突然发现线上还有arm的机器在运行，Google了一番，发现官方居然不提供arm的安装包，且网络中也无存在的对应操作系统可用安装包(OS为Amazone Linux 2)，于是开启了长达一天的踩坑之路，谨以此文献给欲后来者。

# 缘灭
目前源码编译只支持在ubuntu14，16，18，20等LTS的系统进行，且ubuntu 14,16即将被移除支持的OS列表，经测试发现ubuhtu 14已经无法完成编译。
由于arm资源有限，我们选择在x86上进行交叉编译的方式，以下命令均运行在ubuntu 16（下述过程中会拉取chromium编译相关的各种源码，大约27GB，如果网络不“畅通”，建议提前放弃）,期间可能遇到缺少python依赖的问题，直接pip install安装即可
```shell
apt install git ccache -y
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
export PATH="$PATH:/home/ec2-user/depot_tools"
mkdir ~/chromium && cd   ~/chromium
fetch --nohooks --no-history  chromium
cd src
./build/install-build-deps.sh
./build/linux/sysroot_scripts/install-sysroot.py   --arch=arm64
cat <<! >../.gclient
solutions = [
  {
    "name": "src",
    "url": "https://chromium.googlesource.com/chromium/src.git",
    "managed": False,
    "custom_deps": {},
    "custom_vars": {"checkout_pgo_profiles": True},
  },
]
!
gclient runhooks
gn gen out/Default  --args="target_cpu=\"arm64\" enable_nacl=false symbol_level=0 blink_symbol_level=0 v8_symbol_level=0 is_debug=false dcheck_always_on=false is_official_build=true cc_wrapper=\"ccache\" "
autoninja -C out/Default   chrome
```
一段时间后，如需更新最新chromium代码,只需执行：
```shell
git rebase-update
gclient sync
```
到此为止（many hours later），应该可以得到out/Default目录，里面包含了运行chromium所需的"所有"内容，让其打包传送到arm系统且解决了一堆lib依赖后，发现依然存在如下报错信息，究其原因，乃当前操作系统所用glibc版本低于编译执行文件的版本所致。

```[ec2-user@allen chromium-arm64]$ ./chrome
./chrome: /lib64/libm.so.6: version `GLIBC_2.27' not found (required by ./chrome)
./chrome: /lib64/libm.so.6: version `GLIBC_2.29' not found (required by ./chrome)
```
*千万不要听信网上的误人教程--直接编译新版glibc的源码，并安装到根目录*

# 结束语
* 对于Centos 7和Amazone Linux 2建议直接改用firefox,如果是Centos 8 可以通过epel 仓库中安装chromium包。
* 对于Ubuntu 16及以上的ARM系统可以通过snap install chromium,建议使用Ubuntu 20
* 由于我们的系统为Amazone Linux 2，所以直接改用epel中的firefox。
* 使用firefox时，依然会有其它问题，在此列举一、二:
  - Puppeteer使用firefox打开网页时,wait_until参数不支持"networkidle0"值,因此无法保证加载完所有页面内嵌资源(可以使用"domcontentloaded"+sleep不完美替换)

> 参考
> > https://github.com/chromium/chromium/blob/main/build/install-build-deps.sh
> > https://github.com/chromium/chromium/blob/main/docs/linux/build_instructions.md
<script src="{{ "/assets/js/mermaid.min.js" | relative_url }}"></script>
