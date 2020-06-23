cmake_minimum_required(VERSION 3.13)

if (NOT DEFINED VERSION)
    set(VERSION 1.2.2)
endif ()

configure_file(environments.h.in environments.h @ONLY)
set (EXE_NAME deepin-deb-installer)
set(LINK_LIBS
    Qt5::Core
    Qt5::Widgets
    Qt5::Gui
    Qt5::Xml
    Qt5::DBus
    QApt
    Qt5::Concurrent
    ${DtkWidget_LIBRARIES}
    ${DtkCore_LIBRARIES}
    ${DtkGUI_LIBRARIES}
)


set(CMAKE_CXX_STANDARD 11)
set(CMAKE_VERBOSE_MAKEFILE ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -ldtkwidget -ldtkgui -ldtkcore")#${EXE_NAME}

if (${CMAKE_SYSTEM_PROCESSOR} MATCHES "sw_64")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mieee")
endif ()

if (NOT (${CMAKE_BUILD_TYPE} MATCHES "Debug"))
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3")
    # generate qm files
    execute_process(COMMAND bash "assets/translate_generation.sh"
                    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} RESULT_VARIABLE trans_var OUTPUT_VARIABLE trans_out ERROR_VARIABLE trans_error)
                message("trans_var ${trans_var} ")#trans_out ${trans_out} trans_error ${trans_error}
    execute_process(COMMAND deepin-policy-ts-convert "policy2ts" "assets/com.deepin.pkexec.aptInstallDepend.policy.tmp" "../translations/policy"
                    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} RESULT_VARIABLE policy2ts_var OUTPUT_VARIABLE policy2ts_out ERROR_VARIABLE policy2ts_error)
                 message("policy2ts_var ${policy2ts_var} policy2ts_out ${policy2ts_out} policy2ts_error ${policy2ts_error}")
    execute_process(COMMAND deepin-policy-ts-convert "ts2policy" "assets/com.deepin.pkexec.aptInstallDepend.policy.tmp" "policy-install-translation" "assets/com.deepin.pkexec.aptInstallDepend.policy"
                    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} RESULT_VARIABLE ts2policy_var OUTPUT_VARIABLE ts2policy_out ERROR_VARIABLE ts2policy_error)
                 message("ts2policy_var ${ts2policy_var} ts2policy_out ${ts2policy_out} ts2policy_error ${ts2policy_error}")
    execute_process(COMMAND cp "assets/com.deepin.pkexec.aptInstallDepend.policy.tmp" "assets/com.deepin.pkexec.aptInstallDepend.policy"
                    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} RESULT_VARIABLE cp_var OUTPUT_VARIABLE cp_out ERROR_VARIABLE cp_error)
                   message("cp_var ${cp_var} cp_out ${cp_out} cp_error ${cp_error}")
endif ()

add_definitions(-DUSE_POLKIT -DENABLE_INACTIVE_DISPLAY)
# Find the library
find_package(PkgConfig REQUIRED)
find_package(Qt5Widgets REQUIRED)
find_package(Qt5Gui REQUIRED)
find_package(Qt5Core REQUIRED)
find_package(Qt5Xml REQUIRED)
find_package(Qt5Concurrent REQUIRED)
find_package(DtkWidget REQUIRED)
find_package(DtkGui REQUIRED)
find_package(DtkCore REQUIRED)
find_package(Qt5DBus REQUIRED)


set (APP_CPP_FILES
    main.cpp
     choosefilebutton.cpp
     debinfolabel.cpp
     debinstaller.cpp
     deblistmodel.cpp
     droundbgframe.cpp
     filechoosewidget.cpp
     infocontrolbutton.cpp
     installprocessinfoview.cpp
     multipleinstallpage.cpp
     packagelistview.cpp
     packageslistdelegate.cpp
     packagesmanager.cpp
     singleInstallerApplication.cpp
     singleinstallpage.cpp
     uninstallconfirmpage.cpp
     utils.cpp
     workerprogress.cpp
    )
set (APP_QRC_FILES
assets/resources.qrc
)

set (APP_HEADERS_FILES
    choosefilebutton.h
    debinfolabel.h
    debinstaller.h
    deblistmodel.h
    droundbgframe.h
    filechoosewidget.h
    infocontrolbutton.h
    installprocessinfoview.h
    multipleinstallpage.h
    packagelistview.h
    packageslistdelegate.h
    packagesmanager.h
    result.h
    singleInstallerApplication.h
    singleinstallpage.h
    uninstallconfirmpage.h
    utils.h
    workerprogress.h
    )

# 5. 头文件
# 5.1. 定义函数，用于递归添加头文件
function(include_sub_directories_recursively root_dir)
    if (IS_DIRECTORY ${root_dir})               # 当前路径是一个目录吗，是的话就加入到包含目录
        #message("include dir: " ${root_dir})
        include_directories(${root_dir})
    endif()

    file(GLOB ALL_SUB RELATIVE ${root_dir} ${root_dir}/*) # 获得当前目录下的所有文件，让如ALL_SUB列表中
    foreach(sub ${ALL_SUB})
        if (IS_DIRECTORY ${root_dir}/${sub})
            include_sub_directories_recursively(${root_dir}/${sub}) # 对子目录递归调用，包含
        endif()
    endforeach()
endfunction()

find_library(ICUIL18N_LIB icui18n)
find_library(ICUUC_LIB icuuc)

add_executable (${EXE_NAME} ${APP_CPP_FILES} ${APP_QRC_FILES} ${DOCX_CPP_FILES} ${XLSX_CPP_FILES} ${TEST_ACCESSIBLE_HEADERS_FILES})
include_directories(${CMAKE_CURRENT_SOURCE_DIR})
include_directories(${DtkWidget_INCLUDE_DIRS})
include_directories(${DtkCore_INCLUDE_DIRS})
include_directories(${DtkGui_INCLUDE_DIRS})
include_directories( ${Qt5Gui_PRIVATE_INCLUDE_DIRS})



include_directories(${Qt5Widgets_LIBRARIES}
                                               ${Qt5Core_LIBRARIES}
                                              ${Qt5DBus_LIBRARIES}
                                              ${PROJECT_BINARY_DIR}
                                              ${DtkWidget_INCLUDE_DIRS}
                                              ${DtkCore_LIBRARIES}
                                              ${DtkGUI_INCLUDE_DIRS}
                                              ${Qt5Gui_PRIVATE_INCLUDE_DIRS}
                                              ${Qt5Xml_LIBRARIES})
target_include_directories(${EXE_NAME} PUBLIC ${Qt5Widgets_LIBRARIES}
                                               ${Qt5Core_LIBRARIES}
                                              ${Qt5DBus_LIBRARIES}
                                              ${PROJECT_BINARY_DIR}
                                              ${Qt5Gui_PRIVATE_INCLUDE_DIRS}
                                              ${Qt5Xml_LIBRARIES})
install(FILES ${ICUIL18N_LIB} ${ICUUC_LIB} ${SYSTEMD_LIB} DESTINATION lib)
set(CMAKE_INSTALL_PREFIX /usr)
target_link_libraries(${EXE_NAME}  ${LINK_LIBS} -lsystemd -licui18n -licuuc)

file(GLOB APP_QM_FILES "../translations/*.qm")
install(FILES ${APP_QM_FILES} DESTINATION ${CMAKE_INSTALL_PREFIX}/share/deepin-deb-installer/translations)
install(FILES deepin-deb-installer.desktop DESTINATION ${CMAKE_INSTALL_PREFIX}/share/applications)
install(FILES assets/com.deepin.pkexec.aptInstallDepend.policy DESTINATION ${CMAKE_INSTALL_PREFIX}/share/polkit-1/actions)
install(FILES assets/images/deepin-deb-installer.svg
            DESTINATION ${CMAKE_INSTALL_PREFIX}/share/icons/hicolor/scalable/apps/)
install(FILES assets/com.deepin.pkexec.aptInstallDepend.policy
                    DESTINATION ${CMAKE_INSTALL_PREFIX}/share/polkit-1/actions)
                # Install files
                install(TARGETS ${EXE_NAME} DESTINATION bin)





