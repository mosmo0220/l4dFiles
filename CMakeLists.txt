cmake_minimum_required(VERSION 3.10)
project(L4DCombine LANGUAGES CXX)

# 1) Twoja biblioteka lub program
set(SOURCE_FILES
    l4dConfig.cpp
    l4dStorage.cpp
)
add_library(L4DStatic STATIC ${SOURCE_FILES})

# 2) Lista plików do połączenia
set(FILES_TO_COMBINE
    json.hpp
    l4dConfig.h
    l4dConfig.cpp
    l4dStorage.h
    l4dStorage.cpp
)

# 3) Ścieżki
set(OUTPUT_HEADER ${CMAKE_BINARY_DIR}/l4dFiles.hpp)
set(CONCAT_SCRIPT ${CMAKE_BINARY_DIR}/concat_l4dFiles.cmake)

# 4) Na etapie configure: generujemy mały skrypt CMake, który
#    w build-time poskłada wszystkie pliki w jeden nagłówek.
file(WRITE ${CONCAT_SCRIPT}
"#
# Ten skrypt wygenerował: ${OUTPUT_HEADER}
#
file(WRITE \"${OUTPUT_HEADER}\" \"// === l4dFiles.hpp ===\\n\")
file(APPEND \"${OUTPUT_HEADER}\" \"// Zawiera: ${FILES_TO_COMBINE}\\n\\n\")
")

foreach(F ${FILES_TO_COMBINE})
    file(APPEND ${CONCAT_SCRIPT}
"file(APPEND \"${OUTPUT_HEADER}\" \"// ==== ${F} ====\\n\")
file(READ \"${CMAKE_SOURCE_DIR}/${F}\" CONTENT)
file(APPEND \"${OUTPUT_HEADER}\" \"\${CONTENT}\\n\\n\")
")
endforeach()

# 5) Target, który uruchomi ten skrypt w build-time
add_custom_target(combine ALL
    COMMAND ${CMAKE_COMMAND} -P ${CONCAT_SCRIPT}
    DEPENDS ${FILES_TO_COMBINE}
    COMMENT "Generowanie pliku l4dFiles.hpp"
)

# 6) Upewnij się, że biblioteka buduje się przed combine (opcjonalnie)
add_dependencies(combine L4DStatic)
