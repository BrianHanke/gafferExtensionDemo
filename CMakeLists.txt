cmake_minimum_required(VERSION 3.16...4.2)

project(DemoGafferExtension VERSION 2026.7.0 LANGUAGES CXX)

# User-customizable paths
set(CMAKE_INSTALL_PREFIX "C:/Source/gafferExtensionDemo/release")
set(GAFFER_ROOT "C:/onedrive/Software/gaffer")

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(GAFFER_COMPILE_DEFS
    BOOST_SIGNALS_NO_DEPRECATION_WARNING=1
    _GLIBCXX_USE_CXX11_ABI=0
)
if(APPLE)
	set(CMAKE_SHARED_LIBRARY_SUFFIX ".so")
endif()

# Build the library
add_library(DemoGafferExtension SHARED src/DemoGafferExtension/DemoSceneProcessor.cpp)
set_target_properties(DemoGafferExtension PROPERTIES WINDOWS_EXPORT_ALL_SYMBOLS TRUE)
target_compile_definitions(DemoGafferExtension PRIVATE ${GAFFER_COMPILE_DEFS})
target_include_directories(DemoGafferExtension PRIVATE
    include
    ${GAFFER_ROOT}/include
    ${GAFFER_ROOT}/include/Imath
    ${GAFFER_ROOT}/include/OpenEXR
)
target_link_directories(DemoGafferExtension PRIVATE ${GAFFER_ROOT}/lib)
target_link_libraries(DemoGafferExtension PRIVATE Gaffer GafferScene IECore)

install(TARGETS DemoGafferExtension DESTINATION lib)

# Build the Python bindings
add_library(DemoGafferExtensionModule SHARED src/DemoGafferExtensionModule/DemoGafferExtensionModule.cpp)
set_target_properties(DemoGafferExtensionModule PROPERTIES
    PREFIX ""
    OUTPUT_NAME "_DemoGafferExtension"
    SUFFIX ".pyd"
)
target_compile_definitions(DemoGafferExtensionModule PRIVATE ${GAFFER_COMPILE_DEFS})
target_link_libraries(DemoGafferExtensionModule PRIVATE DemoGafferExtension)

if(APPLE) # Not tested, this probably won't work as-is ("Python.framework" in particular)
    target_include_directories(DemoGafferExtensionModule PRIVATE
        include
        ${GAFFER_ROOT}/include
        ${GAFFER_ROOT}/lib/Python.framework/Versions/${PYTHON_VERSION}/include/python${PYTHON_VERSION}
    )
    target_link_libraries(DemoGafferExtensionModule PRIVATE
        GafferBindings
        IECorePython
        boost_python
        Python
    )
else() # Windows/Linux
    target_include_directories(DemoGafferExtensionModule PRIVATE
        include
        ${GAFFER_ROOT}/include
        ${GAFFER_ROOT}/include/Imath
        ${GAFFER_ROOT}/include/OpenEXR
    )
    target_link_directories(DemoGafferExtensionModule PRIVATE ${GAFFER_ROOT}/lib)
    target_compile_definitions(DemoGafferExtensionModule PRIVATE
        BOOST_ALL_NO_LIB
        BOOST_PYTHON_DYNAMIC_LIB
    )
    target_link_libraries( DemoGafferExtensionModule PRIVATE
        DemoGafferExtension
        Gaffer
        GafferScene
        IECore
        GafferBindings
        IECorePython
        boost_python311 # Hard-coded to Python 3.11, included with Gaffer 1.6
    )
endif()

install(TARGETS DemoGafferExtensionModule DESTINATION python/DemoGafferExtension)

# Build the Python modules and install
set(PYTHON_MODULES
    DemoGafferExtension
    DemoGafferExtensionTest
    DemoGafferExtensionUI
    DemoGafferExtensionUITest
)

foreach(module IN LISTS PYTHON_MODULES)
    install(DIRECTORY python/${module}/
        DESTINATION python/${module}
        FILES_MATCHING PATTERN "*.py"
    )
endforeach()

install(DIRECTORY startup/ DESTINATION startup FILES_MATCHING PATTERN "*.py")

install(DIRECTORY resources/
    DESTINATION resources
    FILES_MATCHING PATTERN "*.gfr"
    PERMISSIONS OWNER_READ GROUP_READ WORLD_READ
)