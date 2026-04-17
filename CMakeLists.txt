cmake_minimum_required(VERSION 3.10 FATAL_ERROR)
project(data_security LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 📂 Redirigir ejecutable a /dist
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/dist)

# 🔧 Configuración de rutas para la nueva versión de Raylib
set(RAYLIB_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/libs/raylib/include)
set(RAYLIB_LIB_DIR     ${CMAKE_CURRENT_SOURCE_DIR}/libs/raylib/lib)

# 🔧 Encuentra todos los .cpp dentro de /src
file(GLOB_RECURSE SOURCE_FILES CONFIGURE_DEPENDS src/*.cpp)

# Crea el ejecutable
add_executable(${PROJECT_NAME} ${SOURCE_FILES})

# 🛠️ Agrega las rutas de búsqueda de headers (Tus fuentes + Raylib)
target_include_directories(${PROJECT_NAME} PRIVATE 
    src 
    ${RAYLIB_INCLUDE_DIR}
)

# 📚 Enlaza la librería estática y las dependencias de Windows
# Nota: 'raylib' ahora se busca como archivo físico
target_link_libraries(${PROJECT_NAME} PRIVATE 
    ${RAYLIB_LIB_DIR}/libraylib.a
    opengl32
    gdi32
    winmm
)

# 🚀 Automatización: Evita "Permission Denied"
if(WIN32)
    add_custom_command(TARGET ${PROJECT_NAME} PRE_LINK
        COMMAND cmd /c "taskkill /F /IM ${PROJECT_NAME}.exe /T 2>nul || exit 0"
        COMMENT "Cerrando ejecutable antiguo..."
    )
endif()

# Definición de ruta de assets relativa al ejecutable (para portabilidad)
target_compile_definitions(${PROJECT_NAME} PUBLIC ASSETS_PATH="assets/")

# 📁 Copiar carpeta de assets al directorio de salida (dist)
add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_directory
        "${CMAKE_CURRENT_SOURCE_DIR}/assets"
        "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/assets"
    COMMENT "Sincronizando assets en /dist..."
)