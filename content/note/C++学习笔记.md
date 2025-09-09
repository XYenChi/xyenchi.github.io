---
title: "C++学习笔记"
draft: false
---

最近开始学C++，做一下记录。

#### User-defined conversion function 自定义类型转换函数
Enables implicit convension or explicit conversion from a class type to another type.
给类类型提供显示或隐式转换。

隐式转换 implicit conversion
`operator target_type() const;` 在 RIF 中的用法 `operator int8_t() const { return datum.i8; }`

1.可以对整型进行隐式转换，指针进行显式转换。
2.使用 explicit 和 static_cast<> 进行显式转换。

cpp reference 举的例子：
```cpp
struct X{
	// implicit conversion
	operator int() const { return 7; }

	// explicit conversion
	explicit operator int*() const { return nullptr; }

	// Error: array operator not allowed in conversion-type-id
	// operator int(*)[3]() const { return nullptr; }

	using arr_t = int[3];
	operator arr_t*() const { return nullptr; } // OK if done through typedef
	// operator arr_t () const; // Error: conversion to array not allowed in any case
};

int main(){
	X x;

	int n = static_cast<int>(x); // OK: sets n to 7
	int m = x;                   // OK: sets m to 7

	int* p = static_cast<int*>(x); // OK: sets p to null
	//int* q = x;                  // Error: no implicit conversion

	int (*pa)[3] = x; // OK
}
```