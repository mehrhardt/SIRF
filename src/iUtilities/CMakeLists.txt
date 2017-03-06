#========================================================================
# Author: Kris Thielemans
# Copyright 2016, 2017 University College London
#
#  Licensed under the Apache License, Version 2.0 (the "License");
#  you may not use this file except in compliance with the License.
#  You may obtain a copy of the License at
#
#         http://www.apache.org/licenses/LICENSE-2.0.txt
#
#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS,
#  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#  See the License for the specific language governing permissions and
#  limitations under the License.
#
#=========================================================================

set(CMAKE_POSITION_INDEPENDENT_CODE True)

add_library(iutil iutilities.cpp)
target_include_directories(iutil PUBLIC
  "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>$<INSTALL_INTERFACE:include>"
  )

if(BUILD_PYTHON)

  FIND_PACKAGE(SWIG REQUIRED)
  INCLUDE("${SWIG_USE_FILE}")

  SET_SOURCE_FILES_PROPERTIES(pyiutil.i PROPERTIES CPLUSPLUS ON)
  # find libraries and include files
  # TODO would be better to use target_include_directories

  FIND_PACKAGE(PythonLibs REQUIRED)
  INCLUDE_DIRECTORIES(${PYTHON_INCLUDE_PATH})
  #FIND_PACKAGE(Numpy REQUIRED)
  #INCLUDE_DIRECTORIES(${NUMPY_INCLUDE_DIRS})
  SWIG_ADD_MODULE(pyiutil python pyiutil.i)
  SWIG_LINK_LIBRARIES(pyiutil iutil ${PYTHON_LIBRARIES})

  set(PYTHON_DEST "${CMAKE_INSTALL_PREFIX}/python" CACHE PATH "Destination for python module")
  INSTALL(TARGETS ${SWIG_MODULE_pyiutil_REAL_NAME} DESTINATION "${PYTHON_DEST}")
  INSTALL(FILES "${CMAKE_CURRENT_BINARY_DIR}/pyiutil.py"  DESTINATION "${PYTHON_DEST}")

endif(BUILD_PYTHON)

if(BUILD_MATLAB)

  include_directories(${MATLAB_INCLUDE_DIR})
  add_definitions(${MATLAB_CXXFLAGS})

  add_library(mutilities SHARED mutilities.c)
  # fix output name and link flags
  SET_TARGET_PROPERTIES(mutilities PROPERTIES
        SUFFIX ".${MATLAB_MEX_EXT}" PREFIX "${MATLAB_PREFIX}"
        LINK_FLAGS "${MATLAB_CXXLINKER_FLAGS}") 
  target_link_libraries(mutilities iutil ${MATLAB_LIBRARIES} )

  INSTALL(TARGETS mutilities DESTINATION "${MATLAB_DEST}")
  INSTALL(FILES mutilities.h shrhelp.h DESTINATION "${MATLAB_DEST}")
  #INSTALL(DIRECTORY +mutilities DESTINATION "${MATLAB_DEST}")
  #file(GLOB MatlabFiles "${CMAKE_CURRENT_LIST_DIR}/*.m")
  #INSTALL(FILES ${MatlabFiles} DESTINATION "${MATLAB_DEST}")


endif(BUILD_MATLAB)