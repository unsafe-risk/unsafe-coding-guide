# C++ UTF8 String Get Length

```c++
#include <string>

auto utf8_length_of(std::string const& s) -> size_t {
    size_t count = 0;
    for (int i = 0; i < s.size();) {
        if ((s[i] & 0b10000000) == 0b00000000) {
            count++;
            i+=1;
            continue;
        }
        if ((s[i] & 0b11100000) == 0b11000000) {
            count++;
            i+=2;
            continue;
        }
        if ((s[i] & 0b11110000) == 0b11100000) {
            count++;
            i+=3;
            continue;
        }
        if ((s[i] & 0b11111000) == 0b11110000) {
            count++;
            i+=4;
            continue;
        }
        i++;
    }
    return count;
}
```

#C++ #UTF8 #length