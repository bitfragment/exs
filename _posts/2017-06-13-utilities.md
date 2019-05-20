---
title: "Utilities"
date: 2017-06-13
tags: java
---

```java
public class Utilities {
```

## count()

Count non-overlapping instances of `b` in `a`.

```java
java> count("ababababa", "aba")
java.lang.Integer res0 = 2
```

---

```java
    public static int count(String a, String b) {
        int count = 0;
        int i = 0;
        int j = a.indexOf(b, i);
        while (j >= 0) {
            count++;
            j = a.indexOf(b, j + b.length());
        }
        return count;
    }
```

## isVowel()

```java
java> isVowel('a')
java.lang.Boolean res0 = true
java> isVowel('B')
java.lang.Boolean res1 = false
```

---

```java
    public static boolean isVowel(char ch) {
        String chstr = String.valueOf(Character.toLowerCase(ch));
        return "aeiou".indexOf(chstr) != -1;
    }
```

## countVowels()

```java
java> vowelCount("foobar")
java.lang.Integer res2 = 3
```

---

```java
    public static int vowelCount(String str) {
        int vowelsCount = 0;
        for (int i = 0, l = str.length(); i < l; i++) {
            if (isVowel(str.charAt(i))) vowelsCount++;
        }
        return vowelsCount;
    }
```

## replaceVowels()

```java
java> replaceVowels("foobar", 'x')
java.lang.String res3 = "fxxbxr"
```

---

```java
    public static String replaceVowels(String phrase, char ch) {
        StringBuilder result = new StringBuilder(phrase);
        for (int i = 0, l = result.length(); i < l; i++) {
            if (isVowel(result.charAt(i))) {
                result.setCharAt(i, ch);
            }
        }
        return result.toString();
    }
```

---
```java
} // Utilities
```
