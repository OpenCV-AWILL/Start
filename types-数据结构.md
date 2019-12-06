[TOC]

> 乱序

# Complex

> 类似于`std::complex`，但是它通过字段访问（std使用方法访问）。

* 模板类

```c++
template<typename _Tp> class Complex
{
public:
    //! default constructor
    Complex();
    Complex(_Tp _re, _Tp _im = 0);
    
    //! conversion to another data type
    template<typename T2> operator Complex<T2>() const;
    //! conjugation
    Complex conj() const;
    
    _Tp re, im;	//< the real and the imaginary parts
};
```

* 特化

```c++
typedef Complex<float> Complexf;
typedef Complex<double> Complexd;
```

* 实现

```c++
template<typename _Tp> inline
Complex<_Tp>::Complex() : re(0), im(0)
{
    //
}
```

```c++
template<typename _Tp> inline
Complex<_Tp>::Complex(_Tp _re, _Tp _im) : re(_re), im(_im)
{
    //
}
```

==注意这个模板类的模板成员函数的语法==

*saturate_cast暂时不管*

```c++
template<typename _Tp> template<typename T2> inline
Complex<_Tp>::operator Complex<T2>() const
{
    return Complex<T2>(saturate_cast<T2>(re), saturate_cast<T2>(im));
}
```

**运算符重载**

> 注意：这是一个==完整的==运算符重载组！

```c++
template<typename _Tp> static inline
bool operator == (const Complex<_Tp>& a, const Complex<_Tp>& b)
{
    return a.re == b.re && a.im == b.im;
}
```

```c++
template<typename _Tp> static inline
bool operator != (const Complex<_Tp>& a, const Complex<_Tp>& b)
{
    return a.re != b.re || a.im != b.im;
}
```

```c++
template<typename _Tp> static inline
Complex<_Tp> operator + (const Complex<_Tp>& a, const Complex<_Tp>& b)
{
    return Complex<_Tp>(a.re + b.re, a.im + b.im);
}
```

```c++
template<typename _Tp> static inline
Complex<_Tp>& operator += (Complex<_Tp>& a, const Complex<_Tp>& b)
{
    a.re += b.re;
    a.im += b.im;
    return a;
}
```

```c++
template<typename _Tp> static inline
Complex<_Tp> operator - (const Complex<_Tp>& a, const Complex<_Tp>& b)
{
    return Complex<_Tp>(a.re - b.re, a.im - b.im);
}
```

```c++
template<typename _Tp> static inline
Complex<_Tp>& operator -= (Complex<_Tp>& a, const Complex<_Tp>& b)
{
    a.re -= b.re;
    a.im -= b.im;
    return a;
}
```

```c++
template<typename _Tp> static inline
Complex<_Tp> operator - (const Complex<_Tp>& a)
{
    return Complex<_Tp>(-a.re, -a.im);
}
```

```c++
template<typename _Tp> static inline
Complex<_Tp> operator * (const Complex<_Tp>& a, const Complex<_Tp>& b)
{
    return Complex<_Tp>(a.re * b.re - a.im * b.im, a.re * b.im + a.im * b.re);
}
```

考虑实数与虚数的之间的运算

```c++
template<typename _Tp> static inline
Complex<_Tp> operator * (const Complex<_Tp>& a, _Tp b)
{
    return Complex<_Tp>(a.re * b, a.im * b);
}
```

```c++
template<typename _Tp> static inline
Complex<_Tp> operator * (_Tp b, const Complex<_Tp>& a)
{
    return Complex<_Tp>(a.re * b, a.im * b);
}
```

```c++
template<typename _Tp> static inline
Complex<_Tp> operator + (const Complex<_Tp>& a, _Tp b)
{
    return Complex<_Tp>(a.re + b, a.im);
}
```

```c++
template<typename _Tp> static inline
Complex<_Tp> operator - (const Complex<_Tp>& a, _Tp b)
{
    return Complex<_Tp>(a.re - b, a.im);
}
```

```c++
template<typename _Tp> static inline
Complex<_Tp> operator + (_Tp b, const Complex<_Tp>& a)
{
    return Complex<_Tp>(a.re + b, a.im);
}
```

```c++
template<typename _Tp> static inline
Complex<_Tp> operator - (_Tp b, const Complex<_TP>& a)
{
    return Complex<_Tp>(a.re - b, a.im);
}
```

==复合运算符的返回值是**引用类型**==

```c++
template<typename _Tp> static inline
Complex<_Tp>& operator += (Complex<_Tp>& a, _Tp b)
{
    a.re += b;
    return a;
}
```

```c+
template<typename _Tp> static inline
Complex<_Tp>& operator -= (Complex<_Tp>& a, _Tp b)
{
	a.re -= b;
	return a;
}
```

```c++
template<typename _Tp> static inline
Complex<_Tp>& operator *= (Complex<_Tp>& a, _Tp b)
{
    a.re *= b;
    a.im *= b;
    return a;
}
```

绝对值和除法

```c++
template<typename _Tp> static inline
double abs(const Complex<_Tp>& a)
{
    return std::sqrt((double)a.re * a.re + (double)a.im * a.im);
}
```

==注意这里的细节：处理了类型转换==

```c++
template<typename _Tp> static inline
Complex<_Tp> operator / (const Complex<_Tp>& a, const Complex<_Tp>& b)
{
    double t = 1. / ((double)b.re * b.re + (double)b.im * b.im);
    return Complex<_Tp>((_Tp)((a.re * b.re + a.im * b.im) * t), (_Tp)((-a.re * b.im + a.im * b.re) * t));
}
```

*写到后面他可能觉得麻烦了么？前面写着都不用自己的重载呢。*

```c++
template<typename _Tp> static inline
Complex<_Tp>& operator /= (Complex<_Tp>& a, const Complex<_Tp>& b)
{
    a = a / b;	// 使用了运算符重载
    return a;
}
```

```c++
template<typename _Tp> static inline
Complex<_Tp> operator / (const Complex<_Tp>& a, _Tp b)
{
    _Tp t = (_Tp)1/b;
    return Complex<_Tp>(a.re * t, a.im * t);
}
```

```c++
template<typename _Tp> static inline
Complex<_Tp> operator / (_Tp b, const Complex<_Tp>& a)
{
    return Complex<_Tp>(b)/a;
}
```

```c++
template<typename _Tp> static inline
Complex<_Tp> operator /= (const Complex<_Tp>& a, _Tp b)
{
    _Tp t = (_Tp)1/b;
    a.re *= t;
    a.im *= t;
    return a;
}
```

# Point_



# Rect_

> 1. 使用以下几个参数进行描述：
>     `x`和`y`默认为左上角的坐标。然而，有时我们会用到自底向上的情况，它的x和y表示左下角的坐标。
>     `width`和`height`表示宽和高。
>
> 2. 一般认为包含`左上边界`，不包含`右下边界`。
>     因此一般遍历一个矩形roi会这样做：
>
>     ```c++
>     for(int y = roi.y; y < roi.y + roi.height; ++y)
>     {
>         for(int x = roi.x; x < roi.x + roi.width; ++x)
>         {
>             // ...
>         }
>     }
>     ```
>
> 3. 实现了以下操作：
>
>     ```c++
>     rect += point, rect -= point, rect += size, rect -= size;
>     rect = rect1 & rect2;	// 交集
>     rect = rect1 | rect2;	// 并集
>     rect &= rect1, rect |= rect1;
>     rect == rect1, rect != rect1;	// 比较
>     ```
>
>     

* 模板类

```c++
template<typename _Tp> class Rect_
{
    typedef _Tp value_type;
    
    //! default constructor
    Rect_();
    Rect_(_Tp _x, _Tp _y, _Tp _width, _Tp _height);
    Rect_(const Rect_& r);
    Rect_(Rect_&& r) CV_NOEXCEPT;
    Rect_(const Point_<_Tp>& org, const Size_<_Tp>& sz);
    Rect_(const Point_<_Tp>& pt1, const Point_<_Tp>& pt2);
    
    Rect_& operator = (const Rect_& r);
    Rect_& operator = (Rect_&& r) CV_NOEXCEPT;
    //! the top-left corner
    Point_<_Tp> tl() const;
    //! the bottom-right corner
    Point_<_Tp> br() const;
    
    //! size (width, height) of the rectangle
    Size_<_Tp> size() const;
    //! area (width*height) of the rectangle
    _Tp area() const;
    //! true if empty
    bool empty() const;
    
    //! conversion to another data type
    template<typename _Tp2> operator Rect_<_Tp2>() const;
    
    //! checks whether the rectangle contains the point
    bool contains(const Point_<_Tp>& pt) const;
    
    _Tp x;	//!< x coordinate of the top-left corner
    _Tp y;	//!< y coordinate of the top-left corner
    _Tp width;	//!< width of the rectangle
    _Tp height;	//!< height of the rectangle
};
```

* 特化

```c++
typedef Rect_<int> Rect2i;
typedef Rect_<float> Rect2f;
typedef Rect_<double> Rect2d;
typedef Rect2i Rect;
```

