# For more information about using CMake with Android Studio, read the
# documentation:
# https://developer.android.google.cn/studio/projects/add-native-code#new-project

#########################################################################
#  一
# 这里设置的是最小的cmake版本,一般不需要更改,记得在as更新的时候更新下载的cmake版本
# Sets the minimum version of CMake required to build the native library.
#########################################################################
cmake_minimum_required(VERSION 3.4.1)


#########################################################################
#  二
# 将项目中的c/c++ 源文件打包到指定的库中
# 通过调用个add_library() 函数,可以指定多个源文件,用空格隔开
# 函数需要三个参数值,参数值之间用空格或者回车进行分割
# 1 自定义库名称
# 2 SHARED 共享的库,还有一种选择是 STATIC 静态的,一般使用前者
# 3 库用到的源文件的路径
# 可以通过多次调用次函数,把不同的c/c++ 打包成不同的库
# 4 面对更对的c文件,建议使用函数来处理
#########################################################################

# 查询指定目录下的全部文件, 保存到指定的变量中
aux_source_directory(src/main/c/  ALL_C_FILES)

#加载少数的文件
#add_library(testgo SHARED src/main/c/test.cpp)

add_library(testgo SHARED ${ALL_C_FILES})

# 或者如下
# add_library( 自定义的库的名称
#              SHARED
#              源文件路径
#            )


###########################################################################
#  三
# 指定用到的头文件的地址
# 使用include_directories()函数,为了cmake能找到头文件的路径
# 例如:
# 这里是目录, 所以以/结尾,该目录下应该都是头文件
#
###########################################################################
include_directories(src/main/c/)



############################################################################
#  四
# 使用ndk中已经存在的库/或者说是导入其他库,这里找到指定库并重命名了,并未链接到其他的库中.
# NDK 还以源代码的形式包含一些库，在构建和关联到您的原生库时需要使用这些代码
# 可以使用 add_library() ，将源代码编译到原生库中。要提供本地 NDK 库的路径，
# 可以使用 ANDROID_NDK 路径变量，Android Studio 会自动为您定义此变量.
# 使用find_library() 函数
# 函数2个参数
# 1 自定义的库名称  2 想要cmake加载ndk中定义好的库名称(这个是已经事先存在的)
#
# Searches for a specified prebuilt library and stores the path as a
# variable. Because CMake includes system libraries in the search path by
# default, you only need to specify the name of the public NDK library
# you want to add. CMake verifies that the library exists before
# completing its build.
#############################################################################

# 这里的log-lib 是一个存储了log的路径变量,供后续的target_link_libraries()函数使用
find_library(log-lib log)

# 源码的事例
add_library( app-glue
             STATIC
             ${ANDROID_NDK}/sources/android/native_app_glue/android_native_app_glue.c )

############################################################################
#  五
# 为了确保自定义的原生库可以在需要的库中调用函数，
# 需要使用target_link_libraries() 函数关联库
# 函数有2个参数
# 1 目标库,即你希望哪个库可以调用已经存在的库
# 2 被连接的库,即已经存在的库
# 例如:
# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in this
# build script, prebuilt third-party libraries, or system libraries.
############################################################################

# 此处${} 的不能去掉,表示使用的是log-lib 这个变量的值
target_link_libraries(testgo ${log-lib} )

target_link_libraries( testgo app-glue  )


############################################################################
# 六 添加其他预构建库
#    添加预构建库与为 CMake 指定要构建的另一个原生库类似。
#    由于库已经预先构建，需要使用 IMPORTED 标志告知 CMake 只希望将库导入到项目中
#    然后，需要使用 set_target_properties() 命令指定库的路径
#    为了确保可以在编译时定位您的标头文件，需要使用 include_directories()包含标头文件的路径
#    要将预构建库关联到您自己的原生库，请将其添加到 CMake 构建脚本的 target_link_libraries()
############################################################################

#add_library( imported-lib
#             SHARED
#             IMPORTED )   # 只是导入

#set_target_properties(imported-lib
#                       PROPERTIES IMPORTED_LOCATION          # 属性,导入路径
#                       imported-lib/src/${ANDROID_ABI}/libimported-lib.so )

#include_directories( imported-lib/include/ )
#target_link_libraries(testgo imported-lib  )