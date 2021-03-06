
*************
Inheritance
*************

==============
Introduction
==============

---------------------------------------
Object-Oriented Programming via Types
---------------------------------------

* Most object oriented languages allow user to add fields to derived types
* Objects of a type derived from a base type can be substituted for objects of the base type
* Subprogram (*method*) attached to object type can dispatch at runtime depending on exact type of the object
* Other modules can derive from your object type and define their own behaviors

-------------------------------------
Ada Mechanisms for Type Inheritance
-------------------------------------

* *Primitive* operations on types

   - Standard operations like **+** and **-**
   - Any operation that acts on the type

* Simple type derivation

   - Define types from other types that can add limitations

* Tagged derivation

   - Define types from records to add new fields

============
Primitives
============

---------------------------
The Notion of a Primitive
---------------------------
  
* A type is characterized by two sets of properties

   - Its data structure
   - The set of operations that applies to it

* These operations are called **methods** in C++, or **Primitive Operations** in Ada

   * In Ada

      -  the primitive relationship is implicit
      - The "hidden" parameter **this** is explicit (and can have any name)

      .. code:: Ada
       
          type T is record
             Attrib_Data : Integer;
          end record;
          procedure Attrib_Function(This : T);
       
   * In C++

       .. code:: C++
       
          class T {
            public:
              int Attrib_Data;
              void Attrib_Function (void);
          };

------------------------------
General Rule For a Primitive
------------------------------

* A subprogram `S` is a primitive of type `T` if

   - `S` is declared in the scope of `T`
   - `S` has at least one parameter of type `T` (of any mode, including access) or returns a value of type `T`

      .. code:: Ada

         package P is
            type T is range 1 .. 10;
            procedure P1 (V : T);
            procedure P2 (V1 : Integer; V2 : T);
            function F return T;
         end P;
 
* A subprogram can be a primitive of several types

      .. code:: Ada

         package P is
            type T1 is range 1 .. 10;
            type T2 is (A, B, C);
            procedure Proc (V1 : T1; V2 : T2);
         end P;
 
--------------------------
Beware of Access Types!
--------------------------

* Using a named access type in a subprogram creates a primitive of the access type, **not** the type of the accessed object!

   .. code:: Ada

      package P is
         type T is range 1 .. 10;
         type A_T is access all T;
         procedure Proc (V : A_T); -- Primitive of A_T
      end P;
 
* In order to create a primitive using an access type, the `access` mode should be used

   .. code:: Ada

      package P is
         type T is range 1 .. 10;
         procedure Proc (V : access T); -- Primitive of T
      end P;
 
-------------------------------
Implicit Primitive Operations
-------------------------------

* At type declaration, primitives are implicitly created if not explicitly given by the developer, depending on the kind of the type

      .. code:: Ada

         package P is
            type T1 is range 1 .. 10;
            -- implicit: function "+" (Left, Right : T1) return T1;
            -- implicit: function "-" (Left, Right : T1) return T1;
            -- ...
            type T2 is null record;
            -- implicit: function "=" (Left, Right : T2) return T2;
         end P;
 
* These primitives can be used just as any others

      .. code:: Ada

         procedure Main is
            V1, V2 : P.T1;
         begin
            V1 := P."+" (V1, V2);
         end Main;
 
===================
Simple Derivation
===================

------------------------
Simple Type Derivation
------------------------

* In Ada, any (non-tagged) type can be derived
    
  .. code:: Ada
    
    type Child is new Parent;
     
* A child is a distinct type that inherits from:

   - The data representation of the parent
   - The primitives of the parent
    
   .. code:: Ada
    
      procedure Test is
        type Parent is range 1 .. 10;
        procedure Prim (V : Parent);
        type Child is new Parent;
        --  implicit Prim (V : Child);
        V : Child;
      begin
        V := 5;
        Prim (V);
     
* Conversions are possible for non-primitive operations
    
   .. code:: Ada
    
      package P is
        type Parent is range 1 .. 10;
        type Child is new Parent;
     end P;
       
     procedure Main is
        procedure Not_A_Primitive (V : Parent);
        V1 : Parent;
        V2 : Child;
     begin
        Not_A_Primitive (V1);
        Not_A_Primitive (Parent (V2));
     end Main;
     
--------------------------------------
Simple Derivation and Type Structure
--------------------------------------

* The structure of the type has to be kept

   - An array stays an array
   - A scalar stays a scalar

* Scalar ranges can be reduced

   .. code:: Ada

      type Int is range -100 .. 100;
      type Nat is new Int range 0 .. 100;
      type Pos is new Nat range 1 .. 100;
 
* Constraints on unconstrained types can be specified

   .. code:: Ada

      type Arr is array (Integer range <>) of Integer;
      type Ten_Elem_Arr is new Arr (1 .. 10);
      type Rec (Size : Integer) is record
         Elem : Arr (1 .. Size);
      end record;
      type Ten_Elem_Rec is new Rec (10);

------------------------------------------
Simple Derivation and List of Operations
------------------------------------------

.. admonition:: Language Variant

   Ada 2005

* Operations can be overridden

   + Overriding can be checked by optional `overriding` reserved word

   .. code:: Ada

      type Root is range 1 .. 100;
      procedure Prim (V : Root);
      type Child is new Root;
      overriding procedure Prim (V : Child);
 
* Operations can be added

   + Addition can be checked by optional `not overriding` reserved word

   .. code:: Ada

      type Root is range 1 .. 100;
      procedure Prim (V : Root);
      type Child is new Root;
      not overriding procedure Prim2 (V : Child);
 
* Operations can be removed

   + Removal can be checked by optional `overriding` reserved word

   .. code:: Ada

      type Root is range 1 .. 100;
      procedure Prim (V : Root);
      type Child is new Root;
      overriding procedure Prim (V : Child) is abstract;
 
======================
Signed Integer Types
======================

----------------------------------
Signed Integer Types (Revisited)
----------------------------------

* The *Basic Types* lecture introduced Ada's signed integer types, and the predefined integer types in package `Standard`.
* But ... we missed one important detail.
* A declaration like this:

   .. code:: Ada

      type T is range L .. R;
 
* Is actually a short-hand for:

   .. code:: Ada

      type <Anon> is new Predefined_Integer_Type;
      subtype T is <Anon> range L .. R;
 
----------------------------------
Signed Integer Types Explanation
----------------------------------

.. code:: Ada

   type <Anon> is new Predefined-Integer-Type;
   subtype T is <Anon> range L .. R;
 
* What's going on?

   - The compiler looks at L and R (which must be static) and chooses a predefined signed integer type from `Standard` (e.g. `Integer`, `Short_Integer`, `Long_Integer`, etc.) which at least includes the range L .. R.
   - This choice is implementation-defined.
   - An anonymous type `Anon` is created, derived from that predefined type. `Anon` inherits all of the predefined type's primitive operations, like ``+``, ``-``, ``*`` and so on.
   - A subtype `T` of `Anon` is created with range L .. R

      + `Anon` can be referred to as `T'Base` in your program.

------------------------------
Signed Integer Types Warning
------------------------------

.. code:: Ada

   type <Anon> is new Predefined-Integer-Type;
   subtype T is <Anon> range L .. R;
 
* Warning! The choice of `T'Base` affects whether runtime computations will overflow.

   - Example: on one machine, the compiler chooses `Integer`, which is 32-bit, and your code runs fine with no overflows.
   - On another machine, a compiler might choose `Short_Integer`, which is 16-bit, and your code will fail an *overflow check*
   - Extra care is needed if you have two compilers - e.g. for Host (like Windows or Linux) and Cross targets...

* Good news! GNAT makes consistent and predictable choices on all major platforms.

-------------------------------
Signed Integer Types Guidance
-------------------------------

* You can avoid the implementation-defined choice by deriving your own Base Types explicitly, and using `Assert` to enforce the expected range

   - Something like

   .. code:: Ada

      type My_Base_Integer is new Integer;
      pragma Assert (My_Base_Integer'First = -2**31);
      pragma Assert (My_Base_Integer'Last = 2**31-1);
 
* Then derive further types and subtypes from `My_Base_Integer`
* Don't assume that "Shorter = Faster" for integer maths. On some machines, 32-bit is more efficient than 8- or 16-bit maths!

--------------------------------------
Signed Integer Types Guidance (cont)
--------------------------------------

* If you want to derive from a base type that has a well-defined bit length (for example when dealing with hardware registers that must be a particular bit length), then package Interfaces declares types such as:

.. code:: Ada

   type Integer_8 is range -2**7 .. 2**7-1;
   for Integer_8'Size use 8;
   -- and so on for 16, 32, 64 bit types...
 
===================
Tagged Derivation
===================

-------------------
Tagged Derivation
-------------------

* Simple derivation cannot change the structure of a type
* Tagged derivation applies only to `tagged` record and allows fields to be added
* An Ada `tagged` type is the equivalent of a C++ class in terms of OOP

------------------------------
Tagged Derivation Ada vs C++
------------------------------

.. container:: columns

 .. container:: column

    .. code:: Ada
    
       type T is tagged record
         Attr_D : Integer;
       end record;
       procedure Attr_F (This : T);
       type T2 is new T with record
         Attr_D2 : Integer;
       end record;
       overriding
       procedure Attr_F (This : T2);
       procedure Attr_F2 (This : T2);
     
 .. container:: column
    
    .. code:: C++
    
       class T {
         public:
           int Attr_D;
           virtual void Attr_F(void);
         };
       
       class T2 : public T {
         public:
           int Attr_D2;
           virtual void Attr_F(void);
           virtual void Attr_F2(void);
         };
     
--------------------------------------
Forbidden Operations in Tagged Types
--------------------------------------

* A tagged derivation has to be a type extension
    
   .. code:: Ada
    
      type Root is tagged record
         F1 : Integer;
      end record;
      type Child is new Root; -- illegal
     
* A tagged derivation cannot remove primitives

*  Conversions from child to parent are allowed, but not the other way around (need extra fields to be provided)
    
   .. code:: Ada
    
      type Root is tagged record
          F1 : Integer;
        end record;
      type Child is new Root with record
          F2 : Integer;
        end record;
      V1 : Root  := (F1 => 0);
      V2 : Child := (F1 => 0, F2 => 0);
      ...
      V1 := Root (V2);
      V2 := Child (V1); -- illegal
      V2 := (V1 with F2 => 0);
     
------------
Primitives
------------

* As for regular types, primitives are implicitly inherited, and can be overridden
* A child can add new primitives
    
   .. code:: Ada
    
      type Root is tagged null record;
      procedure Prim1 (V : Root);
      procedure Prim2 (V : Root);
      type Child is new Root with null record;
      overriding procedure Prim1 (V : Child);
      not overriding procedure Prim3 (V : Child);
      -- implicitly inherited:
      -- procedure Prim2 (V : Child);
     
* The parameter which the subprogram is primitive of is called the controlling parameter
* All controlling parameters must be of the same type
    
   .. code:: Ada
    
      type Root1 is tagged null record;
      type Root2 is tagged null record;
      procedure P1 ( V1 : Root1;
                     V2 : Root1);
      procedure P2 ( V1 : Root1;
                     V2 : Root2); -- illegal
 
------------------
Tagged Aggregate
------------------

* Regular aggregate works - values must be given to all fields of the type hierarchy
    
   .. code:: Ada
    
       type Root is tagged record
           F1 : Integer;
         end record;
         type Child is new Root with
           record
           F2 : Integer;
         end record;
         V2 : Child := (F1 => 0, F2 => 0);
     
* Doesn't work if there are private types involved!

* Aggregate extension allows using a copy of parent instance, or default initialization of the parent
* `with null record` can be used when there are no additional components
    
   .. code:: Ada
    
      V  : Root := (F1 => 0);
      V2 : Child := (V with F2 => 0);
      V3 : Child := (Root with F2 => 0);
      V4 : Empty_Child := (Root with null record);
     
--------------
Freeze Point
--------------

* Ada doesn't explicitly identify the end of the list of members
* This end is the implicit "freeze point" occurring whenever:

   - A variable of the type is declared
   - The type is derived
   - The end of the scope is reached

* Declaring primitives on a tagged type past this point is an error

.. code:: Ada
    
   type Root is tagged null record;
   procedure Prim (V : Root);
   type Child is new Root
      with null record; -- freeze root
   procedure Prim2 (V : Root); -- illegal

   V : Child; --  freeze child
   procedure Prim3 (V : Child); -- illegal

-----------------
Prefix Notation
-----------------

.. admonition:: Language Variant

   Ada 2012

* Primitives of tagged types can be called like any other
    
   .. code:: Ada
    
      type Root is tagged record
         F1 : Integer;
      end record;
      procedure Prim1 (V : Root);
      procedure Prim2 (V : access Root; V2 : Integer);
      type Root_Access is access all Root;
      X  : Root_Access := new Root;
      X2 : aliased Root;
      ...
      Prim1 (X.all);
      Prim2 (X2'Access, 5);
     
* When the first parameter is a controlling parameter, the call can be prefixed by the object
    
   .. code:: Ada
    
      X.Prim1;
      X.all.Prim1;
      X.Prim2 (5);
      X2'Access.Prim2 (5);
     
* No `use` or `use type` clause is needed to have visibility over the primitives in this case

========
Lab
========

.. include:: labs/170_inheritance.lab.rst

=========
Summary
=========

---------
Summary
---------

* *Primitive* operations on types

   - An operation is any subprogram that acts on a type
   - Single subprogram can be a primitive for multiple types

      + Any type referenced in the subprogram

* Simple type derivation

   - Types derived from other types can only add limitations

      + Constraints, ranges
      + Cannot change underyling structure

* Tagged derivation

   - Building block for OOP types in Ada
