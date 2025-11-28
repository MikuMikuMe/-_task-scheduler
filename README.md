# ## task-scheduler

Creating a comprehensive task scheduler requires several components. Let's outline the key features and develop a simple yet robust Python program using threads, queues, and the `schedule` library (which is commonly used for task scheduling in Python). This will include adding tasks, prioritizing them, and handling recurring tasks.

We'll also include error handling to ensure the scheduler runs smoothly.

### Steps for the Task Scheduler

1. **Task Management**: Add, remove, and view tasks.
2. **Prioritization**: Allow tasks to be prioritized.
3. **Scheduling**: Schedule tasks at specific intervals or times.
4. **Error Handling**: Gracefully handle errors and edge cases.

### Requirements

- `schedule` library for task scheduling.
- `threading` and `queue` for concurrency.
- Python standard libraries such as `datetime` and `time`.

```bash
# Install the schedule package, if not already installed
pip install schedule
```

### The Complete Program

Here's a basic version of the Task Scheduler:

```python
import schedule
import time
import threading
from datetime import datetime
from queue import PriorityQueue

# Define a task with priority
class Task:
    def __init__(self, name, priority, action):
        self.name = name
        self.priority = priority
        self.action = action

    def __lt__(self, other):
        return self.priority < other.priority

# Task Manager to handle tasks
class TaskManager:
    def __init__(self):
        self.task_queue = PriorityQueue()

    def add_task(self, name, priority, action):
        task = Task(name, priority, action)
        self.task_queue.put(task)
        print(f"Task '{name}' added with priority {priority}.")

    def get_task(self):
        if not self.task_queue.empty():
            return self.task_queue.get()
        else:
            raise Exception("No tasks in queue.")

    def list_tasks(self):
        tasks = []
        while not self.task_queue.empty():
            task = self.task_queue.get()
            tasks.append(task)
            print(f"Task: {task.name}, Priority: {task.priority}")
        # Re-queue the tasks
        for task in tasks:
            self.task_queue.put(task)


# Error Handler
def error_handler(function):
    def wrapper(*args, **kwargs):
        try:
            return function(*args, **kwargs)
        except Exception as e:
            print(f"Error occurred: {e}")
    return wrapper

# Sample tasks
@error_handler
def sample_task():
    print(f"Running sample task at {datetime.now()}")

@error_handler
def another_task():
    print(f"Running another task at {datetime.now()}")

# Scheduler to handle time-based tasks
class Scheduler:
    def __init__(self, task_manager):
        self.task_manager = task_manager

    @error_handler
    def handle_tasks(self):
        task = self.task_manager.get_task()
        task.action()

    def run_scheduler(self):
        schedule.every(10).seconds.do(self.handle_tasks)
        while True:
            schedule.run_pending()
            time.sleep(1)

# Initialize Task Manager and Scheduler
task_manager = TaskManager()
scheduler = Scheduler(task_manager)

# Add tasks to Task Manager
task_manager.add_task("Sample Task", 1, sample_task)
task_manager.add_task("Another Task", 2, another_task)

# List tasks
task_manager.list_tasks()

# Run the scheduler in a separate thread
scheduler_thread = threading.Thread(target=scheduler.run_scheduler)
scheduler_thread.start()

```

### Explanation

- **Task Definition**: A `Task` class holds information about a task (name, priority, action function).
- **Task Manager**: Manages task queue using a priority queue (`PriorityQueue`) for task prioritization.
- **Error Handling**: A decorator `error_handler` is used to manage exceptions gracefully.
- **Task Scheduler**: `Scheduler` uses the `schedule` library to execute tasks every 10 seconds.
- **Threading**: The scheduler runs in its own thread, allowing the main program to remain responsive.

### Notes:

- This example provides a simple scheduler. For real-world applications, more robust time handling and persistence methods (e.g. databases or logging) would be needed.
- Task actions should be thread-safe, especially if they access shared resources.

This simple Python script provides the basic framework to build upon for more complex scheduling and task management systems. It can be extended with more features as needed, such as recurring tasks, different scheduling methods, and enhanced user interfaces.