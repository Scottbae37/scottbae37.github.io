## Modern CMake for modular design(<= ver 3.0)

* 해당 Target에 대한 Build flag를 세팅할 것
* 해당 Target에 대한 dependency를 세팅할 것
* 다른 Target의 내부를 깊게 고려하지 말 것(Keep out of other modules internals)
* Target에 ADD_LIBARY 또는 ADD_EXECUTABLE을 사용
* build flag세팅의 경우: TARGET_XXX를 사용할 것
* Target의 dependency를 선언할 경우: TARGET_LINK_LIBRARIES를 사용할 것
* Don't use macros that affect all targets
 * include\_directories()
 * add\_definitions()
 * link\_libraries()
 * Don't target\_include\_directories() with a path outside your module


### Global setup
* 모든 프로젝트에서 공용으로 사용되어야 할 flag 세팅할 경우
```cmake
cmake_minimum_required(VERSION 3.0)

if(MSVC)
  add_complie_options(/W3 /WX)
else()
  add_complie_options(-W -Wall -Werror)
endif()
```
### Declare module
```cmake
add_library(MyTarget src/file1.cpp)
add_executable(MyTargetExe src/main.cpp)
```

### Declare flags
* target\_include\_directories: 해당 모듈에 대한 include 폴더 설정
 * PUBLIC : 나의 모듈에 대한 public 헤더 include directory를 가르킴
 * PRIVATE : 나의 모듈을 빌드할 때만 include되고, 다른 모듈은 못봄
* target\_compile\_features: 언어 기능 사용
* target\_compile\_definitions: Specify compile definitions
 * PUBLIC의 경우 PUBLIC에 영향을 주고
 * PRIVATE의 경우 implementation쪽에 영향을 주게 할 수 있음.
* target\_compile\_options: Specify compile options

```cmake
target_include_directories(MyTarget PUBLIC inc)
target_include_directories(MyTarget PRIVATE src)

target_compile_features(MyTarget
    PUBLIC
        cxx_variadic_templates
        cxx_nullptr
    PRIVATE
        cxx_lambdas
)

target_compile_definitions (MyTarget PRIVATE -D_XOPEN_SOURCE=600)
target_compile_options (MyTarget PRIVATE -Wextra -Wpedantic)
```

### Declare dependencies
 * PRIVATE: 해당 바이너리를 생성할 때만 유효한 것임.
 * PUBLIC: 내 모듈과 링킹 되는 것들도 알아야할 경우, Client측에서도 알아야 하는 경우
 * INTERFACE: 나는 필요 없지만, 내가 의존하는 애가 필요로 함

```cmake
target_link_libraries(MyTarget
  PUBLIC needs_if_you_use_my_target
  PRIVATE needs_only_impl_inside_my_target)
```

### Header-only library

* Header only library CMakeLists.txt
```cmake
  add_library(my_header_only_lib INTERFACE)
  target_include_directories(my_header_only_lib INTERFACE include)
```

* Client-side CMakeLists.txt
```cmake
  ADD_LIBRARY(MyTarget STATIC src/MyTarget.c)
  TARGET_INCLUDE_DIRECTORIES(MyTarget PUBLIC inc)
  target_link_libraries(MyTarget
      PRIVATE my_header_only_lib)
```

### Comment
```cmake
#[==[
 Multiline
 comments
#]==]
```

### Function


### Macro


### External libraries
* Always like this:
```cmake
find_package(Foo 2.0 REQUIRED)
# ...
target_link_libraries(... Foo::Foo ...)
```



### Example: Private library usage

* Main.c
```c
#include <Calculator.h>
//#include <PrivateLib.h> Can not access because PrivateLib module is CalcLib's private dependency module

int main() {
  addLib(2, 1);
  return 0;
}
```

* Calculator.c

```c
#include <Calculator.h>
#include <Printer.h>

int add2(int a, int i);

int addLib(int a, int b){
  showResult(add2(a, b));
  return add2(a, b);
}

int add2(int a, int i) {
  return 0;
}
```

 * CMakeList.txt
```cmake
cmake_minimum_required(VERSION 3.9)
project(CMakeModuleExam)

set(CMAKE_C_STANDARD 11)

add_library(HeaderOnly INTERFACE)
target_include_directories(HeaderOnly INTERFACE src/header_only)

add_executable(MainModule main.c)

add_library(CalcLib
        src/calc/src/Calculator.c
        src/calc/src/AddFunc.c)
add_library(PrivateLib
        src/private_lib/src/Printer.c)
#PrivateLib module is used internally in CalcLib

target_include_directories(CalcLib
        PUBLIC src/calc/inc)
target_include_directories(PrivateLib
        PUBLIC src/private_lib/inc)

target_link_libraries(CalcLib
        PRIVATE PrivateLib)
#PRIVATE keyword prevent dependent module from propagate outside of CalcLib module

target_link_libraries(MainModule
        PRIVATE CalcLib)
#MainModule doen't know about CalcLib's private dependencies(ex. PrivateLib module)
```

### Google Test 사용

* CMake 3.5: Boost, GTest, GTK, PNG, TIFF

```cmake
find_package(GTest REQUIRED)
include(GoogleTest)

target_link_libraries(exeutable
  PRIVATE GTest::GTest
  PRIVATE GTest::Main
  )

```
