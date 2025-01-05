---
title: Android使用curl+JsonCpp实现http请求
date: 2024-07-04 21:26:21
tags:
categories: Android
---

> 在Android SDK开发过程中，需要进行一些网络鉴权，为了保密需要将http鉴权过程隐藏在so库中。
本文使用curl+JsonCpp库通过c++实现了http请求，源码见 github 项目 [AndroidNative](https://github.com/yadiq/AndroidNative)。

## 1.curl

### 1.1 curl概念 
[curl](https://curl.se/)是一个开源的多协议数据传输开源库，该框架具备跨平台性，开源免费，并提供了包括HTTP、FTP、SMTP、POP3等协议的功能。使用libcurl可以方便地进行网络数据传输操作，如发送HTTP请求、下载文件、发送电子邮件等。如果想让curl支持https，需要依赖openssl，因此需要先编译OpenSSL。

### 1.2 curl的编译
1. 编译过程见上一篇文章
https://yadiq.github.io/2024/07/02/LinuxCompileOpensslCurl/ 
2. 编译结果见本项目源码
https://github.com/yadiq/AndroidNative/tree/main/app/src/main/jni_demo/otherlibs/curl

## 2.JsonCpp

### 2.1 JsonCpp概念 
[JsonCpp](https://github.com/open-source-parsers/jsoncpp)JsonCpp 是一个 C++ 库，允许操作 JSON 值，包括序列化和反序列化字符串。它还可以在反序列化/序列化步骤中保留现有注释，使其成为一种存储用户输入文件的便捷格式。

### 2.2 JsonCpp的编译
1. 我们使用Android NDK CMake 对源码进行编译，和curl静态库编译在一起。
2. 编译过程见本项目源码
https://github.com/yadiq/AndroidNative/tree/main/app/src/main/jni_demo/otherlibs/jsoncpp
https://github.com/yadiq/AndroidNative/blob/main/app/src/main/jni_demo/CMakeLists.txt

## 3. 使用curl+JsonCpp实现http请求

### 3.1 CMake链接库
libcurl(网络协议库)如果要支持https，需要依赖libssl(SSL和TLS协议)。
而libssl(SSL和TLS协议)又依赖libcrypto(密码学算法)。
libcurl(网络协议库)如果要支持压缩传输，需要依赖zlib库。

这里我们使用前文的编译结果，OpenSSL编译后产生的 libcrypto.so libssl.so，curl编译后产生的 libcurl.a 。
使用Android系统自带的zlib库。CMake脚本见：
https://github.com/yadiq/AndroidNative/blob/main/app/src/main/jni_demo/CMakeLists.txt

### 3.2 JsonCpp的使用
已实现Json转string 和 string转Json。

#### 3.2.1 头文件如下：JsonUtil.h
```
#include <string>
#include "json/json.h"

using namespace std;

class JsonUtil {
public:
    /**
     * Json::Value 转 string
     * @param json
     * @return string
     */
    static string jsonToString(Json::Value& json);
    /**
     * string 转 Json::Value
     * @param value
     * @param json
     * @return 成功失败
     */
    static bool stringToJson(string& value, Json::Value& json);
    /**
     * cstring 转 Json::Value
     * @param value
     * @param json
     * @return 成功失败
     */
    static bool stringToJson(char* value, Json::Value& json);
};
```

#### 3.2.2 函数文件如下：JsonUtil.cpp
```
#include "JsonUtil.h"
#include "../../utils/Util.h"

string JsonUtil::jsonToString(Json::Value& json) {
    Json::FastWriter fwriter;//转化器-json压缩
    string str = fwriter.write(json);
    /*string str3 = json.toStyledString();//转化器-json美化1
    LOGD("json美化1, %s", str3.c_str());
    Json::StyledWriter swriter;//转化器-json美化2
    string str2 = swriter.write(json);
    LOGD("json美化2, %s", str2.c_str());*/
    return str;
}

bool JsonUtil::stringToJson(string& value, Json::Value& json) {
    bool result = false;
    Json::Reader reader;//解释器
    if (reader.parse(value, json)) {
        result = true;
        /*LOGD("json解析成功");
        Json::Value::Members mem = json.getMemberNames();
        for (auto iter = mem.begin(); iter != mem.end(); iter++) {
            string itemKey = *iter;
            if (json[itemKey].type() == Json::stringValue) {
                string itemValue = json[itemKey].asString();
                LOGD("%s:%s", itemKey.c_str(), itemValue.c_str());
            }
        }*/
    } else {
        //LOGD("json解析失败");
    }
    return result;
}

bool JsonUtil::stringToJson(char *value, Json::Value &json) {
    bool result = false;
    Json::Reader reader;//解释器
    if (reader.parse(value, json)) {
        result = true;
        //LOGD("json解析成功");
    } else {
        //LOGD("json解析失败");
    }
    return result;
}
```

源码见：
https://github.com/yadiq/AndroidNative/blob/main/app/src/main/jni_demo/otherlibs/jsoncpp/JsonUtil.cpp

### 3.3 curl实现http请求
已实现 http get请求 和 post请求。

#### 3.3.1 头文件如下：CurlHttp.h
```
#include "curl/curl.h"
#include <string>
using namespace std;

class CurlHttp {
public:
    CurlHttp();
    virtual ~CurlHttp();
    /**
     * 全局初始化
     */
    static void init();
    /**
     * 全局释放资源
     */
    static void cleanup();

public:
    /**
     * 发送get请求
     * @return HTTP状态码,200成功
     */
    long getRequest(const string &url, string &response);
    /**
     * 发送post请求
     * @return HTTP状态码,200成功
     */
    long postRequest(const string &url, const string &postParams, string &response);
private:
    CURL* curl;
};
```

#### 3.3.2 函数文件如下：CurlHttp.cpp
请在函数之前，至少调用全局初始化一次。
```
#include "CurlHttp.h"
#include "Util.h"
#include "../utils_android/LogUtil.h"

CurlHttp::CurlHttp() {
    //初始化
    curl = curl_easy_init();

    // 接收响应头数据，0代表不接收 1代表接收
    curl_easy_setopt(curl, CURLOPT_HEADER, 0);
    //设置不使用任何信号/警报处理程序。在多线程场景下设置
    curl_easy_setopt(curl, CURLOPT_NOSIGNAL, 1L);
    //设置超时时间，单位秒
    curl_easy_setopt(curl, CURLOPT_CONNECTTIMEOUT, 10);
    curl_easy_setopt(curl, CURLOPT_TIMEOUT, 10);
    //不验证HTTPS证书的合法性
    curl_easy_setopt(curl, CURLOPT_SSL_VERIFYPEER, 0L);
    //不验证https请求时返回的证书是否与请求的域名相符合，避免被中间者篡改证书文件
    curl_easy_setopt(curl, CURLOPT_SSL_VERIFYHOST, 0L);
    //CURLOPT_VERBOSE的值为1时，会显示详细的调试信息
    //curl_easy_setopt(curl, CURLOPT_VERBOSE, 0);
}

CurlHttp::~CurlHttp() {
    curl_easy_cleanup(curl);
    curl = nullptr;
}

void CurlHttp::init() {
    //设置libcurl需要的程序环境。在程序调用libcurl中的任何其他函数之前，必须在程序中至少调用此函数一次
    curl_global_init(CURL_GLOBAL_ALL);
}

void CurlHttp::cleanup() {
    curl_global_cleanup();
}

//回调函数
size_t write_callback(char *ptr, size_t size, size_t nmemb, void *userdata) {
    ((string *) userdata)->append(ptr, size * nmemb);
    return size * nmemb;
}

long CurlHttp::getRequest(const string &url, string &response) {
    //HTTP状态码
    long http_code = -1;
    if (curl == nullptr)
        return http_code;
    /////////////////通用请求////////////////
    //设置请求的URL地址
    curl_easy_setopt(curl, CURLOPT_URL, url.c_str());
    //设置数据接收函数
    curl_easy_setopt(curl, CURLOPT_WRITEFUNCTION, write_callback);
    curl_easy_setopt(curl, CURLOPT_WRITEDATA, &response);
    //进行网络操作
    CURLcode res = curl_easy_perform(curl);
    /////////////////数据响应////////////////
    //查询HTTP状态码
    curl_easy_getinfo(curl, CURLINFO_RESPONSE_CODE, &http_code);
    //失败时返回错误信息
    if (res != CURLE_OK) {
        response.append(curl_easy_strerror(res));
    }
    //LOGD("GET %ld %s\n%s", http_code, url.c_str(), response.c_str());
    return http_code;
}

long CurlHttp::postRequest(const string &url, const string &postParams, string &response) {
    //HTTP状态码
    long http_code = -1;
    if (curl == nullptr)
        return http_code;
    /////////////////post请求相关////////////////
    //设置请求头
    struct curl_slist *headers = nullptr;
    headers = curl_slist_append(headers, "Content-Type:application/json");
    //headers = curl_slist_append(headers, "Content-Type:application/x-www-form-urlencoded");
    curl_easy_setopt(curl, CURLOPT_HTTPHEADER, headers);
    //设置请求为post请求
    curl_easy_setopt(curl, CURLOPT_POST, 1);
    //设置post请求的参数
    curl_easy_setopt(curl, CURLOPT_POSTFIELDS, postParams.c_str());
    /////////////////通用请求////////////////
    //设置请求的URL地址
    curl_easy_setopt(curl, CURLOPT_URL, url.c_str());
    //设置数据接收函数
    curl_easy_setopt(curl, CURLOPT_WRITEFUNCTION, write_callback);
    curl_easy_setopt(curl, CURLOPT_WRITEDATA, &response);
    //进行网络操作
    CURLcode res = curl_easy_perform(curl);
    /////////////////数据响应////////////////
    //查询HTTP状态码
    curl_easy_getinfo(curl, CURLINFO_RESPONSE_CODE, &http_code);
    //失败时返回错误信息
    if (res != CURLE_OK) {
        response.append(curl_easy_strerror(res));
    }
    //LOGD("POST %ld %s\n%s", http_code, url.c_str(), response.c_str());
    return http_code;
}
```

#### 3.3.3 测试文件如下：CurlTest.cpp
```
#include "CurlTest.h"
#include "../utils/CurlHttp.h"
#include "../utils_android/LogUtil.h"
#include "../otherlibs/jsoncpp/JsonUtil.h"


void CurlTest::curlGet() {
    //get请求
    string urlStr = "http://httpbin.org/get?param1=value1";
    string responseStr;
    CurlHttp curl;
    auto res = curl.getRequest(urlStr, responseStr);
    if (res == 200) {
        LOGD("curlGet success.\n%s", responseStr.c_str());
        //json解析
        Json::Value responseJson;
        bool responseReader = JsonUtil::stringToJson(responseStr, responseJson);
        if (responseReader) {
            string origin = responseJson["origin"].asString();
            LOGD("json解析成功. origin: %s", origin.c_str());
        } else {
            LOGD("json解析失败");
        }
    } else {
        LOGE("curlGet error. code: %ld, response: %s", res, responseStr.c_str());
    }
}

void CurlTest::curlPost() {
    //post请求
    string urlStr = "http://httpbin.org/post";
    Json::Value json;//键值对
    json["key1"] = "value1";
    json["key2"] = "value2";
    string bodyStr = JsonUtil::jsonToString(json);
    string responseStr;
    CurlHttp curl;
    auto res = curl.postRequest(urlStr, bodyStr, responseStr);
    if (res == 200) {
        LOGD("curlPost success.\n%s", responseStr.c_str());
        //json解析
        Json::Value responseJson;
        bool responseReader = JsonUtil::stringToJson(responseStr, responseJson);
        if (responseReader) {
            string origin = responseJson["origin"].asString();
            LOGD("json解析成功. origin: %s", origin.c_str());
        } else {
            LOGD("json解析失败");
        }
    } else {
        LOGE("curlPost error. code: %ld, response: %s", res, responseStr.c_str());
    }
}
```

源码见：
https://github.com/yadiq/AndroidNative/blob/main/app/src/main/jni_demo/utils/CurlHttp.cpp
