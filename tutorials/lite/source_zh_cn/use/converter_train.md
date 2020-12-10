# 训练模型转换

<!-- TOC -->

- [训练模型转换](#训练模型转换)
    - [概述](#概述)
    - [Linux环境](#linux环境)
        - [环境准备](#环境准备)
        - [参数说明](#参数说明)
        - [模型转换示例](#模型转换示例)
    - [Windows环境](#Windows环境)

<!-- /TOC -->

<a href="https://gitee.com/mindspore/docs/blob/master/tutorials/lite/source_zh_cn/use/training_model_converting.md" target="_blank"><img src="../_static/logo_source.png"></a>

## 概述

创建MindSpore端侧模型的步骤：

- 首先基于MindSpore架构使用Python创建网络模型，并导出为`.mindir`文件，参见云端的[保存模型](https://www.mindspore.cn/tutorial/training/zh-CN/master/use/save_model.html#mindir)。
- 然后将`.mindir`模型文件转换成`.ms`文件，`.ms`文件可以导入端侧设备并基于MindSpore端侧框架训练。

## Linux环境

### 环境准备

MindSpore ToD 模型转换工具提供了多个参数，目前工具仅支持Linux系统，环境准备步骤：

- 编译或下载已编译好的模型转换工具。
- 配置模型转换工具的环境变量。

### 参数说明

下表为MindSpore ToD训练模型转换工具使用到的参数：

| 参数                        | 是否必选 | 参数说明                                    | 取值范围    | 默认值 |
| --------------------------- | -------- | ------------------------------------------- | ----------- | ------ |
| `--help`                    | 否       | 打印全部帮助信息                            | -           | -      |
| `--fmk=<FMK>`               | 是       | 输入模型的原始格式                          | MINDIR      | -      |
| `--modelFile=<MODELFILE>`   | 是       | MINDIR模型文件名（包括路径）                | -           | -      |
| `--outputFile=<OUTPUTFILE>` | 是       | 输出模型文件名（包括路径）自动生成`.ms`后缀 | -           | -      |
| `--trainModel=true`         | 是       | 当前模型是否在设备上训练                    | true, false | false  |

> 参数名称和数值之间使用等号连接且不能有空格。

### 模型转换示例

假设待转换的文件为`my_model.mindir`，执行如下转换命令（使用训练版本转换工具，参见[编译MindSpore Lite](https://www.mindspore.cn/tutorial/lite/zh-CN/master/use/build.html#id9)章节）：

```bash
./converter_lite --fmk=MINDIR --trainModel=true --modelFile=my_model.mindir --outputFile=my_model
```

转换成功输出如下：

```text
CONVERTER RESULT SUCCESS:0
```

这表明 MindSpore 模型成功转换为 MindSpore 端侧模型，并生成了新文件`my_model.ms`。如果转换失败输出如下：

```text
CONVERT RESULT FAILED:
```

程序会返回的[错误码](https://www.mindspore.cn/doc/api_cpp/zh-CN/master/errorcode_and_metatype.html)和错误信息。

## Windows环境

后续版本将会增加Windows平台下的训练模型转换工具。
