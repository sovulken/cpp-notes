# Концепты

Предпосылки:

* Хотим навешивать ограничения на шаблонный тип
* SFINAE тяжело понимать человеку, компилятору дороже это компилировать, чем концепты
* Сложно делать перегрузки, используя SFINAE
* Длинные логи ошибок компилирования


##### В каком виде концепты попали в C++20

Концепт - это имя для ограничения.
Ограничение - это шаблонное булево выражение.

* Все концепты implicit.
* От definition check функции по концепту отказались (в proof of concept (conceptGCC) наблюдалось сильное замедление из-за этой проверки). Возможно, не удастся заимплементить эту проверку в будущем, если случится несовместимость.

Объявление концепта:

```cpp
template <typename T>
concept destructible = std::is_nothrow_destructible_v<T>;
```

Использование:

```cpp
template <typename T>
requires destructible<T>
void f(T& t);

// short form
template <destructible T>
void f(T& t);

// ultra-short form, "implicit template"
void f(destructible auto& t);

// concept auto разрешено писать везде, где можно написать просто auto
```

Использование для классов аналогичное:

```cpp
template<destructible T>
class X;

template <typename T>
requires destructible<T>
class Y;
```

Если концепт принимает несколько аргументов:

```cpp
template <A, B, C, D, E>
concept X = ...

// short form
template <X <B, C, D, E> A>
void f(A&);
```

##### Requires expression

```cpp
template <typename T>
concept comparable = requires(T a) {
	{a < a} -> std::convertible_to<bool>;
}

// такой концепт писать не надо, т.к. мы посмотрели только на одну операцию среди остальных возможных

// ограничение в данном случае - convertible_to<bool>
// то есть тип выражения a < a должен быть конвертируемым в bool
```

Requirement'ов бывает 4 вида:

1. требует, чтобы выражение было валидным - ```begin(x)```
2. проверяет наличие типа -  ```typename T::type```
3. compound requirement (пример выше)
4. nested requirement - ```requires(C<T>)```


##### Реальный пример

```cpp
template <typename T>
constexpr bool get_val() {
	return T::value;
}

template <typename T>
requires (sizeof(T) > 1 && get_val<T>())
void f(T); // #1

void f(int); // #2

void g() {
	f('A'); // calls #2
}
```

##### Другие полезные примеры

```cpp
// if couldn't be compiled, then requirement not satisfied

template<class T>
concept Addable = requires (T a, T b) {
	a + b;
};

template<class T, class U = T>
concept Swappable = requires(T&& t, U&& u) {
	swap(std::forward<T>(t), std::forward<U>(u));
	swap(std::forward<U>(u), std::forward<T>(t));
};
```

 ##### Пример потяжелее

 ```cpp
template<class T>
concept C2 = requires(T x) {
	{*x} -> std::convertible_to<typename T::inner>;
	{x * 1} -> std::convertible_to<T>;
};
```

Первое ограничение указывает, что ```*x``` - валидно, тип ```T::inner``` существует и что ```*x``` конвертуруется в ```T::inner```.

Второе указывает, что выражение ```x*1``` корректно и что его результат конвертируется к ```T```.


##### Overload resolution для концептов

Как выбрать более специализированную перегрузку?

Constraint normalization:
* (E) -> E
* E1 && E2 -> conjunction of E1 and E2
* E1 || E2 -> disjunction of E1 and E2
* C<A1, A2, A3, ... An> -> substitute this concept by its normalization
* otherwise -> atomic constraint (неделимый)

Выполним нормализацию концепта A и B и выясним, для каждого ли constraint'а выполняется, что A - true...

заимплементите за меня пж, я так хочу спать.









