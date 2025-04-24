java栈的映射和过滤


架构特点：
1.采用泛型设计增强类型安全性
2.结合函数式编程特性(map/filter)
3.实现迭代器模式支持多种遍历方式
4.通过接口实现多态行为
5.演示了集合操作与业务模型的结合使用


step1:C:\Users\wangrusheng\IdeaProjects\untitled4\src\main\java\org\example\Main.java

```java
package org.example;

import org.example.collections.Stack;
import org.example.model.Bread;  // 如果使用Food示例

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.function.Function;

public class Main {
    public static void main(String[] args) {
        // 示例1: 基本栈操作
        Stack<Integer> intStack = new Stack<>();
        Arrays.asList(1, 2, 3, 4, 5, 6).forEach(intStack::push);

        Stack<Integer> doubled = intStack.map(x -> x * 2);
        Stack<Integer> filtered = intStack.filter(x -> x % 2 == 0);

        System.out.println("Original: " + intStack);
        System.out.println("Doubled: " + doubled);
        System.out.println("Filtered: " + filtered);

        // 示例2: 迭代器测试
        Stack<Integer> testStack = new Stack<>();
        Arrays.asList(10, 20, 30).forEach(testStack::push);
        testStack.forEach(item -> System.out.println("ForEach: " + item));

        // 示例3: Food示例（可选）
        Bread sourdough = new Bread("sourdough");
        System.out.println(sourdough.menuListing());
    }

    // 保持原myMap方法
    public static <T, U> List<U> myMap(List<T> list, Function<T, U> mapper) {
        List<U> result = new ArrayList<>();
        for (T item : list) result.add(mapper.apply(item));
        return result;
    }
}
```

step2:C:\Users\wangrusheng\IdeaProjects\untitled4\src\main\java\org\example\collections\Stack.java

```java
package org.example.collections;

import java.util.*;
import java.util.function.*;


public class Stack<Element> implements Iterable<Element> {
    private List<Element> items = new ArrayList<>();

    public void push(Element newItem) {
        items.add(newItem);
    }

    public Element pop() {
        if (items.isEmpty()) return null;
        return items.remove(items.size() - 1);
    }

    public <U> Stack<U> map(Function<Element, U> transformer) {
        Stack<U> result = new Stack<>();
        for (Element item : items) {
            result.push(transformer.apply(item));
        }
        return result;
    }

    public Stack<Element> filter(Predicate<Element> predicate) {
        Stack<Element> result = new Stack<>();
        for (Element item : items) {
            if (predicate.test(item)) {
                result.push(item);
            }
        }
        return result;
    }

    @Override
    public Iterator<Element> iterator() {
        return new StackIterator(new ArrayList<>(items));
    }

    public void pushAll(Iterable<? extends Element> sequence) {
        for (Element item : sequence) {
            push(item);
        }
    }

    @Override
    public String toString() {
        return items.toString();
    }

    // 迭代器实现
    private class StackIterator implements Iterator<Element> {
        private final List<Element> elements;
        private int current;

        public StackIterator(List<Element> elements) {
            this.elements = new ArrayList<>(elements);
            Collections.reverse(this.elements);
            current = 0;
        }

        @Override
        public boolean hasNext() {
            return current < elements.size();
        }

        @Override
        public Element next() {
            if (!hasNext()) throw new NoSuchElementException();
            return elements.get(current++);
        }
    }
}
```

step3:C:\Users\wangrusheng\IdeaProjects\untitled4\src\main\java\org\example\model\Food.java

```java
package org.example.model;

public interface Food {
    String menuListing();
}
```

step4:C:\Users\wangrusheng\IdeaProjects\untitled4\src\main\java\org\example\model\Bread.java

```java
package org.example.model;

public class Bread implements Food {
    private final String type;

    public Bread(String type) {
        this.type = type;
    }

    @Override
    public String menuListing() {
        return type + " bread";
    }
}
```

step5:运行结果：

```bash
Original: [1, 2, 3, 4, 5, 6]
Doubled: [2, 4, 6, 8, 10, 12]
Filtered: [2, 4, 6]
ForEach: 30
ForEach: 20
ForEach: 10
sourdough bread
```

end