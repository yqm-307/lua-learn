## Lua中的值类型

#### Value 和 TValue
在lua中值类型是一个union，是为了兼容不同类型值的一个联合体。这样就没有继承关系就可以很好的使用


```C
typedef union Value {
  struct GCObject *gc;    
  void *p;         
  lua_CFunction f; 
  lua_Integer i;   
  lua_Number n;    
  /* not used, but may avoid warnings for uninitialized value */
  lu_byte ub; 
} Value;


#define TValuefields	Value value_; lu_byte tt_
// 这个TValue 可以表示几乎所有的lua对象了
typedef struct TValue {
    //   TValuefields;  展开
    Value value_; 
    lu_byte tt_;
} TValue;
```

关于 lu_byte 在 lua 中的定义如下:
```C
/* chars used as small naturals (so that 'char' is reserved for characters) */
typedef unsigned char lu_byte;
typedef signed char ls_byte;
```
其作用就是一个标志位。高位的4比特用来表示 lua 中几个数据类型。低位的4比特中的高位用来表示数据类型，比如number就有integer、float的区分，lua 中相关定义如下:
```C
// lua.h
#define LUA_TNIL		0
#define LUA_TBOOLEAN		1
#define LUA_TLIGHTUSERDATA	2
#define LUA_TNUMBER		3
#define LUA_TSTRING		4
#define LUA_TTABLE		5
#define LUA_TFUNCTION		6
#define LUA_TUSERDATA		7
#define LUA_TTHREAD		8

// lobject.h
/*
** tags for Tagged Values have the following use of bits:
** bits 0-3: actual tag (a LUA_T* constant)
** bits 4-5: variant bits
** bit 6: whether value is collectable
*/

/* add variant bits to a type */
#define makevariant(t,v)	((t) | ((v) << 4))
```
这些几乎可以表示lua中所有的数据类型了，感觉使用union的方式还是挺妙的