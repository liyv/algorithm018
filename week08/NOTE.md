# 初级排序

### 选择排序

```java
public void selectSort(int[] array) {
        int minIndex;
        for (int i = 0; i < array.length; i++) {
            minIndex = i;
            for (int j = i + 1; j < array.length; j++) {
                if (array[j] < array[minIndex]) {
                    minIndex = j;
                }
            }
            int temp = array[i];
            array[i] = array[minIndex];
            array[minIndex] = temp;
        }
}
```

### 插入排序

```java
 public void insertSort(int[] array) {
        for (int i = 1; i < array.length; i++) {
            int t = array[i];
            int j = i - 1;
            for (; j >= 0; j--) {
                if (t < array[j]) {
                    array[j + 1] = array[j];
                } else {
                    break;
                }
            }
            array[j + 1] = t;
        }
    }
```

### 冒泡排序

```java
public void bubbleSort(int[] array) {
        for (int i = array.length - 1; i >= 0; i--) {
            boolean flag = false;//发否发生交换，没有交换，提前退出外层循环
            for (int j = 1; j <= i; j++) {
                if (array[j - 1] > array[j]) {
                    int temp = array[j - 1];
                    array[j - 1] = array[j];
                    array[j] = temp;
                    flag = true;
                }
            }
            if (!flag) break;
        }
    }
```



