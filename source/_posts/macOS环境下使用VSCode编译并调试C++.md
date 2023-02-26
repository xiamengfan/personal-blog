---
title: macOS环境下使用VSCode编译并调试C++
date: 2021-07-19 00:03:27
tags: VSCode
---

通过配置tasks.json完成c++代码的自动编译任务，然后再通过配置launch.json对编译完的文件自动执行并调试。

配置tasks.json的过程很顺利，很快就可以看的项目根目录下编译生成了代码对应的out文件，但是调试过程确一直没法正常运行，
折腾半天才知道mac在更新到Catalina后不再支持lldb调试，还好有便捷的替代方案可以解决：通过安装CodeLLDB插件进行launch.json的配置。

[解决vscode在macos Catalina上配置C/C++环境及调试问题](https://zhuanlan.zhihu.com/p/106935263?utm_source=wechat_sessio)

最后附上自己配置完成后c_cpp_properties.json、tasks.json、launch.json的文件信息。

``` JSON
{
    "configurations": [
        {
            "name": "Mac",
            "includePath": [
                "${workspaceFolder}/**"
            ],
            "defines": [],
            "macFrameworkPath": [
                "/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/System/Library/Frameworks"
            ],
            "compilerPath": "/usr/bin/clang",
            "cStandard": "c11",
            "cppStandard": "c++17",
            "intelliSenseMode": "clang-x64"
        }
    ],
    "version": 4
}
```

``` JSON
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "c++",
            "command": "g++",
            "type": "shell",
            "args": [
                "-g",
                "${file}",
                "-std=c++17",
                "-o",
                "${fileBasenameNoExtension}.out"
            ],
            "presentation": {
                "echo": true,
                "reveal": "always",
                "focus": false,
                "panel": "shared"
            },
            "group": {
                "kind": "build",
                "isDefault": true
            }
        }
    ]
}
```

``` JSON
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(lldb) Launch",
            "preLaunchTask": "c++",
            "type": "lldb",
            "request": "launch",
            "program": "${fileDirname}/${fileBasenameNoExtension}.out",
            "args": [],
            "cwd": "${workspaceFolder}",
        }
    ]
}
```