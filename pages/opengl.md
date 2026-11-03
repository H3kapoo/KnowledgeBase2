# OpengGL
### 1. [Maximum uniform components](#maximum-uniform-components)
GPUs support only a fixed number of uniform components until they refuse to compile.
```cli
1 component = usually 4 bytes
```
Shaders can throw errors at compile time due to exceeding the maximum number of components the current HW can support. <br\>
The fix is to reduce the number of components used. <br\>
Maximum number of components query:
```cpp
int32_t maxUniformComps;
glGetIntegerv(GL_MAX_VERTEX_UNIFORM_COMPONENTS, &maxUniformComps);
```

## References Used
-