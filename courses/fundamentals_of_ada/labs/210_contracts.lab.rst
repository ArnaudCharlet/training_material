---------------
Contracts Lab
---------------

* Overview

   - Create a simplistic test grading system

      + Three types of tests (e.g. quiz, test, final exam)
      + Multiple courses (e.g. math, gym, history, etc )
      + Method to get average of all grades

* Requirements

   - Maximum grade is dependent on type of test
   - Some (non-contiguous) subset of the tests have a higher maximum
   - Cannot take an average of an empty list of grades
   - Average is guaranteed to be between the lowest and highest scores given

* Hints

   - *Subtype Predicate* to determine tests with a higher maximum score
   - *Type Invariant* to ensure record containing course, test type, and score is consistent
   - *Pre-condition* to ensure non-empty array passed into `Average` method
   - *Post-condition* to ensure `Average` result is reasonable

-----------------------------------------
Contracts Lab Solution - Grading (Spec)
-----------------------------------------
.. code:: Ada

   package Grading is

     type Test_T is (Quiz, Test, Final);
     type Course_T is (Arithmetic, Gym, History, Language, Science, Writing);
     subtype Stem_T is Course_T with Static_Predicate => Stem_T in Arithmetic | Science;
     type Grade_T is private;

     function Grade (Kind   : Test_T;
                     Course : Course_T;
                     Score  : Natural)
                     return Grade_T
       with Pre => Score in 0 .. Max_Score (Course, Kind);

     function Scale (Kind  : Course_T; Score : Natural) return Integer is
       (if Kind in Stem_T then Score + 10 else Score);
     function Max_Score (Course : Course_T; Kind   : Test_T) return Natural is
      (case Kind is when Quiz => Scale (Course, 50),
        when Test => Scale (Course, 100), when Final => Scale (Course, 200));

     type Grades_T is array (Integer range <>) of Grade_T;
     function Lowest_Score (Grades : Grades_T) return Integer
       with Pre => Grades'Length > 0;
     function Highest_Score (Grades : Grades_T) return Integer
       with Pre => Grades'Length > 0;
     function Average (Grades : Grades_T) return Integer
       with Pre  => Grades'Length > 0,
            Post => Average'Result >= Lowest_Score (Grades)
                    and then Average'Result <= Highest_Score (Grades);

   private

     type Grade_T is record
       Course : Course_T := Course_T'First;
       Score  : Natural  := 0;
       Kind   : Test_T   := Test_T'First;
     end record with
       Type_Invariant => Score in 0 .. Max_Score (Grade_T.Course, Grade_T.Kind);

   end Grading;

-----------------------------------------
Contracts Lab Solution - Grading (Body)
-----------------------------------------
.. code:: Ada

   package body Grading is

     function Grade (Kind   : Test_T;
                     Course : Course_T;
                     Score  : Natural)
                     return Grade_T is
     begin
       return Ret_Val : Grade_T := (Kind => Kind, Score => Score, Course => Course);
     end Grade;

     function Lowest_Score (Grades : Grades_T) return Integer is
       Lowest : Integer := Grades (Grades'First).Score;
     begin
       for I in Grades'First + 1 .. Grades'Last loop
         Lowest := Integer'Min (Grades (I).Score, Lowest);
       end loop;
       return Lowest;
     end Lowest_Score;

     function Highest_Score (Grades : Grades_T) return Integer is
       Highest : Integer :=
        Scale (Grades (Grades'First).Course, Grades (Grades'First).Score);
     begin
       for I in Grades'First + 1 .. Grades'Last loop
         Highest := Integer'Max (Scale (Grades (I).Course, Grades (I).Score),
                                 Highest);
       end loop;
       return Highest;
     end Highest_Score;

     function Average (Grades : Grades_T) return Integer is
       Average : Integer := 0;
     begin
       for Grade of Grades loop
         Average := Average + Scale (Grade.Course, Grade.Score);
       end loop;
       return Average / Grades'Length;
     end Average;

   end Grading;

----------------------------------------------
Contracts Lab Solution - Main (Declarations)
----------------------------------------------

.. code:: Ada

   with Ada.Exceptions;
   with Ada.Text_IO; use Ada.Text_IO;
   with Grading;     use Grading;
   procedure Main is

     Grades    : Grades_T (1 .. 100);
     Last_Used : Natural := 0;

     function Score return Integer is
     begin
       Put ("  Score: ");
       return Integer'Value (Get_Line);
     end Score;

     generic
       type Ask_T is (<>);
     function Ask return Ask_T;
     function Ask return Ask_T is
     begin
       Put ("  " & Ask_T'Image (Ask_T'First));
       for I in Ask_T'Succ (Ask_T'First) .. Ask_T'Last loop
         Put (" | " & Ask_T'Image (I));
       end loop;
       Put (":");
       return Ask_T'Value (Get_Line);
     end Ask;

     function Kind is new Ask (Test_T);
     function Course is new Ask (Course_T);

--------------------------------------
Contracts Lab Solution - Main (Body)
--------------------------------------

.. code:: Ada

   begin
     loop
       Put_Line ("Grade" & Integer'Image (Last_Used + 1));
       declare
         Grade : Grade_T;
       begin
         Grade := Grading.Grade (Kind   => Kind,
                                 Course => Course,
                                 Score  => Score);
         Last_Used          := Last_Used + 1;
         Grades (Last_Used) := Grade;
       exception
         when The_Err : others =>
           Put_Line (Ada.Exceptions.Exception_Message (The_Err));
           exit;
       end;
     end loop;

     Put_Line ("average: " & Integer'Image (Average (Grades (1 .. Last_Used))));

   end Main;
