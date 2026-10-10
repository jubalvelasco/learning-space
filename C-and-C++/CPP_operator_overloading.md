### Overloading >> and << operator
```c++
#include <iostream>
#include <cstring>

class HexString
{
    public:

    unsigned char* data;
    size_t length;

    HexString(unsigned char* src, size_t n) : data(new unsigned char[n]), length(n)
    {
        memcpy(data,src,n);
    }

    HexString(size_t n) : data(new unsigned char[n]), length(n) {}

    // this class owns a raw pointer, so copying must be disabled (or implemented properly),
    // otherwise two copies would delete the same memory (Rule of Three/Five)
    HexString(const HexString&) = delete;
    HexString& operator=(const HexString&) = delete;

    ~HexString()
    {
        delete [] data;
    }
};

std::ostream& operator<<(std::ostream& outputStream, const HexString& instance)
{
    for(size_t i=0; i<instance.length; ++i)
        outputStream << static_cast<unsigned int>(instance.data[i]) << ' ';
    return outputStream;
}

std::istream& operator>>(std::istream& inputStream, HexString& instance)
{
    // NOTE: reading directly into an `unsigned char` would read a single character,
    // not a number, so read into an `unsigned int` (as hex) first and then narrow it
    for(size_t i=0; i<instance.length; ++i)
    {
        unsigned int value = 0;
        inputStream >> std::hex >> value;
        instance.data[i] = static_cast<unsigned char>(value);
    }
    return inputStream;
}

int main()
{
    unsigned char arr[] = { 0x80, 0xf7, 0x32, 0xfd };

    HexString hexcontain(arr,sizeof(arr));

    std::cout.setf(std::ios::showbase);
    std::cout.setf(std::ios::hex, std::ios::basefield);
    std::cout << hexcontain << "\n";

    std::cin >> hexcontain;
    std::cout << hexcontain << "\n";
}
```

