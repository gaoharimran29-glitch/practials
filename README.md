# PROLOG Practical Programs — Solutions

> All programs are written in Prolog (compatible with SWI-Prolog). Copy each program into a `.pl` file or run it in the Prolog interpreter.

## 1. Family Tree and Family Relationships

**Question:** Write a PROLOG program to implement the family tree and demonstrate the family relationship.

```prolog
% Facts
parent(john, mary).
parent(john, tom).
parent(mary, alice).
parent(tom, bob).

male(john).
male(tom).
male(bob).

female(mary).
female(alice).

% Relationships
father(X, Y) :- parent(X, Y), male(X).
mother(X, Y) :- parent(X, Y), female(X).
child(X, Y) :- parent(Y, X).
sibling(X, Y) :- parent(P, X), parent(P, Y), X \= Y.
grandparent(X, Y) :- parent(X, Z), parent(Z, Y).
grandchild(X, Y) :- grandparent(Y, X).
```

**Sample queries:**
```prolog
?- father(john, mary).
?- sibling(mary, tom).
?- grandparent(john, alice).
```

## 2. Concatenate Two Lists (`conc/3`)

**Question:** Write a PROLOG program to implement `conc(L1, L2, L3)` where `L2` is appended with `L1` to get `L3`.

```prolog
conc([], L, L).
conc([H|T], L2, [H|R]) :-
    conc(T, L2, R).
```

**Sample query:**
```prolog
?- conc([1,2], [3,4], R).
% R = [1,2,3,4].
```

## 3. Reverse a List (`reverse/2`)

**Question:** Write a PROLOG program to implement `reverse(L, R)` where `L` is the original list and `R` is its reverse.

```prolog
reverse([], []).
reverse([H|T], R) :-
    reverse(T, RT),
    conc(RT, [H], R).
```

**Sample query:**
```prolog
?- reverse([1,2,3,4], R).
% R = [4,3,2,1].
```

## 4. Sum of Two Numbers

**Question:** Write a PROLOG program to calculate the sum of two numbers.

```prolog
sum_two(X, Y, S) :-
    S is X + Y.
```

**Sample query:**
```prolog
?- sum_two(5, 7, S).
% S = 12.
```

## 5. Maximum of Two Numbers (`max/3`)

**Question:** Write a PROLOG program to implement `max(X, Y, M)` so that `M` is the maximum of `X` and `Y`.

```prolog
max_num(X, Y, X) :-
    X >= Y.
max_num(X, Y, Y) :-
    X < Y.
```

**Sample query:**
```prolog
?- max_num(8, 5, M).
% M = 8.
```

> Named `max_num/3` to avoid a conflict with built-in predicates in some Prolog implementations.

## 6. Factorial (`factorial/2`)

**Question:** Write a program in PROLOG to implement `factorial(N, F)` where `F` represents the factorial of `N`.

```prolog
factorial(0, 1).
factorial(N, F) :-
    N > 0,
    N1 is N - 1,
    factorial(N1, F1),
    F is N * F1.
```

**Sample query:**
```prolog
?- factorial(5, F).
% F = 120.
```

## 7. Fibonacci (`generate_fib/2`)

**Question:** Write a program in PROLOG to implement `generate_fib(N, T)` where `T` represents the Nth term of the Fibonacci series.

```prolog
% Here, N = 0 gives the first term (0), N = 1 gives the second term (1).
generate_fib(0, 0).
generate_fib(1, 1).
generate_fib(N, T) :-
    N > 1,
    N1 is N - 1,
    N2 is N - 2,
    generate_fib(N1, T1),
    generate_fib(N2, T2),
    T is T1 + T2.
```

**Sample query:**
```prolog
?- generate_fib(6, T).
% T = 8.
```

## 8. Power (`power/3`)

**Question:** Write a PROLOG program to implement `power(Num, Pow, Ans)` where `Num` is raised to the power `Pow` to get `Ans`.

```prolog
power(_, 0, 1).
power(Num, Pow, Ans) :-
    Pow > 0,
    Pow1 is Pow - 1,
    power(Num, Pow1, Ans1),
    Ans is Num * Ans1.
```

**Sample query:**
```prolog
?- power(2, 4, Ans).
% Ans = 16.
```

## 9. Multiplication (`multi/3`)

**Question:** Write a PROLOG program to implement `multi(N1, N2, R)` where `N1` and `N2` are the numbers to be multiplied and `R` represents the result.

```prolog
multi(N1, N2, R) :-
    R is N1 * N2.
```

**Sample query:**
```prolog
?- multi(6, 7, R).
% R = 42.
```

## 10. Membership (`memb/2`)

**Question:** Write a PROLOG program to implement `memb(X, L)` to check whether `X` is a member of list `L` or not.

```prolog
memb(X, [X|_]).
memb(X, [_|T]) :-
    memb(X, T).
```

**Sample queries:**
```prolog
?- memb(3, [1,2,3,4]).
% true.

?- memb(9, [1,2,3,4]).
% false.
```

## 11. Sum of a List (`sumlist/2`)

**Question:** Write a PROLOG program to implement `sumlist(L, S)` so that `S` is the sum of a given list `L`.

```prolog
sumlist([], 0).
sumlist([H|T], S) :-
    sumlist(T, S1),
    S is H + S1.
```

**Sample query:**
```prolog
?- sumlist([1,2,3,4,5], S).
% S = 15.
```

## 12. Even and Odd Length Lists

**Question:** Write a PROLOG program to implement `evenlength(List)` and `oddlength(List)` so that they are true if their argument is a list of even or odd length respectively.

```prolog
evenlength([]).
evenlength([_,_|T]) :-
    evenlength(T).

oddlength([_]).
oddlength([_,_|T]) :-
    oddlength(T).
```

**Sample queries:**
```prolog
?- evenlength([a,b,c,d]).
% true.

?- oddlength([a,b,c]).
% true.
```

## 13. Maximum Element in a List (`maxlist/2`)

**Question:** Write a PROLOG program to implement `maxlist(L, M)` so that `M` is the maximum number in the list.

```prolog
maxlist([X], X).
maxlist([H|T], M) :-
    maxlist(T, M1),
    ( H >= M1 -> M = H ; M = M1 ).
```

**Sample query:**
```prolog
?- maxlist([4,9,2,7,5], M).
% M = 9.
```

> The list must contain at least one element.

## 14. Insert an Item at Nth Position (`insert/4`)

**Question:** Write a PROLOG program to implement `insert(I, N, L, R)` that inserts item `I` into the Nth position of list `L` to generate list `R`.

```prolog
% Positions are 1-based: insert at position 1 means insert at the beginning.
insert(I, 1, L, [I|L]).
insert(I, N, [H|T], [H|R]) :-
    N > 1,
    N1 is N - 1,
    insert(I, N1, T, R).
```

**Sample query:**
```prolog
?- insert(x, 3, [a,b,c,d], R).
% R = [a,b,x,c,d].
```

## 15. Delete the Nth Element (`delete/3`)

**Question:** Write a PROLOG program to implement `delete(N, L, R)` that removes the element at the Nth position from list `L` to generate list `R`.

```prolog
% Positions are 1-based.
delete(1, [_|T], T).
delete(N, [H|T], [H|R]) :-
    N > 1,
    N1 is N - 1,
    delete(N1, T, R).
```

**Sample query:**
```prolog
?- delete(3, [a,b,c,d,e], R).
% R = [a,b,d,e].
```

---

## How to Run in SWI-Prolog

1. Install SWI-Prolog.
2. Save a program into a file with a `.pl` extension (e.g., `practical1.pl`).
3. Open SWI-Prolog and load the file:
   ```prolog
   ?- [practical1].
   ```
4. Run the sample query shown for the practical.
5. To reload after editing, run:
   ```prolog
   ?- make.
   ```

