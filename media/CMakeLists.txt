cmake_minimum_required(VERSION 3.0)

include(GNUInstallDirs)

add_subdirectory(baseset)

set(MENU_NAME "OpenTTD" CACHE STRING "Name of the menu item when placed")
set(MENU_GROUP "Game" CACHE STRING "Category in which the menu item will be placed (Freedesktop based only)")

if (OPTION_INSTALL_FHS)
    if (UNIX AND NOT APPLE AND NOT WIN32)
        # Perform replacements on the openttd.desktop.in (configure step)
        # TODO replace hard-coded "openttd" with "@BINARY_NAME@" once available
        configure_file(openttd.desktop.in openttd.desktop.configured @ONLY)

        find_program(AWK awk mawk gawk)
        find_program(SORT sort)
        find_program(CAT cat)

        if(SORT MATCHES ".+-NOTFOUND")
            message(FATAL_ERROR "FATAL: sort could not be found (${SORT}).")
        endif()
        if(AWK MATCHES ".+-NOTFOUND")
            message(FATAL_ERROR "FATAL: awk (and mawk and gawk) could not be found (${AWK}).")
        endif()
        if(CAT MATCHES ".+-NOTFOUND")
            message(FATAL_ERROR "FATAL: cat could not be found (${CAT}).")
        endif()

        # Add "comment[{lang}]" lines to .desktop file using translations (build step)
        add_custom_target(
            openttd.desktop.comments
            COMMAND
                ${AWK} -f openttd.desktop.translation.awk ${CMAKE_SOURCE_DIR}/src/lang/*.txt |
                ${SORT} |
                ${AWK} -f openttd.desktop.filter.awk > ${CMAKE_CURRENT_BINARY_DIR}/openttd.desktop.comments
            DEPENDS openttd.desktop.configured
            WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        )

        add_custom_target(
            openttd.desktop
            COMMAND ${CAT} openttd.desktop.configured openttd.desktop.comments > openttd.desktop
            DEPENDS openttd.desktop.configured openttd.desktop.comments
        )

        install(FILES "${CMAKE_CURRENT_BINARY_DIR}/openttd.desktop"
            DESTINATION "${CMAKE_INSTALL_DATAROOTDIR}/applications"
            COMPONENT media
        )
    endif (UNIX AND NOT APPLE AND NOT WIN32)
endif (OPTION_INSTALL_FHS)
