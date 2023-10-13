基于jdk21 ffm api的jni封装。


以下示例C++文件，定义打印菲波那切函数和输出最大Point函数

```c
// MathLibrary.h - Contains declarations of math functions
#pragma once

#ifdef MATHLIBRARY_EXPORTS
#define MATHLIBRARY_API __declspec(dllexport)
#else
#define MATHLIBRARY_API __declspec(dllimport)
#endif

// The Fibonacci recurrence relation describes a sequence F
// where F(n) is { n = 0, a
//               { n = 1, b
//               { n > 1, F(n-2) + F(n-1)
// for some initial integral values a and b.
// If the sequence is initialized F(0) = 1, F(1) = 1,
// then this relation produces the well-known Fibonacci
// sequence: 1, 1, 2, 3, 5, 8, 13, 21, 34, ...

// Initialize a Fibonacci relation sequence
// such that F(0) = a, F(1) = b.
// This function must be called before any other function.
extern "C" MATHLIBRARY_API void fibonacci_init(
    const unsigned long long a, const unsigned long long b);

// Produce the next value in the sequence.
// Returns true on success and updates current value and index;
// false on overflow, leaves current value and index unchanged.
extern "C" MATHLIBRARY_API bool fibonacci_next();

// Get the current value in the sequence.
extern "C" MATHLIBRARY_API unsigned long long fibonacci_current();

// Get the position of the current value in the sequence.
extern "C" MATHLIBRARY_API unsigned fibonacci_index();


struct Point {
    int x;
    int y;
};

extern "C" MATHLIBRARY_API Point test_point(Point points[], long count);
```


```c++
// MathLibrary.cpp : Defines the exported functions for the DLL.
#include "pch.h" // use stdafx.h in Visual Studio 2017 and earlier
#include <utility>
#include <limits.h>
#include "MathLibrary.h"
#include <iostream> // Include for using std::cout

// DLL internal state variables:
static unsigned long long previous_;  // Previous value, if any
static unsigned long long current_;   // Current sequence value
static unsigned index_;               // Current seq. position

// Initialize a Fibonacci relation sequence
// such that F(0) = a, F(1) = b.
// This function must be called before any other function.
void fibonacci_init(
    const unsigned long long a,
    const unsigned long long b)
{
    index_ = 0;
    current_ = a;
    previous_ = b; // see special case when initialized
}

// Produce the next value in the sequence.
// Returns true on success, false on overflow.
bool fibonacci_next()
{
    // check to see if we'd overflow result or position
    if ((ULLONG_MAX - previous_ < current_) ||
        (UINT_MAX == index_))
    {
        return false;
    }

    // Special case when index == 0, just return b value
    if (index_ > 0)
    {
        // otherwise, calculate next sequence value
        previous_ += current_;
    }
    std::swap(current_, previous_);
    ++index_;
    return true;
}

// Get the current value in the sequence.
unsigned long long fibonacci_current()
{
    return current_;
}

// Get the current index position in the sequence.
unsigned fibonacci_index()
{
    return index_;
}

Point test_point(Point points[],long count)
{
    if (count <= 0) {
        // Return a default Point with x and y set to 0
        Point defaultPoint = { 0, 0 };
        return defaultPoint;
    }

    Point maxPoint = points[0];
    int maxSum = maxPoint.x + maxPoint.y;

    for (int i = 1; i < count; ++i) {
        int currentSum = points[i].x + points[i].y;
        if (currentSum > maxSum) {
            maxSum = currentSum;
            maxPoint = points[i];
        }
    }

    int x = maxPoint.x;
    int y = maxPoint.y;
    std::cout << "x = " << x << ", y = " << y << std::endl;
    return maxPoint;
}

```



与jna使用相似，继承Library定义接口，使用Native.load加载dll

```java
public record Point(int x,int y) {

}
```
```java
import org.example.jna.Library;
import org.example.jna.Native;
import java.util.List;

public interface MathLibrary extends Library {

    MathLibrary DLL = Native.load("src\\main\\resources\\MathLibrary.dll", MathLibrary.class);

    Long fibonacci_init(Long a, Long b);

    Boolean fibonacci_next();

    Long fibonacci_current();

    Long fibonacci_index();

    Point test_point(Point[] point, long count);
}
```

```java
public class TestNative {
    public static void main(String[] args) {
        // 菲波那切打印
        Long l = MathLibrary.DLL.fibonacci_init(1L, 1L);
        do {
            System.out.println(MathLibrary.DLL.fibonacci_index() + ": " + MathLibrary.DLL.fibonacci_current());
        } while (MathLibrary.DLL.fibonacci_next());

        // 筛选x+y最大值Point
        Point[] pointArray = new Point[10];
        for (int j = 0; j < pointArray.length; j++) {
            pointArray[j] = new Point(j, j);
        }
        Point point = MathLibrary.DLL.test_point(pointArray, pointArray.length);
        System.out.println(point);
    }
}
```