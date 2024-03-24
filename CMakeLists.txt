cmake_minimum_required(VERSION 3.15 FATAL_ERROR)

set(DOCPROJ Doc)
set(DOCPROJ_NAME ${PROJ})
project(${DOCPROJ})

find_package(Doxygen)
find_package(Sphinx)

set(ADD_DOXY_TARGET TRUE)
set(ADD_SPHINX_TARGET TRUE)

if(NOT Doxygen_FOUND)
  set(ADD_DOXY_TARGET FALSE)
  message(STATUS "Doxygen program not found, skipping doxygen target generation")
endif()

if(NOT Sphinx_FOUND)
  set(ADD_SPHINX_TARGET FALSE)
  message(STATUS "Sphinx program not found, skipping sphinx target generation")
endif()

if(ADD_DOXY_TARGET)
  set(DOXYGEN_PROJECT_NAME "${DOCPROJ_NAME}")
  set(DOXYGEN_INPUT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../src)
  set(DOXYGEN_EXCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
  set(DOXYGEN_OUTPUT_DIR ${CMAKE_BINARY_DIR}/GeneratedDoc/doxygen)
  set(DOXYGEN_INDEX_FILE ${DOXYGEN_OUTPUT_DIR}/xml/index.xml)
  set(DOXYFILE_IN ${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile.in)
  set(DOXYFILE_OUT ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile)

  if("${DOCPROJ_PUBLIC_HEADERS} " STREQUAL " ")
    file(GLOB DOCPROJ_PUBLIC_HEADERS
       RELATIVE ${DOXYGEN_INPUT_DIR}
       "*.h"
    )
  endif()

  # Replace variables inside @@ with the current values
  configure_file(${DOXYFILE_IN} ${DOXYFILE_OUT} @ONLY)

  file(MAKE_DIRECTORY ${DOXYGEN_OUTPUT_DIR})  # Doxygen won't create this for us
  add_custom_command(
    OUTPUT
      ${DOXYGEN_INDEX_FILE}
    DEPENDS
      ${DOCPROJ_PUBLIC_HEADERS}
    COMMAND
      ${DOXYGEN_EXECUTABLE} ${DOXYFILE_OUT}
    MAIN_DEPENDENCY
      ${DOXYFILE_OUT} ${DOXYFILE_IN}
    COMMENT
      "Generating documentation with Doxygen"
  )

  add_custom_target(Doxygen DEPENDS ${DOXYGEN_INDEX_FILE})
  add_custom_target(${DOCPROJ})

  if(ADD_SPHINX_TARGET)
    set(SPHINX_SOURCE ${CMAKE_CURRENT_SOURCE_DIR})
    set(SPHINX_BUILD ${CMAKE_BINARY_DIR}/GeneratedDoc/sphinx/${}${PROJ}-${PROJ_VERSION}/html)
    set(SPHINX_INDEX_FILE ${SPHINX_BUILD}/index.html)

    file(COPY "${CMAKE_CURRENT_SOURCE_DIR}/conf.py" DESTINATION ${CMAKE_BINARY_DIR})
    set(RST_PROLOG_FILE "${CMAKE_BINARY_DIR}/global_substitutions.txt")
    set(rst_prolog
        ".. |get_source_cmd| replace:: ${PROJ_GET_SOURCE_CMD}\n"
        ".. |project_name| replace:: ${PROJ}"
    )
    string(REPLACE ";" "" rst_prolog "${rst_prolog}")
    file(WRITE "${RST_PROLOG_FILE}" "${rst_prolog}")

    add_custom_command(
      OUTPUT
        ${SPHINX_INDEX_FILE}
      COMMAND
        ${SPHINX_EXECUTABLE}
          -b html
          -c ${CMAKE_BINARY_DIR}
          -Dbreathe_default_project=${PROJ}
          -Dbreathe_projects.${PROJ}=${DOXYGEN_OUTPUT_DIR}/xml
          -Drelease=${CMAKE_BUILD_TYPE}
          -Dproject=${PROJ}
          -Dversion=${PROJ_VERSION}
          -Dcopyright=${PROJ_COPYRIGHT}
          -Dauthor=${PROJ_AUTHOR}
          ${SPHINX_SOURCE}
          ${SPHINX_BUILD}
      WORKING_DIRECTORY
        ${CMAKE_CURRENT_BINARY_DIR}
      DEPENDS
        ${SPHINX_SOURCE}/index.rst ${DOXYGEN_INDEX_FILE} ${RST_PROLOG_FILE}
      MAIN_DEPENDENCY
        ${CMAKE_BINARY_DIR}/conf.py
      COMMENT
        "Generating final documentation with Sphinx"
    )

    add_custom_target(Sphinx DEPENDS ${SPHINX_INDEX_FILE})
    add_dependencies(${DOCPROJ} Sphinx)

    set(SPHINX_DOC_ZIP_FILE "${CMAKE_BINARY_DIR}/${PROJ}-${PROJ_VERSION}-doc.zip")

    add_custom_command(
        OUTPUT
          ${SPHINX_DOC_ZIP_FILE}
        COMMAND
          ${CMAKE_COMMAND} -E tar "cf" ${SPHINX_DOC_ZIP_FILE} --format=zip -- ${SPHINX_BUILD}
        WORKING_DIRECTORY
          ${CMAKE_BINARY_DIR}/GeneratedDoc/sphinx/
        DEPENDS
          Sphinx
    )
    add_custom_target(SphinxDocZip DEPENDS ${SPHINX_DOC_ZIP_FILE})
    add_dependencies(${DOCPROJ} SphinxDocZip)
  else()
    add_dependencies(${DOCPROJ} Doxygen)
  endif()

  add_dependencies(${PROJ} ${DOCPROJ})
endif()
