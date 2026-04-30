cmake_minimum_required(VERSION 3.27)
project(tool-requires-probe CXX)

find_package(ZLIB REQUIRED)
find_package(Poco REQUIRED)

add_executable(probe src/main.cpp)
target_link_libraries(probe PRIVATE ZLIB::ZLIB Poco::Poco)
