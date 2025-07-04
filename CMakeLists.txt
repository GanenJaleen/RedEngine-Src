cmake_minimum_required(VERSION 3.16)


project(RedEngine-Src LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)


set(SUBPROJECT_DIRS
    boost
    Dependencies/nlohmann-json
    Dependencies/websocketpp
    FiveM/citizen-resources-core
    FiveM/profiles
    FiveM/scripting-server
    FiveM/vfs-core
    GUI/imgui
    GUI/kiero
)


set(GLOBAL_INCLUDE_DIRS "")
set(GLOBAL_SOURCE_FILES "")
set(GLOBAL_TARGETS "")


function(process_project_dir dir)
    get_filename_component(proj_name "${dir}" NAME)
    set(full_path "${CMAKE_CURRENT_SOURCE_DIR}/${dir}")

    file(GLOB_RECURSE SRC_FILES CONFIGURE_DEPENDS
        "${full_path}/src/*.cpp"
        "${full_path}/src/*.c"
        "${full_path}/*.cpp"
        "${full_path}/*.c"
    )


    set(INCLUDE_DIRS "")
    foreach(subdir src include vendor)
        if(EXISTS "${full_path}/${subdir}")
            list(APPEND INCLUDE_DIRS "${full_path}/${subdir}")
        endif()
    endforeach()


    if(EXISTS "${full_path}/CMakeLists.txt")
        add_subdirectory("${full_path}" "${proj_name}_build")
        list(APPEND GLOBAL_TARGETS ${proj_name})
    else()
 
        add_library(${proj_name} STATIC ${SRC_FILES})
        target_include_directories(${proj_name} PUBLIC ${INCLUDE_DIRS})
        list(APPEND GLOBAL_TARGETS ${proj_name})
    endif()


    list(APPEND GLOBAL_INCLUDE_DIRS ${INCLUDE_DIRS})
    list(APPEND GLOBAL_SOURCE_FILES ${SRC_FILES})


    set(GLOBAL_INCLUDE_DIRS ${GLOBAL_INCLUDE_DIRS} PARENT_SCOPE)
    set(GLOBAL_SOURCE_FILES ${GLOBAL_SOURCE_FILES} PARENT_SCOPE)
    set(GLOBAL_TARGETS ${GLOBAL_TARGETS} PARENT_SCOPE)
endfunction()


foreach(subdir IN LISTS SUBPROJECT_DIRS)
    process_project_dir("${subdir}")
endforeach()


if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/redENGINE-INJECTOR.vcxproj")
    message(STATUS "Including redENGINE-INJECTOR.vcxproj...")
    include(ExternalProject)
    include_external_msproject(
        redENGINE_INJECTOR
        "${CMAKE_CURRENT_SOURCE_DIR}/redENGINE-INJECTOR.vcxproj"
        TYPE VCXProj
    )
endif()


set(DUMMY_MAIN "${CMAKE_CURRENT_SOURCE_DIR}/main_dummy.cpp")
if(NOT EXISTS ${DUMMY_MAIN})
    file(WRITE ${DUMMY_MAIN} "int main() { return 0; }\n")
endif()


add_executable(MainApp ${DUMMY_MAIN})

target_include_directories(MainApp PUBLIC ${GLOBAL_INCLUDE_DIRS})


foreach(target_name IN LISTS GLOBAL_TARGETS)
    if(TARGET ${target_name})
        target_link_libraries(MainApp PRIVATE ${target_name})
    endif()
endforeach()
