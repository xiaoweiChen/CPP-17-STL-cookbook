# 将变量作用域限制在if和switch区域内

将变量的生命周期尽可能的限制在指定区域内，是一种非常好的代码风格。有时我们需要在满足某些条件时获得某个值，然后对这个值进行操作。

为了让这个过程更简单，C++17中为if和switch配备了初始化区域。

## How to do it...

这个案例中，我们使用初始化语句，来了解下其使用方式：

- `if`：假设我们要在一个字母表中查找一个字母，我们`std::map`的成员`find`完成这个操作：

```c++
if (auto itr (character_map.find(c)); itr != character_map.end()) {
  // *itr is valid. Do something with it.
} else {
  // itr is the end-iterator. Don't dereference.
}
// itr is not available here at all
```

- `switch`：这个例子看起来像是从玩家输入的字母决定某个游戏中的行为。通过使用`switch`查找字母相对应的操作：

```c++
switch (char c (getchar()); c) {
  case 'a': move_left(); break;
  case 's': move_back(); break;
  case 'w': move_fwd(); break;
  case 'd': move_right(); break;
  case 'q': quit_game(); break;
  case '0'...'9': select_tool('0' - c); break;
  default:
    std::cout << "invalid input: " << c << '\n';
}
```

## How it works...

带有初始化的`if`和`switch`相当于语法糖一样。

```c++
// if: before C++17
{
    auto var(init_value);
    if (condition){
        // branch A. var is accessible
    } else {
        // branch B. var is accessible
    }
    // var is still accessible
}
```

```c++
// if: since C++17
if (auto var (init_value); condition){
    // branch A. var is accessible
} else {
    // branch B. var is accessible
}
// var is not accessible any longer
```

```c++
// switch: before C++17
{
    auto var (init_value);
    switch (var) {
      case 1: ...
      case 2: ...
      ...
    }
    // var is still accessible
}
```

```c++
// switch: since C++17
switch(auto var (init_value); var){
    case 1: ...
    case 2: ...
    ...
}
// var is not accessible any longer
```

这些有用的特性保证了代码的简洁性。C++17之前只能使用外部括号将代码包围，就像上面的例子中展示的那样。减短变量的生命周期，能帮助我们保持代码的整洁性，并且更加易于重构。

## There's more...

另一个有趣的例子是临界区限定变量生命周期。

先来看个栗子：

```c++
if (std::lock_guard<std::mutex> lg {my_mutex}; some_condition) {
  // Do something
}
```

首先，创建一个`std::lock_guard`。这个类接收一个互斥量和作为其构造函数的参数。这个类在其构造函数中对互斥量上锁，之后当代码运行完这段区域后，其会在析构函数中对互斥量进行解锁。这种方式避免了忘记解锁互斥量而导致的错误。C++17之前，为了确定解锁的范围，需要一对额外的括号对。

另一个例子中对弱指针进行区域限制：

```c++
if (auto shared_pointer (weak_pointer.lock()); shared_pointer != nullptr) {
  // Yes, the shared object does still exist
} else {
  // shared_pointer var is accessible, but a null pointer
}
// shared_pointer is not accessible any longer
```

这个例子中有一个临时的`shared_pointer`变量，虽然`if`条件块或外部括号会让其保持一个无用的状态，但是这个变量确实会“泄漏”到当前范围内。

当要使用传统API的输出参数时，`if`初始化段就很有用：

```c++
if (DWORD exit_code; GetExitCodeProcess(process_handle, &exit_code)) {
  std::cout << "Exit code of process was: " << exit_code << '\n';
}
// No useless exit_code variable outside the if-conditional
```

`GetExitCodeProcess`函数是Windows操作系统的内核API函数。其通过返回码来判断给定的进程是否合法的处理完成。当离开条件域，变量就没用了，也就可以销毁这个变量了。

具有初始化段的`if`代码块在很多情况下都特别有用，尤其是在使用传统API的输出参数进行初始化时。

> Note:
>
> 使用带有初始化段的`if`和`switch`能保证代码的紧凑性。这使您的代码紧凑，更易于阅读，在重构过程中，会更容易改动。



