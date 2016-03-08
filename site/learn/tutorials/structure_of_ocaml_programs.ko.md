<!-- ((! set title OCaml 프로그램의 구조 !)) ((! set learn !)) -->

*Table of contents*

# OCaml 프로그램의 구조

이제 고차원적인 관점에서 실제 OCaml 프로그램을 살펴 보는 것에 시간을 투자해 볼 것이다. 여기서는
지역(local)과 전역(global)의 정의, 언제 `;;`나 `;`를 쓰는지, 모듈, 중첩 함수(nested function),
참조(reference) 등에 대해 설명할 생각이다. 이것들을 위해 이전에 본 적이 없어서 이해가 가지 않을 만한
여러 가지 OCaml 개념들부터 살펴보겠다. 아직은 세세한 것에 대해 신경쓰지 않도록 한다. 대신 프로그램의 전체적인
형태와 내가 설명할 기능들에만 집중하기 바란다.

##  지역 "변수(variables)" (*실제로는* 지역 표현식(local expressions))
C에서 `average` 함수를 만들고 여기에 지역 변수를 하나 추가해 보자.
(앞에서 정의했던 것과 비교해 보기 바란다).

```C
double average (double a, double b)
{
  double sum = a + b;
  return sum / 2;
}
```
이제 동일한 함수를 OCaml 버전으로 만들어 보자:

```ocamltop
let average a b =
  let sum = a +. b in
  sum /. 2.0;;
```
표준 구문인 `let name = expression in`이 이름 붙은 지역 표현식(names local exprssion)을
정의하는 데에 사용되었으며 `name`은 이후에 `expression`을 대신하여 코드 블록 끝에 `;;`가 오기
전까지 함수 안에서 쓰일 수 있다. `in` 다음에 들여쓰기를 하지 않았음을 유의하라. 그냥 일반적인 영어
문장인 것처럼 `let ... in`을 쓴다고 생각하면 된다.

이제 C 지역 변수와 이름 붙은 지역 표현식을 비교해 보는 것은 손바닥 뒤집기이다. 사실 이것들은
약간 다르긴 하다. C 변수 `sum`은 스택에 변수를 위해 할당된 공간을 갖는다. 원한다면 나중에 `sum`에
다른 값을 넣을 수도 있고 `sum`이 있는 곳의 주소를 가져올 수도 있다. 이는 OCaml 버전에서는
불가능하다. OCaml 버전에서의 `sum`은 단순히 `a +. b`라는 표현식을 가리키는 축약된 이름일 뿐이다. 
`sum`에 값을 집어 넣거나 변경하는 수단은 존재하지 않는다. (어떻게 변수가 값을 바꿀 수 있는 지는
나중에 볼 것이다).

이를 명백히 확인할 수 있는 다른 예제가 있다. 아래의 두가지 코드는 동일한 값을 리턴해야 한다
(즉 (a+b) + (a+b)²):

```ocamltop
let f a b =
  (a +. b) +. (a +. b) ** 2.
  ;;

let f a b =
  let x = a +. b in
  x +. x ** 2.
  ;;
```

두번째 버전이 보통 훨씬 빠르며(하지만 대부분의 컴파일러들은 "공통 하부식 제거"를 수행해야 할 것이다)
확실히 눈에도 더 잘 들어온다. 두번째 예제의 `x`는 단순히 `a +. b`의 약칭이 되는 것이다.

##  전역 "변수(variables)" (*really* global expressions)
당신은 또한 위에서 얘기한 지역 "변수들"에 대해 최상위 레벨에서 쓰기 위한 전역 이름을 정의하는 것이 가능하다.
하지만 이들은 실제로는 전혀 변수가 아니며 그것들을 가리키는 이름에 불과하다. 아래에 실제(생략 됐지만) 예제가 있다.  

```ocaml
let html =
  let content = read_whole_file file in
  GHtml.html_from_string content
  ;;

let menu_bold () =
  match bold_button#active with
  | true -> html#set_font_style ~enable:[`BOLD] ()
  | false -> html#set_font_style ~disable:[`BOLD] ()
  ;;

let main () =
  (* 코드 생략 *)
  factory#add_item "Cut" ~key:_X ~callback: html#cut
  ;;
```

이 실제 코드 조각에서 `html`은 HTML 프로그램 시작 부분의 첫 `let html =` 구문에서 
딱 한 번 생성되는 HTML 편집 위젯이다(lablgtk 라이브러리의 객체). 이는 나중에 여러 함수에서 참조된다.

위 코드 예제의 `html`이란 이름은 실제로 C나 다른 명령형 언어에서의 실제 전역 변수와는 결코 비교할
수 있는 대상은 아니다. "`html` 포인터"를 "저장"하기 위한 공간은 할당되지 않는다. 게다가 `html`에
다른 무언가, 예를 들면 다른 위젯을 가리키도록 집어 넣는 것도 불가능하다. 다음 섹션에서 실제 변수인
참조에 대해서 얘기할 것이다.

## Let-바인딩
Any use of `let ...`, whether at the top level (globally) or within a
function, is often called a **let-binding**.

## References: real variables
What happens if you want a real variable that you can assign to and
change throughout your program? You need to use a **reference**.
References are very similar to pointers in C/C++. In Java, all variables
which store objects are really references (pointers) to the objects. In
Perl, references are references - the same thing as in OCaml.

Here's how we create a reference to an `int` in OCaml:

```ocamltop
ref 0;;
```
Actually that statement wasn't really very useful. We created the
reference and then, because we didn't name it, the garbage collector
came along and collected it immediately afterwards! (actually, it was
probably thrown away at compile-time.) Let's name the reference:

```ocamltop
let my_ref = ref 0
```
This reference is currently storing a zero integer. Let's put something
else into it (assignment):

```ocamltop
my_ref := 100
```
And let's find out what the reference contains now:

```ocamltop
!my_ref
```
So the `:=` operator is used to assign to references, and the `!`
operator dereferences to get out the contents. Here's a rough-and-ready
comparison with C/C++:

OCaml
```ocamltop
let my_ref = ref 0;;
my_ref := 100;;
!my_ref
```

C/C++
```C
int a = 0; int *my_ptr = &a;
*my_ptr = 100;
*my_ptr;
```
References have their place, but you may find that you don't use
references very often. Much more often you'll be using
`let name = expression in` to name local expressions in your function
definitions.

## 중첩 함수
C doesn't really have a concept of nested functions. GCC supports nested
functions for C programs but I don't know of any program which actually
uses this extension. Anyway, here's what the gcc info page has to say
about nested functions:

A "nested function" is a function defined inside another function.
(Nested functions are not supported for GNU C++.) The nested function's
name is local to the block where it is defined. For example, here we
define a nested function named 'square', and call it twice:

```C
foo (double a, double b)
{
  double square (double z) { return z * z; }

  return square (a) + square (b);
}
```

The nested function can access all the variables of the containing
function that are visible at the point of its definition. This is called
"lexical scoping". For example, here we show a nested function which
uses an inherited variable named `offset`:

```C
bar (int *array, int offset, int size)
{
  int access (int *array, int index)
    { return array[index + offset]; }
  int i;
  /* ... */
  for (i = 0; i < size; i++)
    /* ... */ access (array, i) /* ... */
}
```
You get the idea. Nested functions are, however, very useful and very
heavily used in OCaml. Here is an example of a nested function from some
real code:

```ocamltop
let read_whole_channel chan =
  let buf = Buffer.create 4096 in
  let rec loop () =
    let newline = input_line chan in
    Buffer.add_string buf newline;
    Buffer.add_char buf '\n';
    loop ()
  in
  try
    loop ()
  with
    End_of_file -> Buffer.contents buf;;
```
Don't worry about what this code does - it contains many concepts which
haven't been discussed in this tutorial yet. Concentrate instead on the
central nested function called `loop` which takes just a unit argument.
You can call `loop ()` from within the function `read_whole_channel`,
but it's not defined outside this function. The nested function can
access variables defined in the main function (here `loop` accesses the
local names `buf` and `chan`).

The form for nested functions is the same as for local named
expressions: `let name arguments = function-definition in`.

You normally indent the function definition on a new line as in the
example above, and remember to use `let rec` instead of `let` if your
function is recursive (as it is in that example).

## 모듈과 `open`
OCaml comes with lots of fun and interesting modules (libraries of
useful code). For example there are standard libraries for drawing
graphics, interfacing with GUI widget sets, handling large numbers, data
structures, and making POSIX system calls. These libraries are located
in `/usr/lib/ocaml/` (on Unix anyway). For these examples we're
going to concentrate on one quite simple module called `Graphics`.

The `Graphics` module is installed into 7 files (on my system):

```
/usr/lib/ocaml/graphics.a
/usr/lib/ocaml/graphics.cma
/usr/lib/ocaml/graphics.cmi
/usr/lib/ocaml/graphics.cmx
/usr/lib/ocaml/graphics.cmxa
/usr/lib/ocaml/graphics.cmxs
/usr/lib/ocaml/graphics.mli
```
For the moment let's just concentrate on the file `graphics.mli`. This
is a text file, so you can read it now. Notice first of all that the
name is `graphics.mli` and not `Graphics.mli`. OCaml always capitalizes
the first letter of the file name to get the module name. This can be
very confusing until you know about it!

If we want to use the functions in `Graphics` there are two ways we can
do it. Either at the start of our program we have the `open Graphics;;`
declaration. Or we prefix all calls to the functions like this:
`Graphics.open_graph`. `open` is a little bit like Java's `import`
statement, and much more like Perl's `use` statement.

To use `Graphics` in the interactive toplevel, you must first load the
library with

```ocaml
#load "graphics.cma";;
```
Windows users: For this example to work interactively on Windows, you
will need to create a custom toplevel. Issue the command `ocamlmktop
-o ocaml-graphics graphics.cma` from the command line.
<!-- FIXME: is this last remark still true? -->

A couple of examples should make this clear. (The two examples draw
different things - try them out). Note the first example calls
`open_graph` and the second one `Graphics.open_graph`.

```ocaml
(* To compile this example: ocamlc graphics.cma grtest1.ml -o grtest1 *)
open Graphics;;

open_graph " 640x480";;
for i = 12 downto 1 do
  let radius = i * 20 in
  set_color (if i mod 2 = 0 then red else yellow);
  fill_circle 320 240 radius
done;;
read_line ();;

(* To compile this example: ocamlc graphics.cma grtest2.ml -o grtest2 *)

Random.self_init ();;
Graphics.open_graph " 640x480";;

let rec iterate r x_init i =
  if i = 1 then x_init
  else
    let x = iterate r x_init (i-1) in
    r *. x *. (1.0 -. x);;

for x = 0 to 639 do
  let r = 4.0 *. (float_of_int x) /. 640.0 in
  for i = 0 to 39 do
    let x_init = Random.float 1.0 in
    let x_final = iterate r x_init 500 in
    let y = int_of_float (x_final *. 480.) in
    Graphics.plot x y
  done
done;;

read_line ();;
```
Both of these examples make use of some features we haven't talked about
yet: imperative-style for-loops, if-then-else and recursion. We'll talk
about those later. Nevertheless you should look at these programs and
try and find out (1) how they work, and (2) how type inference is
helping you to eliminate bugs.

## `Pervasives`(기본) 모듈
There's one module that you never need to "`open`". That is the
`Pervasives` module (go and read `/usr/lib/ocaml/pervasives.mli`
now). All of the symbols from the `Pervasives` module are automatically
imported into every OCaml program.

## 모듈 이름 바꾸기
What happens if you want to use symbols in the `Graphics` module, but
you don't want to import all of them and you can't be bothered to type
`Graphics` each time? Just rename it using this trick:

```ocaml
module Gr = Graphics;;

Gr.open_graph " 640x480";;
Gr.fill_circle 320 240 240;;
read_line ();;
```
Actually this is really useful when you want to import a nested module
(modules can be nested inside one another), but you don't want to type
out the full path to the nested module name each time.

## `;;` 와 `;`의 이용과 생략
Now we're going to look at a very important issue. When should you use
`;;`, when should you use `;`, and when should you use none of these at
all? This is a tricky issue until you "get it", and it taxed the author
for a long time while he was learning OCaml too.

Rule #1 is that you should use `;;` to separate statements at the
top-level of your code, and *never* within function definitions or any
other kind of statement.

Have a look at a section from the second graphics example above:

```ocaml
Random.self_init ();;
Graphics.open_graph " 640x480";;

let rec iterate r x_init i =
  if i = 1 then x_init
  else
    let x = iterate r x_init (i-1) in
    r *. x *. (1.0 -. x);;
```

We have two top-level statements and a function definition (of a
function called `iterate`). Each one is followed by `;;`.

Rule #2 is that *sometimes* you can elide the `;;`. As a
beginner you shouldn't worry about this — you should always put in the
`;;` as directed by Rule #1. But since you'll also be reading a lot of
other peoples' code you'll need to know that sometimes we can elide
`;;`. The particular places where this is allowed are:

* Before the keyword `let`.
* Before the keyword `open`.
* Before the keyword `type`.
* At the very end of the file.
* A few other (very rare) places where OCaml can "guess" that the next
 thing is the start of a new statement and not the continuation of
 the current statement.

Here is some working code with `;;` elided wherever possible:

```ocaml
open Random                   (* ;; *)
open Graphics;;

self_init ();;
open_graph " 640x480"         (* ;; *)

let rec iterate r x_init i =
  if i = 1 then x_init
  else
    let x = iterate r x_init (i-1) in
    r *. x *. (1.0 -. x);;

for x = 0 to 639 do
  let r = 4.0 *. (float_of_int x) /. 640.0 in
  for i = 0 to 39 do
    let x_init = Random.float 1.0 in
    let x_final = iterate r x_init 500 in
    let y = int_of_float (x_final *. 480.) in
    Graphics.plot x y
  done
done;;

read_line ()                  (* ;; *)
```

Rules #3 and #4 refer to the single `;`. This is completely different
from `;;`. The single semicolon `;` is what is known as a **sequence
point**, which is to say it has exactly the same purpose as the single
semicolon in C, C++, Java and Perl. It means "do the stuff before this
point first, then do the stuff after this point when the first stuff has
completed". Bet you didn't know that.

Rule #3 is: Consider `let ... in` as a statement, and never put a
single `;` after it.

Rule #4 is: For all other statements within a block of code, follow
them with a single `;`, *except* for the very last one.

The inner for-loop in our example above is a good demonstration. Notice
that we never use any single `;` in this code:

```ocaml
for i = 0 to 39 do
  let x_init = Random.float 1.0 in
  let x_final = iterate r x_init 500 in
  let y = int_of_float (x_final *. 480.) in
  Graphics.plot x y
done
```
The only place in the above code where might think about putting in a
`;` is after the `Graphics.plot x y`, but because this is the last
statement in the block, Rule #4 tells us not to put one there.

## Note about ";"
Brian Hurt writes to correct me on ";"

> The `;` is an operator, just like `+` is. Well, not quite just like
> `+` is, but conceptually the same. `+` has type `int -> int -> int` -
> it takes two ints and returns an int (the sum). `;` has type
> `unit -> 'b -> 'b` - it takes two values and simply returns the second
> one. Rather like C's `,` (comma) operator. You can write
> `a ; b ; c ; d` just as easily as you can write `a + b + c + d`.
> 
> This is one of those "mental leaps" which is never spelled out very
> well - in OCaml, nearly everything is an expression. `if/then/else` is
> an expression. `a ; b` is an expression. `match foo with ...` is an
> expression. The following code is perfectly legal (and all do the same
> thing):
> 
> ```ocamltop
> let f x b y = if b then x+y else x+0
> let f x b y = x + (if b then y else 0)
> let f x b y = x + (match b with true -> y | false -> 0)
> let f x b y = x + (let g z = function true -> z | false -> 0 in g y b)
> let f x b y = x + (let _ = y + 3 in (); if b then y else 0)
> ```
> 
> Note especially the last one - I'm using `;` as an operator to "join"
> two statements. All functions in OCaml can be expressed as:
> 
> ```ocaml
> let name [parameters] = expression
> ```
> 
> OCaml's definition of what is an expression is just a little wider
> than C's. In fact, C has the concept of "statements"- but all of C's
> statements are just expressions in OCaml (combined with the `;`
> operator).
> 
> The one place that `;` is different from `+` is that I can refer to
> `+` just like a function. For instance, I can define a `sum_list`
> function, to sum a list of ints, like:
> 
> ```ocamltop
> let sum_list = List.fold_left ( + ) 0
> ```

## Putting it all together: some real code
In this section we're going to show some real code fragments from the
lablgtk 1.2 library. (Lablgtk is the OCaml interface to the native Unix
Gtk widget library). A word of warning: these fragments contain a lot of
ideas which we haven't discussed yet. Don't look at the details, look
instead at the overall shape of the code, where the authors used `;;`,
where they used `;` and where they used `open`, how they indented the
code, how they used local and global named expressions.

... However, I'll give you some clues so you don't get totally lost!

* `?foo` and `~foo` is OCaml's way of doing optional and named
 arguments to functions. There is no real parallel to this in
 C-derived languages, but Perl, Python and Smalltalk all have this
 concept that you can name the arguments in a function call, omit
 some of them, and supply the others in any order you like.
* `foo#bar` is a method invocation (calling a method called `bar` on
 an object called `foo`). It's similar to `foo->bar` or `foo.bar` or
 `$foo->bar` in C++, Java or Perl respectively.

First snippet: The programmer opens a couple of standard libraries
(eliding the `;;` because the next keyword is `open` and `let`
respectively). He then creates a function called `file_dialog`. Inside
this function he defines a named expression called `sel` using a
two-line `let sel = ... in` statement. Then he calls several methods on
`sel`.

```ocaml
(* First snippet *)
open StdLabels
open GMain

let file_dialog ~title ~callback ?filename () =
  let sel =
    GWindow.file_selection ~title ~modal:true ?filename () in
  sel#cancel_button#connect#clicked ~callback:sel#destroy;
  sel#ok_button#connect#clicked ~callback:do_ok;
  sel#show ()
```

Second snippet: Just a long list of global names at the top level.
Notice that the author elided every single one of the `;;` because of
Rule #2.

```ocaml
(* Second snippet *)
let window = GWindow.window ~width:500 ~height:300 ~title:"editor" ()
let vbox = GPack.vbox ~packing:window#add ()

let menubar = GMenu.menu_bar ~packing:vbox#pack ()
let factory = new GMenu.factory menubar
let accel_group = factory#accel_group
let file_menu = factory#add_submenu "File"
let edit_menu = factory#add_submenu "Edit"

let hbox = GPack.hbox ~packing:vbox#add ()
let editor = new editor ~packing:hbox#add ()
let scrollbar = GRange.scrollbar `VERTICAL ~packing:hbox#pack ()
```

Third snippet: The author imports all the symbols from the `GdkKeysyms`
module. Now we have an unusual let-binding. `let _ = expression` means
"calculate the value of the expression (with all the side-effects that
may entail), but throw away the result". In this case, "calculate the
value of the expression" means to run `Main.main ()` which is Gtk's main
loop, which has the side-effect of popping the window onto the screen
and running the whole application. The "result" of `Main.main ()` is
insignificant - probably a `unit` return value, but I haven't checked -
and it doesn't get returned until the application finally exits.

Notice in this snippet how we have a long series of essentially
procedural commands. This is really a classic imperative program.

```ocaml
(* Third snippet *)
open GdkKeysyms

let () =
  window#connect#destroy ~callback:Main.quit;
  let factory = new GMenu.factory file_menu ~accel_group in
  factory#add_item "Open..." ~key:_O ~callback:editor#open_file;
  factory#add_item "Save" ~key:_S ~callback:editor#save_file;
  factory#add_item "Save as..." ~callback:editor#save_dialog;
  factory#add_separator ();
  factory#add_item "Quit" ~key:_Q ~callback:window#destroy;
  let factory = new GMenu.factory edit_menu ~accel_group in
  factory#add_item "Copy" ~key:_C ~callback:editor#text#copy_clipboard;
  factory#add_item "Cut" ~key:_X ~callback:editor#text#cut_clipboard;
  factory#add_item "Paste" ~key:_V ~callback:editor#text#paste_clipboard;
  factory#add_separator ();
  factory#add_check_item "Word wrap" ~active:false
    ~callback:editor#text#set_word_wrap;
  factory#add_check_item "Read only" ~active:false
    ~callback:(fun b -> editor#text#set_editable (not b));
  window#add_accel_group accel_group;
  editor#text#event#connect#button_press
    ~callback:(fun ev ->
      let button = GdkEvent.Button.button ev in
      if button = 3 then begin
        file_menu#popup ~button ~time:(GdkEvent.Button.time ev); true
      end else false);
  editor#text#set_vadjustment scrollbar#adjustment;
  window#show ();
  Main.main ()
```

