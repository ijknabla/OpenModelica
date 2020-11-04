cmake_minimum_required(VERSION 3.14)

find_package(Java REQUIRED Runtime)
# omc_add_to_report(Java_JAVA_EXECUTABLE)
find_package(Intl REQUIRED)

set(OM_PARSE_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/Parser_omc.c)

set(GNERATED_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/generated)
file(MAKE_DIRECTORY ${GNERATED_DIRECTORY})

include(.cmake/omc_antlr_target_macros.cmake)

omc_add_antlr_base_lexer_target(${CMAKE_CURRENT_SOURCE_DIR}/Modelica_3_Lexer.g ${GNERATED_DIRECTORY})
set(OM_PARSE_SOURCES ${OM_PARSE_SOURCES} ${ANTLR_BASE_LEXER_Modelica_3_Lexer_OUTPUT_SOURCES})

omc_add_antlr_base_lexer_target(${CMAKE_CURRENT_SOURCE_DIR}/ParModelica_Lexer.g ${GNERATED_DIRECTORY})
set(OM_PARSE_SOURCES ${OM_PARSE_SOURCES} ${ANTLR_BASE_LEXER_ParModelica_Lexer_OUTPUT_SOURCES})

omc_add_antlr_base_lexer_target(${CMAKE_CURRENT_SOURCE_DIR}/MetaModelica_Lexer.g ${GNERATED_DIRECTORY})
set(OM_PARSE_SOURCES ${OM_PARSE_SOURCES} ${ANTLR_BASE_LEXER_MetaModelica_Lexer_OUTPUT_SOURCES})

omc_add_antlr_grammar_target(${CMAKE_CURRENT_SOURCE_DIR}/Modelica.g ${GNERATED_DIRECTORY})
set(OM_PARSE_SOURCES ${OM_PARSE_SOURCES} ${GNERATED_DIRECTORY}/ModelicaParser.c)


# ######################################################################################################################
# Library: omparse
add_library(omparse STATIC)
add_library(omc::parser ALIAS omparse)

target_sources(omparse PRIVATE ${OM_PARSE_SOURCES})

target_link_libraries(omparse PUBLIC ${Intl_LIBRARIES})
target_link_libraries(omparse PUBLIC omc::compiler::runtime)
target_link_libraries(omparse PUBLIC omc::simrt::runtime)
target_link_libraries(omparse PUBLIC omc::3rd::omantlr3)

# To find the generated antlr headers. SYSTEM to disable warnings on the generated code.
target_include_directories(omparse SYSTEM PRIVATE ${GNERATED_DIRECTORY})
target_include_directories(omparse PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})

target_include_directories(omparse PRIVATE ${OMCompiler_SOURCE_DIR}) # for revision.h :/

# ######################################################################################################################
# Library: omparse-boot
add_library(omparse-boot STATIC)
add_library(omc::parser-boot ALIAS omparse-boot)

target_sources(omparse-boot PRIVATE ${OM_PARSE_SOURCES})

target_link_libraries(omparse-boot PUBLIC Intl)
target_link_libraries(omparse-boot PUBLIC omc::compiler::runtime)
target_link_libraries(omparse-boot PUBLIC omc::simrt::runtime)
target_link_libraries(omparse-boot PUBLIC omc::3rd::omantlr3)

# Define OMC_BOOTSTRAPPING for the boot lib.
target_compile_definitions(omparse-boot PRIVATE OMC_BOOTSTRAPPING)

# To find the generated antlr headers. SYSTEM to disable warnings on the generated code.
target_include_directories(omparse-boot SYSTEM PRIVATE ${GNERATED_DIRECTORY})
target_include_directories(omparse-boot PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})

target_include_directories(omparse-boot PRIVATE ${OMCompiler_SOURCE_DIR}) # for revision.h