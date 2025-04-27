# Final Report

## 1. What I Found

While reviewing and analyzing the existing codebase, I found the following issues and opportunities for improvement:

- **Lack of Modularity**:  
  The dispatcher or controller logic was not modularized, leading to reduced flexibility and poor separation of concerns.
- **No Unit Tests**:  
  There were no unit tests, and most outputs were handled through basic console prints, limiting the ability to verify individual components in isolation.
- **Input Handling Issues**:  
  After running the game, I noticed that the input controls (e.g., "A" to toggle Auto Fire, "Escape" to quit) didn't work as expected. I wasn't sure if it was a MacOS-specific issue, but the input events were being directly checked in the main loop, which could be problematic across different platforms. This could be improved by making the input handling more modular, which would also make the system more extensible and easier to manage as it grows.
- **Error Handling**:  
  There doesn't seem to be much error handling, especially in cases like input handling or unexpected game state changes. If we are seeing errors or issues, it might be useful to add more debug statements or checks to see if everything is behaving as expected.

## 2. What I Added and Changed

### Logger (logger.h and logger.cpp)

The original `Logger` class provided basic background logging but lacked essential features like timestamping, thread safety, and flushing control. Here’s what I improved:

- **Added `Log(const std::string& message)`**:  
  I added `Log(const std::string& message)`, which adds a timestamp to the message and appends it to a thread-safe queue using `std::mutex`. This allows safe concurrent logging.

- **Added `Flush()`**:  
  The `Flush()` method forces logs to be written to disk immediately, giving control over when logs are saved.

- **Made Logging Thread-Safe with `std::mutex`**:  
  I introduced a `std::mutex` to protect the log queue from race conditions, ensuring thread-safe logging across multiple threads.

- **Improved the `ProcessLogs()` Method**:  
  Logs are now written in the correct order (FIFO) by copying messages to a local vector and writing them outside the critical section, reducing mutex lock time and improving performance.

- **Improved Shutdown Behaviour**:  
  I added an `exitFlag` and ensured the background logging thread finishes before shutdown using `join()`, preventing data loss.

- **General Refinements**:  
  I enforced the singleton pattern, used an initializer list in the constructor, and made sure logs are written fresh on each program run using `std::ios_base::trunc`.

These changes make the `Logger` class thread-safe, reliable, and more efficient, with timestamped, ordered logging and manual flushing control.


### Tower (tower.h and tower.cpp)

The original `Tower` class had minimal functionality and was tightly coupled with low-level socket operations. I refactored it for better modularity and error handling, and implemented essential functionalities:

- **Added Constructor with Port Configuration**:  
  I added a constructor that accepts a port number to bind the tower to a specified port at runtime.

- **Encapsulated Socket Setup Logic**:  
  I moved socket creation, binding, and listening into private methods (`CreateSocket()`, `BindSocket()`, `StartListening()`) to improve code readability and separation of concerns.

- **Error Handling and Logging**:  
  I added error checks and clear `std::runtime_error` exceptions for each step in the socket setup process to make debugging easier.

- **Server Socket Getter**:  
  I added a `GetServerSocket()` method to provide access to the internal server socket when needed by other components.

The `Tower` class is now modular, more flexible, and provides better error reporting, improving its ability to integrate with other components.


### Pooler (pooler.h and pooler.cpp)

The original `Pooler` functioned as a thread pool for client connections, but I redesigned it to handle a memory pool for enemies, improving performance and memory management:

- **Added Thread Pool Initialization**:  
  I added a constructor to initialize worker threads that handle client sockets via a synchronized queue, improving scalability.

- **Synchronized Task Queue**:  
  I implemented a thread-safe queue using `std::mutex` and `std::condition_variable` to safely pass client sockets between the main thread and workers.

- **Start() and Stop() Methods**:  
  I added `Start()` and `Stop()` methods to control the lifecycle of the thread pool, enabling clean startup and shutdown.

- **Connection Handling in Worker Threads**:  
  Worker threads process client connections with a placeholder `HandleClient(int clientSocket)` function, making the logic extensible.

- **Improved Logging and Error Reporting**:  
  Throughout the `Pooler` implementation, added logging (using the improved `Logger`) to track when clients are accepted, processed, or if errors occur.

The `Pooler` class is now a lightweight, scalable request handler, separating concerns between socket listening (`Tower`) and client processing (`Pooler`), which simplifies the overall architecture.

## 3. What Other Changes I'd Recommend

To further improve the codebase and simulation framework, I would recommend the following enhancements:

### Input Handling and Game Logic

- **Modularize Input Handling**:  
  While the existing `getKeyPress()` function works to handle user input, I recommend modularizing the input handling further by moving it to a separate `HandleInput()` function. This would make the input logic easier to maintain, especially as more input controls are added or modified in the future.

- **Cross-Platform Compatibility**:  
  Since I'm using macOS, there could be platform-specific issues with how input is handled, especially with certain key events not being detected correctly. It would be beneficial to test input handling on different platforms to ensure compatibility across all systems.

### Testing

- **Add Unit Tests**:  
  Use a framework like Google Test to write tests for each class (`Logger`, `Tower`, `Pooler`) to validate functionality and catch regressions.

- **Automate Output Comparison**:  
  Create test scenarios and compare actual vs expected log outputs.

### Configurability

- **Use Config Files**:  
  Read settings (like number of towers, timing, client behavior) from a config file (e.g., JSON or TXT) rather than hardcoding them.

### Performance

- **Threading Support**:  
  Add multi-threaded simulation capabilities to model more realistic, concurrent tower processing.

### Error Handling

- **Improve Error Handling**:  
  There doesn't seem to be much error handling, especially in cases like input handling or unexpected game state changes. If we are seeing errors or issues, it might be useful to add more debug statements or checks to see if everything is behaving as expected.
