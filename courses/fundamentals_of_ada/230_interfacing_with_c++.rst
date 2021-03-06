
**********************
Interfacing with C++
**********************

==============
Introduction
==============

--------------
Introduction
--------------

* Lots of C/C++ code out there already

   - Maybe even a lot of reusable code in your own repositories

* Need a way to interface Ada code with existing C/C++ libraries

   - Built-in mechanism to define ability to import objects from C/C++ or export Ada objects

* Passing data between languages can cause issues

   - Sizing requirements
   - Passing mechanisms (by reference, by copy)

=================
Import / Export
=================

------------------------------
Pragma Import / Export (1/2)
------------------------------

* `Pragma Import` allows a C implementation to complete an Ada specification

   - Ada view
    
      .. code:: Ada
    
         procedure C_Proc;
         pragma Import (C, C_Proc, "c_proc");
     
   - C implementation
    
       .. code:: C++
    
          void c_proc (void) {
             // some code
          }
     
* `Pragma Export` allows an Ada implementation to complete a C specification

   - Ada implementation
    
       .. code:: Ada
    
          procedure Ada_Proc;
          pragma Export (C, Ada_Proc, "ada_proc");
          procedure Ada_Proc is
          begin
           -- some code
          end Ada_Proc;
     
   - C view
    
       .. code:: C++
    
          extern void ada_proc (void);
     
------------------------------
Pragma Import / Export (2/2)
------------------------------

* You can also import/export variables

   - Variables imported won't be initialized
   - Ada view

      .. code:: Ada

         My_Var : integer_type;
         Pragma Import ( C, My_Var, "my_var" );
 
   - C implementation

      .. code:: C++

         int my_var;
 
.. code:: Ada

   procedure Ada_Proc;
   pragma Export (
      C, -- convention (could be Asm, Ada, Fortran, etc)
      Ada_Proc, -- To be exported (beware overloading!)
      "ada_proc"); -- Externally referenced name (optional)
 
-----------------------------
Import / Export in Ada 2012
-----------------------------

.. admonition:: Language Variant

   Ada 2012

* In Ada 2012, Import and Export can also be done using aspects:

   .. code:: Ada

      procedure C_Proc
        with Import,
             Convention    => C,
             External_Name => "c_proc";
 
===================
Parameter Passing
===================

-----------------------------
Parameter Passing to/from C
-----------------------------

* The mechanism used to pass formal subprogram parameters and function results depends on:

   - The type of the parameter
   - The mode of the parameter
   - The Convention applied on the Ada side of the subprogram declaration.

* The exact meaning of *Convention C*, for example, is documented in *LRM* B.1 - B.3, and in the *GNAT User's Guide* section 3.11.

-----------------------------------
Passing Scalar Data as Parameters
-----------------------------------

* C types are defined by the Standard
* Ada types are implementation-defined
* GNAT standard types are compatible with C types

   - Implementation choice, use carefully.

* At the interface level, scalar types must be either constrained with representation clauses, or coming from Interfaces.C
  
* Ada view
    
   .. code:: Ada
    
      with Interfaces.C;
      function C_Proc (I : Interfaces.C.Int)
          return Interfaces.C.Int;
      pragma Import (C, C_Proc, "c_proc");
     
* C view
    
   .. code:: C++
    
     int c_proc (int i) {
       /* some code */
     }
     
-----------------------------------
Passing Structures as Parameters 
-----------------------------------

* An Ada record that is mapping on a C struct must:

   - Be marked as convention C to enforce a C-like memory layout
   - Contain only C-compatible types

* C View

   .. code:: C++

     enum Enum {E1, E2, E3};
     struct Rec {
        int A, B;
        Enum C;
     };
 
* Ada View

   .. code:: Ada

     type Enum is (E1, E2, E3);
     Pragma Convention ( C, Enum );
     type Rec is record
       A, B : int;
       C : Enum;
     end record;
     Pragma Convention ( C, Rec );
 
* Using Ada 2012 aspects

   .. code:: Ada

     type Enum is (E1, E2, E3) with Convention => C;
     type Rec is record
       A, B : int;
       C : Enum;
    end record with Convention => C;
 
-----------------
Parameter modes
-----------------

* `in` scalar parameters passed by copy
* `out` and `in out` scalars passed using temporary pointer on C side
* By default, composite types passed by reference on all modes except when the type is marked `C_Pass_By_Copy`

   - Be very careful with records - some C ABI pass small structures by copy!

* Ada View
    
   .. code:: Ada
    
      Type R1 is record
         V : int;
      end record
      with Convention => C;
          
      type R2 is record
         V : int;
      end record
      with Convention => C_Pass_By_Copy;
     
* C View
    
   .. code:: C++
 
      struct R1{
         int V;
      };
      struct R2 {
         int V;
      };
      void f1 (R1 p);
      void f2 (R2 p);
     
====================
Complex Data Types
====================

--------
Unions
--------

* C unions can be bound using the `Unchecked_Union` aspect
* These types must have a mutable discriminant for convention purpose, which doesn't exist at run-time

   - All checks based on its value are removed - safety loss
   - It cannot be manually accessed

* Ada View
    
   .. code:: Ada
    
      type Rec (Flag : Boolean := False) is
      record
         case Flag is
            when True =>
               A : int;
            when False =>
               B : float;
         end case;
      end record
      with Unchecked_Union,
           Convention => C;
     
* C View
    
   .. code:: C++
    
      union Rec {
         int A;
         float B;
      };
     
--------------------
Arrays Interfacing
--------------------

* In Ada, arrays are of two kinds:

   - Constrained arrays
   - Unconstrained arrays

* Unconstrained arrays are associated with

   - Components
   - Bounds

* In C, an array is just a memory location pointing (hopefully) to a structured memory location

   - C does not have the notion of unconstrained arrays

* Bounds must be managed manually

   - By convention (null at the end of string)
   - By storing them on the side

* Only Ada constrained arrays can be interfaced with C

----------------------
Arrays from Ada to C
----------------------

* An Ada array is a composite data structure containing 2 elements: Bounds and Elements

   - **Fat pointers**

* When arrays can be sent from Ada to C, C will only receive an access to the elements of the array

* Ada View
    
   .. code:: Ada
    
      type Arr is array (Integer range <>) of int;
      procedure P (V : Arr; Size : int);
      pragma Import (C, P, "p");
     
* C View
    
   .. code:: C++
    
      void p (int * v, int size)  {
      }
     
----------------------
Arrays from C to Ada
----------------------

* There are no boundaries to C types, the only Ada arrays that can be bound must have static bounds
* Additional information will probably need to be passed

* Ada View
    
   .. code:: Ada
    
      -- DO NOT DECLARE OBJECTS OF THIS TYPE
      type Arr is array (0 .. Integer'Last) of int;
          
      procedure P (V : Arr; Size : int);
      pragma Export (C, P, "p");
          
      procedure P (V : Arr; Size : int) is
      begin
         for J in 0 .. Size - 1 loop
            -- code;
         end loop;
      end P;
     
* C View
    
   .. code:: C++
    
      extern void p (int * v, int size);
      int x [100];
      p (x, 100);
     
---------
Strings
---------

* Importing a `String` from C is like importing an array - has to be done through a constrained array
* `Interfaces.C.Strings` gives a standard way of doing that
* Unfortunately, C strings have to end by a null character
* Exporting an Ada string to C needs a copy!

   .. code:: Ada

      Ada_Str : String := "Hello World";
      C_Str : chars_ptr := New_String (Ada_Str);
 
* Alternatively, a knowledgeable Ada programmer can manually create Ada strings with correct ending and manage them directly

   .. code:: Ada

      Ada_Str : String := "Hello World" & ASCII.NUL;
 
* Back to the unsafe world - it really has to be worth it speed-wise!

========
Lab
========

.. include:: labs/230_interfacing_with_c++.lab.rst

=========
Summary
=========

---------
Summary
---------

* Possible to interface with other languages (typically C/C++)
* Ada provides some built-in support to make interfacing simpler
* Crossing languages can be made safer

   - But it still increases complexity of design / implementation
