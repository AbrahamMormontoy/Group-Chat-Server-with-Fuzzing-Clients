cmake_minimum_required(VERSION 3.10)

include_directories(include)

add_executable(server src/server.c)
add_executable(client src/client.c)

find_package(Threads REQUIRED)
target_link_libraries(server Threads::Threads)
target_link_libraries(client Threads::Threads)
