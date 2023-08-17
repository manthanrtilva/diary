---
layout: post
title:  "GDB: Call function of running process"
date:   2023-08-17 00:37:00 +0000
categories: gdb,daemon,runtime
---
Below code snippet mimic infinitely running process which log to stdout. We will demonstrate how to change log level at runtime.(without restarting running process)
```
#include <cstdio>
#include <cstdarg>
#include <cstring>
#include <iostream>
#include <thread>
#include <chrono>
#include <ctime>   

#define TRACE printf("%s:%d\n",__FILE__,__LINE__);
enum Level {
  DEBUG = 0,
  INFO = 1,
  WARN = 2,
  ERR = 3
};
Level gLevel{Level::DEBUG};
bool stricmp(const char* str1,const char* str2) {
  if(!str1 && !str2) {
    return true;
  }else if(!str1 || !str2) {
    return false;
  }else if(std::strlen(str1) != std::strlen(str2)) {
    return false;
  } else {
    for(size_t i = 0; i < std::strlen(str1); ++i) {
      if(std::tolower(str1[i]) != std::tolower(str2[i])) {
        return false;
      }
    }
  }
  return true;
}
void set_log_level(const char* level) {
  if(stricmp(level,"DEBUG")) {
    gLevel = Level::DEBUG;
  }else if(stricmp(level,"INFO")) {
    gLevel = Level::INFO;
  }else if(stricmp(level,"WARN")) {
    gLevel = Level::WARN;
  }else if(stricmp(level,"ERR")) {
    gLevel = Level::ERR;
  }else {
  }
}
void log_impl(Level level, const char* format,...) {
  if(level >= gLevel) {
    va_list args;
    va_start(args, format);
    vprintf(format,args);
    va_end(args);
  }
}
#define LOG_DEBUG(fmt,...) log_impl(Level::DEBUG,"[DEBUG] : " fmt, ##__VA_ARGS__);
#define LOG_INFO(fmt,...) log_impl(Level::INFO,"[INFO] : " fmt, ##__VA_ARGS__);
#define LOG_WARN(fmt,...) log_impl(Level::WARN,"[WARN] : " fmt, ##__VA_ARGS__);
#define LOG_ERR(fmt,...) log_impl(Level::ERR,"[ERR] : " fmt, ##__VA_ARGS__);
int main() {
  while(1){
    std::time_t end_time = std::chrono::system_clock::to_time_t(std::chrono::system_clock::now());
    printf("%s",std::ctime(&end_time));
    LOG_DEBUG("\n");
    LOG_DEBUG("%d\n",__LINE__);
    LOG_INFO("\n");
    LOG_INFO("%d\n",__LINE__);
    LOG_WARN("\n");
    LOG_WARN("%d\n",__LINE__);
    LOG_ERR("\n");
    LOG_ERR("%d\n",__LINE__);
    std::this_thread::sleep_for(std::chrono::seconds(2));
  }
  return 0;
}
```
Now compile and run above code with below command
```
g++ -g app.cpp -o app && ./app
```
Once application is running now you can change log level from other terminal with below command from other terminal window.
```
gdb -p `pidof app` --batch -ex "set variable ::gLevel=3" -ex c -ex "set confirm off" -ex quit
```
OR
```
gdb -p `pidof app` -batch -ex 'p (void)set_log_level("ERR")' -ex 'c' -ex 'set confirm off' -ex quit
```

If you are compiling code in release mode(without `-g` option), then first command(i.e., setting variable directly) won't work but second variant works.

