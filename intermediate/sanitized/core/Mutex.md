# Mutex

Mutex implements a simple semaphore that can be used to coordinate access to
shared data from multiple concurrent threads.

Example:

    semaphore = Mutex.new

    a = Thread.new {
      semaphore.synchronize {
        # access shared resource
      }
    }

    b = Thread.new {
      semaphore.synchronize {
        # access shared resource
      }
    }