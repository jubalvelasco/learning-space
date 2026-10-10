#### > copy an array

```c
  memcpy(DESTINATION_ARRAY,SOURCE_ARRAY,sizeof(DATA_TYPE)*NUM_ELEMENTS); // add std:: in front for C++
```

#### > printing hex format in printf()

```c++
printf("%08x", num);
```

- the **08** means that the hex value will be zero-padded to a minimum width of 8 hex characters (which represents 4 bytes)
- a lowercase **x** will display lowercase hex letters a-f, and an uppercase **X** will display uppercase letters A-F
