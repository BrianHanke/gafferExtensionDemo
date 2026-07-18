cmake_minimum_required(VERSION 3.0)
project(DemoGafferExtension)

# --- ADD THIS LINE FOR GLOBAL MSVC RUNTIME SAFETY ---
add_definitions(-DBOOST_PYTHON_STATIC_LIB)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_VERBOSE_MAKEFILE OFF)
set(GAFFER_ROOT "C:/onedrive/Software/gaffer")
set(DEMO_INSTALL_PREFIX "C:/Source/gafferExtensionDemo/release")
set(CMAKE_INSTALL_PREFIX "C:/Source/gafferExtensionDemo/release" CACHE PATH "Default install location" FORCE)

if(APPLE)
  set(CMAKE_SHARED_LIBRARY_SUFFIX ".so")
endif(APPLE)

# build the library
file( GLOB DemoGafferExtensionSrc src/DemoGafferExtension/*.cpp )
link_directories( DemoGafferExtension ${GAFFER_ROOT}/lib )
add_library( DemoGafferExtension SHARED ${DemoGafferExtensionSrc} )
set_target_properties( DemoGafferExtension PROPERTIES
    WINDOWS_EXPORT_ALL_SYMBOLS TRUE
)
target_compile_definitions( DemoGafferExtension PRIVATE BOOST_SIGNALS_NO_DEPRECATION_WARNING=1 -D_GLIBCXX_USE_CXX11_ABI=0 )
target_include_directories( DemoGafferExtension PRIVATE
	include
	${GAFFER_ROOT}/include
	${GAFFER_ROOT}/include/Imath
	${GAFFER_ROOT}/include/OpenEXR
)
target_link_libraries( DemoGafferExtension Gaffer GafferScene IECore )
install( TARGETS DemoGafferExtension DESTINATION lib)

# build the python bindings
file( GLOB DemoGafferExtensionModuleSrc src/DemoGafferExtensionModule/*.cpp )
add_library( DemoGafferExtensionModule SHARED ${DemoGafferExtensionModuleSrc} )
set_target_properties( DemoGafferExtensionModule PROPERTIES PREFIX "" OUTPUT_NAME "_DemoGafferExtension" )
target_compile_definitions( DemoGafferExtensionModule PRIVATE BOOST_SIGNALS_NO_DEPRECATION_WARNING=1 -D_GLIBCXX_USE_CXX11_ABI=0 )
target_link_libraries( DemoGafferExtensionModule DemoGafferExtension )

string(REPLACE "." "" PYTHON_VERSION_STRIPPED ${PYTHON_VERSION})
string(REPLACE "3.7" "3.7m" PYTHON_FOLDER ${PYTHON_VERSION})
if(APPLE)
	target_include_directories( DemoGafferExtensionModule PRIVATE include ${GAFFER_ROOT}/include ${GAFFER_ROOT}/lib/Python.framework/Versions/${PYTHON_VERSION}/include/python${PYTHON_VERSION} )
	target_link_libraries( DemoGafferExtensionModule GafferBindings IECorePython boost_python Python)
else()
	target_include_directories( DemoGafferExtensionModule PRIVATE
		include
		${GAFFER_ROOT}/include
		${GAFFER_ROOT}/include/Imath
		${GAFFER_ROOT}/include/OpenEXR
		${GAFFER_ROOT}/include/python${PYTHON_FOLDER}
	)
	# --- ADD THESE DEFINITIONS TO TAME BOOST AUTO-LINKING ---
    target_compile_definitions( DemoGafferExtensionModule PRIVATE
        BOOST_ALL_NO_LIB      # Disables automatic linking completely on Windows
        BOOST_PYTHON_DYNAMIC_LIB # Configures Boost.Python for DLL-to-DLL linking
    )

    # Link against Gaffer's specific library name
    target_link_libraries( DemoGafferExtensionModule
        GafferBindings
        IECorePython
        boost_python311
        python${PYTHON_FOLDER}
    )
endif(APPLE)

install( TARGETS DemoGafferExtensionModule DESTINATION python/DemoGafferExtension )

# build the python modules
file( GLOB DemoGafferExtensionModuleFiles python/DemoGafferExtension/*.py )
install( FILES ${DemoGafferExtensionModuleFiles} DESTINATION python/DemoGafferExtension )
file( GLOB DemoGafferExtensionTestModuleFiles python/DemoGafferExtensionTest/*.py )
install( FILES ${DemoGafferExtensionTestModuleFiles} DESTINATION python/DemoGafferExtensionTest )
file( GLOB DemoGafferExtensionUIModuleFiles python/DemoGafferExtensionUI/*.py )
install( FILES ${DemoGafferExtensionUIModuleFiles} DESTINATION python/DemoGafferExtensionUI )
file( GLOB DemoGafferExtensionUITestModuleFiles python/DemoGafferExtensionUITest/*.py )
install( FILES ${DemoGafferExtensionUITestModuleFiles} DESTINATION python/DemoGafferExtensionUITest )

# build the startup configs
install( DIRECTORY startup DESTINATION . FILES_MATCHING PATTERN "*.py" )

# build the extra resources
install( DIRECTORY resources DESTINATION . FILES_MATCHING PATTERN "*.gfr" PERMISSIONS OWNER_READ GROUP_READ WORLD_READ )


