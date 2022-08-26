+++ 
draft = false
date = 2022-08-26T16:28:58+07:00
title = "Monads Design Pattern"
description = "This article will explain about monads design pattern and why it is useful in programming."
slug = "monads-design-pattern"
authors = ["Dimas Arna"]
tags = ["monads", "functional-programming", "software-engineer", "design-pattern"]
categories = ["design-pattern"]
externalLink = ""
series = []
+++

## Introduction

Monads is important concept in programming and mathematics. But in this article, I want to spesifically explain about monads as the design pattern.

Monads are the ingredient needed to bridge the gap between mathematical (pure) functions (such as + or exponential) and program functions (such as malloc or exception throwing). Roughly:

```
program function = mathematical function + computational monad
```

By solving this equation we obtain

```
computational monad = program function - mathematical function
```

The difference between program functions and mathematical functions is that program functions may have effects, which is whatever they may do meanwhile, like throw exception, write the state, run a random number generator. Monads are thus formal devices to capture effects.

> Note: In this article, I will write some example with **C++** language.

## Monads Components

Monads has 3 components:
1. Wrapper Type
2. Wrap Function
3. Run Function

Let see some example from common monads which is `Writer` and `Option`.

## Writer Monads

Suppose we want to write program function that run mathematical operation and write it to the logs.

First, we need the wrapper type like this:

```cpp
typedef struct number_with_logs {
    int value;
    vector<string> logs;
} number_with_logs_t;
```

Secondly, we need the wrap function to transform the raw type (or primitive type) into wrapper type.
We can write it like this:

```cpp
number_with_logs_t wrap_number_with_logs(int number) {
    vector<string> empty_string_array;
    number_with_logs_t new_type = {
        .value = number,
        .logs = empty_string_array
    };
    return new_type;
}
```

And for the last components we need some run function, which is a function that abstract away the process of transform the monad and write to the log.

```cpp
number_with_logs_t run_with_logs(number_with_logs_t x,
                                function<number_with_logs_t(int)> transform) {
    number_with_logs_t new_data = transform(x.value);
    x.logs.push_back(new_data.logs.back());
    number_with_logs_t return_data = {
        .value = new_data.value,
        .logs = x.logs
    };
    return return_data;
}
```

Now we can just write any transform function that do the job for mathematical operation and write the result to the log. As long as the return type is the wrapper type, we will fine with that.

For example, I write a transform function to square the number and return the squared number along with the logs array.

```cpp
number_with_logs_t square(int x) {
    char log_buffer[100];
    int return_value = x * x;
    vector<string> return_logs;
    sprintf(log_buffer, "Squared %d to get %d.", x, return_value);
    string log = log_buffer;
    return_logs.push_back(log);
    number_with_logs_t return_data = {
        .value = return_value,
        .logs = return_logs
    };
    return return_data;
}
```

And also this function to add one to the number and return the result along with the logs array.

```cpp
number_with_logs_t add_one(int x) {
    char log_buffer[100];
    int return_value = x + 1;
    vector<string> return_logs;
    sprintf(log_buffer, "Added 1 to %d to get %d.", x, return_value);
    string log = log_buffer;
    return_logs.push_back(log);
    number_with_logs_t return_data = {
        .value = return_value,
        .logs = return_logs
    };
    return return_data;
}
```

Now in practice if we want to chain the operation we can just call the run function, give it the variable of wrapper type and also the transform function, then leave a monad do the job behind the scene for writing to the logs.

```cpp
number_with_logs_t a = wrap_number_with_logs(5);
number_with_logs_t b = run_with_logs(a, add_one);
number_with_logs_t result = run_with_logs(b, square);
cout << "Last value: " << result.value << endl;
while(result.logs.size() > 0) {
    cout << result.logs.back() << endl;
    result.logs.pop_back();
}
```

The result is:

```
Last value: 36
Squared 6 to get 36.
Added 1 to 5 to get 6.
```

## Option Monads

`Option` is data structure (or data type) that can contain something or nothing. So we can implement as struct in **C/C++**.

```cpp
typedef struct Option {
    bool is_none;
    void * data;
} Option;
```

Now, for wrap function we need to write two function for two possible "type" which is `Some` or `None`.

```cpp
Option Some(void * a) {
    Option result = {
        .is_none = false,
        .data = a
    };
    return result;
};

Option None() {
    Option result = {
        .is_none = true,
        .data = NULL
    };
    return result;
};
```

Then the run function is simply doing checking before passing the `Option` to the transform function, if the `Option` is contain nothing or `None` then simply return `None`.

```cpp
Option run(Option x, function<Option(int)> transform) {
    if (x.is_none) return None();
    else {
        int * value = (int *) x.data;
        Option result = transform(*value);
        return result;
    }
}
```

The rest is just up to you to create transform function, here I write two functions to do different mathematical operation first to square the number and second to add one to the number. For complement, I also add another function to print the number.

```cpp
Option square(int x) {
    int * value = (int *) malloc(sizeof(int));
    *value = x * x;
    Option result = Some(value);
    return result;
}

Option add_one(int x) {
    int * value = (int *) malloc(sizeof(int));
    *value = x + 1;
    Option result = Some(value);
    return result;
}

Option print(int x) {
    int * value = (int *) malloc(sizeof(int));
    *value = x;
    Option result = Some(value);
    cout << x << endl;
    return result;
}
```

As the result, we can chain together operations like this:

```cpp
int * x = (int *) malloc(sizeof(int));
*x = 5;
Option a = Some(x);
a = run(a, square);
a = run(a, add_one);
a = run(a, print);
```

Console result:

```
26
```

## Summary

Monads are a design pattern that allows a user to chain operations while the monad manages secret work behind the scenes.

Monads has 3 components:
1. Wrapper Type
2. Wrap Function
3. Run Function

References:
1. [Monad-based Programming](https://www8.cs.fau.de/monad-based-programming)
2. [The Absolute Best Intro to Monads For Software Engineers](https://youtu.be/C2w45qRc3aU)