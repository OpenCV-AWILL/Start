[TOC]

> 1. `value_type`的实现细节暂时先不考虑。

# Complex

> 类似于`std::complex`，但是它通过字段访问（std使用方法访问）。

* 模板类

*注意`类型转换`的==模板函数==怎么写！*

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

```c++
template<typename _Tp> inline
Complex<_Tp> Complex<_Tp>::conj() const
{
    return Complex<_Tp>(re, -im);
}
```

**运算符重载**

> 注意：
>
> * 这些重载的运算符函数都==**不是**成员函数==！
>
> * 这是一个==完整的==运算符重载组！

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

> `2D点`类，同样做了一整套运算符重载。

* 模板类

*注意区分`拷贝构造`与`运算符重载`！*

```c++
template<typename _Tp> class Point_
{
public:
    typedef _Tp value_type;
    
    //! default constructor
    Point_();
    Point_(_Tp _x, _Tp _y);
    Point_(const Point_& pt);
    Point_(Point_&& pt) CV_NOEXCEPT;
    Point_(const Size_<_Tp>& sz);
    Point_(const Vec<_Tp, 2>& v);
    
    Point_& operator = (const Point_& pt);
    Point_& operator = (Point_&& pt) CV_NOEXCEPT;
    //! conversion to another data tye
    template<typename _Tp2> operator Point_<_Tp2>() const;
    
    //! conversion to the old-style C structures
    operator Vec<_Tp, 2>() const;
    
    //! dot product
    _Tp dot(const Point_& pt) const;
    //! dot product computed in double-precision arithmetics
    double ddot(const Point_& pt) const;
    //! cross-product
    double cross(const Point_& pt) const;
    //! checks whether the point is inside the specified rectangle
    bool inside(const Rect_<Tp>& r) const;
    _Tp x;	//!< x coordinate of the point
    _Tp y;	//!< y coordinate of the point
};
```

* 特化

```c++
typedef Point_<int> Point2i;
typedef Point_<int64> Point2l;
typedef Point_<float> Point2f;
typedef Point_<double> Point2d;
typedef Point2i Point;
```

* 实现

构造函数

```c++
template<typename _Tp> inline
Point_<_Tp>::Point_():x(0), y(0)
{}
```

```c++
template<typename _Tp> inline
Point_<_Tp>::Point_(_Tp _x, _Tp _y)
    :x(_x), y(_y)
{}
```

```c++
template<typenmae _Tp> inline
Point_<_Tp>::Point_(const Point_& pt)
    :x(pt.x), y(pt.y)
{}
```

```c++
template<typename _Tp> inline
Point_<_Tp>::Point_(Point_&& pt) CV_NOEXCEPT
    :x(std::move(pt.x)), y(std::move(pt.y))
{}
```

```c++
template<typename _Tp> inline
Point_<_Tp>::Point_(const Size_<_Tp>& sz)
    :x(sz.width), y(sz.height)
{}
```

```c++
template<typename _Tp> inline
Point_<_Tp>::Point_(const Vec<_Tp, 2>& v)
    :x(v[0]), y(v[1])
{}
```

拷贝构造

```c++
template<typename _Tp> inline
Point_<_Tp>& Point_<_Tp>::operator = (const Point_& pt)
{
    x = pt.x;
    y = pt.y;
    return *this;
}
```

```c++
template<typename _Tp> inline
Point_<_Tp>& Point_<_Tp>::operator = (Point_&& pt) CV_NOEXCEPT
{
    x = std::move(pt.x);
    y = std::move(pt.y);
    return *this;
}
```

类型转换

```c++
template<typename _Tp> template<typename _Tp2> inline
Point_<_Tp>::operator Point_<_Tp2>() const
{
    return Point_<_Tp2>(saturate_cast<_Tp2>(x), saturate_cast<_Tp2>(y));
}
```

```c++
template<typename _Tp> inline
Point_<_Tp>::operator Vec<_Tp, 2>() const
{
    return Vec<_Tp, 2>(x, y);
}
```

操作

```c++
template<typename _Tp> inline
_Tp Point_<_Tp>::dot(const Point_& pt) const
{
    return saturate_cast<_Tp>(x*pt.x + y*pt.y);
}
```

```c++
template<typename _Tp> inline
double Point_<_Tp>::ddot(const Point_& pt) const
{
    return (double)x*pt.x+(double)y*pt.y;
}
```

```c++
template<typename _Tp> inline
double Point_<_Tp>::cross(const Point_& pt) const
{
    return (double)x*pt.y - (double)y*pt.x;
}
```

```c++
template<typename _Tp> inline
bool Point_<_Tp>::inside(const Rect_<_Tp>& r)const
{
    return r.contains(*this);
}
```

运算符重载（一元）

==注意这个`-`==

```c++
template<typename _Tp> static inline
Point_<_Tp> operator - (const Point_<_Tp>& a)
{
    return Point_<_Tp>( saturate_cast<_Tp>(-a.x), saturate_cast<_Tp>(-a.y) );	// 为什么要使用类型转换呢？
}
```

运算符重载（二元）

```c++
template<typename _Tp> static inline
Point_<_Tp>& operator += (Point_<_Tp>& a, const Point_<_Tp>& b)
{
    a.x += b.x;
    a.y += b.y;
    return a;
}
```

```c++
template<typename _Tp> static inline
Point_<_Tp>& operator -= (Point_<_Tp>& a, const Point_<_Tp>& b)
{
    a.x -= b.x;
    a.y -= b.y;
    return a;
}
```

==注意这里只定义了乘法以及除法的细节！==

```c++
template<typename _Tp> static inline
Point_<_Tp> operator * (const Point_<_Tp>& a, int b)
{
    return Point_<_Tp>( saturate_cast<_Tp>(a.x*b), saturate_cast<_Tp>(a.y*b) );
}
```

```c++
template<typename _Tp> static inline
Point_<_Tp> operator * (int a, const Point_<_Tp>& b)
{
    return Point_<_Tp>( saturate_cast<_Tp>(b.x*a), saturate_cast<_Tp>(b.y*a) );
}
```

```c++
template<typename _Tp> static inline
Point_<_Tp> operator * (const Point_<_Tp>& a, float b)
{
    return Point_<_Tp>( saturate_cast<_Tp>(a.x*b), saturate_cast<_Tp>(a.y*b) );
}
```

```c++
template<typename _Tp> static inline
Point_<_Tp> operator * (float a, const Point_<_Tp>& b)
{
    return Point_<_Tp>( saturate_cast<_Tp>(b.x*a), saturate_cast<_Tp>(b.y*a) );
}
```

```c++
template<typename _Tp> static inline
Point_<_Tp> operator * (const Point_<_Tp>& a, double b)
{
    return Point_<_Tp>( saturate_cast<_Tp>(a.x*b), saturate_cast<_Tp>(a.y*b) );
}
```

```c++
template<typename _Tp> static inline
Point_<_Tp> operator * (double a, const Point_<_Tp>& b)
{
    return Point_<_Tp>( saturate_cast<_Tp>(b.x*a), saturate_cast<_Tp>(b.y*a) );
}
```

```c++
template<typename _Tp> static inline
Point_<_Tp>& operator *= (Point_<_Tp>& a, int b)
{
    a.x = saturate_cast<_Tp>(a.x * b);
    a.y = saturate_cast<_Tp>(a.y * b);	// 这里的a.x, a.y不需要先进行类型转换吗？
    return a;
}
```

```c++
template<typename _Tp> static inline
Point_<_Tp>& operator *= (Point_<_Tp>& a, float b)
{
    a.x = saturate_cast<_Tp>(a.x * b);
    a.y = saturate_cast<_Tp>(a.y * b);
    return a;
}
```

```c++
template<typename _Tp> static inline
Point_<_Tp>& operator *= (Point_<_Tp>& a, double b)
{
    a.x = saturate_cast<_Tp>(a.x * b);
    a.y = saturate_cast<_Tp>(a.y * b);
    return a;
}
```

**其它乘法操作**（这个Matx挺复杂的吧）

```c++
template<typename _Tp> static inline
Point_<_Tp> operator * (const Matx<_Tp, 2, 2>& a, const Point_<_Tp>& b)
{
    Matx<_Tp, 2, 1> tmp = a * Vec<_Tp, 2>(b.x, b.y);
    return Point_<_Tp>(tmp.val[0], tmp.val[1]);
}
```

```c++
template<typename _Tp> static inline
Point3_<_Tp> operator * (const Matx<_Tp, 3, 3>& a, const Point_<_Tp>& b)
{
    Matx<_Tp, 3, 1> tmp = a * Vec<_Tp, 3>(b.x, b.y, 1);
    return Point3_<_Tp>(tmp.val[0], tmp.val[1], tmp.val[2]);
}
```

---

```c++
template<typename _Tp> static inline
Point_<_Tp> operator / (const Point_<_Tp>& a, int b)
{
    Point_<_Tp> tmp(a);
    tmp /= b;
    return tmp;
}
```

```c++
template<typename _Tp> static inline
Point_<_Tp> operator / (const Point_<_Tp>& a, float b)
{
    Point_<_Tp> tmp(a);
    tmp /= b;
    return tmp;
}
```

```c++
template<typename _Tp> static inline
Point_<_Tp> operator / (const Point_<_Tp>& a, double b)
{
    Point_<_Tp> tmp(a);
    tmp /= b;
    return tmp;
}
```

```c++
template<typename _Tp> static inline
Point_<_Tp>& operator /= (Point_<_Tp>& a, int b)
{
    a.x = saturate_cast<_Tp>(a.x / b);
    a.y = saturate_cast<_Tp>(a.y / b);
    return a;
}
```

```c++
template<typename _Tp> static inline
Point_<_Tp>& operator /= (Point_<_Tp>& a, float b)
{
    a.x = saturate_cast<_Tp>(a.x / b);
    a.y = saturate_cast<_Tp>(a.y / b);
    return a;
}
```

```c++
template<typename _Tp> static inline
Point_<_Tp>& operator /= (Point_<_Tp>& a, double b)
{
    a.x = saturate_cast<_Tp>(a.x / b);
    a.y = saturate_cast<_Tp>(a.y / b);
    return a;
}
```

自定义运算符

```c++
template<typename _Tp> static inline
double norm(const Point_<_Tp>& pt)
{
    return std::sqrt((double)pt.x * pt.x + (double)pt.y * pt.y);
}
```

---

**<u>看不懂这些模板函数，自己在vs上复现一下看看实现的是什么。</u>**

[git](git@github.com:CPlusPlus-AWILL/TemplatePractice.git)

```c++
template<typename _AccTp> static inline _AccTp normL2Sqr(const Point_<int>& pt);
template<typename _AccTp> static inline _AccTp normL2Sqr(const Point_<int64>& pt);
template<typename _AccTp> static inline _AccTp normL2Sqr(const Point_<float>& pt);
template<typename _AccTp> static inline _AccTp normL2Sqr(const Point_<double>& pt);
```

```c++
template<> inline int normL2Sqr<int>(const Point_<int>& pt)
{
    return pt.dot(pt);
}
template<> inline int64 normL2Sqr<int64>(const Point_<int64>& pt)
{
    return pt.dot(pt);
}
template<> inline float normL2Sqr<float>(const Point_<float>& pt)
{
    return pt.dot(pt);
}
template<> inline double normL2Sqr<double>(const Point_<int>& pt)
{
    return pt.dot(pt);
}
template<> inline double normL2Sqr<double>(const Point_<float>& pt)
{
    return pt.ddot(pt);
}
template<> inline double normL2Sqr<double>(const Point_<double>& pt)
{
    return pt.ddot(pt);
}
```

# Point3_

> 由`x`、`y`、`z`三个坐标定义。类的结构大体上类似于`Point2_`，也可以与C结构进行类型转换。

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

