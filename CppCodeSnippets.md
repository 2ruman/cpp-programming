# C++ Code Snippets

### Contents
+ [Container](#container)
+ [File](#file)
+ [Macro](#macro)
+ [String](#string)
+ [Thread](#thread)
<br>

## Container

### Safely Get String from Map

```cpp
#include <string>
#include <unordered_map>

const std::string &getOrDefault(const std::unordered_map<int, std::string> &map, int key,
        const std::string &defaultVal) {
    auto iter = map.find(key);
    return iter != map.end() ? iter->second : defaultVal;
}

const std::string &getOrEmpty(const std::unordered_map<int, std::string> &map, int key) {
    return getOrDefault(map, key, "");
}

/* Otherwise, as a single function */
const std::string &getOrEmpty(const std::unordered_map<int, std::string> &map, int key) {
    static const std::string empty{};
    auto iter = map.find(key);
    return iter != map.end() ? iter->second : empty;
}
```

### Safely Get Value from Generic Map

```cpp
#include <unordered_map>

template <typename K, typename V>
const V &getOrDefault(const std::unordered_map<K, V> &map, int key, const V &defaultVal) {
    auto iter = map.find(key);
    return iter != map.end() ? iter->second : defaultVal;
}

// getOrEmpty() re-written using the generic one
const std::string &getOrEmpty(const std::unordered_map<int, std::string> &map, int key) {
    static const std::string empty{};
    return getOrDefault(map, key, empty);
}
```

## File

### Read Nth Record from File

```cpp
#include <cstdlib> // for strtol()
#include <fstream>
#include <string>

std::string getNth(std::ifstream &ifs, int n) {
    std::string ret;
    while ((ifs >> ret) && --n > 0);
    return (n == 0) ? ret : "";
}

std::string readNth(const std::string &filepath, int n) {
    std::ifstream file(filepath);
    return file.is_open() ? getNth(file, n) : "";
}

long readNthLong(const std::string &filepath, int n, long defaultVal) {
    std::string str = readNth(filepath, n);
    if (str.empty()) {
        return defaultVal;
    }
    char *endPtr;
    long ret = std::strtol(str.c_str(), &endPtr, 10);
    if (endPtr == nullptr || *endPtr != '\0') {
        return defaultVal;
    }
    return ret;
}
```

## Macro

### Loop & Condition Comparison

```cpp
/* Check if the flag is set in the given flags */
#define CHECK_FLAGS(flags, flag) if (((flags) & (flag)) > 0)

/* Check if the result is negative */
#define CHECK_NEGATIVE(r) if ((r) < 0)

/* Break the loop if the result is negative */
#define NEGATIVE_BREAK(r) if ((r) < 0) break

/* Oneshot loop that doesn't repeat */
#define ONESHOT_LOOP do {
#define ONESHOT_LOOP_END } while(false);
```

### Loop & Condition Comparison - Example

```cpp
const static int FLAG_A = 0x01;
const static int FLAG_B = 0x02;
const static int FLAG_C = 0x03;

int doA() { return 0; };
int doBC() { return -1; };

int test(int flags) {
    int rc;

    ONESHOT_LOOP

        CHECK_FLAGS(flags, FLAG_A) NEGATIVE_BREAK(rc = doA());
        CHECK_FLAGS(flags, FLAG_B | FLAG_C) NEGATIVE_BREAK(rc = doBC());
        rc = 0;

    ONESHOT_LOOP_END

    return rc;
}

int main() {
    printf("Result : %d\n", test(FLAG_A|FLAG_C));
    return 0;
}
```

## String

### Convert IP address to String

```cpp
#include <arpa/inet.h>
#include <netinet/in.h>
#include <iostream>
#include <string>

std::string ipToString(const unsigned char ip[], bool is_v4_or_v6) {
    char ip_buf[INET6_ADDRSTRLEN] = { 0 };
    return inet_ntop(is_v4_or_v6 ? AF_INET : AF_INET6, ip, ip_buf,
                     is_v4_or_v6 ? INET_ADDRSTRLEN : INET6_ADDRSTRLEN);
}

int main(void) {
    // IPv4 : 192.168.0.1
    // IPv6 : 2001:db8:85a3::8a2e:370:7334
    unsigned char ipv4[4] = { 192, 168, 0, 1 };
    unsigned char ipv6[16] = { 32, 1, 13, 184, 133, 163, 0, 0, 0, 0, 138, 46, 3, 112, 115, 52 };

    std::string ipv4_str = ipToString(ipv4, true);
    std::string ipv6_str = ipToString(ipv6, false);

    std::cout << "IPv4 : " << ipv4_str << std::endl;
    std::cout << "IPv6 : " << ipv6_str << std::endl;
}
```

### Convert String to IP address

```cpp
#include <arpa/inet.h>
#include <netinet/in.h>
#include <string>

int stringToIp(const std::string &ip_str, unsigned char ip[], bool is_v4_or_v6) {
    return inet_pton(is_v4_or_v6 ? AF_INET : AF_INET6, ip_str.c_str(), ip);
}

int main(void) {
    struct in_addr ipv4_addr;
    struct in6_addr ipv6_addr;
    int rc;
    rc = stringToIp("192.168.0.1", reinterpret_cast<unsigned char *>(&ipv4_addr), true);
    printf("rc = %d, { ", rc);
    for (int i = 0 ; i < 4 ; i++) {
        printf("%d%s", ((unsigned char *)&ipv4_addr)[i], (i < 3 ? ", " : " }\n"));
    }
    rc = stringToIp("2001:db8:85a3::8a2e:370:7334", reinterpret_cast<unsigned char *>(&ipv6_addr), false);
    printf("rc = %d, { ", rc);
    for (int i = 0 ; i < 16 ; i++) {
        printf("%d%s", ((unsigned char *)&ipv6_addr)[i], (i < 15 ? ", " : " }\n"));
    }
    return 0;
}
```

### Reverse File Path

```cpp
#include <algorithm>
#include <cstddef>
#include <string>

std::string reversePath(std::string reversed) {
    std::string ret;
    std::reverse(reversed.begin(), reversed.end());

    size_t start = 0;
    size_t end = 0;
    while ((end = reversed.find('/', start)) != std::string::npos) {
        std::string eachFile = reversed.substr(start, end - start);
        std::reverse(eachFile.begin(), eachFile.end());
        ret += eachFile + "/";
        start = end + 1;
    }

    std::string lastFile = reversed.substr(start);
    if (!lastFile.empty()) {
        std::reverse(lastFile.begin(), lastFile.end());
        ret += lastFile;
    }
    return ret;
}

std::string reversedPath = "eEe/DdD/ccc/Bbb/ABC/";
std::string normalPath = reversePath(reversedPath);
```

## Thread

### Sleep
   
```cpp
#include <chrono>
#include <thread>

std::this_thread::sleep_for(std::chrono::seconds(5));

std::this_thread::sleep_for(std::chrono::milliseconds(5000));

std::this_thread::sleep_for(std::chrono::microseconds(5000000));

std::this_thread::sleep_for(std::chrono::nanoseconds(5000000000));
```

