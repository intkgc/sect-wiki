This code generate following code

```java
AsmBuilder builder = new  AsmBuilder();

builder.begin();

Const const = builder.const(HALF, "100");
Const const2 = builder.const(HALF, "200");

Function main = builder.beginFunction("main");
Variable a = main.localVariable();
a.load(const);
Variable b = main.localVariable();
b.load(const2);
Value c = a.plus(b);
builder.printi(c.register());
main.end();


builder.end();
```
### 1000-7

```java
AsmBuilder builder = new  AsmBuilder();
builder.begin();

Const const = builder.localConst(HALF, "1000");
Const const2 = builder.localConst(HALF, "7");
Const zero = builder.localConst(HALF, "0");
Function main = main.beginFunction("main");
Value a = main.localVariable(HALF);
Value b = main.localVariable(HALF);
a.load(const);
b.load(const2);

Label label = main.local("lol");

a.set(a.minus(b));
Value condition = a.bigger(zero);
main.printi(a.register());
label.jump(condition);
main.end();

builder.end();
```

```asm
.code
global main:
  frame
  mov R0, 100
  mov R1, 200
  add R2, R0, R1
  printi R2
  ret
```
