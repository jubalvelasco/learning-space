# Reading & Writing from a binary file
    
## Open Modes

| Open mode | Effects | Must the file Exist? |
| --- | --- | --- |
| ```ios::in \| ios::out``` | To open the file for input and output. | yes |
| ```ios::in \| ios::out \| ios::trunc``` | Opens the file for input and output. If the file already exists, it will be truncated. | no |
| ```ios::in \| ios::out \| ios::app``` | Opens the file for input and output. If the file does not exist, it will be created. Before each write, a seek to the end is performed (C++11 and later). | no |

---------
    
## Position pointers for file streams (or any other stream classes)
    
- ```tellp()``` - returns a `std::streampos` (pos_type) that tells the current position of the put pointer (write)
- ```tellg()``` - returns a `std::streampos` (pos_type) that tells the current position of the get pointer (read)
- ```seekp(pos)``` / ```seekp(offset,posflag)``` - modifies the current position of the put pointer (write) [If you do not specify a positioning flag, the position is an absolute position, i.e. relative to
the beginning of the file]
- ```seekg(pos)``` / ```seekg(offset,posflag)``` - modifies the current position of the get pointer (read) [If you do not specify a positioning flag, the position is an absolute position, i.e. relative to
the beginning of the file]

---------

## Position Flags
    
- ```ios::beg``` - Beginning of the file
- ```ios::cur``` - Current position
- ```ios::end``` - End of the file

---------

## State Flags
    
- `ios::eofbit` - end of file reached
- `ios::failbit` - last read or write operation failed
- `ios::badbit` - an irrecoverable error occurred
- `ios::goodbit` - the stream is ok, i.e. no other state flag is set (its value is `0`).

    
## Methods to discover the current state flags
    
- `eof()` - returns `true` if `eofbit` is set
- `fail()` - returns `true` if `failbit` **or** `badbit` is set
- `bad()` - returns `true` if `badbit` is set
- `good()` - returns `true` if **no** flag is set (`rdstate() == ios::goodbit`)

## Check for end of file

```c++
if( fstr.eof() ) {}
// or
if( fstr.rdstate() & ios::eofbit ) {}
```

## Read current state

```c++
// use bitwise AND, since more than one flag can be set at the same time (e.g. eofbit and failbit)
(stream.rdstate() & <STATUS_FLAG>) != 0
```

## Clear current status flag

```c++
stream.clear()
```

## Write format
    
```c++
file_stream.write(reinterpret_cast<const char*>(&VAR/ARRAY[INDEX]),sizeof(DATA_TYPE_OF_VAR/ARRAY)*NUM_OF_ELEMENT_YOU_WANT_TO_WRITE);
```
    
## Read format
    
```c++
file.read(reinterpret_cast<char*>(&VAR/ARRAY[INDEX]),sizeof(DATA_TYPE_OF_VAR/ARRAY)*NUM_OF_ELEMENT_YOU_WANT_TO_READ);
```
    
> [!NOTE]
> for **single variables** the `NUM_OF_ELEMENT_YOU_WANT_TO_WRITE/READ` is equal to `1`
> or you can omit the `*NUM_OF_ELEMENT_YOU_WANT_TO_WRITE/READ`.

---

## Reading

```C++
std::ifstream binFile("binaryFile", std::ios::in | std::ios::binary);
if (!binFile.is_open()) throw std::invalid_argument("the given file might not exist");

unsigned char binValues[8];
binFile.read(reinterpret_cast<char*>(&binValues[0]),sizeof(unsigned char)*8);
```
    
## Writing

```C++
std::ofstream binFile("binaryFile", std::ios::out | std::ios::binary); // creates the file if it does not exist
if (!binFile.is_open()) throw std::runtime_error("the file could not be created/opened");
    
unsigned char binValues[8];
binFile.write(reinterpret_cast<const char*>(&binValues[0]),sizeof(unsigned char)*8);
```
    
## Sample Program

```c++
#include <iostream>
#include <fstream>
#include <stdexcept>

int main()
{
    #ifdef WRITE_PROG
        // NOTE: std::ios::in | std::ios::out would fail if the file does not exist yet, so only use std::ios::out here
        std::fstream writeBinFile("binaryFile", std::ios::out | std::ios::binary);
        if (!writeBinFile.is_open()) throw std::runtime_error("the file could not be created/opened");

        unsigned char binValues[8] { 't','h','i','s',' ','h','o','t' };
        long negative = -777;

        writeBinFile.write(reinterpret_cast<const char*>(&binValues[0]),sizeof(unsigned char)*4);
        writeBinFile.write(reinterpret_cast<const char*>(&negative),sizeof(long)*1);
        writeBinFile.write(reinterpret_cast<const char*>(&binValues[4]),sizeof(unsigned char)*4);
        writeBinFile.close();
    #endif

    #ifdef READ_PROG
        std::fstream readBinFile("binaryFile", std::ios::in | std::ios::out | std::ios::binary);
        if (!readBinFile.is_open()) throw std::runtime_error("the given file might not exist");

        unsigned char readValues[9] {}; // 8 chars + 1 null terminator, needed because it is printed as a C-string below
        long readNum;
        readBinFile.read(reinterpret_cast<char*>(&readValues[0]),sizeof(unsigned char)*4);
        readBinFile.read(reinterpret_cast<char*>(&readNum),sizeof(long)*1);
        readBinFile.read(reinterpret_cast<char*>(&readValues[4]),sizeof(unsigned char)*4);
        readValues[8] = '\0';
        std::cout << "read = " << readValues << "\n";
        std::cout << "number = " << readNum << "\n";
        readBinFile.close();
    #endif
}
```
    
----------
    
## Reading files by chunk

- calling the `write()` method will automatically change the position of put pointer
- calling the `read()` method will automatically change the position of get pointer

```c++
#define CHUNK_SIZE 1024 // no semicolon here, otherwise `new char[CHUNK_SIZE]` becomes `new char[1024;]`
    
char *tbuffer = new char[CHUNK_SIZE]; // CHUNK_SIZE in bytes
char optional_initial_reads[16];

std::ifstream curr_file(filename,std::ios::binary);

curr_file.read(optional_initial_reads,16);
    
// `read()` returns the stream, which converts to false when it fails (e.g. it hit EOF before reading CHUNK_SIZE bytes),
// so also check gcount() to still process the last partial chunk.
// This avoids the classic `while(!eof())` bug, where an extra empty iteration happens
// when the file size is an exact multiple of CHUNK_SIZE.
while(curr_file.read(tbuffer,CHUNK_SIZE) || curr_file.gcount() > 0)
{
    size_t read_buffer_size = curr_file.gcount();

    if(!curr_file.eof())
    {
        // not last chunk operations (read_buffer_size == CHUNK_SIZE)
    }
    else
    {
        // last chunk operations (read_buffer_size <= CHUNK_SIZE)
        break;
    }
}

delete[] tbuffer;
```

----- 

## Different Examples of Read/Write Operation - ChatGPT generated

```c++
// file_ops.cpp
#include <iostream>
#include <fstream>
#include <vector>
#include <string>
#include <cstdint>
#include <filesystem>
#include <system_error>
#include <algorithm>
#include <functional>
#include <array>
#include <cstring>
#include <cerrno>
#include <stdexcept>

namespace fs = std::filesystem;

// Utility typedef
using byte = uint8_t;

// Default chunk size (64 KiB) — tweak for your IO subsystem
constexpr std::size_t DEFAULT_CHUNK = 64 * 1024;

// --- 1) Chunk / block reading (binary) ---
void read_file_chunked(const fs::path& p,
                       std::size_t chunk_size = DEFAULT_CHUNK,
                       std::function<void(const std::vector<byte>&, std::size_t)> on_chunk = {})
{
    std::ifstream in(p, std::ios::binary);
    if (!in) throw std::system_error(errno, std::generic_category(), "open for read failed");

    std::vector<byte> buf;
    buf.resize(chunk_size);

    std::size_t offset = 0;
    while (in) {
        in.read(reinterpret_cast<char*>(buf.data()), static_cast<std::streamsize>(buf.size()));
        std::streamsize readn = in.gcount();
        if (readn <= 0) break;
        buf.resize(static_cast<std::size_t>(readn));
        if (on_chunk) on_chunk(buf, offset);
        offset += static_cast<std::size_t>(readn);
        buf.resize(chunk_size); // reuse buffer capacity
    }
}

// --- 2) Chunk / block writing (binary) ---
// Append style writing of a stream of chunks (or write from the start after truncation)
void write_file_chunked(const fs::path& p,
                        std::vector<std::vector<byte>> const& chunks,
                        bool truncate_before = true)
{
    // trunc -> erase existing content, app -> every write goes to the end of the file
    // (both modes create the file if it does not exist)
    std::ofstream out(p, std::ios::binary | (truncate_before ? std::ios::trunc : std::ios::app));
    if (!out) throw std::system_error(errno, std::generic_category(), "open for write failed");

    for (auto const& c : chunks) {
        out.write(reinterpret_cast<const char*>(c.data()), static_cast<std::streamsize>(c.size()));
        if (!out) throw std::system_error(errno, std::generic_category(), "write failed");
    }
}

// --- 3) Read B bytes from position N ---
std::vector<byte> read_at(const fs::path& p, std::uint64_t pos, std::size_t length)
{
    std::ifstream in(p, std::ios::binary);
    if (!in) throw std::system_error(errno, std::generic_category(), "open for read failed");

    std::uint64_t sz = fs::file_size(p);
    if (pos >= sz) return {}; // empty

    std::size_t to_read = static_cast<std::size_t>(std::min<std::uint64_t>(length, sz - pos));
    std::vector<byte> buf;
    buf.resize(to_read);

    in.seekg(static_cast<std::streamoff>(pos));
    in.read(reinterpret_cast<char*>(buf.data()), static_cast<std::streamsize>(to_read));
    std::streamsize got = in.gcount();
    buf.resize(static_cast<std::size_t>(got));
    return buf;
}

// --- 4) Insert a sequence of bytes at position N ---
// Approach: resize file larger, then move trailing data in backward-chunks to make space, then write new bytes.
void insert_bytes(const fs::path& p, std::uint64_t pos, const std::vector<byte>& data, std::size_t chunk_size = DEFAULT_CHUNK)
{
    if (data.empty()) return;

    // Validate file exists
    if (!fs::exists(p)) throw std::runtime_error("file does not exist");

    auto size = fs::file_size(p);
    if (pos > size) pos = size;

    // Open read/write fstream
    std::fstream f(p, std::ios::binary | std::ios::in | std::ios::out);
    if (!f) throw std::system_error(errno, std::generic_category(), "open for read/write failed");

    // increase file size to hold the insertion
    fs::resize_file(p, size + data.size());

    // Move blocks from end -> make room
    std::vector<byte> buf;
    buf.resize(chunk_size);

    // We'll copy ranges [read_pos, read_pos + have) to write_pos = read_pos + data.size()
    // iterate read_pos from end-chunk down to pos
    std::uint64_t read_pos = size;
    while (read_pos > pos) {
        std::size_t this_chunk = static_cast<std::size_t>(std::min<std::uint64_t>(chunk_size, read_pos - pos));
        read_pos -= this_chunk; // new read_pos
        f.seekg(static_cast<std::streamoff>(read_pos));
        f.read(reinterpret_cast<char*>(buf.data()), static_cast<std::streamsize>(this_chunk));
        if (f.gcount() != static_cast<std::streamsize>(this_chunk))
            throw std::runtime_error("short read while shifting");

        // write to read_pos + data.size()
        f.seekp(static_cast<std::streamoff>(read_pos + data.size()));
        f.write(reinterpret_cast<const char*>(buf.data()), static_cast<std::streamsize>(this_chunk));
        if (!f) throw std::runtime_error("write failed during shifting");
    }

    // Now write inserted data at pos
    f.seekp(static_cast<std::streamoff>(pos));
    f.write(reinterpret_cast<const char*>(data.data()), static_cast<std::streamsize>(data.size()));
    if (!f) throw std::runtime_error("write failed for inserted data");
    f.flush();
}

// --- 5) Find a sequence of bytes in a file (first occurrence) ---
// Efficient streaming search using Boyer-Moore-Horspool on chunked windows with overlap.
std::uint64_t find_sequence_first(const fs::path& p,
                                  const std::vector<byte>& pattern,
                                  std::size_t chunk_size = DEFAULT_CHUNK)
{
    if (pattern.empty()) return static_cast<std::uint64_t>(-1);

    std::uint64_t filesize = fs::file_size(p);
    if (pattern.size() > filesize) return static_cast<std::uint64_t>(-1);

    // Build BMH bad-char table for bytes
    std::array<std::size_t, 256> bad;
    bad.fill(pattern.size());
    for (std::size_t i = 0; i + 1 < pattern.size(); ++i) {
        bad[pattern[i]] = pattern.size() - 1 - i;
    }

    std::ifstream in(p, std::ios::binary);
    if (!in) throw std::system_error(errno, std::generic_category(), "open for read failed");

    // keep the last (pattern.size() - 1) bytes of each chunk so that matches
    // crossing a chunk boundary are not missed
    const std::size_t overlap = pattern.size() - 1;
    std::vector<byte> buffer;
    buffer.resize(chunk_size + overlap);

    std::uint64_t buffer_start = 0; // file offset of buffer[0]
    std::size_t carried = 0;        // number of bytes carried over from the previous chunk
    while (in) {
        // read the next chunk right after the carried-over bytes
        in.read(reinterpret_cast<char*>(buffer.data() + carried), static_cast<std::streamsize>(chunk_size));
        std::streamsize readn = in.gcount();
        if (readn <= 0) break;
        std::size_t window = carried + static_cast<std::size_t>(readn);

        // run BMH on buffer[0..window)
        std::size_t idx = 0;
        while (idx + pattern.size() <= window) {
            // compare from end
            std::size_t j = pattern.size() - 1;
            while (j < pattern.size() && buffer[idx + j] == pattern[j]) {
                if (j == 0) break;
                --j;
            }
            if (j == 0 && buffer[idx] == pattern[0]) {
                return buffer_start + idx;
            }
            // shift using bad-char
            unsigned char next = buffer[idx + pattern.size() - 1];
            idx += bad[next];
        }

        // move the last `overlap` bytes to the front for the next read
        std::size_t keep = std::min(overlap, window);
        std::memmove(buffer.data(), buffer.data() + window - keep, keep);
        buffer_start += window - keep;
        carried = keep;
    }

    return static_cast<std::uint64_t>(-1); // not found
}

// --- 6) Find a sequence and remove it (first occurrence) ---
// Find the first occurrence, then shift trailing bytes forward and resize file smaller.
bool find_and_remove_first(const fs::path& p, const std::vector<byte>& pattern, std::size_t chunk_size = DEFAULT_CHUNK)
{
    auto pos = find_sequence_first(p, pattern, chunk_size);
    if (pos == static_cast<std::uint64_t>(-1)) return false;

    // shift content after pos+pattern.size() backward to pos
    std::uint64_t file_sz = fs::file_size(p);
    std::uint64_t read_from = pos + pattern.size();
    std::uint64_t write_to = pos;

    std::fstream f(p, std::ios::binary | std::ios::in | std::ios::out);
    if (!f) throw std::system_error(errno, std::generic_category(), "open for r/w failed");

    std::vector<byte> buf;
    buf.resize(chunk_size);

    while (read_from < file_sz) {
        std::size_t this_chunk = static_cast<std::size_t>(std::min<std::uint64_t>(chunk_size, file_sz - read_from));
        f.seekg(static_cast<std::streamoff>(read_from));
        f.read(reinterpret_cast<char*>(buf.data()), static_cast<std::streamsize>(this_chunk));
        if (f.gcount() != static_cast<std::streamsize>(this_chunk)) throw std::runtime_error("short read during removal");
        f.seekp(static_cast<std::streamoff>(write_to));
        f.write(reinterpret_cast<const char*>(buf.data()), static_cast<std::streamsize>(this_chunk));
        if (!f) throw std::runtime_error("write failed during removal");
        read_from += this_chunk;
        write_to += this_chunk;
    }

    // shrink file
    fs::resize_file(p, file_sz - pattern.size());
    return true;
}

// --- 7) Save a file with different permission configuration ---
void set_permissions(const fs::path& p, fs::perms permissions)
{
    fs::permissions(p, permissions, fs::perm_options::replace);
}

// Example helper to set POSIX-like rw-r----- (owner read/write, group read, others none)
void set_posix_rwx_example(const fs::path& p)
{
    fs::perms perms = fs::perms::owner_read | fs::perms::owner_write | fs::perms::group_read;
    set_permissions(p, perms);
}

// --- 8) Read/edit a write-protected file (attempt to change permission, edit, then optionally restore) ---
// Note: Changing permissions may fail if you lack privileges. Always handle errors.
bool edit_write_protected_file(const fs::path& p, std::function<void()> edit_action)
{
    auto st = fs::status(p);
    fs::perms cur = st.permissions();
    bool was_writable = static_cast<bool>(cur & fs::perms::owner_write);

    if (!was_writable) {
        // try add owner write
        try {
            fs::permissions(p, fs::perms::owner_write, fs::perm_options::add);
        } catch (...) {
            return false; // cannot make writable
        }
    }

    // perform edit
    try {
        edit_action();
    } catch (...) {
        // try to restore perms then rethrow
        if (!was_writable) {
            try { fs::permissions(p, cur, fs::perm_options::replace); } catch (...) {}
        }
        throw;
    }

    // restore
    if (!was_writable) {
        try { fs::permissions(p, cur, fs::perm_options::replace); } catch (...) {}
    }
    return true;
}

// ------------------------
// Simple demonstration main
// ------------------------
int main()
{
    try {
        fs::path f = "example.bin";

        // Prepare an example file with 1 MiB of incremental bytes
        {
            std::ofstream out(f, std::ios::binary | std::ios::trunc);
            std::vector<byte> buf(1024);
            for (int i = 0; i < 1024; ++i) buf[i] = static_cast<byte>(i & 0xFF);
            for (int k = 0; k < 1024; ++k) out.write(reinterpret_cast<const char*>(buf.data()), static_cast<std::streamsize>(buf.size()));
        }

        // 1) chunked read: print first 3 chunks sizes
        std::cout << "Chunked read first 3 chunks sizes:\n";
        int cnt = 0;
        read_file_chunked(f, 64*1024, [&](const std::vector<byte>& b, std::size_t off){
            if (cnt < 3) std::cout << " chunk at offset " << off << " size " << b.size() << "\n";
            ++cnt;
        });

        // 2) chunked write: append two small chunks
        std::vector<std::vector<byte>> chunks;
        chunks.push_back(std::vector<byte>{0xAA, 0xBB, 0xCC});
        chunks.push_back(std::vector<byte>{0x11, 0x22});
        write_file_chunked(f, chunks, false); // append

        // 3) read at position
        auto slice = read_at(f, 1024, 16);
        std::cout << "Read 16 bytes at pos 1024: ";
        for (auto b : slice) std::cout << std::hex << (int)b << ' ';
        std::cout << std::dec << '\n';

        // 4) insert bytes
        std::vector<byte> ins = {0xDE, 0xAD, 0xBE, 0xEF};
        insert_bytes(f, 200, ins);
        std::cout << "Inserted 4 bytes at pos 200\n";

        // 5) find a sequence
        std::vector<byte> pat = {0xDE, 0xAD, 0xBE, 0xEF};
        auto found = find_sequence_first(f, pat);
        std::cout << "Found pattern at: " << (found == static_cast<std::uint64_t>(-1) ? -1 : (long long)found) << "\n";

        // 6) find and remove
        bool removed = find_and_remove_first(f, pat);
        std::cout << "Removed pattern? " << (removed ? "yes" : "no") << "\n";

        // 7) set permissions (example)
        set_posix_rwx_example(f);
        std::cout << "Permissions set (owner rw, group r)\n";

        // 8) attempt to edit write-protected file (example: append a byte)
        auto edit_ok = edit_write_protected_file(f, [&](){
            std::ofstream out(f, std::ios::binary | std::ios::app);
            out.put(0x77);
        });
        std::cout << "Edit write-protected file success: " << (edit_ok ? "yes" : "no") << "\n";

    } catch (std::exception& ex) {
        std::cerr << "Error: " << ex.what() << "\n";
        return 2;
    }
    return 0;
}
```