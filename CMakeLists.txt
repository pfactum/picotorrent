cmake_minimum_required(VERSION 3.14 FATAL_ERROR)
cmake_policy(SET CMP0092 NEW) # don't add /W3 as default

project("PicoTorrent")

include(cmake/Git.cmake)
include(cmake/GitVersion.cmake)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_INCLUDE_CURRENT_DIR ON)

set(Boost_USE_STATIC_LIBS ON)
set(OPENSSL_USE_STATIC_LIBS TRUE)

if(DEFINED ENV{OPENSSL_ROOT_DIR})
    set(OPENSSL_ROOT_DIR $ENV{OPENSSL_ROOT_DIR})
    message("Using OpenSSL dir from env variable: ${OPENSSL_ROOT_DIR}")
endif()

# --------- antlr4 options
option(WITH_STATIC_CRT "" Off)
# -----------------------

# --------- libtorrent options
option(BUILD_SHARED_LIBS    "" off)
option(deprecated-functions "" off)

# uncomment here when webtorrent support is usable
# option(webtorrent           "" ON)

# deep inside libtorrent we need to turn off a nlohmann-json dependency
option(NO_EXAMPLES          "" ON)

# ----------------------------

# --------- nlohmann-json options
set(JSON_BuildTests OFF CACHE INTERNAL "")
# ---------

# --------- wx options
set(wxUSE_EXCEPTIONS OFF CACHE BOOL "" FORCE)
# --------------------

find_package(Boost REQUIRED COMPONENTS log)

# vendor libs
add_subdirectory(vendor/fmt)
add_subdirectory(vendor/libtorrent)
add_subdirectory(vendor/nlohmann-json)
add_subdirectory(vendor/sentry-crashpad)
add_subdirectory(vendor/wx)

# pico components
add_subdirectory(src/pql)

gitversion_install()
gitversion_showvariable(BranchName      GITVERSION_VAR_BRANCHNAME)
gitversion_showvariable(MajorMinorPatch GITVERSION_VAR_VERSION)
gitversion_showvariable(Major           GITVERSION_VAR_VERSION_MAJOR)
gitversion_showvariable(Minor           GITVERSION_VAR_VERSION_MINOR)
gitversion_showvariable(Patch           GITVERSION_VAR_VERSION_PATCH)
gitversion_showvariable(SemVer          GITVERSION_VAR_SEMVER)
gitversion_showvariable(ShortSha        GITVERSION_VAR_SHORTSHA)

# Get git commitish from dependencies
git_abbreviated_hash(${CMAKE_SOURCE_DIR}/vendor/fmt           PICO_FMT_GIT_COMMITISH)
git_abbreviated_hash(${CMAKE_SOURCE_DIR}/vendor/libtorrent    PICO_LIBTORRENT_GIT_COMMITISH)
git_abbreviated_hash(${CMAKE_SOURCE_DIR}/vendor/nlohmann-json PICO_NLOHMANN_JSON_GIT_COMMITISH)
git_abbreviated_hash(${CMAKE_SOURCE_DIR}/vendor/wx            PICO_WX_GIT_COMMITISH)

# Generate buildinfo file
configure_file("${CMAKE_SOURCE_DIR}/src/picotorrent/buildinfo.cpp.in" "${CMAKE_SOURCE_DIR}/src/picotorrent/buildinfo.cpp" @ONLY)

# Debug flags
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /Zi")

# Release flags
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /Zi")
set(CMAKE_EXE_LINKER_FLAGS_RELEASE "/DEBUG /INCREMENTAL:NO /MAP /OPT:REF /OPT:ICF")
set(CMAKE_SHARED_LINKER_FLAGS_RELEASE "/DEBUG /INCREMENTAL:NO /MAP /OPT:REF /OPT:ICF")

add_library(
    sqlite
    STATIC
    src/sqlite/sqlite3
)

target_include_directories(
    sqlite
    INTERFACE
    src/sqlite
)

add_executable(
    PicoTorrent-coredb
    src/l10n/main
    lang/ca-ES.json
    lang/de-DE.json
    lang/el-GR.json
    lang/en-US.json
    lang/es-ES.json
    lang/fi-FI.json
    lang/fr-FR.json
    lang/hu-HU.json
    lang/hy-AM.json
    lang/id-ID.json
    lang/it-IT.json
    lang/ka-GE.json
    lang/ko-KR.json
    lang/lt-LT.json
    lang/lv-LV.json
    lang/nl-NL.json
    lang/pl-PL.json
    lang/pt-BR.json
    lang/ro-RO.json
    lang/ru-RU.json
    lang/sv-SE.json
    lang/tr-TR.json
    lang/uk-UA.json
    lang/zh-CN.json
)

target_link_libraries(
    PicoTorrent-coredb
    PRIVATE
    nlohmann_json::nlohmann_json
    sqlite
)

add_executable(
    PicoTorrent
    WIN32

    src/picotorrent/application
    src/picotorrent/buildinfo
    src/picotorrent/crashpadinitializer
    src/picotorrent/main
    src/picotorrent/persistencemanager
    src/picotorrent/resources.rc

    # API
    src/picotorrent/api/libpico

    # BitTorrent
    src/picotorrent/bittorrent/session
    src/picotorrent/bittorrent/torrenthandle

    # Core
    src/picotorrent/core/configuration
    src/picotorrent/core/database
    src/picotorrent/core/environment
    src/picotorrent/core/utils

    # Http
    src/picotorrent/http/httpclient

    # IPC
    src/picotorrent/ipc/applicationoptionsconnection
    src/picotorrent/ipc/server

    # Dialogs
    src/picotorrent/ui/dialogs/aboutdialog
    src/picotorrent/ui/dialogs/addmagnetlinkdialog
    src/picotorrent/ui/dialogs/addtrackerdialog
    src/picotorrent/ui/dialogs/addtorrentdialog
    src/picotorrent/ui/dialogs/createtorrentdialog
    src/picotorrent/ui/dialogs/listeninterfacedialog
    src/picotorrent/ui/dialogs/preferencesadvancedpage
    src/picotorrent/ui/dialogs/preferencesconnectionpage
    src/picotorrent/ui/dialogs/preferencesdialog
    src/picotorrent/ui/dialogs/preferencesdownloadspage
    src/picotorrent/ui/dialogs/preferencesgeneralpage
    src/picotorrent/ui/dialogs/preferenceslabelspage
    src/picotorrent/ui/dialogs/preferencesproxypage

    # Filters
    src/picotorrent/ui/filters/pqltorrentfilter

    # Models
    src/picotorrent/ui/models/filestoragemodel
    src/picotorrent/ui/models/peerlistmodel
    src/picotorrent/ui/models/torrentlistmodel
    src/picotorrent/ui/models/trackerlistmodel

    # Persistence
    src/picotorrent/ui/persistence/persistenttorrentlistview

    # UI
    src/picotorrent/ui/console
    src/picotorrent/ui/mainframe
    src/picotorrent/ui/statusbar
    src/picotorrent/ui/taskbaricon
    src/picotorrent/ui/torrentcontextmenu
    src/picotorrent/ui/torrentdetailsfilespanel
    src/picotorrent/ui/torrentdetailsoverviewpanel
    src/picotorrent/ui/torrentdetailspeerspanel
    src/picotorrent/ui/torrentdetailstrackerspanel
    src/picotorrent/ui/torrentdetailsview
    src/picotorrent/ui/torrentfilelistview
    src/picotorrent/ui/torrentlistview
    src/picotorrent/ui/translator

    # Widgets
    src/picotorrent/ui/widgets/pieceprogressbar

    # Win32 specific stuff
    src/picotorrent/ui/win32/openfiledialog
)

target_compile_definitions(
    PicoTorrent
    PRIVATE
    -D_UNICODE
    -D_WIN32
    -D_WIN32_WINNT=0x0600
    -DNOMINMAX
    -DPICOJSON_USE_INT64
    -DUNICODE
    -DWIN32
    -DWIN32_LEAN_AND_MEAN

    -DLIBPICO_API_EXPORT

    -DPICO_GIT_COMMITISH=${GITVERSION_VAR_SHORTSHA}
    -DPICO_VERSION_MAJOR=${GITVERSION_VAR_VERSION_MAJOR}
    -DPICO_VERSION_MINOR=${GITVERSION_VAR_VERSION_MINOR}
    -DPICO_VERSION_PATCH=${GITVERSION_VAR_VERSION_PATCH}

    -DPICO_FMT_GIT_COMMITISH=${PICO_FMT_GIT_COMMITISH}
    -DPICO_LIBTORRENT_GIT_COMMITISH=${PICO_LIBTORRENT_GIT_COMMITISH}
    -DPICO_NLOHMANN_JSON_GIT_COMMITISH=${PICO_NLOHMANN_JSON_GIT_COMMITISH}
    -DPICO_WX_GIT_COMMITISH=${PICO_WX_GIT_COMMITISH}
)

target_compile_options(PicoTorrent PRIVATE /guard:cf /W4 /WX)

target_include_directories(
    PicoTorrent
    PRIVATE

    include
)

target_link_libraries(
    PicoTorrent

    PRIVATE

    ${Boost_LIBRARIES}

    # wxWidgets
    wxcore wxbase wxpropgrid

    # Windows
    Comctl32
    crypt32
    iphlpapi
    legacy_stdio_definitions
    propsys
    shlwapi
    wininet
    winhttp

    # Crashpad
    crashpad_client
    crashpad_util
    mini_chromium

    # fmt
    fmt

    # nlohmann-json
    nlohmann_json::nlohmann_json

    # Rasterbar-libtorrent
    "torrent-rasterbar"

    # PQL
    PicoTorrentPQL

    sqlite
)

set_property(TARGET PicoTorrent PROPERTY ENABLE_EXPORTS 1)

# Plugins
add_library(
    Plugin_Updater
    SHARED
    src/plugins/updater/updater
)

target_compile_definitions(Plugin_Updater PRIVATE -D_WIN32 -DUNICODE -DWIN32_LEAN_AND_MEAN)
target_include_directories(Plugin_Updater PRIVATE include)
target_link_libraries(Plugin_Updater PRIVATE PicoTorrent Comctl32)

# Copy Crashpad handler
add_custom_command(
    TARGET PicoTorrent POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
        $<TARGET_FILE:crashpad_handler>
        $<TARGET_FILE_DIR:PicoTorrent>
)

# Generate coredb
add_custom_command(
    TARGET PicoTorrent POST_BUILD
    COMMAND PicoTorrent-coredb "${CMAKE_SOURCE_DIR}/lang" $<TARGET_FILE_DIR:PicoTorrent>
)
