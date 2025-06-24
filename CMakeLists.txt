cmake_minimum_required(VERSION 3.10)
project(L4DHeaderOnly LANGUAGES CXX)

set(FILES_TO_COMBINE
  l4dConfig.h
  l4dConfig.cpp
  l4dStorage.h
  l4dStorage.cpp
)

set(OUTPUT_DIR    ${CMAKE_BINARY_DIR})
set(OUTPUT_HEADER ${OUTPUT_DIR}/l4dFiles.hpp)

# 1) skopiuj json.hpp do build/
file(COPY ${CMAKE_SOURCE_DIR}/json.hpp DESTINATION ${OUTPUT_DIR})

# 2) rozpocznij generację nagłówka
file(WRITE ${OUTPUT_HEADER} "#pragma once\n\n")

# 3) zbierz wszystkie systemowe include <...>
set(_angle_includes "")
foreach(F ${FILES_TO_COMBINE})
  file(READ "${CMAKE_SOURCE_DIR}/${F}" _raw)
  string(REGEX MATCHALL "^#include[ \t]*<[^>]+>" _incs "${_raw}")
  foreach(_i IN LISTS _incs)
    list(APPEND _angle_includes "${_i}")
  endforeach()
endforeach()
list(REMOVE_DUPLICATES _angle_includes)
foreach(_i IN LISTS _angle_includes)
  file(APPEND ${OUTPUT_HEADER} "${_i}\n")
endforeach()
file(APPEND ${OUTPUT_HEADER} "\n")

# 4) dopisz zawartość plików bez lokalnych include "..."
foreach(F ${FILES_TO_COMBINE})
  file(APPEND ${OUTPUT_HEADER} "// ==== ${F} ====\n")
  file(READ "${CMAKE_SOURCE_DIR}/${F}" _raw)
  string(REGEX REPLACE "^#include[ \t]*\"[^\"]+\"[ \t]*\n" "" _content "${_raw}")
  file(APPEND ${OUTPUT_HEADER} "${_content}\n")
endforeach()
