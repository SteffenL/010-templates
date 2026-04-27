# Dead Island Save Encoding

## Compression

gzip when there's compression.

## Checksum

PS3 saves have a checksum stored in the `SAVEDATA_LIST_PARAM` in the `PARAM.SFO` file:

```cpp
// Hash length: 8 bytes
void generateHash(const void* data, size_t size, char* hash) {
    if (size == 0) {
        throw std::runtime_error("Zero-length data is not permissible");
    }

    ::memset(hash, 0, 8);
    for (int64_t r7 = 0; r7 < size; ++r7) {
        int64_t r6 = *(reinterpret_cast<const char*>(data) + r7);
        if (!r6) {
            continue;
        }

        int64_t r8 = 0x24924925;
        r8 = (((uint64_t)r8 & 0x00000000ffffffff) * ((uint64_t)r7 & 0x00000000ffffffff));
        r8 = ((uint64_t)r8 & 0xffffffff00000000) >> 32;
        int64_t r9 = ((r7 - r8) >> 1);
        r9 = ((uint64_t)r9 & ~0xffffffff00000000);
        r8 = ((r8 + r9) >> 2);
        r8 = ((uint64_t)r8 & ~0xffffffff00000000);
        r9 = (r8 << 3);
        r8 = (r9 - r8);
        r9 = 0xffffffff828CBFBF;
        r8 = (r7 - r8);
        int64_t r10 = hash[r8];
        r6 = r6 + r10;
        r9 = (((uint64_t)r9 & 0x00000000ffffffff) * ((uint64_t)r6 & 0x00000000ffffffff));
        r9 = ((uint64_t)r9 & 0xffffffff00000000) >> 32;
        r9 = r9 >> 7;
        r10 = r9 << 8;
        int64_t r11 = r9 << 2;
        r9 = r10 - r9;
        r9 = r9 - r11;
        r6 = r6 - r9;
        hash[r8] = static_cast<char>(r6);
    }

    for (int i = 0; i < 7; ++i) {
        uint64_t r4 = hash[i];
        uint64_t r6 = r4 + 0xD3;
        uint64_t r7 = r6 << 6;
        r6 += r7;
        r7 = r6 + r6;
        r6 += r7;
        r6 = r6 >> 12;
        r7 = r6 + r6;
        r6 += r7;
        r7 = r6 << 3;
        r6 = r7 - r6;
        r4 = r4 - r6;
        r4 += 0x113;
        hash[i] = static_cast<char>(r4);
    }
}
```

