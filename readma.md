## Задание 1 (обязательное)

В этом задании мы будем разрабатывать систему планировщика задач.
Вам предстоит разработать и реализовать систему классов задач с использованием наследования и протестировать готовый планировщик задач.

Задачи будут трёх видов:

* `SimpleTask` — простая задача, про неё известны только `id` (число) и `title` (название);
* `Epic` — задача, состоящая из подзадач. Про неё известны `id` (число) и `subtasks` — массив из подзадач, каждая из которых является простым текстом (`String`);
* `Meeting` — задача, описывающая назначенную встречу. Про неё известны `id` — число, `topic` — тема обсуждения, `project` — название проекта, который будут обсуждать, и `start` — дата и время старта текстом.

Все объекты задач должны быть не изменяемы — исходные данные принимать в конструкторе при создании и не иметь сеттеров (только геттеры).

Мы хотим задачи хранить в менеджере. Назовём его класс `Todos` — список дел.
Было бы удобно хранить в нём задачи не по отдельности (т. е. один массив из `SimpleTask`, другой массив из `Epic`, третий массив из `Meeting`), а в одном общем массиве.
Это позволит сильно сократить и упростить код менеджера, а также мы сможем создавать новые виды задач без изменений кода `Todos`.
Чтобы хранить все задачи в одном массиве, нам нужен такой тип ячейки, который позволит в неё положить любую из наших задач.
С этим поможет полиморфизм, для которого мы создадим класс `Task`, в который вынесем общее, что есть во всех задачах, а другие задачи от него отнаследуем:

```java
import java.util.Objects;

public class Task {
    protected int id;

    public Task(int id) {
        this.id = id;
    }

    public int getId() {
        return id;
    }

    // Ниже находятся вспомогательные методы для корректной работы equals
    // Переопределять их в наследниках не нужно

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Task task = (Task) o;
        return id == task.id;
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}
```

А в классах задач будет `public class SimpleTask extends Task {...`, `public class Epic extends Task {...` и `public class Meeting extends Task {...`.

Реализация наследника на примере `SimpleTask`:

```java
public class SimpleTask extends Task {
    protected String title;

    public SimpleTask(int id, String title) {
        super(id); // вызов родительского конструктора
        this.title = title; // заполнение своих полей
    }

    public String getTitle() {
        return title;
    }
}
```

Реализуйте оставшиеся два класса — `Epic` и `Meeting`.

Это позволит нам в менеджере создать единый массив хранения задач, в который мы сможем положить любую из задач:

```java
public class Todos {

    private Task[] tasks = new Task[0]; // <- тут будут все задачи

    /**
     * Вспомогательный метод для имитации добавления элемента в массив
     * @param current Массив, в который мы хотим добавить элемент
     * @param task Элемент, который мы хотим добавить
     * @return Возвращает новый массив, который выглядит как тот, что мы передали,
     * но с добавлением нового элемента в конец
     */
    private Task[] addToArray(Task[] current, Task task) {
        Task[] tmp = new Task[current.length + 1];
        for (int i = 0; i < current.length; i++) {
            tmp[i] = current[i];
        }
        tmp[tmp.length - 1] = task;
        return tmp;
    }

    /**
     * Метод добавления задачи в список дел
     * @param task Добавляемая задача
     */
    public void add(Task task) { // <- вот здесь в параметре может лежать объект и вида SimpleTask, и вида Epic, и вида Meeting
        tasks = addToArray(tasks, task);
    }
    
    public Task[] findAll() {
        return tasks;
    }
}

```

Проверим это следующим тестом. Поместите его в `TodosTest`-класс:
```java
    @Test
    public void shouldAddThreeTasksOfDifferentType() {
        SimpleTask simpleTask = new SimpleTask(5, "Позвонить родителям");

        String[] subtasks = { "Молоко", "Яйца", "Хлеб" };
        Epic epic = new Epic(55, subtasks);

        Meeting meeting = new Meeting(
                555,
                "Выкатка 3й версии приложения",
                "Приложение НетоБанка",
                "Во вторник после обеда"
        );

        Todos todos = new Todos();

        todos.add(simpleTask);
        todos.add(epic);
        todos.add(meeting);

        Task[] expected = { simpleTask, epic, meeting };
        Task[] actual = todos.findAll();
        Assertions.assertArrayEquals(expected, actual);
    }
```

Теперь давайте добавим в менеджер возможность искать задачи по посковому запросу (`query`).
Подходит ли задача запросу, будет решать сама задача, для чего мы в `Task` заведём метод `matches`.

```java
    /**
     * Метод, проверяющий подходит ли эта задача поисковому запросу.
     * Эта логика должна быть определена в наследниках, у каждого она будет своя
     * @param query Поисковый запрос
     * @return Ответ на вопрос, подходит ли эта задача под поисковый запрос
     */
    public boolean matches(String query) {
        return false;
    }
```

В каждом же наследнике мы переопределим этот метод так, чтобы:

* `SimpleTask` подходил, если запрос `query` встречается в `title`;
* `Epic` подходил, если запрос `query` встречается хотя бы в одной из подзадач;
* `Meeting` подходил, если запрос `query` встречается в `topic` или `project`.

Рассмотрим на примере переопределения этого метода для класса `Meeting`.
Мы воспользуемся методом `contains`, который есть у каждого объекта типа `String`: `s1.contains(s2)` отвечает на вопрос, содержится ли текст из `s2` в тексте из `s1`.
В итоге мы может переопределить метод так:

```java
    @Override
    public boolean matches(String query) {
        if (topic.contains(query)) {
            return true;
        }
        if (project.contains(query)) {
            return true;
        }
        return false;
    }
```

Переопределите метод `matches` для двух оставшихся классов.
Для `SimpleTask` это можно сделать одним if, для `Epic` — циклом перебирая подзадачи и также проверяя каждую через `if`.
Напишите тесты на метод `matches` для всех трёх классов, поместите их в `TasksTest`-классе.

Теперь мы можем добавить метод поиска в сам класс `Todos`:
```java
    /**
     * Метод поиска задач, которые подходят под поисковый запрос
     * @param query Поисковый запрос
     * @return Массив из подошедших задач
     */
    public Task[] search(String query) {
        Task[] result = new Task[0]; // массив для ответа
        for (Task task : tasks) { // перебираем все задачи
            if (task.matches(query)) { // если задача подходит под запрос
                result = addToArray(result, task); // добавляем её в массив ответа
            }
        }
        return result;
    }
```

Напишите тесты на метод поиска в классе `TodosTest`.

Как видите, благодаря наследованию и полиморфизму, мы смогли создать менеджер задач, который работал бы с любыми типами задач.
Мы можем создать их ещё хоть сто разных видов, и менеджер менять для этого не потребуется.
