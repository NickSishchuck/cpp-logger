# Logger

A lightweight, configurable C++ logging utility designed for easy extension and customization.

## Features

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

    // Configure the logger (see next section for available settings)

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

// Using direct method calls
Logger::getInstance()->debug("Debug message");  // No file/line info
Logger::getInstance()->info("Info message", __FILE__, __LINE__);  // With file/line info
```

### Configuration

```cpp
// Get logger instance
Logger* logger = Logger::getInstance();

// Initialize (creates log directory and file)
logger->init();

// Set log level
// Note: LogLevel is an internal enum, use these numeric values:
// 0 = DEBUG, 1 = INFO, 2 = WARNING, 3 = ERROR, 4 = FATAL, 5 = TODO
logger->setLogLevel(static_cast<Logger::LogLevel>(0));  // Set to DEBUG level

// Other configuration options
logger->enableColors(true);        // Enable ANSI colors in console
logger->enableTimestamps(true);    // Show timestamps
logger->enableSourceInfo(true);    // Show source file and line
logger->setBasePath("/path/to/project/");  // Base path for shorter file paths
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
        LOG_INFO("ResourceManager initialized");
    }

    bool loadResource(const std::string& path) {
        try {
            LOG_DEBUG("Loading resource: " + path);

            // Resource loading logic here...
            bool success = true;

            if (success) {
                LOG_INFO("Successfully loaded resource: " + path);
                return true;
            } else {
                LOG_ERROR("Failed to load resource: " + path);
                return false;
            }
        } catch (const std::exception& e) {
            // For this specific error, log without using the macro
            // to customize the behavior
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
    logger->setLogLevel(static_cast<Logger::LogLevel>(0));  // DEBUG level
    logger->enableColors(true);

    LOG_INFO("Application starting");

    ResourceManager resourceManager;
    resourceManager.loadResource("textures/background.png");

    LOG_INFO("Application shutting down");
    return 0;
}
```

## Creating Custom Logging Categories

While the Logger class doesn't directly support custom categories, you can create your own wrapper functions or macros for domain-specific logging:

```cpp
// In your domain-specific code
void logNetworkError(int errorCode, const std::string& context, const char* file, int line) {
    std::string errorMsg;

    switch (errorCode) {
        case -1: errorMsg = "Connection refused"; break;
        case -2: errorMsg = "Timeout"; break;
        default: errorMsg = "Unknown error " + std::to_string(errorCode);
    }

    std::string fullMessage = context + ": " + errorMsg;
    Logger::getInstance()->error(fullMessage, file, line);
}

// Create a macro for easy use
#define LOG_NETWORK_ERROR(errorCode, context) \
    logNetworkError(errorCode, context, __FILE__, __LINE__)

// Usage
int status = sendData(socket, data);
if (status < 0) {
    LOG_NETWORK_ERROR(status, "Failed to send data");
}
```

## Implementation Notes

- Log files are automatically created with timestamps in their names
- Thread safety considerations should be added for multi-threaded applications
- Consider memory management for the singleton instance in larger applications
- The log directory (`logs/`) is automatically created if it doesn't exist
