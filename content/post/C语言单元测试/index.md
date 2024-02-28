---
title: C语言单元测试
description: 
date: 2023-01-01 00:00:00+0000
image: 
categories:
    - 2023
tags:
    - C语言
weight: 1
---

*CUnit* （*单元测试*框架），CUnit是一个用C语言编写、管理和运行单元测试的轻量级系统，它为C程序员提供了具有灵活多样用户界面的基本测试功能。

<!--more-->

## 环境安装

```
# 普通版本
apt install libcunit1 libcunit1-dev

# ncurses版本
apt install libcunit1-ncurses libcunit1-ncurses-dev
```

## 四种模式输出简介

| **模式**      | **平台**       | **结果输出方式**     | **使用的接口函数**                                                                                                  | **介绍**                                          |
| ------------------- | -------------------- | -------------------------- | ------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| **Basic**     | **所有**       | **标准输出**         | **#include "CUnit/Basic.h"CU_basic_set_mode(CU_BRM_VERBOSE);CU_basic_run_tests();**                                 | **结果输出到标准输出（stdout）**                  |
| **Automated** | **所有**       | **xml文件**          | **#include "CUnit/Automated.h"CU_set_output_filename("result"); CU_list_tests_to_file();CU_automated_run_tests();** | **生成XML文件**                                   |
| **Console**   | **所有**       | **交互式控制台**     | **#include "CUnit/Console.h"CU_console_run_tests();**                                                               | **比较灵活，可以选择只执行其中某一个测试用例**    |
| **Curses**    | **Linux/Unix** | **交互式curses窗口** | **#include "CUnit/CUCurses.h"CU_curses_run_tests();**                                                               | **跟Console类似，只不过是以Curses窗口的方式展示** |

注：Automated模式生成完XML文件之后，然后再将CUnit-List.dtd、CUnit-List.xsl、CUnit-Run.dtd、CUnit-Run.xsl（这几个文件在CUnit的源码包可以找到，在/usr/share/CUnit/目录下）和XML文件放到同一级目录，再用浏览器打开，就可以看到界面了。

## 例子

### 1.添加测试suite

```c
/**
 * @description: 测试单元初始化函数
 */
int initializeSuite()
{
    printf("\n--- Start CUnit Test ---\n");
    return 0;
}

/**
 * @description: 测试单元释放函数
 */
int finalizeSuite()
{
    printf("\n--- Finish CUnit Test ---\n");
    return 0;
}
```

### 2.添加测试用例case

```c
//case1
void test_1_1()
{
    /* 断言函数返回是否与预期相同 */
    CU_ASSERT_EQUAL_FATAL(3, sum(1,2));
}

//case2
void test_1_2()
{
    CU_ASSERT_EQUAL_FATAL(1, sub(2,1));
}
```

### 3.创建测试模块

**将之前创建的suite和case添加到模块中，可添加多个suite和case。**
模块不是必须的，直接将suite和case添加到主函数也可以实现测试。

```c
/**
 * @description: 构建单元测试套件与用例
 * @return CU_ErrorCode:cunit错误码
 */
CU_ErrorCode TestModule()
{
    CU_pSuite pSuite1 = NULL;
    pSuite1 = CU_add_suite("Suite1", initializeSuite, finalizeSuite);  

    //检测注册Suite情况
    if (NULL == pSuite1) {
        CU_cleanup_registry();
        return CU_get_error();
    }

    /**
     * 添加当前套件的测试用例
     * Test_1-1：用例1-1名 test_1_1：用例函数
     */
    if (NULL == CU_add_test(pSuite1, "Test_1-1", test_1_1))
    {
        CU_cleanup_registry();
        return CU_get_error();
    }
    if (NULL == CU_add_test(pSuite1, "Test_1-2", test_1_2))
    {
        CU_cleanup_registry();
        return CU_get_error();
    }

    return CUE_SUCCESS;
}
```

### 4.测试主函数

```c
#include <CUnit/Basic.h>
#include <CUnit/CUnit.h>

int main(int argc, char const *argv[])
{
    // 初始化测试注册表
    if (CUE_SUCCESS != CU_initialize_registry())
    {
        return CU_get_error();
    }

    //检测是否在执行
    assert(!CU_is_test_running());

    //调用测试模块完成测试用例
    if (CUE_SUCCESS != TestModule())
    {
        CU_cleanup_registry();
        return CU_get_error();
    }

    // 设置运行模式
    /*** 运行结果产成XML文件的模式******** 
     CU_set_output_filename("result"); 
    CU_list_tests_to_file(); 
    CU_automated_run_tests(); 
    ***********************************/
    // CU_console_run_tests(); /* 控制台交互模式 */
    // CU_basic_run_tests(); /* 控制台直接输出 */
    CU_curses_run_tests(); /* ncurses交互模式 */

    //清除注册信息完成单元测试
    CU_cleanup_registry();

    return EXIT_SUCCESS;
}
```

## CUint断言

以下断言可根据测试选择的不同判断条件来抛出异常，以达到单元测试的目的

包含头文件 `#include <CUnit/CUnit.h>`

| **断言**                                                                                                                     | **含义**                                                 |
| ---------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| **CU_ASSERT(int  expression)CU_ASSERT_FATAL(int  expression)CU_TEST(int  expression)CU_TEST_FATAL(int  expression)**         | **断言表达式为TRUE（非零）**                             |
| **CU_ASSERT_TRUE(value)CU_ASSERT_TRUE_FATAL(value)**                                                                         | **断言值为真（非零）**                                   |
| **CU_ASSERT_FALSE(value)CU_ASSERT_FALSE_FATAL(value)**                                                                       | **断言值为假（零）**                                     |
| **CU_ASSERT_EQUAL(actual, expected)CU_ASSERT_EQUAL_FATAL(actual, expected)**                                                 | **断言实际值==期望值**                                   |
| **CU_ASSERT_NOT_EQUAL(actual, expected))CU_ASSERT_NOT_EQUAL_FATAL(actual, expected)**                                        | **断言实际值!=期望值**                                   |
| **CU_ASSERT_PTR_EQUAL(actual, expected)CU_ASSERT_PTR_EQUAL_FATAL(actual, expected)**                                         | **断言指针实际==期待**                                   |
| **CU_ASSERT_PTR_NOT_EQUAL(actual, expected)CU_ASSERT_PTR_NOT_EQUAL_FATAL(actual, expected)**                                 | **断言指针实际!=期待**                                   |
| **CU_ASSERT_PTR_NULL(value)CU_ASSERT_PTR_NULL_FATAL(value)**                                                                 | **指针值==NULL**                                         |
| **CU_ASSERT_PTR_NOT_NULL(value)CU_ASSERT_PTR_NOT_NULL_FATAL(value)**                                                         | **指针值!=NULL**                                         |
| **CU_ASSERT_STRING_EQUAL(actual, expected)CU_ASSERT_STRING_EQUAL_FATAL(actual, expected)**                                   | **断言实际字符串与预期字符串相等**                       |
| **CU_ASSERT_STRING_NOT_EQUAL(actual, expected)CU_ASSERT_STRING_NOT_EQUAL_FATAL(actual, expected)**                           | **断言实际字符串与预期字符串不等**                       |
| **CU_ASSERT_NSTRING_EQUAL(actual, expected, count)CU_ASSERT_NSTRING_EQUAL_FATAL(actual, expected, count)**                   | **断言实际和预期的第一个计数字符相同**                   |
| **CU_ASSERT_NSTRING_NOT_EQUAL(actual, expected, count)CU_ASSERT_NSTRING_NOT_EQUAL_FATAL(actual, expected, count)**           | **断言实际和预期的第一个计数字符不同**                   |
| **CU_ASSERT_DOUBLE_EQUAL(actual, expected, granularity)CU_ASSERT_DOUBLE_EQUAL_FATAL(actual, expected, granularity)**         | **断言（实际-预期）<=（粒度） 此断言必须链接到数学库。** |
| **CU_ASSERT_DOUBLE_NOT_EQUAL(actual, expected, granularity)CU_ASSERT_DOUBLE_NOT_EQUAL_FATAL(actual, expected, granularity)** | **断言（实际-预期）>（粒度） 此断言必须链接到数学库。**  |
| **CU_PASS(message)**                                                                                                         | **用指定的消息注册传递断言。不执行逻辑测试。**           |
| **CU_FAIL(message)CU_FAIL_FATAL(message)**                                                                                   | **用指定的消息注册失败的断言。不执行逻辑测试。**         |
