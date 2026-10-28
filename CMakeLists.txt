cmake_minimum_required(VERSION 3.20)
project(ImGuiFileDialog)

# Option to control installation (disabled by default if used as subproject)
if(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
    set(IGFD_INSTALL_DEFAULT ON)
else()
    set(IGFD_INSTALL_DEFAULT OFF)
endif()

option(IGFD_INSTALL "Install ImGuiFileDialog library and headers" ${IGFD_INSTALL_DEFAULT})
# --- stb の取得 ---
FetchContent_Declare(
        imgui
        GIT_REPOSITORY https://github.com/ocornut/imgui.git
        GIT_TAG        v1.93.0
)
FetchContent_MakeAvailable(imgui)

# -------------------------

# Dear ImGui 用のライブラリターゲットを作成 (コア + Win32 + DirectX 11 バックエンド)
add_library(imgui STATIC
        ${imgui_SOURCE_DIR}/imgui.cpp
        ${imgui_SOURCE_DIR}/imgui_demo.cpp
        ${imgui_SOURCE_DIR}/imgui_draw.cpp
        ${imgui_SOURCE_DIR}/imgui_tables.cpp
        ${imgui_SOURCE_DIR}/imgui_widgets.cpp
        ${imgui_SOURCE_DIR}/backends/imgui_impl_win32.cpp
        ${imgui_SOURCE_DIR}/backends/imgui_impl_dx11.cpp
)

# インクルードパスの設定 (本体とバインド用の両方を通す)
target_include_directories(imgui PUBLIC
        ${imgui_SOURCE_DIR}
        ${imgui_SOURCE_DIR}/backends
)

add_library(ImGuiFileDialog
    ImGuiFileDialog.cpp
    ImGuiFileDialog.h
    ImGuiFileDialogConfig.h
)

# Use generator expressions for proper include directory handling
target_include_directories(ImGuiFileDialog
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
        $<INSTALL_INTERFACE:include>
)

if(UNIX)
    target_compile_options(ImGuiFileDialog PUBLIC -Wno-unknown-pragmas)
endif()

# Installation configuration
if(IGFD_INSTALL)
    include(GNUInstallDirs)

    # Install the library
    install(TARGETS ImGuiFileDialog
       EXPORT ImGuiFileDialogTargets
       ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
       LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
       RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    )

    # Install headers
    install(FILES 
       ImGuiFileDialog.h 
       ImGuiFileDialogConfig.h
       DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    )

    # Install export targets
    install(EXPORT ImGuiFileDialogTargets
       FILE ImGuiFileDialogConfig.cmake
       NAMESPACE ImGuiFileDialog::
       DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/ImGuiFileDialog
    )
endif()
