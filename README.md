# E-Agle TRT
# Serializers

Serializers is a code generation library for C++ to be used with Google's _Protocol Buffers_.

Given the .proto files, it generates a header file and a source file for each proto, besides creating a CMakeLists.txt file
to integrate the code with your main project and a header "serializers.h".

The library wraps the pb files generated by _protoc_, providing a simple and convenient interface to handle, serialize
and deserialize the defined proto messages. It also provides functions to convert and parse the messages to JSON format.

The generated header consist in a struct for each proto Message and the following methods for serialization:
```
std::string serializeAsJsonString() const;
std::string serializeAsProtobufString() const;
bool deserializeFromJsonString(const std::string& str);
bool deserializeFromProtobufString(const std::string& str);
```
The methods implementation is built upon the pb files, which have to be added to the main project and included in the global include path.

## Example
### bar.proto (input)
```
syntax = "proto3";

package bar;

enum MyEnum
{
    MY_ENUM_0 = 0;
    My_ENUM_1 = 1;
}

message MySmallMessage
{
    int64 val = 1;
}

message MyMessage
{
    double val = 1;
    string str = 2;
    MySmallMessage mySmallMessage = 3;
    repeated MyEnum enumVec = 4;
    map<uint32, MySmallMessage> structMap = 5;
}
```

### bar.h (output)
```
#ifndef BAR_H
#define BAR_H

#include "bar.pb.h"

[...]

namespace Serializers
{
enum class MyEnum
{
    MY_ENUM_0 = 0,
    My_ENUM_1 = 1
};

struct MySmallMessage
{
    int64_t val;

    [...]
};

struct MyMessage
{
    double val;
    std::string str;
    MySmallMessage mySmallMessage;
    std::vector<MyEnum> enumVec;
    std::unordered_map<uint32_t, MySmallMessage> structMap;

    MyMessage() = default;
    MyMessage(const bar::MyMessage& proto);
    operator bar::MyMessage() const;

    std::string serializeAsJsonString() const;
    std::string serializeAsProtobufString() const;
    bool deserializeFromJsonString(const std::string& str);
    bool deserializeFromProtobufString(const std::string& str);
};
}

#endif
```

## Usage
- Generate the code with src/generator/main.py *proto_dir* *output_dir*.
- Copy the output directory in your project and generate the pb files with _protoc_, using
the files in output_dir/proto.
- Add add_subdirectory(path/to/output/directory) to your main CMakeLists.txt.
- Include "serializers.h" in your files and use the library code.

```
#include "serializers.h"

#include <iostream>
#include <string>

int main()
{
    Serializers::MySmallMessage mySmallMessage;
    mySmallMessage.val = 3.14;

    Serializers::MyMessage myMessage;
    myMessage.str = "Federico Carbone";
    myMessage.enumVec.push_back(Serializers::MyEnum::MY_ENUM_0);
    myMessage.structMap.emplace(404, mySmallMessage);

    std::string myMessageSerialized = myMessage.serializeAsProtobufString();

    std::cout << myMessage.serializeAsJsonString() << std::endl;

    return 0;
}
```

## Features
### Implemented / Not implemented yet
- [x] Multiple file packages
- [x] All primitive fieldtypes support
- [x] Enum
- [x] "repeated" option
- [x] Maps
- [x] Package check
- [ ] Import external proto definitions
- [ ] Nested types
- [ ] OneOf
- [ ] Custom options
