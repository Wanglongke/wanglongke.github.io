---
layout: post
title:  CPP Template 使用技巧
date:   2022-10-17 11:00:00 +0300
tags:   CPP
description: cpp学习与使用

---


## [Cpp Template](#cpp-template)

### 类调用模板函数

使用**template**关键字进行调用；  
使用**constexpr**修饰构造函数，则可使用**constexpr**修饰此类的对象；

```cpp
#include <iostream>
#include <stdexcept>


class Fraction {
private:
    const int _numerator;
    const int _denominator;

public:
    constexpr Fraction(int num, int denum) :
        _numerator(num), _denominator(denum) {
        if (_denominator == 0) {
            throw std::invalid_argument("denominator must not be zero");
        }
    }

    constexpr Fraction(int parts[2]) :
        Fraction(parts[0], parts[1]) { }

    constexpr Fraction(int num) :
        Fraction(num, 1) { }

    constexpr Fraction() :
        Fraction(0) { }

    template <typename T>
    constexpr T as() const
    {
        return T(_numerator) / T(_denominator);
    }

    template <typename T>
    constexpr T inverseAs() const
    {
        return _numerator != 0 ? T(_denominator) / T(_numerator) : throw std::invalid_argument("inverse of zero is undefined");
    }
};

template<int D, int Q>
constexpr Fraction cs2 = {};

template<>
constexpr Fraction cs2<2, 9> = { 1, 3 };



int main(int argc, char* argv[])
{
    float ret = cs2<2, 9>.template as<float>();
    float ret1 = cs2<1, 3>.template as<float>();

    std::cout << "cs2<2,9>.as() = " << ret << std::endl;
    std::cout << "cs2<1,3>.as() = " << ret1 << std::endl;
	return 0;
}
```

### 动态模板参数

尝试使用动态模板参数，以及类型萃取，来实现面向对象以及泛型；
这里只是简单抽取了一个模板库的代码，仅仅用来理解模板参数的使用；

```cpp
#include <iostream>
#include <array>
#include <tuple>

template<typename WANTED_FIELD,
		 typename CURRENT_FIELD,
         typename... FIELDS,
		 std::enable_if_t<std::is_same<WANTED_FIELD, CURRENT_FIELD>::value, int> = 0
>
constexpr unsigned getIndexInFieldList()
{
	return 0;
}

template<typename WANTED_FIELD,
	typename CURRENT_FIELD,
	typename... FIELDS,
	std::enable_if_t<!std::is_same<WANTED_FIELD, CURRENT_FIELD>::value, int> = 0
>
constexpr unsigned getIndexInFieldList()
{
	return 1 + getIndexInFieldList<WANTED_FIELD, FIELDS...>();
}

template<typename T, unsigned D>
struct FIELD_BASE
{
	FIELD_BASE() = delete;
	typedef T value_type;
	
	static constexpr unsigned size()
	{
		return D;
	}
};

struct VELOCITY : public FIELD_BASE<float, 3> {};
struct DENSITY : public FIELD_BASE<float, 1> {};
struct PRESSURE : public FIELD_BASE<float, 1> {};
struct GEOMETRY : public FIELD_BASE<unsigned, 1> {};

template<typename... FIELDS>
struct  DESCRIPTOR_BASE
{
	DESCRIPTOR_BASE() = delete;
	static constexpr std::size_t field_count = sizeof...(FIELDS);

	template<typename WANTED_FIELD>
	static constexpr int index()
	{
		return getIndexInFieldList<WANTED_FIELD, FIELDS...>();
	}
	template<template<typename...> class COLLECTION>
	using decomposeInto = COLLECTION<FIELDS...>;
};

template<typename FIELD>
struct FieldData
{
	using value_type = typename FIELD::value_type;
	std::array<value_type, FIELD::size()> _data;
};

template<typename... FIELDS>
struct MultiFieldData
{
	std::tuple<FieldData<FIELDS>...> _data;
};

typedef DESCRIPTOR_BASE<VELOCITY, DENSITY, PRESSURE> VEL_DEN_PRE;

template<typename... FIELDS>
using FieldArray = MultiFieldData<FIELDS...>;


int main(int argc, char* argv[])
{
	typename VEL_DEN_PRE::template decomposeInto<FieldArray> _data;
	int denIndex = VEL_DEN_PRE::index<DENSITY>();
	std::cout << "Density Field index is " << denIndex << std::endl;


	return 0;
}
```
