# Persistency 用户手册

## 安装
---

**安装前准备**：

所需环境：
- `Ubuntu 18.04` 及以上版本
- `gcc 9.3` 及以上版本
- `xmake 2.6.9` 及以上版本
- `cmake 3.15` 及以上版本

编译命令：
- boyan整编之后，单独编译Persistency模块，使用以下命令编译的平台版本与boyan整编时相同。即：boyan整编时选择编译x86，Persistency单独编译时也会编译x86版本

```bash
# 在顶层目录, 输入:
source buildtools/envsteup.sh
cd ../per
# 编译
./build.sh all
# 清理
./build.sh clean
```

<br>

- 未进行boyan整编(不推荐)，**在编译之前必须手动编译`Persistency`依赖库**
```bash
# 在顶层目录, 输入:
source buildtools/envsteup.sh
# 选择要编译的平台版本
lunch
cd ../per
# 编译x86版本
xmake f -m [release|debug|coverage|asan] --enable_boyan_sdk=[true|false] -o ${BUILD_OUTPUT_PATH}
# 编译aarch64版本
xmake f -p cross -a [aarch64|arm64] -m [release|debug|coverage|asan] --sdk=${TOOLCHAIN_PATH} --enable_boyan_sdk=[true|false] -o ${BUILD_OUTPUT_PATH}
# 清理
xmake clean -a

```
- 编译选项解释：
  - release： 发布版本
  - debug： 测试版本
  - coverage： 生成代码覆盖率版本
  - asan： asan测试版本
  - enable_boyan_sdk：是否生成doc文档（boyan整编时代表是否生成sdk）
  - sdk：指定交叉编译器路径（仅aarch64编译使用）
  - -o：指定编译产生的文件缓存路径


<br>
<br>

## 快速开始
---

下面使用example中的`simply_read_and_write_files.cpp`为例，进行快速集成演示


1. 准备Persistency运行时需要的配置文件`Manifest.json`，这里提供了默认配置文件供参考

```json
{
	"manifest": {
		"manifest_version": "v1.0.0",
		"executable_vesion": "v1.0.1",
		"central_storage_uri": "./central_storage_uri",
		"general_config": {
			"minimum_sustained_size": 1024,
			"maximum_allowed_size": 2048,
			"collection_update_strategy": 1,
			"element_update_strategy": 1,
			"redundancy_granularity": 1,
			"redundancy_n": 3,
			"redundancy_m": 2,
			"need_encryption": true
		},
		"key_value_storage_list": [
			{
				"key": "keyValueStorage_1",
				"value": {
					"uri": "./var/kvs",
					"default_file": [
						"default_kvs/keyValueStorage_1_default.json"
					],
					"access_mode": "read_write",
					"layout": "json"
				}
			},
			{
				"key": "keyValueStorage_2",
				"value": {
					"uri": "./var/kvs",
					"default_file": [
						"default_kvs/keyValueStorage_2_default.db"
					],
					"access_mode": "read_write",
					"layout": "sqlite"
				}
			}
		],
		"file_storage_list": [
			{
				"key": "fileStorage_1",
				"value": {
					"uri": "./var/fs",
					"default_file": [
						"test1.txt",
						"test2.txt",
						"test3.txt"
					],
					"access_mode": "read_write",
					"layout": "txt"
				}
			},
			{
				"key": "fileStorage_2",
				"value": {
					"uri": "./var/fs",
					"default_file": [
						"test4.bin",
						"test5.bin",
						"test6.bin"
					],
					"access_mode": "read_only",
					"layout": "bin"
				}
			}
		]
	}
}
```

<br>
<br>


2. 编写代码集成到你的项目中
```cpp
/*
 * Copyright (C) Horizon Robotics, Inc. - All Rights Reserved
 * Unauthorized copying of this file, via any medium is strictly prohibited
 * Proprietary and confidential
 */

#include <iostream>
// 引入Persistency中进行文件存储的头文件
#include "persistency/file_storage.h"

// 这里为了方便使用使用了using
using hra::core::InstanceSpecifier;
using hra::per::FileStorage;
using hra::per::OpenFileStorage;
using hra::per::ReadAccessor;

// 定义一个文件名
static const auto kTextFileName{"example_1.txt"};

int main(int argc, char** argv) {
  (void)argc;
  (void)argv;
  // 创建一个InstanceSpecifier实例，该实例名称必须已经在Manifest.json文件中声明
  auto instance = hra::core::InstanceSpecifier("fileStorage_1");
  // 使用OpenFileStorage全局函数打开该实例，获取FileStorage指针
  std::shared_ptr<FileStorage> file_storage = OpenFileStorage(instance).Value();
  // 调用FileStorage对象的OpenFileReadWrite函数打开某个文件，获取其accessor
  auto accessor =
    file_storage->OpenFileReadWrite(kTextFileName,
                               ReadAccessor::OpenMode::kAtTheBeginning).Value();

  // 使用accessor操作该文件内容
  std::string str("Hello, Persistency");
  accessor->WriteText(str);
  accessor->SyncToFile();

  std::cout << "Write Complete ----------------------------------" << std::endl;

  accessor->SetPosition(0);
  auto read_text = accessor->ReadText();
  std::cout << "Read Result: " << read_text.Value().data() << std::endl;
  return 0;
}
```

<br>
<br>

3. 更多用法请参考其他example或api文档
