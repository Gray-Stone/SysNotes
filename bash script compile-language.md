
## Write a C source file, but directly run it in shell.



如何让一个二进制可执行程序同时在Windows与Linux下原生启动？ - Freed-wzy的回答 - 知乎
https://www.zhihu.com/question/566274801/answer/5367438055



```
#if 0
bin="$(basename "$0")" && bin="${bin%%.*}" && cc -g -Wall -o"$bin" "$0" && exec ./"$bin" "$@" || exit
#endif
#include <stdio.h>
#include <stdlib.h>
int main(int argc, char *argv[]) {
  printf("hello\n");
  return EXIT_SUCCESS;
}

```

Main point is Unix will treat a executable as shell script when it doesn't know how to deal with it.