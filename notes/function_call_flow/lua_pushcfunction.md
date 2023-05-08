## 函数  lua_pushcfunction 调用过程

C 需要将 C Function 地址压入到 lua stack 中，以此让 lua 虚拟机可以感知到该函数存在，之后才可以调用。这个函数机制也比较简单。

```C
// lua.h
#define lua_pushcfunction(L,f)	lua_pushcclosure(L, (f), 0)

// lapi.h
LUA_API void lua_pushcclosure (lua_State *L, lua_CFunction fn, int n) {
  lua_lock(L);
  /* 这里调用过来是走 if true 的分支，其实这个操作就是取L的栈顶 */
  if (n == 0) {
    setfvalue(s2v(L->top.p), fn); /*取栈顶并设置值为 light c function*/
    api_incr_top(L);  /* 检测堆栈是否溢出 */
  }
  else { // 暂且不讲
    CClosure *cl;
    api_checknelems(L, n);
    api_check(L, n <= MAXUPVAL, "upvalue index too large");
    cl = luaF_newCclosure(L, n);
    cl->f = fn;
    L->top.p -= n;
    while (n--) {
      setobj2n(L, &cl->upvalue[n], s2v(L->top.p + n));
      /* does not need barrier because closure is white */
      lua_assert(iswhite(cl));
    }
    setclCvalue(L, s2v(L->top.p), cl);
    api_incr_top(L);
    luaC_checkGC(L);
  }
  lua_unlock(L);
}

// lobject.h
#define setfvalue(obj,x) \
{ \ 
    TValue *io=(obj); \ 
    val_(io).f=(x); \   /*对 struct Value 中 f 进行赋值，其实就是吧最外面的C Function 赋给栈顶元素 */
    settt_(io, LUA_VLCF); \ /* 设置struct TValue 的 tt_(类型标识) 为 light C function*/
}

#define api_incr_top(L)	
{ \
    L->top.p++; \
		api_check(L, L->top.p <= L->ci->top.p, \
					"stack overflow"); \
}

```

整个流程很清晰，取操作的 lua_State 的栈顶，并将 c function 设置为栈顶值，将栈增加一位并检测是否 stack overflow。