# Call_Rust_From_C
example to call rust from c , tested under linux mint 64bit


# how  : 
- **create cargo pkg :**  
```cargo new rustcalls --lib```

- **add to toml  :** 
```
[lib]
crate-type =["cdylib"]
```

- **build library with command  :** 
```
cargo build

``` 

- **compile c program with  :**   
```
gcc -g call_rust.c -o call_rust  -lrustcalls -L./../target/debug -Wl,-rpath=./../target/debug

```


# src/lib.rs 
```
use std::ffi::{c_void, CStr};
use std::os::raw::c_char;
use std::slice;

// A Rust struct mapping the C struct
#[repr(C)]
#[derive(Debug)]
pub struct RustStruct {
    pub c: char,
    pub ul: u64,
    pub c_string: *const c_char,
}

macro_rules! create_function {
    // This macro takes an argument of designator `ident` and
    // creates a function named `$func_name`.
    // The `ident` designator is used for variable/function names.
    ($func_name:ident, $ctype:ty) => {
        #[no_mangle]
        pub extern "C" fn $func_name(v: $ctype) {
            // The `stringify!` macro converts an `ident` into a string.
            println!(
                "{:?}() is called, value passed = <{:?}>",
                stringify!($func_name),
                v
            );
        }
    };
}

// create simple functions where C type is exactly mapping a Rust type
create_function!(rust_char, char);
create_function!(rust_wchar, char);
create_function!(rust_short, i16);
create_function!(rust_ushort, u16);
create_function!(rust_int, i32);
create_function!(rust_uint, u32);
create_function!(rust_long, i64);
create_function!(rust_ulong, u64);
create_function!(rust_void, *mut c_void);

// for NULL-terminated C strings, it's a little bit clumsier
#[no_mangle]
pub extern "C" fn rust_string(c_string: *const c_char) {
    // build a Rust string from C string
    let s = unsafe { CStr::from_ptr(c_string).to_string_lossy().into_owned() };

    println!("rust_string() is called, value passed = <{:?}>", s);
}

// for C arrays, need to pass array size
#[no_mangle]
pub extern "C" fn rust_int_array(c_array: *const i32, length: usize) {
    // build a Rust array from array & length
    let rust_array: &[i32] = unsafe { slice::from_raw_parts(c_array, length as usize) };
    println!(
        "rust_int_array() is called, value passed = <{:?}>",
        rust_array
    );
}

#[no_mangle]
pub extern "C" fn rust_string_array(c_array: *const *const c_char, length: usize) {
    // build a Rust array from array & length
    let tmp_array: &[*const c_char] = unsafe { slice::from_raw_parts(c_array, length as usize) };

    // convert each element to a Rust string
    let rust_array: Vec<_> = tmp_array
        .iter()
        .map(|&v| unsafe { CStr::from_ptr(v).to_string_lossy().into_owned() })
        .collect();

    println!(
        "rust_string_array() is called, value passed = <{:?}>",
        rust_array
    );
}

// for C structs, need to convert each individual Rust member if necessary
#[no_mangle]
pub unsafe extern "C" fn rust_cstruct(c_struct: *mut RustStruct) {
    let rust_struct = &*c_struct;
    let s = CStr::from_ptr(rust_struct.c_string)
        .to_string_lossy()
        .into_owned();

    println!(
        "rust_cstruct() is called, values passed = <{} {} {}>",
        rust_struct.c, rust_struct.ul, s
    );
}



```


# src/call_rust.c : 

```


#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <wchar.h>

// sample struct to illustrate passing a C-struct to Rust
struct CStruct {
    char c;
    unsigned long ul;
    char *s;
};

// functions called in the Rust library
extern void rust_char(char c);
extern void rust_wchar(wchar_t c);
extern void rust_short(short i);
extern void rust_ushort(unsigned short i);
extern void rust_int(int i);
extern void rust_uint(unsigned int i);
extern void rust_long(long i);
extern void rust_ulong(unsigned long i);
extern void rust_string(char *s);
extern void rust_void(void *s);
extern void rust_int_array(const int *array, int length);
extern void rust_string_array(const char **array, int length);
extern void rust_cstruct(struct CStruct *c_struct);

int main() {
    // pass char to Rust
    rust_char('A');
    rust_wchar(L'ζ');

    // pass short to Rust
    rust_short(-100);
    rust_ushort(100);

    // pass int to Rust
    rust_int(-10);
    rust_uint(10);

    // pass long to Rust
    rust_long(-1000);
    rust_ulong(1000);    

    // pass a NULL terminated string
    rust_string("hello world");

    // pass a void* pointer
    void *p = malloc(1000);
    rust_void(p);  

    // pass an array of ints
    int digits[10] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9}; 
    rust_int_array(digits, 10);

    // pass an array of c strings
    const char *words[] = { "This", "is", "an", "example"};
    rust_string_array(words, 4);   

    // pass a C struct
    struct CStruct c_struct;
    c_struct.c = 'A';
    c_struct.ul = 1000;
    c_struct.s = malloc(20);
    strcpy(c_struct.s, "0123456789");
    rust_cstruct(&c_struct);

    // don't forget to clean up ;-)
    free(p); 
    free(c_struct.s);

    return 0;
}


```

# library from rust can called dierctly to nim : 
- **Exmaple code nim** :       
```
const libName* = "full path to ............. librustcalls.so"

proc rust_int*(flags: int32)  : cint {.importc: "rust_int", dynlib: libName.}

proc rust_void*(flags: pointer)  : cint {.importc: "rust_void", dynlib: libName.}

echo rust_int(16)

```






