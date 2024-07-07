cmake_minimum_required(VERSION 3.16)

project(FunASRWebscoket) 

set(CMAKE_CXX_STANDARD 14 CACHE STRING "The C++ version to be used.")
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)


option(ENABLE_GLOG "Whether to build glog" ON)
option(ENABLE_WEBSOCKET "Whether to build websocket server" ON)
option(BUILD_SHARED_LIBS "Build shared libraries" ON)
option(ENABLE_PORTAUDIO "Whether to build websocket server" ON)
option(ENABLE_FST "Whether to build openfst" ON) # ITN need openfst compiled
 
if(ENABLE_WEBSOCKET)
  # cmake_policy(SET CMP0135 NEW)
  include(FetchContent)
  FetchContent_Declare(websocketpp
  GIT_REPOSITORY https://github.com/zaphoyd/websocketpp.git
    GIT_TAG 0.8.2
    SOURCE_DIR ${PROJECT_SOURCE_DIR}/third_party/websocket
    )
  
  FetchContent_MakeAvailable(websocketpp)
  include_directories(${PROJECT_SOURCE_DIR}/third_party/websocket)
   
  FetchContent_Declare(asio
     URL   https://github.com/chriskohlhoff/asio/archive/refs/tags/asio-1-24-0.tar.gz
   SOURCE_DIR ${PROJECT_SOURCE_DIR}/third_party/asio
  )
  
  FetchContent_MakeAvailable(asio)
  include_directories(${PROJECT_SOURCE_DIR}/third_party/asio/asio/include)
 
  FetchContent_Declare(json
     URL   https://github.com/nlohmann/json/archive/refs/tags/v3.11.2.tar.gz
   SOURCE_DIR ${PROJECT_SOURCE_DIR}/third_party/json
  )
  
  FetchContent_MakeAvailable(json)
  include_directories(${PROJECT_SOURCE_DIR}/third_party/json/include)

endif()

if(ENABLE_GLOG)
    include_directories(${PROJECT_SOURCE_DIR}/third_party/glog/src)
    set(BUILD_TESTING OFF)
    add_subdirectory(${PROJECT_SOURCE_DIR}/third_party/glog glog)
    include_directories(${glog_BINARY_DIR})

endif()

if(ENABLE_PORTAUDIO)
  include(FetchContent)

  set(portaudio_URL  "http://files.portaudio.com/archives/pa_stable_v190700_20210406.tgz")
  set(portaudio_URL2 "https://isv-data.oss-cn-hangzhou.aliyuncs.com/ics/MaaS/ASR/dep_libs/pa_stable_v190700_20210406.tgz")
  set(portaudio_HASH "SHA256=47efbf42c77c19a05d22e627d42873e991ec0c1357219c0d74ce6a2948cb2def")

  FetchContent_Declare(portaudio
    URL
      ${portaudio_URL}
      ${portaudio_URL2}
    URL_HASH          ${portaudio_HASH}
    SOURCE_DIR ${PROJECT_SOURCE_DIR}/third_party/portaudio
  )

  FetchContent_MakeAvailable(portaudio)
  include_directories(${PROJECT_SOURCE_DIR}/third_party/portaudio/include)

endif()

if(ENABLE_FST)
    # fst depend on glog and gflags
    include_directories(${PROJECT_SOURCE_DIR}/../onnxruntime/third_party/gflags)
    add_subdirectory(${PROJECT_SOURCE_DIR}/../onnxruntime/third_party/gflags gflags)
    include_directories(${gflags_BINARY_DIR}/include)
      
    # the following openfst if cloned from https://github.com/kkm000/openfst.git
    # with some patch to fix the make errors. 
    add_subdirectory(${PROJECT_SOURCE_DIR}/../onnxruntime/third_party/openfst openfst)
    include_directories(${openfst_SOURCE_DIR}/src/include)
    if(WIN32)
    include_directories(${openfst_SOURCE_DIR}/src/lib)
    endif() 
endif()

include_directories(${CMAKE_SOURCE_DIR}/include)

# install openssl first apt-get install libssl-dev
# find_package(OpenSSL REQUIRED)
# if(APPLE)
#     set(OPENSSL_ROOT_DIR $ENV{OPENSSL_ROOT_DIR})
#     include_directories(${OPENSSL_INCLUDE_DIR})
#     link_directories(${OPENSSL_ROOT_DIR}/lib)
# endif()

if(CMAKE_SYSTEM_PROCESSOR MATCHES "arm64")
    add_compile_options(-arch arm64)
endif()



set(ONNXRUNTIME_DIR /Users/zxh/code-repo/silero-vad/onnxruntime-osx-x86_64-1.12.1)
set(FFMPEG_DIR /usr/local/ffmpeg)
include_directories(/Users/zxh/code-repo/silero-vad/onnxruntime-osx-x86_64-1.12.1/include)
link_directories(/Users/zxh/code-repo/silero-vad/onnxruntime-osx-x86_64-1.12.1/lib/)
link_directories(${FFMPEG_DIR}/lib/)
find_library(ONNXRUNTIME_LIBRARY onnxruntime PATHS ${ONNXRUNTIME_DIR})


# include_directories(${PROJECT_SOURCE_DIR}/third_party/gflags)
# add_subdirectory(third_party/gflags)
# include_directories(${gflags_BINARY_DIR}/include)

include_directories(${PROJECT_SOURCE_DIR}/../onnxruntime/include/)
include_directories(${PROJECT_SOURCE_DIR}/../onnxruntime/src)
include_directories(${PROJECT_SOURCE_DIR}/../onnxruntime/third_party/yaml-cpp/include/)
include_directories(${PROJECT_SOURCE_DIR}/../onnxruntime/third_party/kaldi-native-fbank)
include_directories(${PROJECT_SOURCE_DIR}/../onnxruntime/third_party/jieba/include)
include_directories(${PROJECT_SOURCE_DIR}/../onnxruntime/third_party/jieba/include/limonp/include)
include_directories(${PROJECT_SOURCE_DIR}/../onnxruntime/third_party)
include_directories(${PROJECT_SOURCE_DIR}/../onnxruntime/third_party/kaldi)
include_directories(${PROJECT_SOURCE_DIR}/../onnxruntime/third_party/openfst/src/include)

add_subdirectory(${PROJECT_SOURCE_DIR}/../onnxruntime/third_party/yaml-cpp yaml-cpp)
add_subdirectory(${PROJECT_SOURCE_DIR}/../onnxruntime/third_party/kaldi-native-fbank/kaldi-native-fbank/csrc csrc)
add_subdirectory(${PROJECT_SOURCE_DIR}/../onnxruntime/src src)
add_subdirectory(${PROJECT_SOURCE_DIR}/../onnxruntime/third_party/kaldi kaldi)



file(GLOB FUNASR_SOURCES
    ${PROJECT_SOURCE_DIR}/../onnxruntime/src/*.cpp
)

# add_executable(funasr-wss-client "funasr-wss-client.cpp" "audio.cpp" "resample.cpp")
# add_executable(funasr-wss-client-2pass "funasr-wss-client-2pass.cpp" "audio.cpp" "resample.cpp" "microphone.cpp")
add_executable(funasr-wss-client "funasr-wss-client.cpp" "../onnxruntime/src/audio.cpp" "../onnxruntime/src/resample.cpp")
add_executable(funasr-wss-client-2pass "funasr-wss-client-2pass.cpp" "../onnxruntime/src/audio.cpp" "../onnxruntime/src/resample.cpp" "microphone.cpp")
# add_executable(funasr-wss-client "funasr-wss-client.cpp"  ${FUNASR_SOURCES})
# add_executable(funasr-wss-client-2pass "funasr-wss-client-2pass.cpp"  "microphone.cpp" ${FUNASR_SOURCES})
# 查找 OpenSSL 库
find_package(OpenSSL REQUIRED)
# 设置 macOS 和 Linux 平台的库路径及链接
if(APPLE)
    # OpenSSL 通过 Homebrew 安装，需要指定路径
    set(OPENSSL_ROOT_DIR /usr/local/opt/openssl@1.1)
    set(OPENSSL_INCLUDE_DIR ${OPENSSL_ROOT_DIR}/include)
    set(OPENSSL_LIBRARIES ${OPENSSL_ROOT_DIR}/lib)

    include_directories(${OPENSSL_INCLUDE_DIR})
    link_directories(${OPENSSL_LIBRARIES})

    # PortAudio 在 macOS 上的路径（假设通过 Homebrew 安装）
    # find_package(PkgConfig REQUIRED)
    # pkg_search_module(PORTAUDIO REQUIRED portaudio-2.0)
    
    # if (PORTAUDIO_FOUND)
    #     include_directories(${PORTAUDIO_INCLUDE_DIRS})
    #     link_directories(${PORTAUDIO_LIBRARY_DIRS})
    # else()
    #     message(FATAL_ERROR "PortAudio library not found!")
    # endif()
    target_link_libraries(funasr-wss-client PUBLIC funasr OpenSSL::SSL OpenSSL::Crypto pthread glog ${ONNXRUNTIME_LIBRARY})
    target_link_libraries(funasr-wss-client-2pass PUBLIC funasr OpenSSL::SSL OpenSSL::Crypto pthread ${PORTAUDIO_LIBRARIES} glog ${ONNXRUNTIME_LIBRARY})
else()
    # 如果在 Linux 上
    target_link_libraries(funasr-wss-client PUBLIC OpenSSL::SSL OpenSSL::Crypto pthread)
    target_link_libraries(funasr-wss-client-2pass PUBLIC OpenSSL::SSL OpenSSL::Crypto pthread portaudio)
endif()

