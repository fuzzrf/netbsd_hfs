NetBSD hfs local overflow

Bug details:
```
size_t
hfslib_reada_node_offsets(void* in_bytes, uint16_t* out_offset_array)
{
        void*           ptr;

        if (in_bytes == NULL || out_offset_array == NULL)
                return 0;

        ptr = in_bytes;

        /*
         * The offset for record 0 (which is the very last offset in the node) is
         * always equal to 14, the size of the node descriptor. So, once we hit
         * offset=14, we know this is the last offset. In this way, we don't need
         * to know the number of records beforehand.
         */
        out_offset_array--;
        do {
                out_offset_array++;
                *out_offset_array = be16tohp(&ptr);
        } while (*out_offset_array != (uint16_t)14);

        return ((uint8_t*)ptr - (uint8_t*)in_bytes);
}
```

while loop has no range checks, so we can overflow out_offset_array array.


How to test:
```
1. enable user mounts
# sysctl -w vfs.generic.usermount=1


2. mount test.img
```

