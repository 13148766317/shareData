WebRTC iOS 编译

# 思路
1. 根据官方文档
2. 参考网上其它脚本

WebRTC development - Prerequisite software
https://webrtc.googlesource.com/src/+/refs/heads/master/docs/native-code/development/prerequisite-sw/index.md

Install the Chromium depot tools.
https://commondatastorage.googleapis.com/chrome-infra-docs/flat/depot_tools/docs/html/depot_tools_tutorial.html#_setting_up

WebRTC development
https://webrtc.googlesource.com/src/+/refs/heads/master/docs/native-code/development/index.md

gn 编译脚本文档
https://gn.googlesource.com/gn/+/refs/heads/main/docs

https://www.re2x.com/WebRTC-wiki/zh-CN/

# 工具下载

## 下载depot_tools 

新建目录
`mkdir webrtcproject &&  cd webrtcproject`

下载工具
`git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git`

配置环境变量 ~/.bashrc or ~/.zshrc
`export PATH=/Users/appkle/hope2008/code/other/webrtcproject/depot_tools:$PATH` 

配置git config

```
git config --global user.name "John Doe"
git config --global user.email "jdoe@email.com"
git config --global core.autocrlf false
git config --global core.filemode false
# and for fun!
git config --global color.ui true
```

# 获取代码

```
mkdir webrtc && cd webrtc
fetch --nohooks webrtc_ios
gclient sync
```

# 指定Release

查看log 

https://webrtc.googlesource.com/src/+log/refs/heads/master


使用gclient sync -r eeab9ccb2417cab18ae1681c6644c25fa4eadcd3 指定版本

# 创建分支 

```
git checkout -b kdsrtc
or 
git new-branch kdsrtc
gclient sync
```

# 生成编译文件 
```
gn gen out/ios_64 --args='target_os="ios" target_cpu="arm64" ios_enable_code_signing=false'

#使用xcode 
gn gen out/ios_64_xcode --args='target_os="ios" target_cpu="arm64" ios_enable_code_signing=false' --ide=xcode
open -a Xcode.app out/ios_64_xcode/all.xcodeproj
```

如果要签名，修改参数，主要是签名大多了，见build/config/ios/ios_sdk.gni与build/config/ios/find_signing_identity.py 程序找不到签名，可查看源码


 

```

gn gen out/ios_64_test --args='target_os="ios" target_cpu="arm64" ios_code_signing_identity="8507A08DF58F8E31E470DEAA197EAE0EDBEF0F51"'

#进行查找
gn gen out/ios_64_test --args='target_os="ios" target_cpu="arm64" ios_code_signing_identity_description="Apple Development: Jing Rao (Z289S7XPAJ)"'


#使用这个成功
gn gen out/ios_64_test --args='target_os="ios" target_cpu="arm64" ios_code_signing_identity_description="Apple Development"'

#is_component_build 这个没什么用
gn gen out/ios_64_no --args='target_os="ios" target_cpu="arm64" is_component_build=false ios_enable_code_signing=false' --ide=xcode


```

其它参数
```
target_os = "ios"
ios_enable_code_signing = false
use_xcode_clang = true
is_component_build = false
rtc_include_tests = false
is_debug = false
target_cpu = "arm64"
ios_deployment_target = "10.0"
rtc_libvpx_build_vp9 = false
enable_ios_bitcode = false
use_goma = false
rtc_enable_symbol_export = true
enable_dsyms = true
enable_stripping = true
```

# 编译
```
ninja -C out/ios_64 framework_objc
ninja -C out/ios_64 AppRTCMobile

```

# 查看签名

`codesign -vv -d  WebRTC.framework`

# 手动签名

`codesign -f -s 'iPhone Developer: Thomas Kollbach (7TPNXN7G6K)' Example.app`

# 查看签名证书 

`xcrun security find-identity -v -p codesigning `

# 使用build_ios_libs.py编译
比较费时，会编译所有cpu架构与lipo整合
```
cd tools_webrtc/ios
python3 build_ios_libs.py
```
输出目录out_ios_libs


# 使用编译的WebRTC.framework
出现错误提示
```
 error: Building for iOS, but the linked and embedded framework 'WebRTC.framework' was built for iOS + iOS Simulator. (in target 'WebRTC-Demo' from project 'WebRTC-Demo')

```
更改Build Settings->Validate Workspace为YES
# 服务器下载代码脚本

```
#!/bin/bash

sudo apt-get update
cd /data/ && sudo mkdir webrtcproject && cd webrtcproject && sudo chmod a+w .

git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git

echo "export PATH=/data/webrtcproject/depot_tools:\$PATH" >> ~/.bashrc
export PATH=/data/webrtcproject/depot_tools:\$PATH
git config --global user.name "John Doe"
git config --global user.email "jdoe@email.com"
git config --global core.autocrlf false
git config --global core.filemode false
# and for fun!
git config --global color.ui true

screen 
cd /data/webrtcproject
mkdir webrtc && cd webrtc
fetch --nohooks webrtc_ios
gclient sync --jobs=4

```

# 其它

fetch --nohooks webrtc_ios 源代码13G

运行gclient sync 后 代码是 16G


```
gclient是用来同步代码，在和src同级目录会有一个隐藏.gclient文件，里面记录了基本的代码拉取设置
src里面的各个目录、甚至子目录，基本上都是一个独立的git库
 gclient sync的命令回去检查整个项目的完整情况，并同步代码
如果gclient sync无法通过，一般都不是代码的问题，是因为工具链或依赖库和当前代码需要的不一致
gclient是用来同步代码和工具链的
gn 是用来产生ninja所需的配置文件
ninja 才是编译的
代码里面有很多*.gni，可以认为是和make脚本差不多的，是告诉ninja，我要编译某个项目
例如AppRTCMobile，需要哪些代码文件、以来哪些库
```
# 出错记录

### gn gen out/ios_64 --args='target_os="ios" target_cpu="arm64"'

default 开启了签名参数，可以禁止签名 `ios_enable_code_signing=false`


```
gn gen out/ios_64 --args='target_os="ios" target_cpu="arm64"'
ERROR at //build/config/ios/ios_sdk.gni:181:33: Script returned non-zero exit code.
    ios_code_signing_identity = exec_script("find_signing_identity.py",
                                ^----------
Current dir: /Users/appkle/hope2008/code/other/webrtcproject/webrtc/src/out/ios_64/
Command: python3 /Users/appkle/hope2008/code/other/webrtcproject/webrtc/src/build/config/ios/find_signing_identity.py --matching-pattern Apple Development
Returned 1 and printed out:

Automatic code signing identity selection was enabled but could not
find exactly one codesigning identity matching "Apple Development".

Check that the keychain is accessible and that there is exactly one
valid codesigning identity matching the pattern. Here is the parsed
output of `xcrun security find-identity -v -p codesigning`:

  1) 8C24D***********************************: "Apple Development: fantem fu (MNB6N*****)"

See //build/config/sysroot.gni:74:5: whence it was imported.
    import("//build/config/ios/ios_sdk.gni")
    ^--------------------------------------
See //build/config/linux/pkg_config.gni:5:1: whence it was imported.
import("//build/config/sysroot.gni")
^----------------------------------
See //BUILD.gn:15:1: whence it was imported.
import("//build/config/linux/pkg_config.gni")
^-------------------------------------------
➜  src git:(kdsrtc)

See //build/config/sysroot.gni:74:5: whence it was imported.
    import("//build/config/ios/ios_sdk.gni")
    ^--------------------------------------
See //build/config/linux/pkg_config.gni:5:1: whence it was imported.
import("//build/config/sysroot.gni")
^----------------------------------
See //BUILD.gn:15:1: whence it was imported.
import("//build/config/linux/pkg_config.gni")
^-------------------------------------------
➜  src git:(kdsrtc)
  17) 712C6***********************************: "Apple Development: Jing Rao (73DWP*****)"
    17 valid identities found

See //build/config/sysroot.gni:74:5: whence it was imported.
    import("//build/config/ios/ios_sdk.gni")
    ^--------------------------------------
See //build/config/linux/pkg_config.gni:5:1: whence it was imported.
import("//build/config/sysroot.gni")
^----------------------------------
See //BUILD.gn:15:1: whence it was imported.
import("//build/config/linux/pkg_config.gni")
^-------------------------------------------
➜  src git:(kdsrtc)
  17) 712C6***********************************: "Apple Development: Jing Rao (73DWP*****)"
    17 valid identities found

See //build/config/sysroot.gni:74:5: whence it was imported.
    import("//build/config/ios/ios_sdk.gni")
    ^--------------------------------------
See //build/config/linux/pkg_config.gni:5:1: whence it was imported.
import("//build/config/sysroot.gni")
^----------------------------------
See //BUILD.gn:15:1: whence it was imported.
import("//build/config/linux/pkg_config.gni")
^-------------------------------------------
```

### ninja -C out/ios_64 framework_objc

```
/bin/sh: ../../third_party/llvm-build/Release+Asserts/bin/clang++: cannot execute binary file
ninja: build stopped: subcommand failed.
```

下载了错误的文件clang，因为是在linux上，而不是在macos上下载的代码

```
file third_party/llvm-build/Release+Asserts/bin/clang++
third_party/llvm-build/Release+Asserts/bin/clang++: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.24, stripped
```

删除，重新同步

```
rm -rvf ./third_party/llvm*
gclient sync
```

再看一次，已经正常

```
file third_party/llvm-build/Release+Asserts/bin/clang++
third_party/llvm-build/Release+Asserts/bin/clang++: Mach-O 64-bit executable x86_64
```

参考：
https://blog.csdn.net/Martin_chen2/article/details/107047778