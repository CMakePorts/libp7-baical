include_directories(${P7_Headers})

set(SOURCE
   C_Example.c
)

add_executable(p7-c-example ${SOURCE})
target_link_libraries(p7-c-example P7)
