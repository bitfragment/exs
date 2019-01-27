---
title: "Numeronym"
date: 2019-01-26
tags: c
---

* TOC
{:toc}

## Source

[Problem - 71A - Codeforces](https://codeforces.com/problemset/problem/71/A)

## My solution

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include "../lib/unity/src/unity.h" // throwtheswitch.org


char * solution(char * str, int str_len, char * abbrev) {
    if (str_len <= 10) {
        strcpy(abbrev, str);
        return abbrev;
    };

    // Maximum length of the input string is 100 characters.
    // Get the length of the input string, minus its first and
    // last characters, and convert the number representing that
    // length to a string.
    char buf[3];
    snprintf(buf, 3, "%d", str_len - 2);
    int buf_len = strlen(buf);

    // Build the abbreviation.
    abbrev[0] = str[0];
    for (int i = 0; i <= buf_len; i++) {
        abbrev[i + 1] = buf[i];
    }
    abbrev[buf_len + 1] = str[str_len - 1];
    abbrev[buf_len + 2] = '\0';
    
    return abbrev;
}


void test_mytest() {
    char abbrev[5];
    TEST_ASSERT_EQUAL_STRING("", solution("", 0, abbrev));
    TEST_ASSERT_EQUAL_STRING("a", solution("a", 1, abbrev));
    TEST_ASSERT_EQUAL_STRING("abcdefghij", solution("abcdefghij", 10, abbrev));
}


void test_provided() {
    char abbrev[5];
    TEST_ASSERT_EQUAL_STRING("word", solution("word", 4, abbrev));
    TEST_ASSERT_EQUAL_STRING("l10n", solution("localization", 12, abbrev));
    TEST_ASSERT_EQUAL_STRING("i18n", solution("internationalization", 20,  abbrev));
    TEST_ASSERT_EQUAL_STRING("p43s", solution("pneumonoultramicroscopicsilicovolcanoconiosis", 45, abbrev));
}


int run_tests(void) {
    UNITY_BEGIN();
    RUN_TEST(test_mytest);
    RUN_TEST(test_provided);
    return UNITY_END();
}


int main(void) {
    run_tests();
    
    int LINEMAX = 100;
    
    // Max line length in OLJ input = 100.
    char buf[LINEMAX];
    
    // Maximum abbreviation of line length in input = "a98z".
    char abbrev[5];

    // First line of OLJ input contains number of test cases.
    int test_cases = atoi(fgets(buf, LINEMAX, stdin));

    while (test_cases--) {
        if (fgets(buf, LINEMAX, stdin) != NULL) {
            size_t buf_len = strlen(buf);
            if (buf_len > 0 && buf[buf_len - 1] == '\n') {
                buf[--buf_len] = '\0';
            }
        solution(buf, buf_len, abbrev);
        printf("%s\n", abbrev);
        }
    }

    return 0;
}
```
