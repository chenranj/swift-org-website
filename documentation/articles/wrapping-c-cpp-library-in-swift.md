---
layout: page
date: 2023-11-08 12:00:00
title: 在 Swift 中封装 C/C++ 库
author: [etcwilde, ktoso, yim-lee]
---

有许多优秀的 C/C++ 库。在 Swift 代码中使用这些库是可能的,而无需将它们重写为 Swift。本文将解释实现这一点的几种方法以及在 Swift 中使用 C/C++ 时的最佳实践。

## Package

1. 如果需要,使用 `Package.swift`、`Sources` 目录等创建一个新的 Swift 包。
2. 在 `Sources` 下为 C/C++ 库创建一个新的模块/目录。假设在本节的其余部分中它被命名为 `CMyLib`。
    - 一个约定是在模块名称前加上 `C` 前缀。例如,`CDataStaxDriver`。
3. 将 C/C++ 库的源代码目录作为 Git 子模块添加到 `Sources/CMyLib` 下。
    - 如果设置正确,Swift 包的根目录中应该有一个 `.gitmodules` 文件,其内容如下:

	```
	[submodule "my-lib"]
        	path = Sources/CMyLib/my-lib
        	url = https://github.com/examples/my-lib.git
	```

4. 修改 `Package.swift` 以添加 `CMyLib` 作为目标,并指定源文件和头文件的位置。

	```swift
	.target(
      name: "CMyLib",
      dependencies: [],
      exclude: [
          // 要排除的文件和/或目录相对于 'CMyLib' 的路径。例如:
          // "./my-lib/src/CMakeLists.txt",
          // "./my-lib/tests",
      ],
      sources: [
          // 源文件和/或目录相对于 'CMyLib' 的路径。例如:
          // "./my-lib/src/foo.c",
          // "./my-lib/src/baz",
      ],
      cSettings: [
          // .headerSearchPath("./my-lib/src"),
      ]
	),
	```

	对于 C++ 库,使用 `cxxSettings` 而不是 `cSettings`。目标定义还有其他选项和参数可用。有关详细信息,请参阅 SwiftPM API 文档。

5. 尝试使用 `swift build` 编译 Swift 包。根据需要调整 `Package.swift`。

### 模块映射

除非存在自定义模块映射,否则会自动为 Clang 目标(例如 `CMyLib`)生成模块映射。(即头目录中存在 `module.modulemap` 文件)

模块映射生成的规则可以在[这里](https://github.com/swiftlang/swift-package-manager/blob/main/Sources/PackageLoading/ModuleMapGenerator.swift)找到。

### 包含 C/C++ 库构建生成的文件

某些 C/C++ 库在其构建中生成额外的必需文件(例如配置文件)。要在 Swift 包中包含这些文件:

1. `cd` 进入 C/C++ 库的根目录(例如 `Sources/CMyLib/my-lib`),然后构建它。
    - 回想一下上面的步骤 3,这是 Git 子模块的目录。<u>不应该在此目录中进行任何修改。</u>C/C++ 库构建生成的输出文件/目录应添加到 `.gitignore` 中。
2. 在 `Sources/CMyLib/` 下创建一个目录,用于存放必需的文件。(例如 `Sources/CMyLib/extra`)
3. 将 C/C++ 构建输出中生成的必需文件复制到上一步创建的目录中。
4. 更新 `Package.swift`,将步骤 2 中创建的目录(即 `extra`)的路径或单个文件路径(例如 `./extra/config.h`)添加到目标(即 `CMyLib`)的 `sources` 数组中,用于源文件,或作为 `.headerSearchPath`,用于头文件。

### 覆盖 C/C++ 库中的文件

要使用自定义实现而不是 C/C++ 库附带的实现:

1. 在 `Sources/CMyLib` 下创建一个目录,用于存放自定义代码文件。(例如 `Sources/CMyLib/custom`)
2. 将自定义代码文件添加到上一步创建的目录中。
    - 如果需要,为源文件和头文件创建单独的子目录。
3. 更新 `Package.swift`:
    - 将步骤 1 中创建的目录(即 `custom`)的路径或单个文件路径(例如 `./custom/my_impl.c`)添加到目标(即 `CMyLib`)的 `sources` 数组中,用于源文件,或作为 `.headerSearchPath`,用于头文件。
    - 将 C/C++ 库的文件路径添加到目标(即 `CMyLib`)的排除数组中。(例如 `./my-lib/impl.c`)

## CMake

此示例旨在将 C 库导入 Swift。您需要获取库,提供一个模块映射,以便 Swift 可以导入它,然后链接它。对于 C++,机制基本相同,有关如何在单个项目中构建的 C++ 库进行双向互操作的示例,请参阅 [Swift-CMake 示例存储库](https://github.com/apple/swift-cmake-examples/tree/main/3_bidirectional_cxx_interop)中的双向 cxx 互操作项目。


### 获取库

如果您不是与 Swift 库一起构建 C 库,则需要以某种方式获取库的副本。

* ExternalProject
    * 在构建时运行,可配置性最有限,但也将 C/C++ 库的构建与您的构建隔离开来。
    * 当您的项目与依赖项之间的耦合度较低,并且该库不太可能安装在您的项目预期运行的位置,或者当您需要对依赖项构建进行某种程度的可配置性时,这是一个不错的选择。
    * 更多详细信息请参见 [External Project](https://cmake.org/cmake/help/latest/module/ExternalProject.html)。


```cmake
include(ExternalProject)
ExternalProject_Add(ZLIB
    GIT_REPOSITORY "https://www.github.com/madler/zlib.git"
    GIT_TAG "09155eaa2f9270dc4ed1fa13e2b4b2613e6e4851" # v1.3
    GIT_SHALLOW TRUE

    UPDATE_COMMAND ""

    CMAKE_ARGS
      -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
)
ExternalProject_Get_Property(ZLIB INSTALL_DIR)
add_library(zlib STATIC IMPORTED GLOBAL)
set_target_properties(zlib PROPERTIES
  INTERFACE_INCLUDE_DIRECTORIES "${INSTALL_DIR}/include"
  IMPORTED_LOCATION "${INSTALL_DIR}/lib/libz.a"
)

add_executable(example example.c)
target_link_libraries(example PRIVATE zlib)
```

此示例从 GitHub 下载 zlib v1.3 并构建它。由于我们设置了固定的标签,因此不需要 CMake 尝试更新它,提交哈希的内容永远不会更改。External Project 创建的 `ZLIB` 目标是一个 CMake "实用程序"库,因此我们不能直接链接它。相反,我们可以在设置 `ZLIB_DIR` 到构建目录后使用 `find_package` 来找到它,或者由于我们已经知道它存在的位置,我们可以创建一个导入的静态库。将 `INTERFACE_INCLUDE_DIRECTORIES` 设置为安装 zlib 头文件的位置,将 `IMPORTED_LOCATION` 设置为静态存档,会生成一个我们可以链接代码的目标。然后,CMake 将告诉编译器在哪里查找头文件,并为链接到导入的 `zlib` 目标的任何目标链接静态存档。

* `FetchContent`
    * 在配置时运行,并导致合并的构建图。这最适合拉入作为库实现细节的外部部分。请注意,由于构建图是合并的,因此变量名称和目标需要适当地命名空间,否则它们将发生冲突,并且可能无法按预期构建。
    * 当您的项目与依赖项之间存在紧密耦合时,这是一个不错的选择。由于构建图是合并的,您的项目可以依赖于依赖项中的单个构建目标,而不是依赖于整个项目,这可以提高构建性能。
    * 更多详细信息请参见 [FetchContent](https://cmake.org/cmake/help/latest/module/FetchContent.html)。


* `find_package`
    * 从 sysroot 中查找库和头文件。默认情况下,CMake 将操作系统的根目录视为 sysroot,但可以隔离到其他 sysroot 以进行交叉编译。
    * 此选项适用于从基本系统或 sysroot 中获取系统依赖项,或者通过使用 `<PackageName>_ROOT` 为项目的分发者提供使用预构建项目的选项。
    * 更多详细信息请参见 [find_package](https://cmake.org/cmake/help/latest/command/find_package.html)。

 使用 CMake 将现有 C 库封装在 Swift 中的示例将使用 `find_package`、自定义模块映射文件和虚拟文件系统 (VFS) 覆盖,以及帮助程序层,以将 SQLite 代码库的部分迁移到 Swift 可以导入的内容。

### 入门

从基本的 CMake 设置开始:

```cmake
cmake_minimum_required(VERSION 3.26)
project(SQLiteImportExample LANGUAGES Swift C)
```

这将创建一个名为 "SQLiteImportExample" 的 CMake 项目,该项目使用 Swift 和 C,并需要 CMake 3.26 或更高版本。

在此示例中,我们不会构建 SQLite,而是从系统或提供的 sysroot 中提取它。

```cmake
find_package(SQLite3 REQUIRED)
```

这告诉 CMake 根据 `FindSQLite3.cmake` 包文件查找 SQLite3。由于我们将其标记为必需的依赖项,如果找不到包的某些部分,CMake 将停止构建的配置。

找到后,CMake 定义以下变量:
* `SQLite3_INCLUDE_DIRS` — 找到 `sqlite3.h` 的文件路径
* `SQLite3_LIBRARIES` — sqlite 的使用者需要链接的库
* `SQLite3_VERSION` — 找到的 sqlite3 版本
* `SQLite3_FOUND` — 用于告诉 `find_package` 找到了 SQLite。请注意,如果我们没有将其标记为 `REQUIRED` 包,我们可以稍后检查此变量以查看是否找到它,如果没有找到,则回退到 `ExternalProject` 以单独构建它。

CMake 还将定义 `SQLite::SQLite3` 构建目标,我们稍后将使用它来更轻松地通过构建图传播依赖项和搜索位置信息。有关 SQLite3 包的文档,请参见此处:[FindSQLite3](https://cmake.org/cmake/help/latest/module/FindSQLite3.html)。


### 将 SQLite 导入 Swift

Swift 无法直接导入头文件。SwiftPM 和 Xcode 等一些工具有时可以为桥接头文件生成模块映射,但其他工具(如 CMake)则不能。手动编写模块映射可以让您更好地控制如何将 C 库导入 Swift。有关如何编写模块映射文件的详细信息,请参阅[模块映射语言](https://clang.llvm.org/docs/Modules.html#module-map-language)规范。

对于我们的示例,我们只需要将 `sqlite3.h` 头文件公开给 Swift。

我们的 `sqlite3.modulemap` 文件的内容如下:

```c
module CSQLite {
  header "sqlite3.h"
}
```

模块名称表示我们用于将此模块导入 Swift 的名称。对于我们的示例,相应的 Swift 导入语句将是 `import CSQLite`。

我们可以包含其他指令,例如 `link "sqlite3"` 以指示自动链接机制它应该自动链接到 sqlite3 库,但这对我们的目的来说是不必要的,因为当我们告诉我们的程序链接到 sqlite 库时,CMake 将自动为我们执行此操作。

现在,我们需要将模块映射文件放在正确的位置。我们希望模块映射文件与 `sqlite.h` 文件放在一起,但根据 `sqlite.h` 的位置,我们可能无法访问它。这就是虚拟文件系统的用武之地。虚拟文件系统(或 VFS)是编译器对文件系统的视图。VFS 覆盖文件允许我们覆盖该视图,以便我们可以从编译器的视图更改文件名并将文件放置在文件系统中的任何位置,而无需实际将它们放在物理驱动器上。

输入 VFS 覆盖的格式是 YAML(请注意,JSON 是 YAML 的子集,因此如果您愿意,可以将其表示为 JSON 对象)。缺点是此文件需要根的绝对路径,或者要覆盖的位置。根据您要写入的位置,该位置可能不可移植,因此硬编码这些文件可能无法工作。但是,我们可以使用 CMake 动态生成适用于我们系统的覆盖。我们将向项目添加以下模板并将其称为 `sqlite-vfs-overlay.yaml`。

```yaml
---
version: 0
case-sensitive: false
use-external-names: false
roots:
  - name: "@SQLite3_INCLUDE_DIR@"
    type: directory
    contents:
      - name: module.modulemap
        type: file
        external-contents: "@SQLite3_MODULEMAP_FILE@"
```

但是,该文件是不完整的。我们将将覆盖模板与以下 CMake 配对,以发出与我们的环境匹配的最终覆盖文件。

```cmake
# 设置 VFS 覆盖以将自定义模块映射注入到 Swift 中导入 SQLite
set(SQLite3_MODULEMAP_FILE "${CMAKE_CURRENT_SOURCE_DIR}/sqlite3.modulemap")
configure_file(sqlite-vfs-overlay.yaml "${CMAKE_CURRENT_BINARY_DIR}/sqlite3-overlay.yaml")

target_compile_options(SQLite::SQLite3 INTERFACE
  "$<$<COMPILE_LANGUAGE:Swift>:SHELL:-vfsoverlay ${CMAKE_CURRENT_BINARY_DIR}/sqlite3-overlay.yaml>"
)
```

结果是一个 VFS 覆盖文件,它将自定义模块映射文件注入到 `sqlite.h` 所在的目录中,同时将 `sqlite.3.modulemap` 重命名为 `module.modulemap`。所有使用 SQLite3 库的 Swift 程序都需要使用相关的 VFS 覆盖文件才能找到模块映射。我们使用 `target_compile_options` 来添加它。由于 `SQLite::SQLite3` 是一个导入的库,它无法影响构建 SQLite 本身,因此我们将其添加为 `INTERFACE` 选项,确保它传播到所有依赖它的目标。

现在在此项目上运行 CMake 应该会报告您缺少 SQLite,在这种情况下,您需要安装它才能使用它,或者将 `sqlite3-overlay.yaml` 发送到构建目录的顶部。

```yaml
---
version: 0
case-sensitive: false
use-external-names: false
roots:
  - name: "/usr/include"
    type: directory
    contents:
      - name: module.modulemap
        type: file
        external-contents: "/home/ewilde/sqlite-import-example/sqlite3.modulemap"
```

这是在我的 Linux 系统上发出的,其中 `sqlite3.h` 位于 `/usr/include`,项目源位于我主目录中的一个目录中。

这应该足以导入内容。总结一下,我们的项目中总共有四个文件:

* `sqlite3.modulemap` 告诉 Swift 哪些 C 头文件与哪些导入的模块相关联。
* `sqlite-vfs-overlay.yaml` 告诉 Swift 将 sqlite3 模块映射文件注入到正确的位置以进行导入,而无需更改实际系统。
* `CMakeLists.txt` 组织配置 VFS 覆盖,然后构建项目。
* `hello.swift` 调用 C SQLite 库。

```c
// sqlite3.modulemap
module CSQLite {
  header "sqlite3.h"
}
```

```yaml
# sqlite-vfs-overlay.yaml
---
version: 0
case-sensitive: false
use-external-names: false
roots:
  - name: "@SQLite3_INCLUDE_DIR@"
    type: directory
    contents:
      - name: module.modulemap
        type: file
        external-contents: "@SQLite3_MODULEMAP_FILE@"
```

```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.26)
project(SQLiteImportExample LANGUAGES Swift C)

find_package(SQLite3 REQUIRED)

# 设置 VFS 覆盖以注入自定义模块映射文件
set(SQLite3_MODULEMAP_FILE "${CMAKE_CURRENT_SOURCE_DIR}/sqlite3.modulemap")
configure_file(sqlite-vfs-overlay.yaml
               "${CMAKE_CURRENT_BINARY_DIR}/sqlite3-overlay.yaml")
target_compile_options(SQLite::SQLite3 INTERFACE
  "$<$<COMPILE_LANGUAGE:Swift>:SHELL:-vfsoverlay ${CMAKE_CURRENT_BINARY_DIR}/sqlite3-overlay.yaml>")

add_executable(Hello hello.swift)
target_link_libraries(Hello PRIVATE SQLite::SQLite3)
```

```swift
// hello.swift
import CSQLite

public class Database {
    var dbCon: OpaquePointer!

    public struct Flags: OptionSet {
        public let rawValue: Int32

        public init(rawValue: Int32) {
            self.rawValue = rawValue
        }

        public static let readonly = Flags(rawValue: SQLITE_OPEN_READONLY)
        public static let readwrite = Flags(rawValue: SQLITE_OPEN_READWRITE)
        public static let create = Flags(rawValue: SQLITE_OPEN_CREATE)
        public static let deleteOnClose = Flags(rawValue: SQLITE_OPEN_DELETEONCLOSE)
    }

    public init?(filename: String, flags: Flags = [.create, .readwrite]) {
        guard sqlite3_open_v2(filename, &dbCon, flags.rawValue, nil) == SQLITE_OK,
              dbCon != nil else {
            return nil
        }
    }

    deinit {
        sqlite3_close_v2(dbCon)
    }
}

guard let database = Database(filename: ":memory:") else {
    fatalError("由于某种原因无法加载数据库")
}
```

## 使用 C/C++

### 管理封装的 C/C++ 类型的生命周期

当封装具有指定生命周期的 C/C++ 类型时,例如由某种初始化和稍后的 "destroy" 调用概述的生命周期,在 Swift 中有两种方法可以实现这一点。当封装具有某些 `resource_init()` 和 `resource_destroy(the_resource)` API 的 C 类型时,这种情况尤其常见。

第一种方法是使用 Swift 类来封装资源并通过类的 `init`/`deinit` 管理其生命周期。下面是一个封装 RocksDB 中 C 管理的设置对象的示例:

```swift
public final class WriteOptions {
    let underlying: OpaquePointer!

    public init() {
        underlying = rocksdb_writeoptions_create()
    }

    deinit {
        rocksdb_writeoptions_destroy(underlying)
    }
}
```

第二种方法是使用 "不可复制"类型(您可能从其他语言中知道它们是仅移动类型)。要使用不可复制类型声明类似的 `WriteOptions` 包装器,您可以执行以下操作:

```swift
public struct WriteOptions: ~Copyable {
    let underlying: OpaquePointer!

    public init() {
        underlying = rocksdb_writeoptions_create()
    }

    deinit {
        rocksdb_writeoptions_destroy(underlying)
    }
}
```

不可复制类型的缺点是目前它们不能在所有上下文中使用。例如,在 Swift 5.9 中,不可能将不可复制类型存储为字段,或通过闭包传递它们(因为闭包可以多次使用,这会破坏不可复制类型需要保证的唯一性)。优点是,与类不同,不可复制类型不执行引用计数。