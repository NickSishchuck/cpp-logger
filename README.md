# Logger Class

A lightweight, configurable C++ logging utility designed for easy extension and customization.

## Features

- Singleton pattern for easy global access
- Multiple log levels (DEBUG, INFO, WARNING, ERROR, FATAL, TODO)
- Console output with optional ANSI colors
- Automatic file logging with timestamps
- Source file and line information
- Configurable behavior (timestamps, colors, source info)
- Extensible design for custom logging needs

## Usage

### Basic Setup

```cpp
#include "Logger.h"

int main() {
    // Get the logger instance
    Logger* logger = Logger::getInstance();
    
    // Initialize the logger
    logger->init();
    
    // Configure the logger
    logger->setLogLevel(Logger::LogLevel::DEBUG);   // Show all log levels including debug
    logger->enableColors(true);                     // Enable colored console output
    
    // Start logging
    LOG_INFO("Application started");
    
    return 0;
}
```

### Basic Logging

The Logger provides both direct method calls and convenient macros:

```cpp
// Using macros (automatically includes file and line information)
LOG_DEBUG("Initializing renderer");
LOG_INFO("Application started");
LOG_WARNING("Low memory condition detected");
LOG_ERROR("Failed to load resource");
LOG_FATAL("Critical system failure");
LOG_TODO("Implement better error handling here");

// Using direct method calls (manual file/line info)
Logger::getInstance()->debug("Debug message");
Logger::getInstance()->info("Info message", __FILE__, __LINE__);
```

### Configuration

```cpp
// Get logger instance
Logger* logger = Logger::getInstance();

// Initialize (creates log directory and file)
logger->init();

// Configure settings
logger->setLogLevel(Logger::LogLevel::DEBUG);  // Show all messages (including debug)
logger->enableColors(true);                    // Enable ANSI colors in console
logger->enableTimestamps(true);                // Show timestamps
logger->enableSourceInfo(true);                // Show source file and line
logger->setBasePath("/path/to/project/");      // Base path for shorter file paths
```

### Extending the Logger: OpenGL Error Example

The Logger can be extended for domain-specific logging needs. Here's an example of how it's been extended for OpenGL error logging:

```cpp
// Macro for OpenGL error logging
#define LOG_GLERROR(context) { \
    GLenum glErr = glGetError(); \
    if (glErr != GL_NO_ERROR) { \
        std::string errorMsg = std::string(context) + ": " + Logger::getInstance()->glErrorToString(glErr); \
        Logger::getInstance()->error(errorMsg, __FILE__, __LINE__); \
    } \
}

// Usage example
void initializeShaders() {
    // Create shader
    GLuint vertexShader = glCreateShader(GL_VERTEX_SHADER);
    LOG_GLERROR("After creating vertex shader");
    
    // Compile shader
    glCompileShader(vertexShader);
    LOG_GLERROR("After shader compilation");
    
    // Log shader info messages
    Logger::getInstance()->enableSourceInfo(false);  // Disable source info for cleaner shader logs
    
    GLint success;
    glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);
    if (!success) {
        GLchar infoLog[512];
        glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
        LOG_ERROR(std::string("Shader compilation failed: ") + infoLog);
    }
    
    Logger::getInstance()->enableSourceInfo(true);  // Re-enable source info
}
```

## Log Levels

Log levels are processed in order of severity. If you set a higher level, lower levels will be filtered out:

- `DEBUG` - Detailed information, typically useful only for diagnosing problems
- `INFO` - Confirmation that things are working as expected
- `WARNING` - Indication that something unexpected happened, but the application still works
- `ERROR` - A more serious problem that prevented an operation from completing
- `FATAL` - A very severe error that will likely lead to application termination
- `TODO` - Markers for code that needs to be revisited or completed

## Display Options

- `enableTimestamps(bool)` - Show/hide timestamps in log output
- `enableSourceInfo(bool)` - Show/hide source file and line numbers
- `enableColors(bool)` - Enable/disable ANSI color codes in console output

## Files and Paths

- `setBasePath(string)` - Set the base path for shorter file paths in logs
- Log files are automatically created in a `logs` directory with date-time in the filename

## Complete Example

```cpp
#include "Logger.h"
#include <stdexcept>

class ResourceManager {
private:
    Logger* logger;
    
public:
    ResourceManager() {
        logger = Logger::getInstance();
        logger->info("ResourceManager initialized", __FILE__, __LINE__);
    }
    
    bool loadResource(const std::string& path) {
        try {
            logger->debug("Loading resource: " + path);
            
            // Resource loading logic here...
            bool success = true;
            
            if (success) {
                logger->info("Successfully loaded resource: " + path);
                return true;
            } else {
                logger->error("Failed to load resource: " + path);
                return false;
            }
        } catch (const std::exception& e) {
            // Temporarily disable source info for this specific error
            logger->enableSourceInfo(false);
            logger->error("Exception during resource loading: " + std::string(e.what()));
            logger->enableSourceInfo(true);
            
            return false;
        }
    }
};

int main() {
    // Initialize and configure logger
    Logger* logger = Logger::getInstance();
    logger->init();
    logger->setLogLevel(Logger::LogLevel::DEBUG);
    logger->enableColors(true);
    
    LOG_INFO("Application starting");
    
    ResourceManager resourceManager;
    resourceManager.loadResource("textures/background.png");
    
    LOG_INFO("Application shutting down");
    return 0;
}
```

## Extending with Your Own Categories

You can extend the Logger to add your own custom logging categories or domain-specific error handling:

```cpp
// In your domain-specific code
std::string networkErrorToString(int errorCode) {
    // Conversion logic here
    return "Network error " + std::to_string(errorCode);
}

// Create a macro for network error logging
#define LOG_NETWORK_ERROR(errorCode, context) { \
    std::string errorMsg = std::string(context) + ": " + networkErrorToString(errorCode); \
    Logger::getInstance()->error(errorMsg, __FILE__, __LINE__); \
}

// Usage
int status = sendData(socket, data);
if (status < 0) {
    LOG_NETWORK_ERROR(status, "Failed to send data");
}
```

## Implementation Notes

- The Logger uses a singleton pattern for global access
- Log files are automatically created with timestamps in their names
- Thread safety considerations should be added for multi-threaded applications
- Consider memory management for the singleton instance in larger applications
