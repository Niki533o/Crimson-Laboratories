# OpenGL (Windows Platform)
- Creating a Window and rendering a triangle (using SHADERS)

# Setting up OpenGL in Visual Studio 2019
## 1. GLFW
### Download GLFW
Go to https://www.glfw.org/download.html and
download 32-bit Windows Binaries under 'Windows pre-compiled binaries'.

### GLFW library setup
1. Extract glfw.zip folder.
2. Go to your Project Folder and make a new folder named 'Dependencies'. In that folder, add another new folder named 'GLFW'.
3. From the extracted glfw folder, copy the 'include' file and the 'lib-vc2019' file and paste it into the 'GLFW' folder that was made.
4. In Visual Studio, right click on the project folder and go to 'properties'.
5. Configuration should be set to 'All configurations' and Platform to be on 'Win32'.
```
separate all directories with a semicolon
```
6. Under 'Configuration Properties' --> 'C/C++' --> 'General', add "$(SolutionDir)Dependencies\GLFW\include" in 'Additional Library Directories'.
7. Under 'Configuration Properties' --> 'Linker' --> 'General', add "$(SolutionDir)Dependencies\GLFW\lib-vc2019" in 'Additional Library Directories'.
8. Under 'Configuration Properties' --> 'Linker' --> 'Input', erase everything and add "glfw3.lib; opengl32.lib; User32.lib; Gdi32.lib; Shell32.lib;".

### Creating a Window
In your project .cpp file, copy & paste the example code given below.
Build the project and Debug. A black window should open.

#### Example Code of how to create a window
Obtained from https://www.glfw.org/documentation.html
```
#include <GLFW/glfw3.h>

int main(void)
{
    GLFWwindow* window;

    // Initialize the library
    if (!glfwInit())
        return -1;

    // Create a windowed mode window and its OpenGL context
    window = glfwCreateWindow(640, 480, "Hello World", NULL, NULL);
    if (!window)
    {
        glfwTerminate();
        return -1;
    }

    // Make the window's context current 
    glfwMakeContextCurrent(window);

    // Loop until the user closes the window 
    while (!glfwWindowShouldClose(window))
    {
        // Render here 
        glClear(GL_COLOR_BUFFER_BIT);

        // Swap front and back buffers 
        glfwSwapBuffers(window);

        // Poll for and process events 
        glfwPollEvents();
    }

    glfwTerminate();
    return 0;
}
```
## 2. GLEW
### Download GLEW
Go to http://glew.sourceforge.net/ and download 'Binaries Windows 32-bit and 64-bit'.

### GLEW library setup
1. Extract the glew.zip folder.
2. Copy the extracted glew folder, paste it into your 'Dependencies' folder and rename the folder to 'GLEW'.
3. In Visual Studio, right click on the project folder and go to 'properties'.
4. Configuration should be set to 'All configurations' and Platform to be on 'Win32'.
```
separate all directories with a semicolon
```
5. Under 'Configuration Properties' --> 'C/C++' --> 'General', add "$(SolutionDir)Dependencies\GLEW\include" in 'Additional Library Directories'.
6. Under 'Configuration Properties' --> 'Linker' --> 'General', add "$(SolutionDir)Dependencies\GLEW\lib\Release\Win32" in 'Additional Library Directories'.
7. Under 'Configuration Properties' --> 'Linker' --> 'Input', add "glew32s.lib".
8. Under 'Configuration Properties' --> 'C/C++' --> 'Preprocessor', add "GLEW_STATIC".

### Initializing GLEW and rendering a triangle

In your project .cpp file, add ``` #include <GL/glew.h> ``` and ``` #include <iostream> ``` before the glfw header.
Then Add the code below after OpenGL Context creation.
```
if (glewInit() != GLEW_OK)
        std::cout << "Error!" << std::endl;
```
Create an array of 3 vectors representing 3 vertices (which in my case is)
```
float bufferdata[9] = {
       -1.0f, -1.0f, 0.0f,
        1.0f, -1.0f, 0.0f,
        0.0f,  1.0f, 0.0f

    };
 ```
 Give the vertices to OpenGL by creating buffers (example below)
 ```
 unsigned int buffer;
    glGenBuffers(1, &buffer);

    glBindBuffer(GL_ARRAY_BUFFER, buffer);
    glBufferData(GL_ARRAY_BUFFER, sizeof(bufferdata), bufferdata, GL_STATIC_DRAW);

    glEnableVertexAttribArray(0);
    glVertexAttribPointer(
        0,                  // attribute 0. No particular reason for 0, but must match the layout in the shader.
        3,                  // size
        GL_FLOAT,           // type
        GL_FALSE,           // normalized?
        sizeof(float) * 3,  // stride
        (const void*)0     // array buffer offset
    );
```
Draw the triangle in the main loop after clearing screen.
```
glDrawArrays(GL_TRIANGLES, 0, 3);
```
And a window displaying a white triangle should appear.

### Shader Compilation
Create shaders. A Vertex Shader, which will be executed for each vertex, and a Fragment Shader, which will be executed for each sample by adding the code below right after the buffers and attribs functions in the main function.
```
std::string vertexShader =
        "#version 330 core\n"
        "layout(location = 0) in vec4 position;\n"
        "void main()\n"
        "{\n"
        "gl_Position = position;\n"
        "}\n";

    std::string fragmentShader =
        "#version 330 core\n"
        "layout(location = 0) out vec4 color;\n"
        "void main()\n"
        "{\n"
        "color = vec4(1.0, 0.0, 0.0, 1.0);\n"
        "}\n";
```
Write a function to create the shaders and another function to compile them.
```
static unsigned int CreateShader(const std::string& vertexShader, const std::string& fragmentShader) {

    unsigned int program = glCreateProgram();
    unsigned int vs = CompileShader(GL_VERTEX_SHADER, vertexShader);
    unsigned int fs = CompileShader(GL_FRAGMENT_SHADER, fragmentShader);

    glAttachShader(program, vs);
    glAttachShader(program, fs);
    glLinkProgram(program);
    glValidateProgram(program);

    glDeleteShader(vs);
    glDeleteShader(fs);

    return program;
}

static unsigned int CompileShader(unsigned int type, const std::string& source) {

    unsigned int id = glCreateShader(type);
    const char* src = source.c_str();
    glShaderSource(id, 1, &src, nullptr);
    glCompileShader(id);

    int result;
    glGetShaderiv(id, GL_COMPILE_STATUS, &result);
    if (result == GL_FALSE) {

        int length;
        glGetProgramiv(id, GL_INFO_LOG_LENGTH, &length);
        char* message = (char*)alloca(length * sizeof(char));
        glGetShaderInfoLog(id, length, &length, message);

        std::cout << "Failed to compile " << (type == GL_VERTEX_SHADER ? "vertex" : "fragment") << "shader!" << std::endl;
        std::cout << message << std::endl;

        glDeleteShader(id);
        return 0;
    }

    return id;
}
```
Call the createshader function (before the main loop)
```
unsigned int shader = CreateShader(vertexShader, fragmentShader);
```
Tell OpenGL that you want to use the shader (before the main loop)
```
glUseProgram(shader);
```
# ScreenShot 
![screenshot](https://user-images.githubusercontent.com/39631095/89414579-43015780-d748-11ea-9927-27134792f0e2.png)
