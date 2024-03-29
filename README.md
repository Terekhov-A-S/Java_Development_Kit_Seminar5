# Java Development Kit (семинары)

![picture for project](https://raw.githubusercontent.com/Terekhov-A-S/Java_Development_Kit_Seminar5/main/src/main/resources/E3C09A05-7AA7-42E1-9DAB-1643AB9F52CA.png)

## Урок 5. Многопоточность


### Задача.

1. Пять безмолвных философов сидят вокруг круглого стола, перед каждым философом стоит тарелка спагетти.
2. Вилки лежат на столе между каждой парой ближайших философов.
3. Каждый философ может либо есть, либо размышлять.
4. Философ может есть только тогда, когда держит две вилки — взятую справа и слева.
5. Философ не может есть два раза подряд, не прервавшись на размышления (можно не учитывать).

Описать в виде кода такую ситуацию. Каждый философ должен поесть три раза.

#### Решение

Эта задача обычно называется "Проблема обедающих философов". Она является классическим примером синхронизации в многозадачных программах. Будем использовать ReentrantLock, чтобы решить проблему взаимного исключения:

<details>

  <summary>Нажмите, чтобы открыть код</summary>

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class Philosopher implements Runnable {
    private final int id;
    private final Lock leftFork;
    private final Lock rightFork;

    public Philosopher(int id, Lock leftFork, Lock rightFork) {
        this.id = id;
        this.leftFork = leftFork;
        this.rightFork = rightFork;
    }

    private void think() throws InterruptedException {
        System.out.println("Философ " + id + " размышляет.");
        Thread.sleep(100); // Представим, что философ размышляет некоторое время
    }

    private void eat() throws InterruptedException {
        System.out.println("Философ " + id + " ест.");
        Thread.sleep(100); // Представим, что философ ест некоторое время
    }

    @Override
    public void run() {
        try {
            for (int i = 0; i < 3; i++) {
                think();
                leftFork.lock();
                try {
                    rightFork.lock();
                    try {
                        eat();
                    } finally {
                        rightFork.unlock();
                    }
                } finally {
                    leftFork.unlock();
                }
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}

class DiningPhilosophers {
    public static void main(String[] args) {
        int numPhilosophers = 5;
        Lock[] forks = new Lock[numPhilosophers];
        for (int i = 0; i < numPhilosophers; i++) {
            forks[i] = new ReentrantLock();
        }

        Thread[] philosophers = new Thread[numPhilosophers];
        for (int i = 0; i < numPhilosophers; i++) {
            philosophers[i] = new Thread(new Philosopher(i, forks[i], forks[(i + 1) % numPhilosophers]));
            philosophers[i].start();
        }

        // Ждем завершения всех философов
        try {
            for (Thread philosopher : philosophers) {
                philosopher.join();
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
    
```

</details>


Этот код создает 5 философов и предоставляет им вилки. Философы будут размышлять и есть поочередно, избегая взаимной блокировки благодаря использованию ReentrantLock. Программа завершится, когда каждый философ поест три раза.

Все работает корректно, согласно задания:

![work is correct](https://raw.githubusercontent.com/Terekhov-A-S/Java_Development_Kit_Seminar5/main/src/main/resources/2024-02-24_20-35-56.png)

---


*Подготовил студент Geek Brains* [**`Терехов Александр`**](https://gb.ru/users/1db43d0f-6c3d-46d1-bf5e-974b49af6f0d), Java_Development_Kit_Seminar5