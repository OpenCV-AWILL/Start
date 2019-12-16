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

* 模板类

```c++
template<typename _Tp> class Point3_
{
public:
    typedef _Tp value_type;
    
    //! default constructor
    Point3_();
    Point3_(_Tp _x, _Tp _y, _Tp _z);
    Point3_(const Point3_& pt);
    Point3_(const Point3_&& pt) CV_NOEXCEPT;
    explicit Point3_(const Point_<_Tp>& pt);
    Point3_(const Vec<_Tp, 3>& v);

    Point3_& operator = (const Point3_& pt);
    Point3_& operator = (Point3_&& pt) CV_NOEXCEPT;
    //! conversion to another data type
    template<typename _Tp2> operator Point3_<_Tp2>() const;
    //! conversion to cv::Vec<>
    operator Vec<_Tp, 3>() const;
    
    //! dot product
    _Tp dot(const Point3_& pt) const;
    //! dot product computed in double-precision arithmetics
    double ddot(const Point3_& pt) const;
    //! cross product of the 2 3D points
    Point3_ cross(const Point3_& pt) const;
    _Tp x;	//!< x coordinate of the 3D point
    _Tp y;	//!< y coordinate of the 3D point
    _Tp z;	//!< z coordinate of the 3D point
};
```

* 特化

```c++
typedef Point3_<int> Point3i;
typedef Point3_<float> Point3f;
typedef Point3_<double> Point3d;
```

* 实现

构造函数

```c++
template<typename _Tp> inline
Point3_<_Tp>::Point3_()
    :x(0), y(0), z(0)
{}
```

```c++
template<typename _Tp> inline
Point3_<_Tp>::Point3_(_Tp _x, _Tp _y, _Tp _z)
    :x(_x), y(_y), z(_z)
{}
```

```c++
template<typename _Tp> inline
Point3_<_Tp>::Point3_(const Point3_& pt)
    :x(pt.x), y(pt.y), z(pt.z)
{}
```

```c++
template<typename _Tp> inline
Point3_<_Tp>::Point3_(Point3_&& pt) CV_NOEXCEPT
    :x(std::move(pt.x)), y(std::move(pt.y)), z(std::move(pt.z))
{}
```

```c++
template<typename _Tp> inline
Point3_<_Tp>::Point3_(const Point_<_Tp>& pt)
    :x(pt.x), y(pt.y), z(_Tp())
{}
```

```c++
template<typename _Tp> inline
Point3_<_Tp>::Point3_(const Vec<_Tp, 3>& v)
    :x(v[0]), y(v[1]), z(v[2])
{}
```

类型转换

```c++
template<typename _Tp> template<typename _Tp2> inline
Point3_<_Tp>::operator Point3_<_Tp2>() const
{
    return Point3_<_Tp2>(saturate_cast<_Tp2>(x), saturate_cast<_Tp2>(y), saturate_cast<_Tp2>(z));
}
```

```c++
template<typename _Tp> inline
Point3_<_Tp>::operator Vec<_Tp, 3>() const
{
    return Vec<_Tp, 3>(x, y, z);
}
```

拷贝构造

```c++
template<typename _Tp> inline
Point3_<_Tp>& Point3_<_Tp>::operator = (const Point3_& pt)
{
    x = pt.x;
    y = pt.y;
    z = pt.z;
    return *this;
}
```

```c++
template<typename _Tp> inline
Point3_<_Tp>& Point3_<_Tp>::operator = (Point3_&& pt) CV_NOEXCEPT
{
    x = std::move(pt.x);
    y = std::move(pt.y);
    z = std::move(pt.z);
    return *this;
}
```

操作

==这个模板类型<>省略和不省略的区别是什么？看一下C++ Primer吧。==

省略与否应该是不影响函数签名的。

因为在编译器里面我已经试过了，编译出来的函数签名是完全一致的。

***这样写反而让我很迷惑。***

```c++
template<typename _Tp> inline
_Tp Point3_<_Tp>::dot(const Point3_& pt) const
{
    return saturate_cast<_Tp>(x*pt.x + y*pt.y + z*pt.z);
}
```

```c++
template<typename _Tp> inline
double Point3_<_Tp>::ddot(const Point3_& pt) const
{
    return (double)x*pt.x + (double)y*pt.y + (double)z*pt.z;
}
```

cross的数学意义是什么？*可能需要补充一些高数的知识了。*

```c++
template<typename _Tp> inline
Point3_<_Tp> Point3_<_Tp>::cross(const Point3_<_Tp>& pt) const
{
    return Point3_<_Tp>(y*pt.z - z*pt.y, z*pt.x - x*pt.z, x*pt.y - y*pt.x);
}
```

运算符重载

粗略看一遍直接跳过吧。都差不多一个样。

# Size_

> 这个简单的模板类用来指定图像/矩形的尺寸——使用`width`和`height`。

* 模板类

```c++
template<typename _Tp> class Size_
{
public:
    typedef _Tp value_type;
    
    //! default constructor
    Size_();
    Size_(_Tp _width, _Tp _height);
    Size_(const Size_& sz);
    Size_(Size_&& sz) CV_NOEXCEPT;
    //! the area (width*height)
    _Tp area() const;
    //! aspect ratio (width/height)
    double aspectRatio() const;
    //! true if empty
    bool empty() const;
    
    //! conversion of another data type
    template<typename _Tp2> operator Size_<_Tp2>() const;
    
    _Tp width;	//!< the width
    _Tp height;	//!< the height
};
```

* 特化

```c++
typedef Size_<int> Size2i;
typedef Size_<int64> Size2l;
typedef Size_<float> Size2f;
typedef Size_<double> Size2d;
typedef Size2i Size;
```

* 实现

构造函数

```c++
template<typename _Tp> inline
Size_<_Tp>::Size_()
    :width(0), height(0) {}
```

...

操作

```c++
template<typename _Tp> inline
_Tp Size_<_Tp>::area() const
{
    const _Tp result = width * height;
    CV_DbgAssert(!std::numeric_limits<_Tp>::is_integer || width == 0 || result / width == height);	// make sure the result fits in the return value
    return result;
}
```

```c++
template<typename _Tp> inline
double Size_<_Tp>::aspectRatio() const
{
    return width / static_cast<double>(height);
}
```

```c++
template<typename _Tp> inline
bool Size_<_Tp>::empty() const
{
    return width <= 0 || height <= 0;
}
```

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
> 4. 注意它的一些扩展操作。如平移、膨胀、&、|、+、-、>=

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

* 实现

构造

```c++
template<typename _Tp> inline
Rect_<_Tp>::Rect_(const Point_<_Tp>& pt1, const Point_<_Tp>& pt2)
{
    x = std::min(pt1.x, pt2.x);
    y = std::min(pt1.y, pt2.y);
    width = std::max(pt1.x, pt2.x) - x;
    height = std::max(pt1.y, pt2.y) - y;
}
```

操作

```c++
template<typename _Tp> inline
Point_<_Tp> Rect_<_Tp>::tl() const
{
    return Point_<_Tp>(x, y);
}
```

```c++
template<typename _Tp> inline
Point_<_Tp> Rect_<_Tp>::br() const
{
    return Point_<_Tp>(x + width, y + height);
}
```

```c++
template<typename _Tp> inline
Size_<_Tp> Rect_<_Tp>::size() const
{
    return Size_<_Tp>(width, height);
}
```

```c++
template<typename _Tp> inline
_Tp Rect_<_Tp>::area() const
{
    const _Tp result = width *height;
    CV_DbgAssert(!std::numeric_limits<_Tp>::is_integer || width == 0 || result / width == height);	// make sure the result fits in the return value
    return result;
}
```

```c++
template<typename _Tp> inline
bool Rect_<_Tp>empty() const
{
    return width <= 0 || height <= 0;
}
```

```c++
template<typename _Tp> inline
bool Rect_<_Tp>::contains(const Point_<_Tp>& pt) const
{
    return x <= pt.x && pt.x < x + width && y <= pt.y && pt.y < y + height;
}
```

运算符的操作

```c++
template<typename _Tp> static inline
Rect_<_Tp>& operator += (Rect_<_Tp>& a, const Point_<_Tp>& b)
{
    a.x += b.x;
    a.y += b.y;
    return a;
}
```

```c++
template<typename _Tp> static inline
Rect_<_Tp>& operator += (Rect_<_Tp>& a, const Size_<_Tp>& b)
{
    a.width += b.width;
    a.height += b.height;
    return a;
}
```

与或操作==注意为什么&和|的条件判断为什么会不一样？==

```c++
template<typename _Tp> static inline
Rect_<_Tp>& operator &= (Rect_<_Tp>& a, const Rect_<_Tp>& b)
{
    _Tp x1 = std::max(a.x, b.x);
    _Tp y1 = std::max(a.y, b.y);
    a.width = std::min(a.x + a.width, b.x + b.width) - x1;
    a.height = std::min(a.y + a.height, b.y + b.height) - y1;
    a.x = x1;
    a.y = y1;
    if(a.width <= 0 || a.height <= 0)
    	a = Rect();
    return a;
}
```

```c++
template<typename _Tp> static inline
Rect_<_Tp>& operator |= (Rect_<_Tp>& a, const Rect_<_Tp>& b)
{
    if(a.empty()){
        a = b;
    }
    else if(!b.empty())
    {
        _Tp x1 = std::min(a.x, b.x);
        _Tp y1 = std::min(a.y, b.y);
        a.width = std::max(a.x + a.width, b.x + b.width) - x1;
        a.height = std::max(a.y + a.height, b.y + b.height) - y1;
        a.x = x1;
        a.y = y1;
    }
    return a;
}
```

# RotatedRect

> 平面中旋转矩形的一个简单描述——center point、length of each side、rotation angle in degrees。

* 模板类

```c++

```

