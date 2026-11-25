---
title: "使用 nlohmann/json 序列化"
description: "nlohmann/json 序列化 struct、enum 和 std::optional 等自定义类型的用法记录。"
pubDate: 2024-01-01
tags: ['Tech']
---

[https://github.com/nlohmann/json](https://github.com/nlohmann/json)

```
#include <iostream>
#include <fstream>
#include <vector>
#include <optional>

#include "json.hpp"

namespace nl = nlohmann;

enum class TestEnum : int {
    Left = 0,
    Right
};

struct SubTestStruct {
    int test = 0;
};

struct TestStruct {
    int test = 0;
    bool testBool = false;
    TestEnum testEnum = TestEnum::Left;
    std::optional<bool> testOpt;
    std::vector<int> testVec;
    SubTestStruct subTestStruct;
};

// nl 不直接支持 optional， 使用 adl_serializer 可以支持任意类型的序列化
// https://github.com/nlohmann/json/pull/2117
// https://github.com/nlohmann/json#how-do-i-convert-third-party-types
namespace nlohmann {
    template <typename T>
    struct adl_serializer<std::optional<T>> {
        static void to_json(json& j, const std::optional<T>& opt) {
            if (opt == std::nullopt) {
                j = nullptr;
            } else {
              j = *opt; // this will call adl_serializer<T>::to_json which will
                        // find the free function to_json in T's namespace!
            }
        }

        static void from_json(const json& j, std::optional<T>& opt) {
            if (j.is_null()) {
                opt = std::nullopt;
            } else {
                opt = j.get<T>(); // same as above, but with
                                  // adl_serializer<T>::from_json
            }
        }
    };
}

// 序列化struct,class
NLOHMANN_DEFINE_TYPE_NON_INTRUSIVE(SubTestStruct, test)
NLOHMANN_DEFINE_TYPE_NON_INTRUSIVE(TestStruct, test, testBool, testEnum, testOpt, testVec, subTestStruct)

// 序列化enum
NLOHMANN_JSON_SERIALIZE_ENUM(TestEnum, {
    {TestEnum::Left, "Left"},
    {TestEnum::Right, "Right"},
})

void printTest(const TestStruct& s) {
    nl::json jsonValue;
    nl::to_json(jsonValue, s);

    std::cout << "TestStruct test: " << s.test
              << ", testBool: " << s.testBool
              << ", testEnum: " << (int) s.testEnum
              << ", testOpt: " << (s.testOpt.has_value() && s.testOpt.value())
              << ", testVec: { ";
    for(auto val : s.testVec) {
        std::cout << val << ",";
    }
    std::cout << "\b" << " }";
    std::cout << ", subTestStruct.test: " << s.subTestStruct.test;
    std::cout << std::endl;
}

int main() {
    // 写入
    std::ofstream sf("config.json");
    nl::json j;
    TestStruct s;
    s.test = 100;
    s.testOpt = true;
    for(int i = 0; i < 5; ++i) {
        s.testVec.push_back(i);
    }
    s.subTestStruct.test = 99;
    printTest(s);
    nl::to_json(j, s);
    // 单独获取一个值
    std::cout << "Json value: " << j["test"] << std::endl;
    //  写入文件
    sf << j;
    sf.close();

    //  读取
    std::ifstream  is("config.json");
    nl::json readJson;
    is >> readJson;
    TestStruct newS = readJson.get<TestStruct>();
//    nl::from_json(readJson, newS);
    printTest(newS);
    is.close();
    return 0;
}
```
