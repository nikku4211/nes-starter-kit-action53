find_package(Python3 REQUIRED)

set(CONVERTER ${CMAKE_CURRENT_SOURCE_DIR}/make_chr.py)

set(BIN2H ${CMAKE_CURRENT_SOURCE_DIR}/bin2h.py)

# Create commands to convert PNGs to CHRs
set(CHR_FILES "")
file(GLOB PNG_FILES ${CMAKE_CURRENT_SOURCE_DIR}/*.png)
foreach(PNG_FILE ${PNG_FILES})
    get_filename_component(PNG_NAME ${PNG_FILE} NAME_WE)
    set(CHR_FILE ${CMAKE_CURRENT_BINARY_DIR}/${PNG_NAME}.chr)

    add_custom_command(
        OUTPUT ${CHR_FILE}
        COMMAND ${Python3_EXECUTABLE} ${CONVERTER} ${PNG_FILE} ${CHR_FILE}
        DEPENDS ${CONVERTER} ${PNG_FILE}
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
        COMMENT "Converting ${PNG_NAME}.png to ${PNG_NAME}.chr"
    )

    list(APPEND CHR_FILES ${CHR_FILE})
endforeach()

add_custom_target(convert_chrs DEPENDS ${CHR_FILES})

# Create commands to convert CHRs to Hs
set(H_FILES "")
foreach(CHR_FILE ${CHR_FILES})
    get_filename_component(CHR_NAME ${CHR_FILE} NAME_WE)
    set(H_FILE ${CMAKE_CURRENT_SOURCE_DIR}/${CHR_NAME}.h)

    add_custom_command(
        OUTPUT ${H_FILE}
        COMMAND ${Python3_EXECUTABLE} ${BIN2H} -o ${H_FILE} ${CHR_FILE}
        # Ensure that `pngchr.s` is reassembled
        COMMAND ${CMAKE_COMMAND} -E touch ${CHR_ASM}
        DEPENDS ${BIN2H} ${CHR_FILE}
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
        COMMENT "Converting ${CHR_NAME}.chr to ${CHR_NAME}.h"
    )

    list(APPEND H_FILES ${H_FILE})
endforeach()

add_custom_target(convert_hs DEPENDS ${H_FILES})