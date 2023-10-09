#  CMake最低版本
CMAKE_MINIMUM_REQUIRED(VERSION 3.10)

#  项目名 语言
PROJECT(study_project C CXX)

#  C++版本
SET(CMAKE_CXX_STANDARD 20)

#  可执行程序的输出目录
SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)

#  生成文件的输出目录
SET(GENERATED_OUT_PATH ${PROJECT_SOURCE_DIR}/generated)

#  查找OpenCV
FIND_PACKAGE(OpenCV REQUIRED)
FIND_PACKAGE(Protobuf CONFIG REQUIRED)
FIND_PACKAGE(absl CONFIG REQUIRED)
FIND_PACKAGE(gRPC CONFIG REQUIRED)

# HCNetSDK

# 查找本地Protobuf模块的库信息，实际上CMake就是在找Protobuf-config.cmake文件
SET(protobuf_MODULE_COMPATIBLE TRUE)
FIND_PACKAGE(Protobuf CONFIG REQUIRED)
MESSAGE(STATUS "Using Protobuf ${Protobuf_VERSION}")

SET(_PROTOBUF_LIBPROTOBUF protobuf::libprotobuf)
SET(_REFLECTION gRPC::grpc++_reflection)
if(CMAKE_CROSSCOMPILING)
    FIND_PROGRAM(_PROTOBUF_PROTOC protoc)
ELSE()
    SET(_PROTOBUF_PROTOC $<TARGET_FILE:protobuf::protoc>)
ENDIF()

# 查找本地gRPC模块的库信息
FIND_PACKAGE(gRPC CONFIG REQUIRED)
MESSAGE(STATUS "Using gRPC ${gRPC_VERSION}")

SET(_GRPC_GRPCPP gRPC::grpc++)
IF(CMAKE_CROSSCOMPILING)
    FIND_PROGRAM(_GRPC_CPP_PLUGIN_EXECUTABLE grpc_cpp_plugin)
ELSE()
    SET(_GRPC_CPP_PLUGIN_EXECUTABLE ${TARGET_FILE:gRPC::grpc_cpp_plugin})
ENDIF()

#  proto文件生成代码文件

#  设置生成文件路径
SET(data_proto ${PROJECT_SOURCE_DIR}/resources/protos/study.proto)
SET(data_proto_srcs "${GENERATED_OUT_PATH}/protos/study_project/study.pb.cc")
SET(data_proto_hdrs "${GENERATED_OUT_PATH}/protos/study_project/study.pb.h")
SET(data_grpc_srcs "${GENERATED_OUT_PATH}/protos/study_project/study.grpc.pb.cc")
SET(data_grpc_hdrs "${GENERATED_OUT_PATH}/protos/study_project/study.pb.h")

#  生成文件
ADD_CUSTOM_COMMAND(
    OUTPUT "${data_proto_srcs}" "${data_proto_hdrs}" "${data_grpc_srcs}" "${data_grpc_hdrs}"
    COMMAND protoc
        ARGS --grpc_out "${GENERATED_OUT_PATH}/protos/study_project"
        --cpp_out "${GENERATED_OUT_PATH}/protos/study_project"
        -I "${PROJECT_SOURCE_DIR}/resources/protos"
        --plugin=protoc-gen-grpc=`which grpc_cpp_plugin`
        "${data_proto}"
    DEPENDS "${data_proto}"
)

# 将grpc的四个文件编译成一个库
ADD_LIBRARY(data_grpc_proto
    ${data_grpc_srcs}
    ${data_grpc_hdrs}
    ${data_proto_srcs}
    ${data_proto_hdrs}
)

# 链接库data_grpc_proto，会生成链接文件
TARGET_LINK_LIBRARIES(data_grpc_proto
    ${_REFLECTION}
    ${_GRPC_GRPCPP}
    ${_PROTOBUF_LIBPROTOBUF}
)

# include
include_directories("${GENERATED_OUT_PATH}")
include_directories("${PROJECT_SOURCE_DIR}/include")

SET(SOURCES

)

SET(HEADERS

)

ADD_EXECUTABLE(${PROJECT_NAME} ${SOURCES} ${HEADERS})

# OpenCV
TARGET_INCLUDE_DIRECTORIES(${PROJECT_NAME} PRIVATE ${OpenCV_INCLUDE_DIRS})
TARGET_LINK_LIBRARIES(${PROJECT_NAME} ${OpenCV_LIBS})

# HCNetSDK

# proto
TARGET_LINK_LIBRARIES(${PROJECT_NAME}
    data_grpc_proto
    ${_REFLECTION}
    ${_GRPC_GRPCPP}
    ${_PROTOBUF_LIBPROTOBUF}
)