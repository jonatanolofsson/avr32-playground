include_directories(${CMAKE_CURRENT_SOURCE_DIR})
include_directories(${CMAKE_SOURCE_DIR}/config)

set(ELFFILE wii.elf)
set(HEXFILE wii.hex)

add_executable(${ELFFILE}
    ../../Framebuffer/Framebuffer.cpp
    ../../Framebuffer/FramebufferConsole.cpp
    ../../Framebuffer/SSD1306.cpp
    ../../Framebuffer/Scope.cpp
    ../../Stream/formatting.cpp
    ../../Stream/BaseOutStream.cpp
    ../../Stream/USARTStream.cpp
    ../../system/system_init.c
    ../../system/tick.c
    ../../system/console.cpp
    ../../system/framebuffer.cpp
    ../../system/streams.cpp
    ../../sensors/Wii.cpp
    ../../sensors/IMU.cpp
    ${AVR32_USER_BOARD_DIR}/system_init.cpp
    wii.cpp
    ${AVR32_USER_BOARD_DIR}/init.c
    ${ASF_SOURCE_FILES}
    ${AVR32_TRAMPOLINE}
    ${ASF_PATH}/avr32/drivers/intc/exception.S
    #~ /usr/avr32-elf/lib/crt0.o
    #~ /usr/lib/gcc/avr32-elf/4.4.7/crti.o /usr/lib/gcc/avr32-elf/4.4.7/crtbegin.o
    #~ /usr/lib/gcc/avr32-elf/4.4.7/crtend.o /usr/lib/gcc/avr32-elf/4.4.7/crtn.o
)


add_custom_command(OUTPUT ${HEXFILE}
    COMMAND ${SIZE} ${ELFFILE}
    COMMAND ${OBJCOPY} -O ihex ${ELFFILE} ${HEXFILE}
    DEPENDS ${ELFFILE}
    COMMENT "Creating .hex file")
add_custom_target(hex ALL DEPENDS ${HEXFILE})

add_custom_target(flash DEPENDS ${HEXFILE} erase)
add_custom_target(erase)
add_custom_target(start)
add_custom_command(TARGET erase POST_BUILD
    COMMAND ${DFUPROGRAM} ${DFUPART} erase
    )

add_custom_command(TARGET start POST_BUILD
    COMMAND ${DFUPROGRAM} ${DFUPART} start
    )

add_custom_command(TARGET flash
    POST_BUILD
    COMMAND ${DFUPROGRAM} ${DFUPART} flash --suppress-bootloader-mem ${HEXFILE}
    )
