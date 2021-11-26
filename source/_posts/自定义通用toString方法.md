---
title: 自定义通用toString方法
date: 2021-07-20 23:17:40
tags: 反射
---

## 自定义toString

```java
import java.lang.reflect.AccessibleObject;
import java.lang.reflect.Array;
import java.lang.reflect.Field;
import java.lang.reflect.Modifier;
import java.util.ArrayList;

/**
 * @Description: 通用toString
 * @author: yts
 * @date: 2021年07月20日 22:31
 */
public class ObjectAnalyzer {

    public ArrayList<Object> visited = new ArrayList<>();

    public String toString(Object obj) throws IllegalAccessException {
        if (obj == null) {
            return "null";
        }
        if (visited.contains(obj)) {
            return "....";
        }
        visited.add(obj);
        Class<?> cl = obj.getClass();

        if (cl == String.class) {
            return (String) obj;
        }

        if (cl.isArray()) {
            String r = cl.getComponentType() + "[]{";
            for (int i = 0; i < Array.getLength(obj); i++) {
                if (i > 0) {
                    r += ",";
                }
                Object val = Array.get(obj, i);
                //是否为基本类型
                if (cl.getComponentType().isPrimitive()) {
                    r += val;
                }else {
                    r += toString(val);
                }
            }
            return r += "}";
        }
        String r = cl.getName();
        do {
            r += "[";
            Field[] fields = cl.getDeclaredFields();
            AccessibleObject.setAccessible(fields, true);
            for (Field field : fields) {
                if (!Modifier.isStatic(field.getModifiers())) {
                    if (!r.endsWith("[")) {
                        r += ",";
                    }
                    r += field.getName() + "=";
                    Class<?> t = field.getType();
                    Object val = field.get(obj);
                    if (t.isPrimitive()) {
                        r += val;
                    }else {
                        r += toString(val);
                    }
                }
            }
            r += "]";
            cl = cl.getSuperclass();
        }
        while (cl != null);
        return r;
    }
}

```

```java
	import java.util.ArrayList;

/**
 * @Description: TODO
 * @author: yts
 * @date: 2021年07月20日 22:28
 */
public class ObjectAnalyzerTest {

    public static void main(String[] args) throws IllegalAccessException {
        ArrayList<Object> list = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            list.add(i);
        }
        ObjectAnalyzer objectAnalyzer = new ObjectAnalyzer();
        System.out.println(objectAnalyzer.toString(list));
    }
}

```

