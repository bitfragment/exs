---
title: "Sum values"
date: 2018-03-05
tags: c
---

Source: [CodeAbbey](https://www.codeabbey.com/) problem 1: [Sum "A+B"](https://www.codeabbey.com/index/task_view/sum-of-two)

```c
#include <stdio.h>
#include <string.h>
#include "../lib/unity/src/unity.h" // throwtheswitch.org


int sum(const int a, const int b) {
    return a + b;
}
```


## Tests

```c
void test_sum(void) {
    TEST_ASSERT_EQUAL_INT(1, sum(0, 1));
    TEST_ASSERT_EQUAL_INT(0, sum(-1, 1));
    TEST_ASSERT_EQUAL_INT(3, sum(1, 2));
}

int run_tests(void) {
    UNITY_BEGIN();
    RUN_TEST(test_sum);
    return UNITY_END();
}
```


## Interaction

```c
int main(int argc, char *argv[]) {
    run_tests();
    
    if (argc > 1 && !strcmp(argv[1], "-i")) {
        int a ,b;
        printf("Enter two integer values separated by a space. "
               "For example: 3 1\n"
               "> ");

        while (1) {
            scanf("%d %d", &a, &b);
            printf("%d\n> ", sum(a, b));
        }
    }

}
```

```md
Enter two integer values separated by a space. For example: 3 1
> -1 0
-1
> -1 1
0
> 4 5
9
> -999 1000
1
```
