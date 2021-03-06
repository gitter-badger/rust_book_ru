% Приведение типов

Rust, со своим акцентом на безопасность, обеспечивает два различных способа
преобразования различных типов между собой. Первый - `as`, для безопасного
приведения. Второй - `transmute`, в отличие от первого, позволяет произвольное
приведение типов и является одной из самых опасных фич Rust!

# `as`

Ключевое слово `as` выполняет обычное приведение типов:

```rust
let x: i32 = 5;

let y = x as i64;
```

Оно допускает только определенные виды приведения типов:

```rust,ignore
let a = [0u8, 0u8, 0u8, 0u8];

let b = a as u32; // four eights makes 32
```

Это приведет к ошибке:

```text
error: non-scalar cast: `[u8; 4]` as `u32`
let b = a as u32; // four eights makes 32
        ^~~~~~~~
```

Это ’нескалярное преобразование’, потому что у нас здесь преобразуются
множественные значения: четыре элемента массива. Такие виды преобразований очень
опасны, потому что они делают предположения о том, как реализованы множественные
нижележащие структуры. Поэтому нам нужно что-то более опасное.

# `transmute`

Функция `transmute` предоставляется [внутренними средствами компилятора]
[intrinsics], и то, что она делает, является очень простым, но в то же время
очень опасным. Она сообщает Rust, чтобы он воспринимал значение одного типа, как
будто это значение другого типа. Это делается независимо от системы проверки
типов, и поэтому полностью на ваш страх и риск.

[intrinsics]: intrinsics.html

В предыдущем примере, мы знаем, что массив из четырех `u8` отображается в массив
`u32` должным образом, и поэтому мы хотим выполнить приведение. Если вместо `as`
использовать `transmute`, то Rust позволит это сделать:

```rust
use std::mem;

unsafe {
    let a = [0u8, 0u8, 0u8, 0u8];

    let b = mem::transmute::<[u8; 4], u32>(a);
}
```

Для того чтобы компиляция прошла успешно, мы должны обернуть эту операцию в
`unsafe` блок. Технически, только вызов `mem::transmute` должен быть выполнен в
небезопасном блоке, но в данном случае хорошо было бы поместить в этот блок все
необходимое, связаное с этим вызовом, чтобы было удобнее искать. В данном
примере связаной необходимой переменной является `a`, и поэтому она находится в
блоке. Код может быть в любом стиле, иногда контекст расположен слишком далеко,
и тогда упаковка всего кода в `unsafe` не будет такой уж хорошей идеей.

Хотя при использовании `transmute` и выполняется очень мало проверок, но как
минимум будет проверяться, что типы имеют одинаковый размер. Нижеприведенный код
завершится ошибкой:

```rust,ignore
use std::mem;

unsafe {
    let a = [0u8, 0u8, 0u8, 0u8];

    let b = mem::transmute::<[u8; 4], u64>(a);
}
```

со следующим описанием:

```text
error: transmute called on types with different sizes: [u8; 4] (32 bits) to u64
(64 bits)
```

Все, кроме этой одной проверки, на ваш страх и риск!
