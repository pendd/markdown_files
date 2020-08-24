## Big O

> What is Big O ?

==**We use Big O to describe the performance of an algorithm**==

Big O 关注的是性能消耗的增长

![](https://github.com/pendd/picture/raw/master/Data_Structure/Big O.png)

![](https://github.com/pendd/picture/raw/master/Data_Structure/Big_O_Graph.png)

### O(1)

> 算法的性能消耗是恒定的

```java
public void log(int[] numbers) {
    // 无论数组numbers的长度有多大，里面有多少项，都只取第一个，恒定的时间
	System.out.println(numbers[0]);
}
```

### O(n)

> 算法的性能消耗是线性的

```java
public void log(int[] numbers) {
    // 线性增长 O(n)
    for(int number : numbers) {
        System.out.println(number);
    }
}

public void log(int[] numbers) {
    // 线性增长 O(1 + n + 1)  => O(n)  当n趋于无穷时，就相当于O(n)
    System.out.println();
    for(int number : numbers) {
        System.out.println(number);
    }
    System.out.println();
}

public void log(int[] numbers) {
    // 线性增长 O(n + n) = O(2n) n 和 2n 都是线性增长 所以相当于 O(n) 
    for(int number : numbers) {
        System.out.println(number);
    }
    
    for(int number : numbers) {
        System.out.println(number);
    } 
}

public void log(int[] numbers, String[] names) {
    // 线性增长 O(n + m) => O(n)
    for(int number : numbers) {  // O(n)
        System.out.println(number); 
    }
    
    for(int name : names) {  // O(m)
        System.out.println(name);
    } 
}
```

![](https://github.com/pendd/picture/raw/master/Data_Structure/O_N_Graph.png)

### O(n ^ 2)

```java
public void log(int[] numbers) {
   // O(n * n) = O(n ^ 2)
    for(int first : numbers) {  // O(n)
        for(int second : numbers) // O(n)
        System.out.println(first + "," + second); 
    }
}

public void log(int[] numbers) {
    // O(n ^ 2 + n)  一个数的平方总散大于等于它自己  所以相当于 O(n ^ 2)
    for( int number : numbers) {   // O(n)
        System.out.println(number);
    }
 	// O(n ^ 2)
    for(int first : numbers) {  // O(n)
        for(int second : numbers) // O(n)
        System.out.println(first + "," + second); 
    }
}
```

### O(log n)

> Binary Search 就是一种 O(log n)

![](https://github.com/pendd/picture/raw/master/Data_Structure/O_LOGN_Graph.png)

### O(2^n)

![](https://github.com/pendd/picture/raw/master/Data_Structure/O_2N_Graph.png)

### Space Complexity 

```java
public void greet(String[] names) {
    // 输入数组的项数与输出是无关的  空间复杂度为 O(1)
    for( int i = 0; i < names.length; i ++) 
        System.out.println("Hi " + names[i]);
}

public void greet(String[] names) {
    // 输入数组的项数与输出是想关的  空间复杂度为 O(n)
    String[] copy = new String[names.length];
    
    for( int i = 0; i < names.length; i ++) 
        System.out.println("Hi " + names[i]);
}
```

## Arrays

**Lookup by Index  O(1)**

**Lookup by Value  O(n)**

**Insert                      O(n)**

**Delete                     O(n)**

### 自己实现数组

```java
public class Array {

    private int[] data;

    // 记录添加的元素个数   例如 int[10]  但是添加了5个元素进来  
    // 这个时候应该输出5个元素的值 后面5个0不应该输出
    private int count;

    Array(int size) {
        data = new int[size];
    }

    public void print() {
        // 这里为count 而不是data.length
        for (int i = 0; i < count; i++) {
            System.out.print(data[i] + "  ");
        }
    }

    public void insert(int item) {
        expandArray();
        data[count ++] = item;
    }

    /**
     * 时间复杂度：
     *  最好的情况：O(1)
     *  最坏的情况：O(n)
     *  而时间复杂度考虑的情况是最坏的情况，所以indexOf方法的时间复杂度为O(n)
     */
    public int indexOf(int item) {
        for (int i = 0; i < data.length; i++)
            if (data[i] == item)
                return i;
        // 数组中不存在该元素
        return -1;
    }

    public int length() {
        return count;
    }

    public void removeAtIndex(int index)  {
        count --;
        if (index >= count || index < 0) throw new IllegalArgumentException();
        for (int i = index; i < count; i++)
            data[i] = data[i + 1];
    }

    private void expandArray() {
        if (data.length == count) {
            copyArray();
        }
    }
    
    private void copyArray() {
        int[] newData = new int[count * 2];
        for (int i = 0; i < data.length; i++) {
            newData[i] = data[i];
        }
        data = newData;
    }
}

```

## Linked Lists

> 由于链表中的节点是离散分布于内存中，它们可能并不是一个挨着另一个，这就是他们必须保存下一个节点引用的原因，所以我们无法通过序号快速查找节点，我们需要从头找到尾，直到找到对应序号，所以时间复杂度为**O(n)**

Lookup

​		By Value    O(n)

​		By Index    O(n)

Insert

​		At the End  O(1)

​		At the Beginning   O(1)

​		In the Middle   O(n)  因为查找到节点是O(n)  更新节点是O(1)

DELETE

​		From the Beginning   O(1)

​		From the End.             O(n)

​		From the Middle.       O(n)

### 自己的

```java
public class Node {
  private int value;
  private Node next;

  Node() {}

  Node(int value) {
    this.value = value;
  }

  Node(Node next) {
    this.next = next;
  }

  public void setNext(Node next) {
    this.next = next;
  }

  public Node getNext() {
    return next;
  }

  public int getValue() {
    return value;
  }
}
```

```JAVA
public class LinkedList {
  private Node first;
  private Node last;

  LinkedList() {
    last = new Node();
    first = new Node(last);
  }

  public int indexOf(int value) {
    Node node = first;
    int index = 0;
    while (node.getNext() != last) {
      if (node.getValue() == value) return index;
      node = node.getNext();
      index ++;
    }
    return -1;
  }

  public boolean contains(int value) {
    Node node = first;
    while (node.getNext() != last) {
      if (node.getValue() == value) return true;
      node = node.getNext();
    }
    return false;
  }

  public void deleteFirst() {
    Node node = first.getNext();
    first.setNext(node.getNext());
    node = null;
  }

  public void deleteLast() {
    if (first.getNext() == last) return;
    Node node = first;
    while (node.getNext().getNext() != last) {
      node = node.getNext();
    }
    Node lastNode = node.getNext();
    node.setNext(last);
    lastNode = null;
  }

  public void addFirst(int value) {
    Node node = new Node(value);

    Node tempNode = first.getNext();
    first.setNext(node);
    node.setNext(tempNode);
  }

  public void addLast(int value) {
    Node node = new Node(value);

    Node loopNode = first;
    while (loopNode.getNext() != last) {
      loopNode = loopNode.getNext();
    }

    loopNode.setNext(node);
    node.setNext(last);
  }


  @Override
  public String toString() {
    Node node = first.getNext();
    StringBuilder builder = new StringBuilder("[");
    while (node != null) {
      builder.append(node.getValue());
      node = node.getNext();
      if (node.getNext() == null) break;
      builder.append(" ");
    }
    builder.append("]");
    return builder.toString();
  }
}

```

### 老师的

```java
public class LinkedListTea {

    // 由于这个内仅有自己用，不暴露给外界 所以使用private 加内部类方式
    private class Node {
        private int value;
        private Node next;

        Node(int value) {
            this.value = value;
        }
    }

    private Node first;
    private Node last;
    private int size;

    public void addLast(int element) {
        Node node = new Node(element);
        if (isEmpty()) first = last = node;
        else {
            // 这里的last 指的是没有加入当前element元素之前的最后一个
            // 把它的next指向当前element
            last.next = node;
            // 设置当前的element元素为最后一个
            last = node;
        }
        size ++;
    }

    public void addFirst(int element) {
        Node node = new Node(element);
        if (isEmpty()) first = last = node;
        else {
            // 把当前这个element元素指向之前的第一个元素
            // 之前的第一个元素就变为第二个元素了
            node.next = first;
            // 把当前元素设置为first
            first = node;
        }
        size ++;
    }

    public int indexOf(int element) {
        Node node = first;
        int index = 0;
        while (node != null) {
            if (node.value == element) return index;
            node = node.next;
            index ++;
        }
        return -1;
    }

    public void removeFirst() {
        if (isEmpty()) throw new NoSuchElementException();

        if (first == last) first = last = null;
        else {
            Node toFirstNode = first.next;
            first.next = null;
            first = toFirstNode;
        }
        size --;
    }

    public void removeLast() {
        Node previous = getPrevious(last);
        if (isEmpty()) throw new NoSuchElementException();
        if (previous == null) first = last = null;
        else {
            last = previous;
            last.next = null;
        }
        size --;
    }
    
    public int[] toArray() {
        int index = 0;
        int[] array = new int[size];
        Node current = first;
        if (current != null) {
            array[index ++] = current.value;
            current = current.next;
        }
        return array;
    }

    public boolean contains(int element) {
        return indexOf(element) != -1;
    }

    public int size() {
        return size;
    }
    

    @Override
    public String toString() {
        StringBuilder builder = new StringBuilder("[");
        Node node = first;
        while (node != null) {
            builder.append(node.value);
            if (node != last) builder.append(" ");
            node = node.next;
        }
        builder.append("]");
        return builder.toString();
    }

    private boolean isEmpty() {
        return first == null;
    }

    private Node getPrevious(Node node) {
        Node loopNode = first;
        while (loopNode != null) {
            if (loopNode.next == node) return loopNode;
            loopNode = loopNode.next;
        }
        return null;
    }
}

```

### 面试题1：反转链表

```java
// 反转链表 [10 -> 20 -> 30] => [30 -> 20 -> 10]
    public void reverse() {
        if (isEmpty()) return;

        Node previous = first;
        Node current = first.next;
        while (current != null) {
            Node next = current.next;
            current.next = previous;
            previous = current;
            current = next;
        }
        last = first;
        last.next = null;
        first = previous;
    }
```

### 面试题2：一次遍历获取倒数第k个节点

```java
// 一次遍历获取倒数第k个node
    public int getKthFromTheEnd(int k) {
        if (isEmpty()) throw new IllegalStateException();
		// 通常在面试的时候是没有size这个大小可以获取的  所以用下方循环里的判断
        // if (k > size) throw new IllegalArgumentException();
        Node previous = first;
        Node current = first;
        for (int i = 0; i < k-1; i++) {
            current = current.next;
            if (current == null) throw new IllegalArgumentException();
        }

        while (current != last) {
            previous = previous.next;
            current = current.next;
        }
        return previous.value;
    }
```

### Arrays vs Linked Lists

- Static arrays have a fixed size

- Dynamic arrays grow bu 50-100%    Vector: 100%  ArrayList: 50%  

  动态数组扩容 会浪费一些内存，因为没有使用

- Linked lists do not waste memory

  链表不浪费内存，因为保存的是实际占用，但是会消耗多的内存，因为要保存下一个节点的地址信息

- use arrays if you know the number of items to store

- new ArrayList(100)     DEFAULT_CAPACITY 10

### Double Linked List

双向链表比单向链表多消耗空间，但是在删除最后一个节点时，单向链表的时间复杂度为O(n)，而双向链表的时间复杂度为O(1)， 因为双向链表保留了last的前一个节点。

> 双向链表环形结构可用于 音乐播放器的循环播放，登录页面tab键的切换，把几个输入框放入双向链表即可

## Stacks

**可以使用数组、链表来构建栈**

**Last In First Out (LIFO)**

用数组或者链表保存栈

> 栈的所有操作时间复杂度都是 O(1)

> 栈的用途

- Implement the undo feature  

  实现程序的重做功能

- Build compilers(eg syntax checking)

  用来创建编译器

  例如：栈可以帮助编译器实现纠正表达中的语法

- Evaluate expressions(eg 1 + 2 * 3)

  实现数学表达式

  例如：创建一个计算器，允许用户输入一些需要计算的数

  然后我们可以在栈中计算它们

- Build navigation(eg forward/back)

  创建浏览器的导航功能

  例如：web浏览器中的前进和后退功能

### 面试题1：字符串反转

```java
public class StackOperation {

    // 字符串反转
    public static String reverseStr(String str) {
        if (str == null) throw new IllegalArgumentException();

        Stack<Character> stack = new Stack();
        for (char c : str.toCharArray())
            stack.push(c);

        StringBuilder builder = new StringBuilder();
        while (!stack.isEmpty())
            builder.append(stack.pop());

        return builder.toString();
    }
}
```

### 面试题2：匹配符号

```java
// 我的
public class StackOperation {
    private String str;

    StackOperation(String str) {
        this.str = str;
    }

    public String reverseStr() {
        if (str == null) throw new IllegalArgumentException();

        Stack<Character> stack = new Stack<>();
        for (char c : str.toCharArray())
            stack.push(c);

        StringBuilder builder = new StringBuilder();
        while (!stack.isEmpty())
            builder.append(stack.pop());

        return builder.toString();
    }

    public boolean isBalanced() {
        Stack<Character> stack = new Stack<>();
        Map<Character,Character> map = new HashMap<>();
        map.put('(' , ')');
        map.put('[' , ']');
        map.put('<' , '>');
        map.put('{' , '}');
        for (Character c : str.toCharArray()) {
            if (map.containsKey(c)) stack.push(c);
            if (map.containsValue(c)) {
                if (stack.empty()) return false;
                Character right = stack.pop();
                if (!c.equals(map.get(right))) return false;
            }
        }
        return stack.empty();
    }
}

```

```java
// 老师的
public class StackOperation {

    private final List<Character> leftBrackets = Arrays.asList('(', '[', '{', '<');
    private final List<Character> rightBrackets = Arrays.asList(')', ']', '}', '>');

    private String str;

    StackOperation(String str) {
        this.str = str;
    }

    public boolean isBalanced() {
        Stack<Character> stack = new Stack<>();

        for (char c : str.toCharArray()) {
            if (isLeftBracket(c)) stack.push(c);
            if (isRightBracket(c)) {
                if (stack.empty()) return false;

                Character left = stack.pop();
                if (!bracketsMatch(left, c)) return false;
            }
        }

        return stack.empty();
    }

    private boolean isLeftBracket(char ch) {
        // 数组leftBrackets 不要在方法中声明，否则每次调用方法都会分配内存空间
        // 应该放到类级别上声明
        return leftBrackets.contains(ch);
    }

    private boolean isRightBracket(char ch) {
        return rightBrackets.contains(ch);
    }

    private boolean bracketsMatch(char left, char right) {
        return leftBrackets.indexOf(left) == rightBrackets.indexOf(right);
    }
}

```

### 数组实现Stack

```java
public class Stack {

    private int size;

    private final int capacity = 10;

    private int[] array;

    Stack() {
        array = new int[capacity];
    }

    Stack(int capacity) {
        array = new int[capacity];
    }

    public void push(int element) {
        // 当添加元素超过当前数组大小时，可以进行扩容或报异常处理
        // expendArray();
        if (array.length == size)
            throw new StackOverflowError();
        array[size ++] = element;
    }

    public int pop() {
        if (size == 0)
            throw new EmptyStackException();
        return array[--size];
    }

    public int peek() {
        if (size == 0)
            throw new EmptyStackException();
        return array[size - 1];
    }

    public boolean isEmpty() {
        return size == 0;
    }

    public void expendArray() {
        if (array.length != size) return;
        array = Arrays.copyOf(array, size * 2);
    }

    @Override
    public String toString() {
        /*StringBuffer buffer = new StringBuffer("[");
        for (int i = 0; i < size; i++) {
            buffer.append(array[i]);
            if (i != size -1)
                buffer.append(" ");
        }
        buffer.append("]");
        return buffer.toString();*/
        return Arrays.toString(Arrays.copyOfRange(array, 0, size));
    }
}

```

## Queues

**可以使用数组、链表、栈来构建队列**

java中的实现ArrayDeque

**First In First Out (FIFO)**

- enqueue    O(1)
- dequeue    O(1)
- peek           O(1)
- isEmpty      O(1)
- isFull           O(1)

### 反转队列

```JAVA
public static Queue reverse(Queue<Integer> queue) {
        Stack<Integer> stack = new Stack<>();

        while (!queue.isEmpty())
            stack.push(queue.remove());

        while (!stack.empty())
            queue.add(stack.pop());

        return queue;
}
```

### 用数组实现环形队列

```java
public class ArrayQueueForArray {

    private int first;
    private int last;

    private static final int CAPACITY = 5;
    private int size;
    private int[] array;

    ArrayQueueForArray() {
        array = new int[CAPACITY];
    }

    public void enqueue(int element) {
        if (isFull()) throw new RuntimeException("队列已满");

        array[last] = element;
        last = (last + 1) % CAPACITY;
        size ++;
    }

    public int dequeue() {
        if (isEmpty()) throw new RuntimeException("队列为空，没有可出队的元素");

        int current = array[first];
        array[first] = 0;
        first = (first + 1) % CAPACITY;
        size --;
        return current;
    }

    public int peek() {
        if (isEmpty()) throw new RuntimeException("队列为空，没有可出队的元素");

        return array[first];
    }

    public boolean isFull() {
        return array.length == size;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    @Override
    public String toString() {
        return Arrays.toString(array);
    }
}
```

### 用栈实现队列

```java
public class ArrayQueueForStack {

    private Stack<Integer> input;
    private Stack<Integer> output;

    ArrayQueueForStack() {
        input = new Stack<>();
        output = new Stack<>();
    }

    // O(1)
    public void enqueue(int element) {
        input.push(element);
    }

    // O(n)
    public int dequeue() {
        if (isEmpty())
            throw  new IllegalStateException();

        if (output.isEmpty())
            while (! input.isEmpty())
                output.push(input.pop());

        return output.pop();
    }

    public boolean isEmpty() {
        return input.empty() && output.empty();
    }

}
```

### 优先级队列

**优先级队列整形默认从小到大存放元素，即使添加的时候是乱序的**

Java中的实现有PriorityQueue

可以使用 Arrays 和 Heap堆来实现

#### 使用数组实现

```java
package com.pd.queues;

/**
 * @description:
 * @author: pd
 * @date: 2020-08-13 11:01 下午
 */
public class PriorityQueue {

  private int[] array;
  private int size;

  PriorityQueue() {
    array = new int[5];
  }

  // 我的，很粗糙，难读
  public void insert(int element) {
    if (isFull())
      throw new RuntimeException("队列已满");
    if (isEmpty()) {
      array[size++] = element;
      return;
    }

    for (int i = size - 1; i >= 0; i--) {
      if (array[i] >= element) {
        array[i + 1] = array[i];

        if (i == 0) {
          array[0] = element;
        }
      }
      else {
        array[i + 1] = element;
        break;
      }
    }
    size ++;
  }

  public int remove() {
    if (isEmpty())
      throw new RuntimeException("队列为空");
    int result = array[0];
    for (int i = 0; i < size - 1; i++) {
      array[i] = array[i + 1];
    }
    array[--size] = 0;
    return result;
  }

  public boolean isEmpty() {
    return size == 0;
  }

  public boolean isFull() {
    return size == array.length;
  }

  // 老师的
  public void add(int item) {
    if (isFull())
      throw new RuntimeException("队列已满");

    int i = shiftItemsToInsert(item);
    array[i] = item;
    size ++;
  }

  private int shiftItemsToInsert(int item) {
    int i;
    for (i = size -1; i >= 0; i--) {
      if (array[i] > item)
        array[i + 1] = array[i];
      else
        break;
    }
    return i + 1;
  }
}
```

## Hash Tables

Lookup   O(1)

Insert      O(1)

Delete     O(1)

> 面试题1：在一个字符串中找到第一个不重复的字符

```java
public static Character getFirstNonRepeated(String str) {
        Map<Character, Integer> map = new LinkedHashMap<>();
        char[] chars = str.toCharArray();
        for (char c : chars)
            map.merge(c, 1, Integer::sum);

        // 使用这个必须要使用linkedHashMap
//        return map.entrySet().stream().filter(m -> m.getValue() == 1).map(Entry::getKey).findFirst().orElse(Character.MIN_VALUE);

    // 可以使用HashMap 乱序  上面的return 和下面这三行等同
        for (char c : chars)
            if (map.get(c) == 1) return c;

        return Character.MIN_VALUE;

    }
```

> 面试题2: 在一个字符串中找到第一个重复的字符

```java
public static Character findFirstRepeatedChar(String str) {
        Set<Character> set = new HashSet<>();

        for (char c : str.toCharArray()) {
            if (set.contains(c))
                return c;

            set.add(c);
        }
        return Character.MIN_VALUE;
}
```

> hash 的一种简单实现

```java
// hash的一种简单实现方式
    public static int hash(String str) {
        int hash = 0;
        
        for (char ch : str.toCharArray())
            hash += ch;
        
        // 100 表示hash内部数组大小
        return hash % 100;
    }
```

需要回看  整理笔记  碰撞的几种解决方式、 实现hash table的方式