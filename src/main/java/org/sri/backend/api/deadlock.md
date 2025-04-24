# 🔒 Java Deadlock Example with Detection Using jconsole, jstack, and VisualVM

This project demonstrates a **deadlock scenario in Java** and how to detect it using JVM tools like `jconsole`, `jstack`, and `VisualVM`.

---

## 🧠 What is a Deadlock?

A **deadlock** happens when two or more threads are blocked forever, waiting for each other to release locks. This typically occurs in multi-threaded applications when:
- Thread A holds Lock 1 and waits for Lock 2
- Thread B holds Lock 2 and waits for Lock 1

Since neither thread can proceed, they’re stuck forever—this is a deadlock.

---

## 📌 Java Deadlock Example

Copy this code into a file named `DeadlockDemo.java`.

```java
public class DeadlockDemo {
    private static final Object lockA = new Object();
    private static final Object lockB = new Object();

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            synchronized (lockA) {
                System.out.println("Thread 1: Holding lock A...");
                try { Thread.sleep(100); } catch (InterruptedException ignored) {}
                System.out.println("Thread 1: Waiting for lock B...");
                synchronized (lockB) {
                    System.out.println("Thread 1: Acquired lock A and B");
                }
            }
        });

        Thread t2 = new Thread(() -> {
            synchronized (lockB) {
                System.out.println("Thread 2: Holding lock B...");
                try { Thread.sleep(100); } catch (InterruptedException ignored) {}
                System.out.println("Thread 2: Waiting for lock A...");
                synchronized (lockA) {
                    System.out.println("Thread 2: Acquired lock B and A");
                }
            }
        });

        t1.start();
        t2.start();
    }
}
```

---

## ▶️ How to Run the Program

1. Compile and run the code:
   ```bash
   javac DeadlockDemo.java
   java DeadlockDemo
   ```

2. Keep the application running while using the tools below.

---

## 🛠️ Tools to Detect Deadlock

### ✅ 1. jconsole (Graphical Tool)
- Run `jconsole` from terminal:
  ```bash
  jconsole
  ```
- Attach it to the Java process.
- Go to the **"Threads"** tab.
- If a deadlock is detected, it will show you the involved threads and locked resources.

### ✅ 2. jstack (Command-line)
- Get the Java process ID:
  ```bash
  jps
  ```
- Run:
  ```bash
  jstack <PID> > thread-dump.txt
  ```
- Search the file:
  ```bash
  grep -A 20 BLOCKED thread-dump.txt
  ```
- Look for circular waits: one thread is `waiting to lock` something that another thread is holding.

### ✅ 3. VisualVM (Advanced GUI Tool)
- Download: https://visualvm.github.io/download.html
- Extract and run:
  ```bash
  ./visualvm/bin/visualvm
  ```
- Attach to your Java process.
- Go to the **"Threads"** tab:
    - See all thread states.
    - VisualVM will **highlight** deadlocked threads and show **stack traces**.

---

## 🧪 Bonus: Programmatic Deadlock Detection

You can even write code to detect deadlocks at runtime using Java's built-in `ThreadMXBean`.

```java
import java.lang.management.*;

public class DeadlockChecker {
    public static void main(String[] args) {
        ThreadMXBean bean = ManagementFactory.getThreadMXBean();
        long[] ids = bean.findDeadlockedThreads();

        if (ids != null) {
            ThreadInfo[] infos = bean.getThreadInfo(ids, true, true);
            for (ThreadInfo info : infos) {
                System.out.println("🔴 Deadlock detected involving: " + info.getThreadName());
            }
        } else {
            System.out.println("✅ No deadlocks detected");
        }
    }
}
```

---

## 📚 Summary

| Tool        | Use Case                          | Strengths                       |
|-------------|-----------------------------------|----------------------------------|
| `jconsole`  | Simple GUIs for thread info       | Auto-detects deadlocks          |
| `jstack`    | CLI tool for thread dumps         | Lightweight, works anywhere     |
| `VisualVM`  | Advanced GUI with thread insights | Great for many threads + live analysis |
| `ThreadMXBean` | Programmatic detection in code | Use in apps or admin dashboards |

---

## 👨‍💻 Author

Created for learning and debugging deadlocks in real-world Java applications. Feel free to fork and extend!
