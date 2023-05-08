## lua_push_number 调用

这里就以number举例说明如何将值从 C 压入到 lua stack

```C
LUA_API void lua_pushnumber (lua_State *L, lua_Number n) {
  lua_lock(L);
  setfltvalue(s2v(L->top.p), n); // 将 n 设置为 L 的栈顶元素
  api_incr_top(L);               // +1 ，并检测堆栈溢出
  lua_unlock(L);
}

/* 可以看到是一样的流程 */
LUA_API void lua_pushinteger (lua_State *L, lua_Integer n) {
  lua_lock(L);
  setivalue(s2v(L->top.p), n);
  api_incr_top(L);
  lua_unlock(L);
}
```