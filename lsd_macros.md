LSD Macro Language

Search the Macros of interest below, considering that most of them are part of a family with postfixes “L”, “S”, “LS” etc. For example, the Macro V(…) is part of a family of Macros together with VL(…), VS(…) and VLS(…).

LSD equation file structure
A simulation run consists in a sequence of time steps, during each of which every Variable in the model is “updated”. That is, a piece of code is executed, normally returning a numerical value that is associated to the Variable for that time step. The LSD Equations are these pieces of codes.

The parts of the code for the Equations are separated in blocks (see below). Usually, they are located all in a single file, called the equation file, which is compiled together with the rest of the LSD source code. Normally, this is done using LMM and therefore the user should find the Equation file automatically pre-filled with the minimum required code.

The only required lines in the Equation file are the following:


#include "../src/fun_head_fast.h"

MODELBEGIN

MODELEND

void close_sim( )
{
}

The user should place the Equations only after the keyword MODELBEGIN and before MODELEND.

Every Variable has its own Equation, although a model normally contains many copies of a Variable. In this case, the same piece of code is executed as many times as many copies (instances) of the Variable exist, but, of course, the values used in each execution may change. In fact, Variables (and Parameters) are located “within” Objects, so each copy (or instance) of an Object contains a different copy (or instance) of the Variable, under the same names. The Equations’ code always refers to an Object. For example, suppose an Object named Firm contains a Variable called Q that depends on another Variable, named K. If there are many copies of the Object Firm, each instance of the Equation for Q, that require K for its computation, will refer to its own copy of K (a particular value), that is, the copy of K contained in the same Object as the copy of Q currently under computation.

The Equations’ code for all the Variables is usually located in a single file. The order in which the Equations appear in the file is irrelevant, since LSD automatically evaluates which sequence of Variable computations needs to be executed. Therefore, each Variable and the associated Equation must be thought of as a difference equation, written independently from one another and computed at the generic time t:

Xt = f ( Yt-lag, Zt-lag, ... )

Equation code may also use LSD Parameters. Parameters are numeric values defined by LSD to be used in Equations in the same way as variables. Parameters are not defined by specific Equations, instead, they are set by the LSD configuration in use when the simulation starts and often remain constant along the entire run. However, Parameters can be still changed by other Variables’ Equations “on-the-flight”, see Macros like WRITE(…) or INCR(…), being the modeler responsible for the control of the update timing and value.

LSD Equation files are composed by standard C++ code plus LSD-specific keywords (Macro, Object, Variable and Parameter names). All LSD Macros are named in upper case. The labels of LSD Objects, Variables and Parameters must be represented within straight quotation marks (""), as in "Lab", without any trailing or leading blank characters (like spaces). Labels not inside quotation marks are interpreted as regular C++ identifiers and not as LSD names, more likely leading to compilation errors. In C++ terms, LSD labels for Variables and Parameters must be null terminated (likely constant) character arrays. In the LSD Macros presented below, all the included quotation marks are compulsory for proper simulation operation. Also, all identifiers not inside quotation marks are intended to be C++ identifiers, most usually regular C++ variable names (like v[n], cur etc.) used in LSD to create the Equations code. There is no connection of LSD Variables (and Parameters) with C++ variables, even if they share the same name (which may be confusing but is perfectly legal).

LSD also accepts no Macro, standard C++ code to be used. In this case the actual LSD functions must be expressed in pure C++, please refer to the manual for the original LSD language for details. This is not the recommended approach to most users, in particular less experienced programmers.



EQUATION( "Lab" )
The keyword EQUATION(…) starts a block of code in the equation file, to be associated to a Variable (or a Function), as in the example below:

EQUATION( "Lab" )
/*
Normally here should be place a comment, specifying what the equation does
*/

RESULT( 3 ) // This Equation assigns always the value 3 to any copy of variable Lab

The first line indicates that the code refer to the Variable (or Function) labelled Lab. After the initial line, normally, it is added a comment (highlighted in green). The following lines can contain any LSD (or C++) code. The last line must be indicated with the keyword RESULT(…), where in between the parenthesis it is contained the value that the Variable (or Function) is to be assigned.

If EQUATION( "Lab" ) is associated to a Variable named Lab, each instance (copy) of the Variable must be computed once and only once at every time step. Therefore, for example, if other Variables uses its value in other Equations, then the Equation provides always the same value during the same time step. That is, the numerical value of an instance of Lab at time t is always identical.

Notice that, because of this particular updating scheme, the concept of Equation when used together with a LSD Variable is very different from the operation of a regular subroutine or procedure in most computer languages. Subroutines and procedures are usually reevaluated every time they are invoked, irrespective to the relative timing between calls. A similar updating scheme, despite not commonly used in LSD models, can be performed if the Equation is associated with a Function instead of a Variable (this association is defined during the model structure configuration).

LSD offers numerous Macros to be used when creating Equations. The most frequently used is V(…) that simply computes or retrieves the value of another Variable, Function or Parameter in the model. When the computations required for an Equation or a Function are slightly less than trivial, it is common to use temporary variables to store intermediate results. For example, consider the following Equation:

EQUATION( "PROF" )
/***************************
The equation computes the profit rate:
PROF(t) = P(t) * A(t-1) - C - RIM – RIN * Inn
profits per unit of capital are equal current price times lagged productivity
minus the cost for research (innovative firms spend for both type
of research) and fixed costs.
***************************/
RESULT( V("Price") * VL("A", 1) - V("C") - V("RIM") - V("RIN") * V("Inn") )

Warning
The deprecated FUNCTION Macro no longer force an element defined as a Variable in the LSD Browser to operate as a Function, irrespective of the code in the equation file. Model configurations created for old versions of LSD depending on this behavior must be updated to properly set Functions as such in the LSD Browser. The FUNCTION macro is not available when #include "fun_head_fast.h" is used in the equation file, the default, and it is not necessary in new models (an EQUATION Macro can be used to define both Variables and Functions).



Temporary numeric storage: v[n]
The standard way to express an Equation is to collect a set of data from the model (Variables and Parameters) and then to elaborate on them to provide the desired result value for the current Variable. Since the LSD Macros to collect the required data may be quite long, it is considered good practice to store the intermediary values to be used in numerical temporary storage. It may be preferred to store the intermediate values used in the final computation in the LSD provided temporary storage called v[0], v[1], v[2] etc.

Up to 1,000 temporary storage positions are available (from v[0] to v[999]) to each Equation. The values of v[…] are reset for each Equation, and therefore cannot be used to transfer information from one Equation to another or between different time steps of the same Equation. Technically, this storage is configured as a volatile C++ double precision array.

Example
The example above can be rewritten using the temporary storage:

EQUATION( "PROF" )
/***************************
The equation computes the profit rate:
PROF(t) = P(t) * A(t-1) - C - RIM – RIN * Inn
profits per unit of capital are equal current price times lagged productivity
minus the cost for research (innovative firms spend for both type
of research) and fixed costs.
***************************/
v[0] = V( "Price" );
v[1] = VL( "A",1 );
v[2] = V( "C" );
v[3] = V( "RIM" );
v[4] = V( "RIN" );
v[5] = V( "Inn" );

RESULT( v[0] * v[1] - v[2] - v[3] - v[4] * v[5] )

This practice has several advantages. Firstly, the code generally is more readable when using shorter symbols for values. Secondly, the intermediate values can be observed using the LSD Debugger, so tracing errors get easier. Thirdly, if a value is used many times in the same Equation, it is faster and less prone to errors to store the value in a v[…] temporary variable and just re-use it instead of re-calling the Macro providing it.



Object structure
Most LSD Macros are usually operated “within” the Object instance, as in the examples above, implicitly defined by the LSD Macro THIS (or the predefined C++ pointer p). The reason is that LSD Macros in Equations by default operate on Variables, Parameters, and Functions which are located in the same Object instance as the Variable or Function being computed is. For example, consider a model with an Object Firm containing Variables Profit, Price and Quantity, and the Equation for Profit is (neglecting the temporal indexes):

Profit = Price × Quantity

In general, the model may have many copies of Object Firm, and each copy of Variable Profit must use the values of Quantity and Price referring to the same instance of Object Firm. LSD takes care automatically of identifying the correct values in the code of the Equation, using the Object the LSD Variable is attached to as the reference. In particular:

·         THIS (or C++ p) refers to the Object instance containing the Variable whose Equation is being computed

·         CALLER (or C++ c) refers to the Object instance that requested the computation of the Variable

·         cur, cur1, cur2 etc. are temporary C++ pointers, which can store the current addresses of Object instances obtained from other LSD Macros

 

Warning: Despite the availability of C++ predefined pointers to operate on the Object structure, their use should be avoided. Very importantly, the corresponding LSD Macros offer pointer checking to the operations, preventing hard-to-find crashes and easily pointing error locations to the modeler. If required, for a small performance gain, pointer checking can be disabled (see NO_POINTER_CHECK) after the simulation model is free of bugs.

Current Object: THIS (or C++ p)
THIS Macro (and the predefined C++ pointer p) refers to the Object instance containing the Variable or Function whose Equation is being executed (as “this” instance). Since most of the operations in an Equation concern the values in the same Object instance, this is the most frequently used in the Equations, and in most of cases the LSD Macros allow to use it by default without specifying it. Only advanced code should require direct access to it.

Caller Object: CALLER (or C++ c)
In most cases, Equations computing some Variable's values are executed because other Equations needed those values. The pointer referred by Macro CALLER (and the predefined C++ pointer c) points to the Object instance which caused that Equation to be computed (the “caller” of the computation), if any. If an Equation is computed only because LSD requested so, because no other Equation triggered the computation, then CALLER is defined as NULL, a conventional value.

Typically, the object pointed by CALLER is used in a Function to acquire more contextual information, since these are not computed unless another Equation explicitly requests their calculation.

Object’s relatives: PARENT(S), GRANDPARENT(S), NEXT(S), ROOT (or C++ up, up->up, next, root)
The information in this paragraph is not necessarily relevant for writing basic LSD models. But it explains the technical implementation of an LSD model that can be used to optimize a model’s code or create more sophisticated data structures.
Every Object instance (for example, THIS, CALLER etc.) is related to its relatives in the model hierarchy tree. Every Object instance can therefore connect to its direct relatives using the links to them. The linking pointers can then be constructed, using C++ syntax, using the following components:

·        The single-instanced top Object in the model hierarchy is given by Macro ROOT which has no PARENT or NEXT relatives (these Macros return NULL in this case)

·        PARENT is the Object instance containing the Object instance (or the parent to) in which the current Variable is being calculated; relatedly, PARENTS( CALLER ) and PARENTS( cur ) points to the parent of the caller instance and of the one pointed by cur, respectively

·        GRANDPARENT, GRANDPARENTS( CALLER ) and GRANDPARENTS( cur ) point to the grandparents of the Object instances THIS, CALLER and cur, and so on

·        NEXT is the Object instance immediately following the current Object THIS (or a brother to) in the list of descendants from its parent PARENT; all the instances in the brotherhood chain share the same parent Object and the last instance in the chain has NEXT equal to NULL

·         Similarly, NEXTS( CALLER ) and NEXTS( cur ) are brothers of CALLER and cur, respectively

Any “younger“ brother instances in the model hierarchy tree can be reached by arbitrary combinations of NEXTS(…) Macros departing from a given Object instance. However, the exploration of previous or “older” brother instances require first the use of the SEARCH(…) Macro family, to identify the “first“ brother instance.

The best way to remember the object's components is to keep in mind a geometrical representation of Objects:

   ROOT
    /\
    ||
     .
     .
     .
    /\
    ||
  PARENT
    /\
    ||
 +------+
 | THIS | => NEXT => ... => NULL
 +------+

The linking pointers provided by the Macros, like the above example, are proper Object instances and therefore can be used as LSD Macro operands. For example, consider a model where you have 100 instances of Object Firm descending from a single-instance Object named Market. In turn, each Firm Object instance has about 1,000 instances of Object Worker descending from it, as in the representation below:

      ROOT
       /\
       ||
   +--------+
   | Market | (contains variable Price)
   +--------+
     /\  /\
     ||  ||
     ||  ++========++
     ||            ||
 +--------+    +--------+    +--------+    +--------+             +----------+
 | Firm 1 | => | Firm 2 | => | Firm 3 | => | Firm 4 | => . . . => | Firm 100 |
 +--------+    +--------+    +--------+    +--------+             +----------+
     /\            /\            /\            /\                      /\
     ||            ||             . . .         . . .                   . . .
     ||            ||
     ||       +----------+    +----------+             +-------------+ 
     ||       | Worker 1 | => | Worker 1 | => . . . => | Worker 1000 |
     ||       +----------+    +----------+             +-------------+
 +----------+    +----------+    +----------+             +-------------+ 
 | Worker 1 | => | Worker 1 | => | Worker 2 | => . . . => | Worker 1000 |
 +----------+    +----------+    +----------+             +-------------+

If a Variable in Object Firm uses a common Variable from Market, say, Price, you can write the line of code:

v[0] = V( "Price" );

However, it will take significant time to work out, because LSD will have to visit and discard all the instances of Worker for each of the instances of Firm before moving “up” to look for the Variable Price in Object Market. In fact, the Macro V(…) will initially search down, into Worker, for a Variable or Parameter Price. Only if it does not find it downward in the tree, it will look then upwards, trying to find Price in the parent Market. This means that LSD must skip 100,000 Object instances, in total, before computing the 100 instances of the Firm’s Variable. Instead, using the code line:

v[0] = VS( PARENT, "Price" );

directly provides the desired value, since the first Object searched is PARENT, which is Market. Note that, for using pointer constructs, the modeler must be sure that final pointed instance exists. If, in the example above, the Equation is placed in Object Market then PARENT now points to ROOT and there is no Variable Price there, so the simulation will issue an error and stop.

Temporary Object's pointers: cur, cur1, cur2, …
Besides temporary variables for intermediate numerical values, LSD also provides a predefined group of temporary pointers to Object instances: cur and from cur1 to cur9. These elements are generic pointers to Object instances, used to temporary store the addresses of required instances, like v[…] is used to stored temporary numerical values.

Typically, the user calls LSD Macros returning Object pointers and assigns the results to cur, cur1, … and uses the pointer in another Macro requiring it as an operand. See the examples of LSD Macros returning Object pointers. Predefined pointer variables are usually initialized to the NULL value, to prevent severe errors when used incorrectly. However, this adds a small overhead to LSD. Pointer pre-initialization can be disabled by adding a #define NO_POINTER_INIT statement before the #include fun_head_fast.h line. Disabling pointer initialization also disable pointer checking (see NO_POINTER_CHECK Macro for details).

For example, when creating a new Object instance, one wants to initialize its Parameters. This is done by storing the new instance “address” in a temporary pointer and applying the LSD required Macros on it. For example, the following lines within an Equation create a new instance of Object Firm and set its Variable K to 0 (see the details in the description of the LSD Macros listed below).

...
cur = ADDOBJ( "Firm" );
WRITELS( cur, "K", 0, t );
...

Hook pointers: HOOK(…)
Any Object instance in an LSD model may contain one or more special elements called hooks. A hook is a pointer to another Object instance, like temporary pointer variable cur, or the result of Macros like PARENT and NEXT, but which is not temporary, existing for the life of the Object instance. Hooks allow connecting one instance to any other in the model. A hook is a modeler-controlled pointer and is not used by LSD except if the Equation’s code manipulates it explicitly.

Each Object is endowed with one static hook plus unlimited optional dynamic hooks. Dynamic hooks may must be created before usage, by means of the ADDHOOK(…), and after that it can be initialized (WRITE_HOOK(…)) and used (HOOK(…)) with corresponding Macros. The current number of dynamic hooks allocated to an Object instance may be obtained using COUNT_HOOK(…).

Hooks are available to the modelers for special purposes, allowing the creation of heterogeneous, multi-layer, ad-hoc networks. Any two Object instances in a model can be “hooked”, irrespective of where they are in the model structure. Also, a single Object may have as many hooks to other Object instances as required. Typically, hooks are employed to efficiently speed up the access to frequently used instances, preventing the need of repeated slow search operations. This way, they allow the creation of highly flexible and efficient data structures embedded by the model structure.

Temporary Link's pointers: curl, curl1, curl2, …
Similarly to pointers to Objects, these elements are special temporary pointer variables to Links connecting Nodes represented by instances of a same Object, defining an homogeneous, sibling-only LSD network (see LSD Networks for details). They are used to temporarily store the addresses of required network Links for LSD Macros requiring or returning Links as operands or as results. See the examples of LSD Macros returning Links.

The temporary Link pointer variables are usually initialized to NULL, to allow detecting errors, at a slight performance cost. Pointer pre-initialization can be disabled by adding a #define NO_POINTER_INIT statement before the #include fun_head_fast.h line. Disabling pointer initialization also disable pointer checking (see NO_POINTER_CHECK Macro for details).

Model-specific objects and other C++ data structures
The Object instances pointed by the Macros and variables discussed above (THIS, CALLER, cur, etc.) are all “local”, in the sense that they represent a different instance depending on the Equation in whose they are used. That is, depending on the Variable whose Equation is computed, they change the pointed instance as the Variable being computed changes: THIS and hooks refers to the Object instance whose Variables are under computation; CALLER to the instance that caused the Variable to be computed; cur and the others are assigned within the specific Equations code.

However, the modeler can create and use global and local Object-instance pointers and other pure C++ data structures (global, local, or associated to LSD Objects), that is, pointers (or data structures) that have persistent content throughout a simulation run or not. The use of global and local Object pointers or pure C++ data structures should be avoided by unexperienced modelers because they frequently create errors difficult to be catch.

To create a global Object pointer, or any C++ data structure, visible to all Equations and with persistent values throughout the simulation, declare it using the regular C++ syntax outside the scope of the Equation area, that is, on the beginning of the Equation file and just before the keyword MODELBEGIN.  For instance, this may be done for optimization purposes, when a very large model (many Objects and hierarchical layers) contain Variables that refer frequently to one specific Object copy (or particular data structure), and an adequate structure of hooks is not sufficient. Example:

#include "../src/fun_head_fast.h"

object *market;

MODELBEGIN

EQUATION( "Init" )
/*
Technical initialization function. It is computed only once and then it is
transformed in a parameter and never computed again.
Sets the global pointer 'market' pointing to the Object Market, so
to speed up the access to this object
*/

market = SEARCH( "Market" ); //assign the C++ object pointer "market" to point to the LSD Object Market
PARAM; //optional; transform "Init" in a parameter so to not compute again this equation

RESULT( 1 )

where the name of the Object ("Market" in the example) is not one of the easily accessible Object instances provided by the existing Macros (THIS, CALLER, cur, etc.). The global pointer must be assigned with any of the LSD Macros returning Object pointers, within any Equation. For example, there may be a Equation for a Variable called Init making such an assignment. The Variable Init must be computed for sure before market is used in any other Equation. This can be ensured placing Init in the Object ROOT, since any simulation time step starts the computation always from Variables contained in the topmost level (i.e., ROOT) of the model structure tree. In the example above, the modeler can now use C++ pointer market as any other Object-instance pointer, knowing that it refers to the LSD Object Market. For example, the Equation for a Variable placed anywhere in the model may use the line:

v[0] = VS( market, "Price" );

and be sure that the value of Variable Price is returned quickly. Note that the usual line

v[0] = V( "Price" );

would also work, but likely less efficiently, as it would cost computation time in case the Variable whose Equation containing the line for Price is located in an Object placed “far” away from Market.

Modelers using global objects or other C++ data structures should be careful when assigning configurations with multiple instances. If, for example, the model configuration uses many instances of the Object Market, any use of the global object market would refer only to the very first instance of the Object Market, certainly undermining the model operation. In this case, for instance, a global C++ vector of object pointers could be used, as in vector < object * > market[ 20 ] for a case using 20 Market instances (and properly initialized in Init).

Local (temporary) C++ data structures can be created in the scope of any Equation using regular C++ syntax. To simplify the creation of local and temporary C++ data types/structures in all Equations, declare them by defining EQ_USER_VARS before the keyword MODELBEGIN. Multiple C++ data types/structures can be separated by “;” and multi-line definitions can be created using “\” as the line terminator.  The declaration line must be something like:

#include "../src/fun_head_fast.h"

#define EQ_USER_VARS object *this_market; \
                     int id_market; \
                     double sales_market;

MODELBEGIN
...

Other than global or local C++ data structures, Object-specific extensions may be also used. This provides a powerful tool for modelers to extend LSD data handling capabilities. For details, please check the extended Object data Macros.



Postfixes and Macro families
Most of the LSD Macros, like V(…) used in the examples above, come as a family where the name of the Macro can be expressed with no postfix, or with “L”, “S”, “LS” etc. as postfix. The postfix “L” stands for Lag, indicating that the operation refers to past values. The postfix “S” indicates a Specific Object instance. Consider that any LSD operation is performed in each Equation, at every time step. Therefore, the time “now” and the Object instance containing the Variable whose Equation is being executed are used by default when no postfix is supplied. Otherwise, for operations that must refer to past time steps, or to Object instances different from the one containing the Variable whose Equation is under execution, the modeler must use one of the post-fixed versions of the LSD Macro and provide the necessary details:

·         If no postfix is specified, the Macro is supposed to refer to the Object instance containing the Variable under computation and to the current time step.

·         The postfix “L” means that the Macro apply to the same Object instance containing the Variable under computation but must be specified the lag to which the operation refers to.

·         The postfix “S” means that the Macro contains the specification to the Object instance to which it must be applied, and it refers to the current time step.

·         The postfix “LS” means that the Macro is applied to the lag and to the Object instance specified.

The Equation's code can always use any Object pointers that are available to the modeler, using Macros, hooks, temporary pointers, etc. One example is the Macro THIS, which provides a pointer to the Object instance containing the Variable under computation. Another is CALLER, which refers to the Object instance containing the Variable that triggered the computation of the Variable, if any (NULL otherwise).



V( "Lab" ), VS( obj, "Lab" )
VL( "Lab", lag ), VLS( obj, "Lab", lag )
These are the most used LSD Macros. They return the value of the Variable or Parameter with label Lab. The forms VL(…) and VLS(…) permit to specify a lag, so that the values returned concern the value of Lab with lag lags. The forms VS(…) and VLS(…) specify in which specific obj Object instance the Parameter or Variable must be searched for.

When no specific obj instance is provided, the search starts from THIS (see available Objects), the current instance containing the Variable under computation. In case the model contains many instances of a Variable named Lab, that is, many instances of an Object containing the Variable, the Macro returns the value of the instance "closer" to the Object where the search started from. By closer, it is meant the first instance found by searching the model using the following strategy:

1)    Search in the current or obj Object instance’s Variables and Parameters

2)    Search in the current or obj descendants’ instances

3)    Search in the current or obj descendants’ descendants’ instances recursively, to the lowest level in the model structure tree

4)    Search in the current or obj parent’s instances

5)    Search in the current or obj Object parent’s parent’s instances recursively, up to the ROOT level

In each Object instance explored, the same strategy is applied recursively, so that every instance in the model structure is visited, if necessary, except if NO_SEARCH Macro is used. This strategy is similar to the one employed when searching for Objects, like in SEARCH(…).

So, for example, if there is only one instance of Variable Lab in the model, this can be found in any upstream or downstream Object, irrespective of where the search is started.

Example
Consider the Equation Qt = Kt-1 × At

The code may be as follows:

EQUATION( "Q" )
/*
Compute the quantity Q as the product of lagged capital K and current productivity A
*/

RESULT( VL( "K", 1 ) * V( "A" ) )

or, equivalently:

EQUATION( "Q" )
/*
Compute the quantity Q as the product of lagged capital K and current productivity A
*/

v[0] = VL( "K", 1 );
v[1] = V( "A" );

RESULT( v[0] * v[1] )

In the second case, the two temporary variables v[0] and v[1] are used to store the intermediate results, which can be observed during a debugging session, so that modeler can trace possible unexpected results.

If the Object containing Lab has no instance (see  USE_ZERO_INSTANCE) the macro returns the NAN (not a number) value.



MAVE( "Lab", per ), MAVES( obj, "Lab", per )
MAVEL( "Lab", per, lag ), MAVELS( obj, "Lab", per, lag )
The MAVE[LS](…) Macros compute the time moving average of Variable named Lab for per periods, optionally starting after the specified lag (effectively using the period from lag to lag+per) the. Similar to V(…), these Macros search for Lab over the entire model structure, starting by the descendants of the current Object (THIS) or obj, except if NO_SEARCH Macro is used.

By default, the moving average considers only Variable values computed during the simulation (for t = 1, 2, …), that is, it does not consider the initial value(s) set in the configuration (the t = 0 value, for instance), if any. However, if a negative value for per is provided, the initial values provided, if any, are all considered for the moving average computation. Naturally, only the absolute value of per is considered in any case.

Please note that, despite the name similarity, MAVE[LS](…) and AVE[LS](…) are completely different Macro families. The former operates the averaging across simulation cases/times, while the latter performs the average computation over Object instances.



WHTMAVE( "Lab", per, weight ), WHTMAVES( obj, "Lab", per, weight )
WHTMAVEL( "Lab", per, weight, lag ), WHTMAVELS( obj, "Lab", per, weight, lag )
The WHTAVE[LS](…) Macros compute the time weighted moving average of Variable named Lab for per periods using the array weight to define importance of each lagged period, optionally starting after the specified lag (effectively using the period from lag to lag+per) the. Similar to V(…), these Macros search for Lab over the entire model structure, starting by the descendants of the current Object (THIS) or obj, except if NO_SEARCH Macro is used.

The weight C++ array of type double must have, at least, length equal to per. The values in the array are used to weight the moving average in the direct order: the first array element (weight[0]) is used for the first lag considered (equal to the current case 0 if lag is not specified), the second element (weight[1]) weights the second effective lag, as so on.

By default, the weighted moving average considers only Variable values computed during the simulation (for t = 1, 2, …), that is, it does not consider the initial value(s) set in the configuration (the t = 0 value, for instance), if any. However, if a negative value for per is provided, the initial values provided, if any, are all considered for the moving average computation. Naturally, only the absolute value of per is considered in any case.

Please note that, despite the name similarity, WHTMAVE[LS](…) and WHTAVE[LS](…) are completely different Macro families. The former operates the averaging across simulation cases/times, while the latter performs the average computation over Object instances.



SUM( "Lab" ), SUMS( obj, "Lab" )
SUML( "Lab", lag ), SUMLS( obj, "Lab", lag )
SUM_CND( "Lab1", "Lab2", "lop", value ), SUM_CNDS( obj, "Lab1", "Lab2", "lop", value )
SUM_CNDL( "Lab1", "Lab2", "lop", value, lag ), SUM_CNDLS( obj, "Lab1", "Lab2", "lop", value, lag )
The SUM[LS](…) Macros search, with the same strategy described in V(…) (see also the different forms), an instance of the Variable or Parameter Lab. Then, they keep on summing up all the values of instances of the Variable or Parameter named Lab (with the lag lag, if specified), found in the set of Object instances which are brothers to the first found. Usually, they are used to sum up the values of a given Variable of descending Object instances.

The SUM_CND[LS](…) variants perform similarly to the SUM[LS](…) case but only consider for summing the Variable or Parameter Lab1 contained in an Object instance which has the Variable or Parameter Lab2 content satisfying the conditional expression defined by the logical relational operator lop and the supplied reference value. That is, the Variable or Parameter named Lab1 is only summed for Object instances where the expression ( V( "Lab2" ) lop value ) evaluates to true. lop can be any of the following standard C++ logical operators: ==, !=, >, >=, <, <=. There is no restriction to the special case of using the same Variable or Object for Lab1 and Lab2. Both Lab1 and Lab2 must be contained in the same Object. If the parameter lag is supplied, it applies to both Lab1 and Lab2, which must have the appropriate number of lags configured.

If the Object containing Lab/Lab1 has no instance (see  USE_ZERO_INSTANCE), or there is no Object instance which satisfies the conditional expression in SUM_CND[LS](…), the Macros return the 0 (zero) value.

Example
If Q_TOT is contained in an Object named Market, from which descend a set of Object instances named Firm containing a Variable with label Q, its Equation can be coded as:

EQUATION( "Q_TOT" )
/*
Compute the sum of all Q's in the market
*/

RESULT( SUM( "Q" ) )
 

Note that, if your model contains many Object instances named Market, this Equation will sum up only the Q's contained in the set of descendants of Market, and not all the Q's existing in the model.

If, otherwise, just some of the instances of Variable Q are to be summed, say, for Object instances of Firm located in a particular country identified by the value of a Parameter named country (contained in the same instance), the corresponding Equation may be written as:

EQUATION( "Q_US" )
/*
Compute the sum of Q's of firms in the US market (country code 1)
*/

RESULT( SUM_CND( "Q", "country", "==", 1 ) )




MAX( "Lab" ), MAXS( obj, "Lab" )
MAXL( "Lab", lag ), MAXLS( obj, "Lab", lag )
MAX_CND( "Lab1", "Lab2", "lop", value ), MAX_CNDS( obj, "Lab1", "Lab2", "lop", value )
MAX_CNDL( "Lab1", "Lab2", "lop", value, lag ), MAX_CNDLS( obj, "Lab1", "Lab2", "lop", value, lag )
Similar to SUM(…), but, instead of the sum, these Macros compute the maximum value of the Variable or Parameter named Lab or Lab1 among the descendants of the current Object (THIS) or obj, using the appropriate lag.

If MAX_CND[LS](…) forms are used, only Object instances satisfying the conditional expression, defined by Variable or Parameter Lab2, the logical operator lop and the reference value, are used in computation. Both Lab1 and Lab2 must be contained in the same Object. See SUM(…) for details and examples.

If the Object containing Lab/Lab1 has no instance (see  USE_ZERO_INSTANCE), or no instance satisfies the conditional expression, the Macros return the NAN (not a number) value. NAN values can be checked using isnan(…)(see USE_NAN).



MIN( "Lab" ), MINS( obj, "Lab" )
MINL( "Lab", lag ), MINLS( obj, "Lab", lag )
MIN_CND( "Lab1", "Lab2", "lop", value ), MIN_CNDS( obj, "Lab1", "Lab2", "lop", value )
MIN_CNDL( "Lab1", "Lab2", "lop", value, lag ), MIN_CNDLS( obj, "Lab1", "Lab2", "lop", value, lag )
Similar to SUM(…), but, instead of the sum, these Macros compute the minimum value of the Variable or Parameter named Lab or Lab1 among the descendants of the current Object (THIS) or obj, using the appropriate lag.

If MIN_CND[LS](…) forms are used, only Object instances satisfying the conditional expression, defined by Variable or Parameter Lab2, the logical operator lop and the reference value, are used in computation. Both Lab1 and Lab2 must be contained in the same Object. See SUM(…) for details and examples.

If the Object containing Lab/Lab1 has no instance (see  USE_ZERO_INSTANCE), or no instance satisfies the conditional expression, the Macros return the NAN (not a number) value. NAN values can be checked using isnan(…)(see USE_NAN).



AVE( "Lab" ), AVES( obj, "Lab" )
AVEL( "Lab", lag ), AVELS( obj, "Lab", lag )
AVE_CND( "Lab1", "Lab2", "lop", value ), AVE_CNDS( obj, "Lab1", "Lab2", "lop", value )
AVE_CNDL( "Lab1", "Lab2", "lop", value, lag ), AVE_CNDLS( obj, "Lab1", "Lab2", "lop", value, lag )
Similar to SUM(…), but, instead of the sum, these Macros compute the average (arithmetic mean) of the Variable or Parameter named Lab or Lab1 among the descendants of the current Object (THIS) or obj, using the appropriate lag.

If AVE_CND[LS](…) forms are used, only Object instances satisfying the conditional expression, defined by Variable or Parameter Lab2, the logical operator lop and the reference value, are used in computation. Both Lab1 and Lab2 must be contained in the same Object. See SUM(…) for details and examples.

If the Object containing Lab/Lab1 has no instance (see  USE_ZERO_INSTANCE), or no instance satisfies the conditional expression, the Macros return the NAN (not a number) value. NAN values can be checked using isnan(…)(see USE_NAN).



WHTAVE( "Lab1", "Lab2" ), WHTAVES( obj, "Lab1", "Lab2" )
WHTAVEL( "Lab1", "Lab2", lag ), WHTAVELS( obj, "Lab1", "Lab2", lag )
WHTAVE_CND( "Lab1", "Lab2", "Lab3", "lop", value ), WHTAVE_CNDS( obj, "Lab1", "Lab2", "Lab3", "lop", value )
WHTAVE_CNDL( "Lab1", "Lab2", "Lab3", "lop", value, lag ), WHTAVE_CNDLS( obj, "Lab1", "Lab2", "Lab3", "lop", value, lag )
Similar to SUM(…), but, instead of the sum, these Macros compute the weighted average of the Variable or Parameter named Lab1 among the descendants of the current Object (THIS) or obj, using the Variable or Parameter Lab2 as the weight, that is, the sum of the products between the values of Lab1 and Lab2, over the appropriate lag. There is no enforcement or requirement that the values contained in element Lab2 add up to one. So, it is up to the modeler to ensure this condition to ensure the produced result represents the desired value. Otherwise, the WHTAVE[LS](…) macros can be used to sum arbitrary products of the elements Lab1 and Lab2.

If WHTAVE_CND[LS](…) forms are used, only Object instances satisfying the conditional expression, defined by Variable or Parameter Lab3, the logical operator lop and the reference value, are used in computation. See SUM(…) for details and examples.

Elements Lab1, Lab2 and Lab3, if present, must be contained in the same Object.

Differently from AVE(…), If the Object containing Lab/Lab1 has no instance (see  USE_ZERO_INSTANCE), or no instance satisfies the conditional expression, the Macros return the 0 (zero) value.



MED( "Lab" ), MEDS( obj, "Lab" )
MEDL( "Lab", lag ), MEDLS( obj, "Lab", lag )
MED_CND( "Lab1", "Lab2", "lop", value ), MED_CNDS( obj, "Lab1", "Lab2", "lop", value )
MED_CNDL( "Lab1", "Lab2", "lop", value, lag ), MED_CNDLS( obj, "Lab1", "Lab2", "lop", value, lag )
Similar to SUM(…), but, instead of the sum, these Macros compute the median of the Variable or Parameter named Lab or Lab1 among the descendants of the current Object (THIS) or obj, using the appropriate lag.

If MED_CND[LS](…) forms are used, only Object instances satisfying the conditional expression, defined by Variable or Parameter Lab2, the logical operator lop and the reference value, are used in computation. Both Lab1 and Lab2 must be contained in the same Object. See SUM(…) for details and examples.

If the Object containing Lab/Lab1 has no instance (see  USE_ZERO_INSTANCE), or no instance satisfies the conditional expression, the Macros return the NAN (not a number) value. NAN values can be checked using isnan(…)(see USE_NAN).



PERC( "Lab", perc ), PERCS( obj, "Lab", perc )
PERCL( "Lab", perc, lag ), PERCLS( obj, "Lab", perc, lag )
PERC_CND( "Lab1", perc, "Lab2", "lop", value ), PERC_CNDS( obj, "Lab1", perc, "Lab2", "lop", value )
PERC_CNDL( "Lab1", perc, "Lab2", "lop", value, lag ), PERC_CNDLS( obj, "Lab1", perc, "Lab2", "lop", value, lag )
Similar to SUM(…), but, instead of the sum, these Macros compute the percentile defined by perc (p) of the Variable or Parameter named Lab or Lab1 among the descendants of the current Object (THIS) or obj, using the appropriate lag. The percentile indicates the value below which a given percentage (P) of observations in a group of observations falls. The percentile perc should be informed in the format P = p / 100, and must be in the range 0 ≤ p ≤ 1.

If PERC_CND[LS](…) forms are used, only Object instances satisfying the conditional expression, defined by Variable or Parameter Lab2, the logical operator lop and the reference value, are used in computation. Both Lab1 and Lab2 must be contained in the same Object. See SUM(…) for details and examples.

If the Object containing Lab/Lab1 has no instance (see  USE_ZERO_INSTANCE), or no instance satisfies the conditional expression, the Macros return the NAN (not a number) value. NAN values can be checked using isnan(…)(see USE_NAN).



SD( "Lab" ), SDS( obj, "Lab" )
SDL( "Lab", lag ), SDLS( obj, "Lab", lag )
SD_CND( "Lab1", "Lab2", "lop", value ), SD_CNDS( obj, "Lab1", "Lab2", "lop", value )
SD_CNDL( "Lab1", "Lab2", "lop", value, lag ), SD_CNDLS( obj, "Lab1", "Lab2", "lop", value, lag )
Similar to SUM(…), but, instead of the sum, these Macros compute the (population) standard deviation of the Variable or Parameter named Lab or Lab1 among the descendants of the current Object (THIS) or obj, using the appropriate lag.

If SD_CND[LS](…) forms are used, only Object instances satisfying the conditional expression, defined by Variable or Parameter Lab2, the logical operator lop and the reference value, are used in computation. Both Lab1 and Lab2 must be contained in the same Object. See SUM(…) for details and examples.

If the Object containing Lab/Lab1 has no instance (see  USE_ZERO_INSTANCE), or no instance satisfies the conditional expression, the Macros return the NAN (not a number) value. NAN values can be checked using isnan(…)(see USE_NAN).



COUNT( "obj_Lab" ), COUNTS( obj, "obj_Lab" )
COUNT_CND( "obj_Lab", "Lab", "lop", value ), COUNT_CNDS( obj, "obj_Lab", "Lab", "lop", value )
COUNT_CNDL( "obj_Lab", "Lab", "lop", value, lag ), COUNT_CNDLS( obj, "obj_Lab", "Lab", "lop", value, lag )
COUNT_ALL( "obj_Lab" ), COUNT_ALLS( obj, "obj_Lab" )
COUNT_ALL_CND( "obj_Lab", "Lab", "lop", value ), COUNT_ALL_CNDS( obj, "obj_Lab", "Lab", "lop", value )
COUNT_ALL_CNDL( "obj_Lab", "Lab", "lop", value, lag ), COUNT_ALL_CNDLS( obj, "obj_Lab", "Lab", "lop", value, lag )
These Macros explore the model counting for the number of instances of the Object named obj_Lab. That is, they count only within the descendants of obj, if specified in COUNTS(…), or in THIS if obj not specified in COUNT(…), and their descendants. Therefore, the search is not exhaustive over the entire model, unless it is started from the ROOT of the model.

The COUNT[S](…) versions only explore a single group of Object instances, that is, a set of Object instances named obj_Lab under a single parent Object. If there is more than a group, it considers only the first one. The COUNT_ALL[S](…) versions consider Object instances in all groups of Objects descending from the selected Object type (obj or THIS) including all of its instances.

If COUNT_CND[LS](…) and COUNT_ALL_CND[LS](…) forms are used, only Object instances satisfying the conditional expression, defined by Variable or Parameter Lab using the appropriate lag, the logical operator lop and the reference value, are considered for counting. Lab must be contained in Object obj_Lab. See SUM(…) for details and examples.

The Macros return the number of instances of the Object or zero if no Object or instance is found (see  USE_ZERO_INSTANCE), or no instance satisfies the conditional expression.



STAT( "Lab" ), STATS( obj, "Lab" )
STATL( "Lab", lag ), STATLS( obj, "Lab", lag )
STAT_CND( "Lab1", "Lab2", "lop", value ), STAT_CNDS( obj, "Lab1", "Lab2", "lop", value )
STAT_CNDL( "Lab1", "Lab2", "lop", value, lag ), STAT_CNDLS( obj, "Lab1", "Lab2", "lop", value, lag )
These Macros works as the Macro SUM(…), but, instead of the sum, they compute a set of descriptive statistics of the Variable or Parameter named Lab or Lab1 among the descendants of the current Object (THIS) or obj, using the appropriate lag.

If STAT_CND[LS](…) forms are used, only Object instances satisfying the conditional expression, defined by Variable or Parameter Lab2, the logical operator lop and the reference value, are used in computation. See SUM(…) for details and examples.

These Macros return the number of instances found for the Variable or Parameter named Lab and also store a set of values in the temporary storage v[…]. Namely, it places in v[…] the following values:

·         v[0] : number of instances/copies of Lab/Lab1 (same as the Macro return value)

·         v[1] : average of Lab/Lab1

·         v[2] : variance of Lab/Lab1

·         v[3] : maximum value of Lab/Lab1

·         v[4] : minimum value of Lab/Lab1

·         v[5] : median of Lab/Lab1

·         v[6] : standard deviation of Lab/Lab1

If the Object containing Lab/Lab1 has no instance (see  USE_ZERO_INSTANCE), or no instance satisfies the conditional expression, the Macros return the NAN (not a number) value for all statistics, except the number of instances (which is set to zero). NAN values can be checked using isnan(…)(see USE_NAN).



INTERACT( "Comment", value ), INTERACTS( obj, "Comment ", value )
These Macros interrupt the simulation run and open the LSD Debugger window, presenting the message Comment. The Object shown in the debugger is the same Object instance containing the Variable under computation (which has the INTERACT(…) call) or the specified Object obj.

Contrary to the debugger usual behavior, additionally to the name of the Variable just computed, the interface presents the value chosen (the default to be used) in a numeric data entry field. User can input a new value as any C++ numeric constant or variable, typically one of the v[…] temporary elements. The value typed in by the user will be returned by the Macro as its result.

This Macro inserted in an Equation allows the run to be modified according to the choice of an interacting user.

Example
For example, an Equation may contain the following code:

v[1] = V( "MinMarketShare" ); //minimum m.s. below which a firm should be removed
CYCLE_SAFE( cur, "Firm" )
{ //Cycle through each firm, allowing for the removal of the pointer
  v[0] = VS( cur, "MarketShare" ); //read the market share
  if ( v[0] < v[1] )
    v[5] = INTERACTS( cur, "Remove this firm? (1=yes; 2=no)", v[4] );

  if ( v[5] == 1 )
  { //remove if the answer was 1
    DELETE( cur );
    v[3]++;
  }
}

The cycle removes all Firm’s having market shares smaller than a given value and that the user permitted to.

Warning
Beware that such a Macro used in a model requires the user to interact with it. See LOG(…) and SLEEP(…) for an alternative to show information to the user without fully stopping the simulation or requiring effective user interaction.



SEARCH( "obj_Lab" ), SEARCHS( obj, "obj_Lab" )
These Macros explore the model searching for the first instance of the Object named obj_Lab, starting at the Object instance where is the Variable is under computation, or at the Object instance pointed by obj. If one instance is found, the Macros return a pointer to it, otherwise NULL is returned,

When an specific obj instance is not provided, the search starts from THIS (see available Objects), the current instance containing the Variable under computation. In case the model contains many instances of an Object named obj_lab, the Macro returns a pointer to the instance "closer" to the Object where the search started from. By closer, it is meant the first instance found by searching the model using the following strategy:

1)    Search in the current or obj Object descendants’ instances

2)    Search in the current or obj descendants’ descendants’ instances recursively, to the lowest level in the model structure tree

3)    Search in the current or obj Object parent’s instances

4)    Search in the current or obj Object parent’s parent’s instances recursively, up to the ROOT level

In each Object instance explored, the same strategy is applied recursively, so that every instance in the model structure is visited, if necessary, except if NO_SEARCH Macro is used. This strategy is similar to the one employed when searching for Variables, like in V(…).

If an specific instance of a multi-instanced Object is required, the advanced Macro family TSEARCH(…) may be a complementary option to quickly find it, in particular when multiple instances must be found.

Example
The result from SEARCH(…) is usually attributed to a temporary pointer for later usage, like in the following piece of code where an Object looks for a grandparent Object named Market, saves its address to temporary pointer cur and uses it to retrieve the grandparent’s Variable TotalSales:

…
cur = SEARCH( GRANDPARENT, "Market" );
v[1] = VS( cur, "TotalSales" );
…



SEARCH_CND( "Lab", value ), SEARCH_CNDS( obj, "Lab", value )
SEARCH_CNDL( "Lab", value, lag ), SEARCH_CNDLS( obj, "Lab", value, lag )
These Macros are used to find an Object instance in the model which contains the Variable or Parameter named Lab with the specific numeric value value. The search can optionally consider obj as the starting point for the search (instead of the current Object under calculation, THIS) and the specific lag lag if different from current time step. When the first Object instance that satisfies the condition is found, a pointer to it is returned to the calling Equation, and can be used with other LSD Macros like VS(…). If no Object instance containing the searched value is found, SEARCH_CND(…) family returns a NULL pointer.

The search strategy for Lab extends across the entire model structure as necessary, like in V(…), except if NO_SEARCH is used.

If there are many Object instances to search for Variable or Parameter Lab, the Macros may demand significant time for finding the desired value. In such cases, particularly when searches are performed repeatedly, the advanced Macro family TSEARCH_CND(…)

Example
Suppose you want to extract randomly a Firm from a group of 100 instances, and that these contain a Parameter IdFirm set from 1 to 100, you can use the following code to set a pointer to it in cur:

v[0] = uniform_int( 1, 100 );
cur = SEARCH_CND( "IdFirm", v[0] );



SEARCH_INST( obj ), SEARCH_INSTS( obj_from, obj )
These Macros find the Object instance position/number of the specific Object pointed by obj. The search can optionally consider obj_from as the starting point for the search (instead of the current Object under calculation, THIS). The search process for obj is similar to SEARCH(…).

If a matching Object instance is found, the Macro returns the instance number representing the position of the specific instance pointed by obj in the brotherhood chain, starting from 1. If the Object is not found, SEARCH_INST(…) returns zero.

If obj is set to NULL, the Macros will return the instance number of the Object containing the Variable currently under computation.

Example
If you want to check if the IdFirm Parameter is set to the number (position) of each Firm Object instance:

CYCLE( cur, "Firm" )
{
  v[1] = VS( cur, "IdFirm" );
  v[2] = SEARCH_INST( cur );
  if ( v[1] != v[2] )
    LOG( "\n IdFirm=%g different from instance number=%g", v[1], v[2] );
}



RNDDRAW( "obj_Lab", "Lab" ), RNDDRAWS( obj, "obj_Lab", "Lab" )
RNDDRAWL( "obj_Lab", "Lab", lag ), RNDDRAWLS( obj, "obj_Lab", "Lab", lag )
These Macros search for a group of Object instances named obj_Lab and returns a pointer to one of them, randomly chosen with probability proportional to the values of the Variable labeled Lab in the same Object. The Macros can optionally consider obj as the starting point for the search (instead of the current Object under calculation, THIS) and the specific lag lag if different from current time step. The search process for obj_Lab is similar to SEARCH(…).

Example
Consider the Equation for DrawAFirm which has to return the value of Parameter IdFirm in one randomly selected Firm among the instances of Firm descending from the Object where DrawAFirm is contained in. The Equation could be:  

EQUATION( "DrawAFirm" )
/***************************
Return the Id of a Firm chosen randomly with probability
equal to the square of the market shares.
***************************/

CYCLE( cur, "Firm" )
{
  v[0] = V( "ms" );
  WRITES( cur, "Prob", v[0] * v[0] );
}

cur = RNDDRAW( "Firm", "Prob" );

RESULT( VS( cur, "IdFirm" ) )

The code assigns to the Parameter Prob in each Firm the square of their market shares (ms), so drawing probability is defined by Firm’s squared ms, and then draws randomly one of them, returning its address to be stored in cur.



RNDDRAW_FAIR( "obj_Lab" ), RNDDRAW_FAIRS( obj, "obj_Lab" )
These Macros are identical to RNDDRAW(…) but use the same probability for all Object instances, that is, all instances named Lab have the same probability of being drawn. The Macro can optionally consider obj as the starting point for the search (instead of the current Object under calculation, THIS). The search process for obj_Lab is similar to SEARCH(…).

Example
The following Equation returns the Parameter IdFirm from a randomly chosen Object Firm.

EQUATION( "DrawAFirm" )
/***************************
Return the Id of a Firm chosen randomly with identical
probabilities for each object
***************************/

cur = RNDDRAW_FAIR( "Firm" );

RESULT( VS( cur, "IdFirm" ) )



RNDDRAW_TOT( "obj_Lab", "Lab", total ), RNDDRAW_TOTS( obj, "obj_Lab", "Lab", total )
RNDDRAW_TOTL( "obj_Lab", "Lab", lag, total ), RNDDRAW_TOTLS( obj, "obj_Lab", "Lab", lag, total )
These Macros are identical, but faster, to RNDDRAW(…) as the modeler explicitly provide the (correct) sum of all the weights, that is, the sum of the Variable named Lab among all Object instances with label obj_Lab. Unpredictable drawing behavior is expected if total is not equal to the sum of all pertinent weights. The Macros can optionally consider obj as the starting point for the search (instead of the current Object under calculation, THIS) and the specific lag lag if different from current time step. The search process for obj_Lab is similar to SEARCH(…).



WRITE( "Lab", new_value ), WRITES( obj, "Lab", new_value )
WRITEL( "Lab", new_value, time ), WRITELS( obj, "Lab", new_value, time )
WRITELL( "Lab", new_value, time, lag ), WRITELLS( obj, "Lab", new_value, time, lag )
These Macros write the new_value to one instance of a Variable or Parameter named Lab. In the case of a Variable, it is possible to make appear as if the time of latest computation for the overwritten Variable is time. That is, after the Macro is executed, the Variable Lab will appear to LSD as if its last computation had been executed at time time, and the result of the Equation were new_value. If the label Lab corresponds to a Parameter, the field time is saved (to be read by LAST_CALC(…)) but not used by LSD. The Macros return the new element value after the update.

Additionally, the “LL” versions of the Macro allow also modifying lagged values of a Variable or the initial value of a Parameter (only lag = 1 is valid in this case). This means that one can “rewrite the past”, changing the value LSD would retrieve if a lagged value of the Variable is subsequently requested (in a VL(…) command, for instance) or in the corresponding saved values, if the Variable or Parameter is marked to be saved. When lag is used, setting time equal to or ahead of current time (T) may produce unexpected results (setting time to T-1 is usually safe). Rewriting the past can be a very dangerous practice and is only recommended to correctly initialize (lagged) values of Variables and Parameters in Object instances created in the middle of the simulation, see details below.

If WRITE(…) Macros are used to update the current (t) value of a Variable configured with a non-default updating scheme, any initial update delay is cancelled and a new update period is immediately started, according to the Variable updating options used.

The Variable or Parameter must be contained in the Object instance obj or in the current instance (THIS) of the Variable being computed, otherwise the Macros produce an error and stop the simulation. Differently from most other Macros, WRITE(…)never performs searching across the model structure, unlike VL(…), irrespective of USE_SEARCH being called beforehand.

Warning
These Macros can be very useful to implement complex situations, but they should be used with extreme care because they potentially disrupt the automatic updating system of execution of the Equations. The best practice is to use WRITE(…) only to change Parameter values during the simulation. Another adequate use of WRITE(…), when used with Variables, concerns the initialization of newly added Object instances.

See RECALC(…) for another, less invasive Macro to force the reevaluation of already calculated (or just created) Variables.



INCR( "Lab", value ), INCRS( obj, "Lab", value )
These Macros add value to the numeric content of one instance of a Variable or Parameter named Lab. The Macros return the new value after the increment.

The Variable or Parameter must be contained in the Object instance obj or in the current instance (THIS) of the Variable being computed, otherwise the Macros produce an error and stop the simulation. Differently from most other Macros, INCR(…)never performs searching across the model structure, unlike VL(…), irrespective of USE_SEARCH being called beforehand.

INCR(…) Macro family should be used preferably with Parameters, for the same reasons explained in the WRITE(…) family.

Warning
If Lab is a Variable, these Macros will not trigger the computation of the Equation for Lab before the addition is performed. That is, if Lab was not yet computed in the current period (t), the last available value for Lab is used as the base for the addition and the result will be saved as the new value for Lab in t. No further recomputation will be automatically performed in t. If the previous computation of Lab is required, the user must do it explicitly before calling INCR(…).



MULT( "Lab", value ), MULTS( obj, "Lab", value )
These Macros multiply by value the numeric content of one instance of the Variable or Parameter named Lab. The Macros return the new value after the multiplication.

The Variable or Parameter must be contained in the Object instance obj or in the current instance (THIS) of the Variable being computed, otherwise the Macros produce an error and stop the simulation. Differently from most other Macros, MULT(…)never performs searching across the model structure, unlike VL(…), irrespective of USE_SEARCH being called beforehand.

MULT(…) Macro family should be used preferably with Parameters, for the same reasons explained in the WRITE(…) family.

Warning
If Lab is a Variable, these Macros will not trigger the computation of the Equation for Lab before the multiplication is performed. That is, if Lab was not yet computed in the current period (t), the last available value for Lab is used as the base for the multiplication and the result will be saved as the new value for Lab in t. No further recomputation will be automatically performed in t. If the previous computation of Lab is required, the user must do it explicitly before calling MULT(…).



CYCLE( cur, "obj_Lab" ) { }, CYCLES( obj_from, cur, "obj_Lab" ) { }
CYCLE_SAFE( cur, "obj_Lab" ) { }, CYCLE_SAFES( obj_from, cur, "obj_Lab" ) { }
These Macros scan all the Object instances labelled obj_Lab descending from current Object THIS or from the specified Object obj_from, assigning cyclically the pointer cur specified to the address of each target Object. Nested CYCLE(…)’s can be used, provided that the pointers (cur, cur1, cur2,…) differ for the different levels. The search process for obj_Lab is similar to SEARCH(…).

Example
For example, consider a model including Object Market containing Object instances named Firm. An Equation in an Object containing a set of Markets, for example, representing countries, can use the following nested code:

v[0] = 0; // set to v[0] to be initially 0
CYCLE( cur, "Market" )
{ // for each market
  CYCLES( cur, cur1, "Firm" )
  { // for each firm, contained in cur
    v[1] = VS( cur1, "Profit" );
    v[0] = v[0] + v[1];
  }
}
// from here v[0] contains the cumulated profits

The cur pointer cycles through all Object instances named Market (supposing there is more than one). In turn, for each Market the pointer cur1 cycles through each Object labelled Firm that is contained in this specific Market. For each Firm, v[1] receives the value of the Variable named Profit related to that particular Firm, which is subsequently accumulated in v[0].

See also the example for RNDDRAW(…).

Warning
Never use DELETE(…) inside a CYCLE(…) to remove one of the Object instances being cycled at any nesting level. In fact, deleting any Object pointed to by cur prevents the CYCLE(…)from identifying the next one in the brotherhood chain. Instead, use the alternative Macros CYCLE_SAFE(…). In this case, it is possible to delete the Object pointed by the cycling pointer cur. However, it is not possible to nest different CYCLE_SAFE(…) levels.

The example below is a mistake because cur1 is deleted within a CYCLE(…) command:

ERROR - ERROR - ERROR
CYCLE( cur, "Market" )
{ // for each market
  CYCLES( cur, cur1, "Firm" )
  { // for each firm, contained in cur
    v[1] = VS( cur1, "Profit" );
    if ( v[1] < 0 )
      DELETE( cur1 ); // ERROR: this undermines the inner CYCLE
  }
}
ERROR - ERROR - ERROR

Instead, the correct code should be:

CYCLE( cur, "Market" )
{ // for each market
  CYCLE_SAFES( cur, cur1, "Firm" )
  { // for each firm, contained in cur
    v[1] = VS( cur1, "Profit" );
    if ( v[1] < 0 )
      DELETE( cur1 );
  }
}



ADDOBJ( "obj_Lab" ), ADDOBJS( obj, "obj_Lab" )
ADDOBJL( "obj_Lab", t_upd ), ADDOBJLS( obj, "obj_Lab", t_upd )
ADDNOBJ( "obj_Lab", num ), ADDNOBJS( obj, "obj_Lab", num )
ADDNOBJL( "obj_Lab", num, t_upd ), ADDNOBJLS( obj, "obj_Lab", num, t_upd )
These Macros add one (or more) new instance(s) to the Object named obj_Lab descending from obj, or to THIS if obj is not specified. ADDOBJ(…) returns a pointer to the (first) newly created Object. More than one Object instance can be created at once if their number num is informed in the ADDNOBJ(…) formats. In this case, a pointer is returned only to the first created Object instance. The additional instances can be easily reached by using the NEXTS(…) Macro over the first instance (i.e., by using NEXTS( CUR ), NEXTS( NEXTS( cur ) ) etc.).

The ADDOBJ(…) versions initializes the newly created Object instance(s), that is, sets the values for its Parameters and lagged Variables, according to the values defined in the model configuration (and modified by the simulation until the current time step).

The Macros without the “L” suffix set the new Object instance(s) as modified by the simulation during the current time step, that is, its (their) Variables are going to be updated only in the next time step. Instead, the Macros with the “L” suffix allow for the definition of the time of the last update t_upd for all of the Object’s Variables, if t_upd is positive or zero. If t_upd is negative, the effective time of last update from each Variable of the original copied Object instance (or current time, if the Variable was never updated).

The Object obj_Lab must be contained in (a son of) the Object instance obj or in the current instance (THIS) of the Variable being computed, otherwise the Macros produce an error and stop the simulation. Differently from most other Macros, ADDOBJ(…)never performs searching across the model structure, unlike SEARCH(…), irrespective of USE_SEARCH being called beforehand.

In case the original Object named obj_Lab was defined as having descendants (at any level), the Macros builds the whole structure of descendants also under the new Object instance(s). As the Macros return the address of the newly created Object(s), the modeler can easily add in the creation Equation some customized initialization code. See Macro family WRITES(…) for details on changing the initial values of Variables (including lagged values) and Parameters.

If the original (and so, the new) Object is set to save the values of all or some of its Variables, these will be available for the post-simulation Analysis of Results. The data concerning the periods before an Object introduction in the simulation is filled with missing values (“NA”).

Example
For example, you can have an Equation for a Variable in the Object Market that contains the following:

…
cur = ADDOBJ( "Firm" );
WRITES( cur, "DateBirth", T );
…

The lines above create a new Object Firm, which has the same initial data as the first Object in the model file at the current time step. Then, the code modifies the new Object Parameter named DateBirth, storing there the current time step.

Warning
The newly created Object can only be added to a parent Object which already has, at least, one descending Object (son) labelled as obj_Lab.



ADDOBJ_EX( "obj_Lab", obj_ex ), ADDOBJ_EXS( obj, "obj_Lab", obj_ex )
ADDOBJ_EXL( "obj_Lab", obj_ex, t_upd ), ADDOBJ_EXLS( obj, "obj_Lab", obj_ex, t_upd )
ADDNOBJ_EX( "obj_Lab", num, obj_ex ), ADDNOBJ_EXS( obj, "obj_Lab", num, obj_ex )
ADDNOBJ_EXL( "obj_Lab", num, obj_ex, t_upd ), ADDNOBJ_EXLS( obj, "obj_Lab", num, obj_ex, t_upd )
The ADDOBJ_EX(…) are special versions of ADDOBJ(…) Macro which use the specific modeler supplied example Object pointer obj_ex to initialize the newly created Object instance(s), generating one or num new instances, each one a clone of obj_ex. A pointer is returned only to the first created Object instance. The additional instances can be easily reached by using the NEXTS(…) Macro over the first Object (i.e., by using NEXTS( cur ), NEXTS( NEXTS( cur ) ), etc.).

The Object obj_Lab must be contained in (a son of) the Object instance obj or in the current instance (THIS) of the Variable being computed, otherwise the Macros produce an error and stop the simulation. Differently from most other Macros, ADDOBJ_EX(…)never performs searching across the model structure, unlike SEARCH(…), irrespective of USE_SEARCH being called beforehand.

Please check more details about the new Object(s) creation, including initialization, in ADDOBJ(…).

Example
For example, suppose that in Market Object one wishes to add a new Firm that is a clone of the most productive existing Firm. The following code identifies the Object Firm with the maximum productivity and adds a new Firm with the same initialization data of this Firm:

…
cur = SEARCH( "Firm" );
v[0] = MAXS( cur, "A" );
cur = SEARCH_CND( "A", v[0] );
cur = ADDOBJ_EX( "Firm", cur );
…

The first line retrieves the address of the instance of first Firm in the brotherhood chain. The second line finds the value of the highest productivity among the existing Firm’s. Then, the Macro SEARCH_CND(…) returns a pointer to the first Object instance that has the maximum productivity. Finally, this instance is used as an example for the creation of the new Firm. Note that the same temporary pointer cur is used (1) to store the address to the first Firm, (2) the pointer to most productive Firm and then, (3) the new Object instance. This may seem strange to who is not accustomed to C++, but it is perfectly safe. In fact, the content of the temporary pointer cur has already been used when the Macro returns the newly created Object.



DELETE( obj )
This Macro deletes the Object instance pointed by obj from the model, including all descending Object instances, if any, and freeing the allocated memory. Deleting a NULL pointer does nothing.

The Object instance obj must be a valid pointer to an existing exact instance, otherwise the Macro produces an error and stop the simulation. Differently from most other Macros, DELETE(…)never performs searching across the model structure, unlike SEARCH(…), irrespective of USE_SEARCH being called beforehand.

While its use is very simple, DELETE(…) should be used with care to avoid the elimination of data structures still used in other parts of the model, including any descending Object. The data produced by the Object until deletion (and marked for saving) will be still available for the Analysis of Results. Missing values (“NA”) fill data for the periods after the deletion.

By default, LSD ensures that at least one instance of any Object is preserved and cannot be deleted. In special situations, this behavior can be disabled using USE_ZERO_INSTANCE Macro. The modeler must carefully handle the results coming from other Macros in the case of zero-instanced Objects, usually NAN or NULL values, to avoid potentially serious errors.

Warnings
The modeler must be careful when deleting Object instances which may contain Variables or Parameters which may be in use in a chained computation (i.e., SUM(…), STAT(…)) at the same time, that is, “suicidal” Equations. If an instance with a Variable or Parameter currently in use is deleted (or commits “suicide”), the result of the corresponding computation in which the DELETE(…) Macro was invoked is still considered. This behavior can potentially lead to inconsistency in computations which assume the “suicidal” Object instance still exists, like in a SEARCH_CND[…], and should be used with extreme care.

Actual Object instance deletion may be postponed if the Variable Equation in which the Macro DELETE(…) is invoked as part of the computation of another Variable in the same Object instance. In this case, only when the computation of the most upstream Variable under computation in this Object is completed then the Object instance is effectively deleted. All upstream Equations, to the one triggering the deletion, must account for this delay, during which the to be deleted instance is still present and affecting calculations. The DELETING(…) Macro can be used to detect if an Object instance is currently waiting to be deleted.

It is not possible to delete Object instances inside a regular CYCLE(…) loop Macro when the Object controlling it is of the same type (label) of the instance being deleted. This may produce hard crashes. Whenever an Object instance needs to be deleted inside such type of loop, it is required the use of the CYCLE_SAFE(…) Macros. See above for details and an example.



SORT( "obj_Lab", "Lab1", "Dir" ), SORTS( obj, "obj_Lab", "Lab1", "Dir" )
SORTL( "obj_Lab", "Lab1", "Dir", lag ), SORTLS( obj, "obj_Lab", "Lab1", "Dir", lag )
SORT2( "obj_Lab", "Lab1", "Lab2", "Dir" ), SORT2S( obj, "obj_Lab", "Lab1", "Lab2", "Dir" )
SORT2L( "obj_Lab", "Lab1", "Lab2", "Dir", lag ), SORT2LS( obj, "obj_Lab", "Lab1", "Lab2", "Dir", lag )
These Macros sort (with the quick sort method) a chain of Object instances labeled obj_Lab according to the values of Variable named Lab1 contained in each Object being sorted. The option "Dir" must be either UP or DOWN. SORT(…) produces an error in case Variable Lab1, though defined in the model, is not contained in the Object named obj_Lab. In case there are more than one chain of Object instances with label obj_Lab, the Macro sorts only the first set found by exploring the model with the usual full search strategy (see SEARCH(…)). If obj is specified, the search for the group of instances to be sorted starts from obj. The Macros can optionally consider the specific lag for Variable Lab1 (and Lab2) if different from current time step.

The SORT2(…) Macros are available for ranking on two dimensions, firstly on the Variable named Lab1 and then on Variable Lab2.

Example
The following line, used in a Variable’s Equation contained in an Object from which descend many Object instances labelled Firm, sorts these descendants according to decreasing values of their Variable Q.

SORT( "Firm", "Q", DOWN );

In the second example, the Object instances labelled as Market are sorted according to their increasing values of their Variable or Parameter named X. If two or more Markets have identical values on X, then the Variable or Parameter Y is used to sort them.

SORT2( "Market", "X", "Y", UP );

Warning
The Object instances being sorted must not delete themselves in the Equations associated with the involved Variables.



HOOK( num ), HOOKS( obj, num )
SHOOK, SHOOKS( obj )
These Macros return a pointer to another Object previously “hooked” to the current Object instance THIS (the one containing the Variable/Function being computed) or to the one pointed by obj. The non-negative integer num in HOOK(…) indicates the desired allocated dynamic hook to use. The always-available, un-numbered, single static hook can be accessed by SHOOK/SHOOKS(…) forms.

Every Object is automatically endowed with a single (un-numbered) static hook, accessed by the HOOK Macro. Additionally, more hooks can be dynamically allocated. Dynamic hooks are numbered from zero to the number of allocated hooks minus one. Dynamic hook pointers must be allocated before usage using ADDHOOK(…). All hooks must assigned before usage using WRITE_HOOK(…).

Hook pointers allow for the construction of efficient data structures based on the model structure. See more details here.

Example
Suppose you have a model where the Object Consumer stores the Parameter IdUsed the ID of the Object product currently used. Moreover, suppose that the products can break at each time step with a probability contained in the Object Firm. The Equation of the Consumer to check if the product breaks may be the following:

EQUATION( "IdUsed" )
/*
Id of the product used by the consumer.
Look whether the product breaks down or not. In the first case choose a new product.
*/

v[0] = VL( "IdUsed", 1 ); // product used by the consumer
cur = SEARCH_CND( "IdFirm", v[0] ); // find the object with IdFirm equal to my IdUsed
v[2] = VS( cur, "BD" ); // read the probability of breaking the product
if ( RND < v[2] )
{ // product broken
  cur1 = RNDDRAW_FAIRS( PARENT, "Firm" ); // choose a new firm from the father of the former firm
  v[1] = VS( cur1, "IdFirm" );
}
else
  v[1] = v[0]; // product not broken, used the same product as before

RESULT( v[1] )

This Equation can slow down the simulation in case it contains thousands of Firms and of Consumers, since the system has to scan many Firm’s to find the one required by the Consumer. The same code can be written as follows:

EQUATION( "IdUsed" )
/*
Id of the product used by the consumer.
Look whether the product breaks down or not. In the first case choose a new product.
Dynamic HOOK( 0 ) is the consumers' pointer containing the firm used by the consumer.
*/

v[2] = VS( HOOK( 0 ), "BD" ); // read the probability of breaking the product using hook
if ( RND < v[2] )
{ // product broken
  cur1 = RNDDRAW_FAIRS( PARENT, "Firm" ); // choose a new firm from the father of the former firm
  WRITE_HOOK( 0, cur1 );
  v[1] = VS( cur1, "IdFirm" );
}
else
  v[1] = v[0]; // product not broken, used the same product as before

RESULT( v[1] )

The code is faster because the Equation does not need to scan all Firm’s to find the desired one.

Beware that the modeler must ensure that any hook is always correctly assigned. In the example, you need to consider that at the very first time step the hook number 0 (HOOK( 0 )) is not created or assigned. At the start of a simulation run all the hooks must be created and initialized (or they will be set to NULL). Therefore, you may fix the code above as follows:

EQUATION( "IdUsed" )
/*
Id of the product used by the consumer.
Look whether the product breaks down or not. In the first case choose a new product.
Dynamic HOOK( 0 ) is the consumers' free pointer containing the firm used by the consumer.
*/
if ( COUNT_HOOK == 0 )
{  // executed the very first time step, when hook was not created
   v[0] = VL( "IdUsed", 1 ); // product used by the consumer
   cur = SEARCH_CND( "IdFirm", v[0] ); // find the object with IdFirm equal to my IdUsed
   v[2] = VS( cur, "BD" ); // read the probability of breaking the product
   ADDHOOK( 1 ); // create the hook pointer
   WRITE_HOOK( 0, cur ); // initialize the hook pointer
}
else
   v[2] = VS( HOOK( 0 ), "BD" ); // read the probability of breaking the product

if ( RND < v[2] )
{  // product broken
   cur1 = RNDDRAW_FAIRS( PARENT, "Firm" ); // choose a new firm from the father of the former firm
   WRITE_HOOK( 0, cur1 ); // update the hook
   v[1] = VS( cur1, "IdFirm" );
}
else
   v[1] = v[0]; // product not broken, used the same product as before

RESULT( v[1] )

Of course, as only a single hook is needed for each Consumer object, in this example one could have used the static (non-numbered) hook, which is pre-allocated. To do so, simply replace HOOK( 0 ) with SHOOK, WRITE_HOOK( 0, cur ) with WRITE_SHOOK( cur ), and eliminate ADDHOOK( 1 ).



WRITE_HOOK( num, obj_dest ), WRITE_HOOKS( obj, num, obj_dest )
WRITE_SHOOK( obj_dest ), WRITE_SHOOKS( obj, obj_dest )
The WRITE_HOOK(…) Macros set the dynamic hook numbered num to the Object pointed by obj_dest. Hook assignment can be performed on the current Object instance THIS (the one containing the Variable/Function being computed) or to the one pointed by obj. The variants WRITE_SHOOK(…) set the always-available, un-numbered, single static hook.

Dynamic hooks are numbered from zero to the number of allocated hooks minus one. Dynamic hook pointers must be allocated before assignment using ADDHOOK(…). The static hook does not have to be allocated before usage.

After hook assignment, Macros HOOK(…)/SHOOK can be used to read/access the dynamic hook pointer number num or the static pointer. The modeler is responsible to ensure valid hook pointers before using them for addressing Object instances in other Macros.

Check the example above for more details.



ADDHOOK( num ), ADDHOOKS( obj, num )
The modeler may allocate as many dynamic hooks as necessary for any Object by informing the desired number num in ADDHOOK(…). Dynamic hooks are numbered stating from zero so num must be an integer equal or larger than zero. The allocated dynamic hooks will be numbered from zero to num – 1. Hooks may be added to the current Object instance THIS (the one containing the Variable/Function being computed) or to the one pointed by obj.

The always available single, un-numbered static hook does not need to and cannot be allocated using ADDHOOK(…).

Newly allocated hooks are initialized to NULL (no Object), including the static hook. Dynamic/static hooks must be assigned using WRITE_HOOK(…)/WRITE_SHOOK(…) before usage.

Increasing of the number of allocated dynamic hooks is possible at any time, by using the ADDHOOK(…) Macros again, with a larger num. The existing hooks are not affected by the increase. The number of dynamic hooks may be also reduced using a smaller num than the last used value. The excess higher-numbered hooks will be destroyed. num equal to zero deallocates all dynamic hooks, preserving just the un-numbered static hook.

Check the example above for more details.



COUNT_HOOK, COUNT_HOOKS( obj )
Macro COUNT_HOOK returns the number of allocated dynamic hooks (initialized or not). Hooks may be counted for the current Object instance THIS (the one containing the Variable/Function being computed) or to the one pointed by obj. Counting for an Object without hooks added by ADDHOOK(…) return zero.

Please note that hooks are numbered from zero to the number of allocated hooks minus one. The single, un-numbered static hook is not considered in the number returned by these Macros.

Check the example above for more details.



DOWN_LAT, DOWN_LATS( obj )
LEFT_LAT, LEFT_LATS( obj )
RIGHT_LAT, RIGHT_LATS( obj )
UP_LAT, UP_LATS( obj )
These Macros simplify moving in an Object structure created in lattice format (n instances of a top Object type, each one composed by a single descending Object which also has a single descending and so on, up to m levels). If an Object instance obj is specified, or the current Object instance THIS otherwise, a pointer to the corresponding down, left, right, or up neighbor instance in the lattice is returned. If obj or THIS are on the lattice border, and the requested neighbor does not exist, a NULL pointer is returned.

The Object instance obj must be a valid pointer to an existing exact instance, otherwise the Macro produces an error and stops the simulation. Differently from most other Macros, these never performs searching across the model structure, unlike SEARCH(…), irrespective of USE_SEARCH being called beforehand.

Despite the Macros are designed to be used in a lattice organized Object structure, they can be used at any time to navigate across the neighbor structure (parents, descendants and siblings). LSD does not enforce that the model structure actually has a lattice format.

There is no direct association of these Macros with the lattice graphical window commands (see INIT_LAT(…)). However, they can be used to simplify the association of a graphical lattice to a lattice-shaped Object structure.



INIT_LAT( )
INIT_LAT( color )
INIT_LAT( color, rows, cols )
INIT_LAT( color, rows, cols, width, height )
These Macros create a bi-dimensional lattice graphical window available for simulation run time updating. The lattice is formed by rows lines and cols columns, set to 100 × 100 by default. The user can also set the physical window size as width × height pixels, set to 400 × 400 by default, and the background color, set to white if not explicitly defined (see the available colors in WRITE_LAT(…)).

Just one lattice window may be active in each simulation run. Creating a new lattice will destroy the existing one.



DELETE_LAT
Close the lattice graphical window, if one exists. All data associated with the lattice is lost.

See INIT_LAT(…) on the details about the lattice window.



V_LAT( row, col )
These Macros read the current color of one cell in the lattice. The color is read from the lattice cell in line row and column col (see the color values in WRITE_LAT(…)).

The lattice origin (1, 1) is at the upper-left corner of the graphical window. Only positive values are accepted for row and col. The maximum values for row and col are the lattice number of rows and columns, respectively (see INIT_LAT(…)). Decimal parts of the two values are ignored.

Negative color values represent the (positive) 24-bit RGB equivalent (0xrrbbgg, being rr, bb and gg from 00 to 0xff in hexadecimal format).

Before V_LAT(…) is used, INIT_LAT(…) has to be explicitly called once, if not the command is ignored.



WRITE_LAT( row, col )
WRITE_LAT( row, col, color )
These Macros write to one cell in the lattice, modifying its color. The change is applied to the lattice cell in line row and column col to the color value defined by color. If color is not specified, it is set to black (1).

The lattice origin (1, 1) is at the upper-left corner of the graphical window. Only positive values are accepted for row and col. The maximum values for row and col are the lattice number of rows and columns, respectively (see INIT_LAT(…)). Decimal parts of the two values are ignored.

Some possible values for color (many more are possible) are:

0.            Black

1.            Red

2.            Green (dark)

3.            Yellow (dark)

4.            Pink

5.            Blue (dark)

6.            Blue (normal)

7.            Gray

1000.   White

Negative color values set color to the (positive) 24-bit RGB equivalent (0xrrbbgg, being rr, bb and gg from 00 to 0xff in hexadecimal format).

Before WRITE_LAT(…) is used, INIT_LAT(…) has to be explicitly called once, if not the command is ignored.



SAVE_LAT( "filename" )
This Macro saves a graphical snapshot of the lattice to the disk as an .eps (encapsulated PostScript) file, in the LSD model directory, named as filename.eps. If filename is not provided, the default name is lattice.eps. The filename may contain a path if saving to a particular directory is required. If no directory is included in filename, the file is saved to the current LSD model configuration directory.

Before SAVE_LAT(…) is used, INIT_LAT(…) has to be explicitly called once, if not the command is ignored. Multiple snapshots to different files is possible but requires the user to choose a different filename in each Macro call (see CONFIG/PATH for an example).



LSD Networks
LSD Network tools provide Macros to configure a set of LSD Object instances, typically a brotherhood chain, as a network. Such Object instances become the network nodes and any topology of directed links can be built over them. All Network Macros operate on a special data structure that can be added to any LSD Object. This structure is named Node. Only Object instances with a Node added can be part of a Network. Networks are represented in LSD as directed graphs by default. Undirected networks can be represented by using two directed arcs (links) in opposite directions.

Each LSD Object can have a single Node and so participate in a single Network. Also, only instances which are part of a single brotherhood chain, that is, have the same name (label) and parent Object, can be handled as an entity by network-level Macros. However, multiple Networks can coexist in a model, given that each of them is based on Nodes associated to different chains. Additionally, Object instances which are not brothers can use some of the basic Network Macros that do not operate over an entire Network/brotherhood chain (typically the ones which do not use the chain name “node_Obj”).

To create a Network, the modeler can start from any of the automatic generators available in INIT_NET(…), including a disconnected Network (just Nodes, no link). From any starting configuration (including no Node at all), it is possible to incrementally add, remove and modify Nodes and links over the simulation time, by including the Macros presented below in the Equation code of regular LSD Variables.

LSD Networks can be also loaded from and saved to Pajek-formatted files (Pajek is a popular free network analysis software), at any time before, during or after the simulation, using the Macro families LOAD_NET(…), SAVE_NET(…) and SNAP_NET(…) or the menu options of LSD Browser. It is highly recommended to use Pajek (or other network analysis software) together with LSD Networks to be able to visualize them and generate the required network metrics, for instance.



V_NODEID( ), V_NODEIDS( node_obj )
V_NODENAME( ), V_NODENAMES( node_obj )
These Macros return the ID (as a double precision integer value) or the name (as a null-terminated character array pointer) of the Node indicated by Object pointer node_obj, or the current Node THIS otherwise, respectively. Unnamed Nodes are indicated as a null string. IDs are not required to be unique in the Network. If the selected Object is not a Node, the Macros return zero or NULL, respectively.

Please note that the Node name is not related to the name (label) of the Object hosting the Node.



V_LINK( link )
This Macro returns the weight of the Link pointed by pointer link, as set by WRITE_LINK(…).



STAT_NET( "node_Lab" ), STAT_NETS( obj, "node_Lab" )
STAT_NODE( ), STAT_NODES( node_obj )
The STAT_NET(…) Macros provides statistics of a Network composed by the Object instances labelled node_Lab, descending from the Object instance pointed by obj or the current Object THIS otherwise. The search process for node_Lab is similar to SEARCH(…).

The results are stored in the temporary storage as follows:

·         v[0]: number of Nodes

·         v[1]: number of Links (directed arcs)

·         v[2]: average out-degree

·         v[3]: minimum out-degree

·         v[4]: maximum out-degree

·         v[5]: density (including loops)

The STAT_NODE(…) Macros return the number of outgoing Links from the Node indicated by Object pointer node_obj, or the current Node THIS otherwise.



SEARCH_NODE( "node_Lab", id ), SEARCH_NODES( obj, "node_Lab", id )
SEARCH_LINK( id ), SEARCH_LINKS( node_obj, id )
The SEARCH_NODE(…) Macros search for a Node from a Network composed by the Object instances labelled node_Lab, descending from the Object instance pointed by obj or the current Object THIS otherwise, for the first occurrence of an ID id. The Macros return an Object pointer to the first found Node or NULL if no Node is found. The search process for node_Lab is similar to SEARCH(…).

SEARCH_NODE(…) can be slow for large networks, a customized turbo search approach is recommended in such cases.

The SEARCH_LINK(…) Macros search for outgoing Links from a Node indicated by Object pointer node_obj, or the current Node THIS otherwise, for first occurrence of a Link destined to a Node with ID equal to id. The Macros return a Link pointer to the found Link or NULL if no link exists to the searched destination.



RNDDRAW_NODE( "node_Lab" ), RNDDRAW_NODES( obj, "node_Lab" )
RNDDRAW_LINK( ), RNDDRAW_LINKS( node_obj )
The RNDDRAW_NODE(…) Macros random draw one Node from a Network composed by the Object instances labelled node_Lab, descending from the Object instance pointed by obj or the current Object THIS otherwise. All Nodes have the same drawing probability by default but the individual Node probability may be reset using DRAWPROB_NODE(…). The Macros return an Object pointer to the drawn Node or NULL if no Node exists. The search process for node_Lab is similar to SEARCH(…).

The RNDDRAW_LINK(…) Macros random draw one outgoing Link from a Node indicated by Object pointer node_obj, or the current Node THIS otherwise. All Links have the same drawing probability by default but the individual outgoing Link probability may be reset using DRAWPROB_LINK(…). The Macros return a Link pointer to the drawn Link or NULL if no link exists.



DRAWPROB_NODE( prob ), DRAWPROB_NODES( node_obj, prob )
DRAWPROB_LINK( link, prob )
The DRAWPROB_NODE(…) Macros reset to prob the random draw probability of the Node indicated by Object pointer node_obj, or the current Node THIS otherwise. When a random draw is performed, the probability values attributed to each Node are always rescaled to add up to 1.

The DRAWPROB_LINK(…) Macros reset to prob the random draw probability of the Link indicated by the Link pointer link. When a random draw is performed, the probability values attributed to each Link are always rescaled to add up to 1.



LINKTO( link )
LINKFROM( link )
The Macros return an Object pointer to the destination or to the origin of the Link indicated by the Link pointer link, respectively.



WRITE_NODEID( id ), WRITE_NODEIDS( node_obj, id )
WRITE_NODENAME( "name" ), WRITE_NODENAMES( node_obj, "name" )
These Macros reset the ID (to an integer value) or the name (to a null-terminated character array) of the Node indicated by Object pointer node_obj, or the current Node THIS otherwise, to id or name, respectively. name can be set to "" (null string). id must be provided but it is not required to be unique in the Network. If the selected Object is not a Node, the Macros do nothing.

Please note that the Node name is not related to the name (label) of the Object hosting the Node.



WRITE_LINK( link, weigth )
This Macro reset the weight of the Link pointed by Link pointer link to weigth. The weight of links can be retrieved with V_LINK(…).



INIT_NET( "node_Lab", "gen", nodes, par1, par2 ), INIT_NETS( obj, "node_Lab", "gen", nodes, par1, par2 )
These Macros create a Network using as Nodes the Object instances with label node_Lab that are descendants from current Object instance THIS, or from obj, if supplied. The search process for node_Lab is similar to SEARCH(…).

The Network topology is created using one of the available pseudorandom network generators (see below) configured with parameters par1 and par2 to include a number nodes of Nodes. If the existing number of instances of Object node_Lab is less than the required number of nodes, the missing instances are automatically created (excess instances are not deleted nor included in the Network). If nodes is not defined or set to zero, the Network will use all the existing instances of Object node_Lab as nodes.

The available network generator algorithms and the respective parameters are:

·         Disconnected network (gen = "DISCONNECTED") : does not require parameters par1/par2

·         Fully connected network (gen = "CONNECTED") : does not require parameters par1/par2

·         Directed random network (gen = "RANDOM-DIR") : par1 is the total number of directed links, par2 not required

·         Undirected random network (gen = "RANDOM-UNDIR") : par1 is two times the total number of undirected links (two arcs per link), par2 not required

·         Uniform directed random network (gen = "UNIFORM") : par1 is the out-degree, par2 not required

·         Renyi-Erdos undirected random network (gen = "RENYI-ERDOS") : par1 is the average out-degree, par2 not required
Star network (gen = "STAR") : does not require parameters par1/par2 (hub is first Node in the chain)

·         Circle network (gen = "CIRCLE") : par1 is the out-degree, par2 not required

·         Small World random network (gen = "SMALL-WORLD") : par1 is the out-degree, par2 is the rewiring probability

·         Scale-free random network (gen = "SCALE-FREE") : par1 is the average out-degree, par2 is the exponential scale factor

·         Lattice network (gen = "LATTICE") : par1 is the number of lattice rows/columns, par2 is 0 for 4 neighbors and 1 for 8 neighbors per cell

The INIT_NET(…) Macro return the effective number of directed links (arcs) of the generated network. According to the generator used, the Network may have to be reshuffled before use, applying SHUFFLE_NET(…).



LOAD_NET( "node_Lab", "filename" ), LOAD_NETS( obj, "node_Lab", "filename" )
SAVE_NET( "node_Lab", "filename" ), SAVE_NETS( obj, "node_Lab", "filename" )
The LOAD_NET(…) Macros read a network in Pajek format from file named “filename_xx.net” (where xx is the current pseudorandom number generator seed, see RND_SEED) using as Nodes the Object instances with label node_Lab that are descendants from current Object instance THIS, or from obj, if supplied. The search process for node_Lab is similar to SEARCH(…).

If the existing number of instances node_Lab is less than the required number, the missing ones are automatically created (excess instances are not deleted nor included in the Network).

The SAVE_NET(…) Macros save one LSD Network to a file in Pajek format named “filename_xx.net” (where xx is the current seed, as before) using as Nodes the Object instances with label node_Lab that are descendants from current Object THIS, or from Object obj, if supplied. The Network is not affected by these Macros. The filename may contain a path if saving to a particular directory is required. If no directory is included in filename, the file is saved to the current LSD model configuration directory.

If multiple simulation runs are going on xx (the seed) is automatically incremented sequentially, forcing the loading from or the saving to a different network file for each run. This is particularly useful when performing Monte Carlo analysis.



SNAP_NET( "node_Lab", "filename" ), SNAP_NETS( obj, "node_Lab", "filename" )
The SNAP_NET(…) Macros add a new LSD Network snapshot to a multi-network file in Pajek format named filename_xx.paj (where xx is the current seed, as in LOAD_NET(…)) using as Nodes the Object instances with label node_Lab that are descendants from current Object instance THIS, or from obj, if supplied. The search process for node_Lab is similar to SEARCH(…).

The Network is not affected by these Macros. The filename may contain a path if saving to a particular directory is required. If no directory is included in filename, the file is saved to the current LSD model configuration directory.

Multi-network files allow the analysis of the network dynamics in Pajek (or other software). If multiple simulation runs are going on xx (the seed) is automatically incremented sequentially, forcing the loading from or the saving to a different multi-network file for each run.



ADDNODE( id, "name" ), ADDNODES( obj, id, "name" )
These Macros add the Node data structure to the Object instance indicated by Object pointer obj, or the current Object THIS otherwise, defining also its Node ID to id (an integer value) and the Node name to name (a character string). name can be set to "" (null string). ID must be provided but it is not required to be unique in the Network. The search process for node_Lab is similar to SEARCH(…).

If Node already exists in the selected Object, it is reset (all existing Links are destroyed) to the new values (but preserving the unique system controlled serial number). Please note that the Node name is not related to the name (label) of the Object hosting the Node.

The ADDNODE(…) Macro returns an Object pointer to the newly created Node.



ADDLINK( node_obj ), ADDLINKS( obj, node_obj )
ADDLINKW( node_obj, weigth ), ADDLINKWS( obj, node_obj, weigth )
These Macros add a new Link from the Node indicated by Object pointer obj, or the current Node THIS otherwise, to the Node pointed by node_obj. Both Nodes must be in the same brotherhood chain. If any of the Object instances is not yet a Node, it has the Node added automatically. Link may have its weight set to weight when using the ADDLINKW(…) Macros, or have the weight set to 1 by default.

The ADDLINK(…) Macro returns a Link pointer to the newly created Link.



DELETE_NET( "node_Lab" ), DELETE_NETS( obj, "node_Lab" )
DELETE_NODE( ), DELETE_NODES( node_obj )
DELETE_LINK( link )
The DELETE_NET(…) Macros delete a Network composed by the Object instances labelled node_Lab, descending from the Object instance pointed by obj or the current instance THIS otherwise. The search process for node_Lab is similar to SEARCH(…).

All Nodes and Links associated with the brotherhood chain defined by node_Lab are deleted but no Object in the chain is deleted.

The DELETE_NODE(…) Macros delete the Node pointed by Object pointer node_obj. The Object hosting the Node is not deleted.

The DELETE_LINK(…) Macro delete the Link pointed by Link pointer link. The Nodes/instances pointed by link are not deleted.



SHUFFLE_NET( "node_Lab" ), SHUFFLE_NETS( obj, "node_Lab" )
These Macros shuffle the Nodes of a Network composed by the Object instances labelled node_Lab, descending from the Object instance pointed by obj or the current instabce THIS otherwise, using the Fischer-Yates shuffling algorithm. The search process for node_Lab is similar to SEARCH(…).

Shuffling reassigns Node IDs but does not change original node's serial numbers or the network structure.



CYCLE_LINK( curl ), CYCLE_LINKS( node_obj, curl )
This Macro creates a cycle (loop) through the set of all Links departing from a Node pointed by Object pointer node_obj or the current Object THIS otherwise, using Link pointer curl as iterator.

The following example shows an example. An Equation in the current Object Node scans all linked Nodes to calculate the average income of its neighbors:

…
v[1] = 0;                   // reset the income accumulator
CYCLE_LINK( curl1 )              // scan all network neighbors
{
  cur = LINKTO( curl1 );         // get an object link to current neighbor
  v[2] = VS( cur, "Income" );    // get neighbor income
  v[1] += v[2];                  // accumulate income
}

V[3] = STAT_NODE;           // get current node total outgoing links
v[4] = v[1] / v[3];              // calculate the average income
…



Mathematical and statistical functions
Besides the LSD specific Macros, modelers can use the several mathematical and statistical functions, as listed below.

Constants
·         pi: the number π

Functions and operators
·         abs( a ): the absolute value of a

·         exp( a ): the exponential of a

·         fact( a ): the factorial of a

·         log( a ), log10( a ): the natural or base 10 log of a

·         min( a, b ): the minimum between a and b

·         max( a, b ): the maximum between a and b

·         pow( a, b ): the power b of a

·         round( a ): the integer closest to the real value a

·         round_digits( a, b ): the real value a rounded to b significant digits

·         sin( a ), cos( a ), tan( a ), asin( b ), acos( b ), atan( b ): the trigonometric functions (a in radians)

·         sqrt( a ), cbrt( a ): the square or cubic root of a

·         tgamma( a ), lgamma( a ): the gamma or the log-gamma function of a

Tests
·         is_finite( a ), is_inf( a ), is_nan( a ): logical tests to check a for being finite, infinite or NaN (not a number), returning a zero if false

Random number drawing
·         alapl( med, alpha1, alpha2 ): a random draw from an asymmetric Laplace distribution with median med, parameters alpha1 and alpha2

·         bernoulli( p ): a random draw from a Bernoulli distribution with probability p

·         beta( alpha, beta ): a random draw from a beta distribution with parameters alpha and beta

·         binomial( p, t ): a random draw from a binomial distribution with probability p and t trials.

·         bpareto( alpha, min, max ): a random draw from a bounded Pareto distribution with shape parameter alpha in the interval [min, max]

·         cauchy( a, b ): a random draw from a Cauchy distribution with mode a and scale b

·         chi_squared( n ): a random draw from a chi-squared distribution with n degrees of freedom

·         exponential( lambda ): a random draw from an exponential distribution with lambda mean frequence

·         fisher( m, n ): a random draw from a Fisher F distribution with m and n degrees of freedom

·         gamma( alpha, beta ): a random draw from a gamma distribution with parameters alpha and beta

·         geometric( p ): a random draw from a geometric distribution with probability p

·         lnorm( mean, dev ): a random draw from a lognormal distribution with mean mean and standard deviation dev

·         norm( mean, dev ): a random draw from a normal distribution with mean mean and standard deviation dev

·         pareto( mu, alpha ): a random draw from a Pareto distribution with scale mu and shape parameter alpha

·         poisson( mean ): a random draw from a Poisson distribution with mean mean

·         student( n ): a random draw from a Student T distribution with n degrees of freedom

·         uniform( min, max ): a random uniform draw in the interval [min, max]

·         uniform_int( min, max ): a random integer draw in the interval [min, max] with uniform probability

·         weibull( a, b ): a random draw from a Weibull distribution with parameters a and b

Cumulative distribution functions
·         alaplcdf( med, alpha1, alpha2, x ): the cumulative asymmetric Laplace distribution with parameters alpha1 and alpha2 from minus infinity to x

·         betacdf( alpha, beta, x ): the beta cumulative distribution with parameters alpha and beta from minus infinity to x

·         bparetocdf( alpha, min, max, x ): the bounded Pareto cumulative distribution with shape parameter alpha in the interval [min, max] from minus infinity to x

·         gammacdf( mean, x ): the gamma cumulative distribution with mean mean from minus infinity to x

·         lnormcdf( mean, dev, x ): the lognormal cumulative distribution with mean mean and deviation dev from minus infinity to x

·         normcdf( mean, dev, x ): the normal cumulative distribution with mean mean and deviation dev from minus infinity to x

·         paretocdf( mu, alpha ): the Pareto cumulative distribution with scale mu and shape parameter alpha from minus infinity to x

·         poissoncdf( mean, x ): the Poisson cumulative distribution with mean mean from minus infinity to x

·         unifcdf( min, max, x ): the uniform cumulative distribution in the interval [min, max] from minus infinity to x

Other C++ functions defined in <cmath> may also available, according to the compiler version. Of course, it is always possible to declare new functions or link any C++ library to the LSD model.

Eigen linear algebra library*
LSD includes the powerful Eigen library for linear algebra, including native C++ support to matrix containers, numerical solvers, and related algorithms. Eigen namespace classes, methods and functions are available for use in the Equations’ code when the definition below is in the first line of the equations file:

#define EIGENLIB

By default, the above line is commented out, please remove the // in front of it to enable the Eigen library. Please notice that enabling the Eigen library substantially increase the size of the src subfolder created inside the model home folder when creating a LSD no-window version of the model.

Eigen objects, like matrices, can be easily associated with LSD Objects using the ADDEXT(…) family of macros. For instance, suppose you want to add a 2 by 2 floating point matrix to each Object named Firm, initialize the matrices with zeros, and then change some of the matrix elements:

EQUATION( "Init" )
/*
Initialization code, run just once in the beginning of the simulation
*/

CYCLE( cur, "Firm" )
{
  ADDEXTS( cur, MatrixXd );  // adds an empty matrix extension to each Firm
  DO_EXTS( cur, MatrixXd, resize, 2, 2 );  // resize the matrix to 2x2
  EXTS( cur, MatrixXd ) << 0, 0, 0, 0;  // set each matrix element to 0
  EXTS( cur, MatrixXd ) ( 0, 0 ) = 1;  // set element a11 to 1
  EXTS( cur, MatrixXd ) ( 0, 1 ) = 2;  // set element a12 to 2
  EXTS( cur, MatrixXd ) ( 1, 0 ) = 3;  // set element a21 to 3
  EXTS( cur, MatrixXd ) ( 1, 1 ) = 4;  // set element a22 to 4
}

PARAMETER;  // turns the variable "Init" into a parameter (run only once)

RESULT( 1 )

Notice that Eigen follows the C-language convention of starting indexes from zero, so the matrix element a11 is addressed by ( 0, 0 ) and so on.

To compute the matrix determinant and to directly access elements in the matrix created above in a Firm Object Equation:

…
v[0] = DO_EXT( MatrixXd, determinant );  // compute the determinant of matrix
v[11] = EXT( MatrixXd )( 0, 0 );  // pick element in the first row and column
…

To use the library many functions, please refer to Eigen website and documentation at http://eigen.tuxfamily.org.

Eigen requires a recent C++ compiler and is not supported on a Windows 32-bit LSD installation. Eigen may have problems when compiled with the –O0 compilation flag for optimizing (automatically selected then the Debug option is set in the LMM menu Model>Model Options). In this case, use at least –O1 or –Og when compiling using Eigen (the LSD defaults should be safe.

In some circumstances, Eigen may produce segmentation faults (SIGSEGV errors) in Windows due to some unsupported optimizations. These optimizations can be turned off by adding the following lines to the very beginning of your equation file:

#define EIGEN_DONT_VECTORIZE
#define EIGEN_DISABLE_UNALIGNED_ARRAY_ASSERT

Eigen is free software distributed under the MPL2 license.



Advanced LSD coding
The commands listed here provide special operations rarely necessary for basic models. They normally concern only advanced modelers.

#define EIGENLIB
This statement enables the usage of the Eigen math library. It must be placed/uncommented before the #include fun_head_fast.h line in the equation file. See details above on Eigen usage.

Eigen library is large and slows down the equation file compilation. Only enable Eigen if it is being used in the equation file.

#define EQ_USER_VARS var1; var2; …
The statement allows the definition of user temporary local variables which are visible in all Equations. Multiple variable definitions (var1, var2, …) must be separated by semicolons (“;”). Multi-line definitions are possible by using the backslash line break terminator (“\”).

Those variables are not global, in the sense that values stored to them are visible only in the current Equation. If C++ global user variables are required, please check above.

#define NO_POINTER_INIT
By default, LSD initializes all predefined temporary pointers (cur, curl etc.) to the NULL value, to make detection of mistakes simpler. This is a recommended practice in C++ programing, considering the language does not check for pointers validity before they are used. Usage of non-initialized pointers is a common cause of models crashing LSD. However, pointer initialization adds overhead to the simulation execution. The definition of NO_POINTER_INIT disables the default LSD pointer initialization behavior and eliminates the associated performance penalty.

The #define NO_POINTER_INIT statement must be placed/uncommented before the #include fun_head_fast.h line in the equation file.  Please note that pointer checking is always disabled when the legacy LSD header file fun_head.h is used instead of fun_head_fast.h.

Additionally, checking for NULL (or invalid) pointers before their usage also add overhead to the simulation execution. To disable the pointer check mechanism, please check the Macro NO_POINTER_CHECK below. When NO_POINTER_INIT is defined, the Macro USE_POINTER_CHECK is disabled and will be ignored if used. This is the lowest overhead configuration in LSD.

Warning: when the #define NO_POINTER_INIT statement is not used, in some special circumstances a LSD Macro argument may be evaluated more than one time during its execution. This may likely produce undesirable behaviors when the modeler does any kind of C++ variable assignment in a Macro call. For instance, in:

ERROR - ERROR - ERROR
ADDHOOK( COUNT( "Firm" ) );
i = 0;

CYCLE( cur, "Firm" )
  WRITE_HOOK( cur, i++ );  // read the number of hook pointers assigned to object

ERROR - ERROR - ERROR

The variable i gets incremented twice in a single invocation of Macro COUNT_HOOKS(…), which is not the intended behavior of storing a pointer to each Firm Object to a sequential hook of the Market parent Object. After adding half of the Firm instances, the code above will crash as i will go over the number of allocated hooks. Instead, the code should be written as:

ADDHOOK( COUNT( "Firm" ) );
i = 0;

CYCLE( cur, "Firm" )
{
   WRITE_HOOK( cur, i );  // read the number of hook pointers assigned to object
   ++i;
}

close_sim( )
At the end of each simulation run modelers can execute some specific code. Typically, this is used to free some memory explicitly allocated during the simulation run (using new, malloc etc.). The C++ void function close_sim( ) should be located at the end of the Equation file. By default this function does nothing as it has an empty body ({…}), but has to be declared once.

ABORT
This Macro instructs LSD to stop the simulation run at the end of the current simulation step. When it is executed in one Equation, LSD is dynamically reconfigured to assume that the last simulation time step is the current one (time T). This means that all equations still not evaluated during the current time step are still evaluated before the simulation stops. All saved data is available for the Analysis of Results. If multiple simulation runs are configured, LSD execution will continue with the next run, if any.

CALLER
CALLER Macro points to the Object containing the Variable or Function which requested the computation of the current Variable or Function under computation, or NULL if the computation was requested by LSD because the Variable was not used by any Equation. This is the same Object in predefined C++ pointer c.

See the details about Object’s organization for more information.

CONFIG
PATH
These Macros return the current LSD model configuration name (CONFIG) and the corresponding model configuration directory (folder) in the operating system storage (PATH), respectively, as C++ constant character arrays. Note that PATH does not end in a slash (“/”) character, so adding one when concatenating PATH and CONFIG is compulsory, as in the example:

…
char[500] filename;   // allocate space for the file name character array (string)
sprint( filename, "%s/%s", PATH, CONFIG );  // concatenate with the "/" between
SAV_LAT( filename );
…

This information is usually used by the modeler for saving information to the computer disk while computing the Equations code, as in SAVE_LAT(…), SAVE_NET(…) or SNAP_NET(…).

CURRENT
This keyword contains the current value of the Variable under computation, irrespective of the time of last computation of the Variable. It is the same value that can be obtained with the Macro VL( "Lab", 1 ), used in the Equation for Lab. The result of the Equation will be reported in CURRENT at the subsequent time step.

EQUATION( "Up" )
/*
This function returns a progressive value each time it is requested
*/

v[0] = CURRENT;
v[1] = VL( "Up", 1 );

/* v[0] and v[1] are identical */
RESULT( v[0] )

This can be used even when a Variable is not defined to store past values, since it accesses directly the LSD memory location storing the variables’ values.

DEBUG_START
DEBUG_START_AT( step )
DEBUG_STOP
DEBUG_STOP_AT( step )
When Macro DEBUG_START is invoked within an Equation, from that moment on LSD will enable the LSD Debugger and will save all the names of the Variables computed in a file named log.txt in the model home directory, together with some additional information. Macro DEBUG_STOP reverses that action, disabling the LSD Debugger and stop recording to log.txt. This file may help spotting problems in the simulation due to the order in which the Variables are being evaluated by LSD. Optionally, the Debugger and the log file can be started or stopped only at a specific time step in the simulation by using DEBUG_START_AT(…) and DEBUG_STOP_AT(…).

These Macros do not open the LSD Debugger by themselves (if this is required, see INTERACT). Instead, they enable the Debugger to open as soon as a Variable with the Debug flag set (menu Model>Change Element), if any, is computed. They perform similarly to setting the Start debugger at step option in menu Run>Simulation Settings, with one important difference, as only the Macros enable the creation of a log file. However, the file creation causes LSD to enter into a significantly slower mode as at any time a Variable needs to be computed, LSD must add data to log.txt. So, if a log file is not required, using the Start debugger at step option is recommended.

The Macros are disabled when fast modes or parallel updating of Variables is enabled. If debugging is need in those situations, it is possible to temporarily disable the FAST and FAST_FULL modes by using the Macro OBSERVE, or temporarily disabling parallel processing before the simulation starts by selecting the Disable parallel computation checkbox in menu Run>Simulation Settings.

DEFAULT_RESULT( value )
Defines the default result for an EQUATION(…) which fails in producing a valid RESULT(…)/ END_EQUATION(…) to value. The standard default value is 0 (zero).

DELETING, DELETING( obj )
Test if the current the Object instance currently under computation, or the one pointed by obj, has been already set for deletion but was not yet effectively deleted because one or more of its Variables are still under computation (see DELETE(…) for details). A zero result (true) indicates the Object instance is not set for deletion (yet). Values different than zero (false) mean the instance is going to be deleted soon and should not normally be considered.

All pending computations, upstream to the one triggering the deletion, must account for this delay, during which the to be deleted instance is still present and affecting calculations. The DELETING(…) Macros can be used to detect if an Object instance is currently waiting to be deleted.

END_EQUATION( value )
Immediately ends an EQUATION(…) execution, assigning value as the computed value for the current time step.

EQUATION_DUMMY( “Lab”, “main_Lab”)
This macro declares an Equation which has no code, to be associated with a Variable labelled Lab contained in the same Object. The value for this Variable is supposed to be computed by the Equation for another Variable named main_Lab and updated by the use of an explicit WRITE(…) command. Differently from EQUATION(…), this macro is used without a matching RESULT(…) statement.

The retrieval of Variable Lab value automatically triggers the computation of Variable main_Lab, if not already done. The Variable main_Lab must exist and be contained in the same Object as Lab. The effective execution of the WRITE(…) command in the Equation for main_Lab is not checked by LSD. If the Equation for Variable main_Lab does not update the value of Lab Variable, the previous period value is automatically replicated.

Warning: when parallel updating is enabled for the main_Lab Variable whose Equation updates Lab, LSD may hang (deadlock) in some circumstances, according to the order the Variables are updated. As a workaround, the user may define main_Lab as an empty string (""). This workaround can be also applied if Variable Lab is not contained in the same Object as main_Lab.

FAST
FAST_FULL
OBSERVE
These Macros dynamically change the simulation execution mode, as if the modeler had clicked on the respective buttons of LSD Log window while a simulation is running. OBSERVE mode is the default when one simulation starts to run and it creates a run-time plot window (if there is at least one Variable set to show during run time) and show extra (verbose) information in LSD Log window during the run. FAST mode prevents the update of the run-time plot window (and hide it) as well discards the presentation of most of the non-critical information in LSD Log window (or log file), effectively speeding up the simulation execution. FAST_FULL further prevents the presentation of any non-error message.

The FAST and FAST_FULL modes reduce the information available to LSD when an error occurs. It is recommended to turn it off during the debugging of Equations’ code.

LAST_CALC( "lab" ), LAST_CALCS( obj, "lab" )
Return the last time the Variable or Parameter named Lab contained in the Object instance currently under computation was calculated or updated. If the element was never calculated or updated LAST_CALC(…) returns zero. If Object pointer obj is supplied, Variable labelled Lab is supposed to be located in it. The search process for Lab is similar to V(…).

LOG( "Comment" )
LOG( "format", var1, var2, … )
PLOG( "Comment" )
PLOG( "format", var1, var2, … )
The LOG(…) Macro family produces a message in the LSD Log window. If a regular string Comment is provided, it is presented “as is”. However, if a specially formatted string format is provided, then the modeler can also present the numerical contents of C++ variables (like temporary storage v[…]) and other data. format is defined exactly like for the standard C/C++ printf(…) function and is very flexible in terms of what can be presented and how, please see the C/C++ standard library documentation.

Any message submitted with LOG(…) is only presented while LSD simulation is running in OBSERVE(…) mode. Use PLOG(…) (as “priority” log) to be able to present messages that are required in any running mode (OBSERVE(…) or FAST(…)).

NAME, NAMES( obj )
NAME Macro returns a pointer to a C string containing the name of the Object containing the Variable under computation. NAMES(…) returns a pointer to a C string with the name of the Object pointed by obj, if valid, or NULL otherwise. The provided string is read only, and its content must not be changed.

NEXT, NEXTS( obj )
NEXT Macro points to the Object instance which is the next to the current Object under computation in its brotherhood, if any, or NULL otherwise. NEXTS(…) returns a pointer to the next Object instance to the Object pointed by obj., if any, or NULL otherwise.

Differently from the predefined C++ pointer next, the Macro NEXTS(…) do check its argument for NULL and invalid Object instances (if NO_POINTER_CHECK is not defined) and so is preferred to directly using the predefined pointer.

See the details about Object’s organization for more information.

Warning: The next Object is not guaranteed to exist, as this depends on the position of the Object where the Variable/Function under computation is located in the model structure. It is the modeler responsibility checking when the last Object instance in a brotherhood is reached and prevent using NULL pointers in other Macros like VS(…), for instance.

NO_NAN
USE_NAN
Numeric values in LSD are always stored as IEEE 754 double-precision binary floating-point format. This format, beyond the representation of decimal numbers between 10-308 and 10+308 with 16 significant digits precision, allows for special representations of the non-numeric value positive infinite, negative infinite and NAN (not a number). By default, LSD does not allow Variables and Parameters to be set to any of the special floating-point representations, stopping showing an error if any such values are attributed to a Variable by an Equation.

However, in some special situations the modeler may want to use the value NAN to indicate that the Variable’s value is not a number (or, generically, not available) for some specific time steps, to prevent biases in statistical calculations for instance. The Macro USE_NAN instructs LSD to accept Variables to be set to NAN (but still forbidding infinite values), while NO_NAN reverts it to the default behavior. In any case, LSD will treat NAN’s in results data as “not available” for plot generation, statistics calculation and data export.

NO_POINTER_CHECK
USE_POINTER_CHECK
By default, LSD now checks all pointers used in Macros for validity. Whenever a Macro which requires an Object instance pointer is called, if the pointed address does not correspond to an existing instance LSD will issue an error message and stop the simulation, pointing to the originating Macro location. As pointer checking may have significant performance impact for some models, particularly when parallel computation is used, these Macros allow for the control the pointer checking mechanism.

After Macro NO_POINTER_CHECK is called, only NULL pointer checking is performed. This way, if access to an invalid non-NULL pointer is performed, LSD will immediately crash (segmentation fault, SIGSEGV). Despite speeding up the simulation, it is recommended that this Macro be only used after the model code is fully debugged.

Every time USE_POINTER_CHECK is invoked, full pointer checking is enabled but, before, LSD has to (re) build a list with all model Object instances, which may be computationally expensive for large models. Therefore, the switching between check/no-check modes during simulation should be kept to a minimum. Also, switching must not be done in Equations which are computed in parallel.

Alternatively, the statement #define NO_POINTER_CHECK may be also used at the beginning of the equation file (before the #include fun_head_fast.h line). This way, pointer checking is completely disabled, which provides a small performance gain when compared to using Macro NO_POINTER_CHECK in the equations’ code. However, in this case pointer checking is permanently disabled at the compilation time, so the use of USE_POINTER_CHECK will be ignored. Disabling pointer checking this way also turn off the NULL pointer checking in most commands.

Pointer checking is always completely disabled when the legacy LSD header file fun_head.h is used instead of fun_head_fast.h.

NO_SAVED
USE_SAVED
By default, LSD can only access lagged values of a Variable (like in Macro VL(…)) when such Variable is defined with the corresponding number of lags in the model configuration. However, if the Variable is set to be saved, a LSD model can optionally access any lag of it after the Macro USE_SAVED is invoked. To restore the default behavior, use Macro NO_SAVED. Please note that Variables requiring the initialization of lagged values must still be explicitly defined using the appropriate number of lags in the model configuration, irrespective of being saved or not.

NO_SEARCH
USE_SEARCH
Most LSD Macros which does not modify the values of Variables and Parameters (like V(…)) search for the specified label using a recursive strategy. According to this scheme, if a Variable or Parameter named as the given label does not exist in the current Object (the one containing the Variable or Function under computation), LSD ends up searching the entire model structure. This default behavior is usually desired, as relieves the modeler from specifying where the desired element is located, also making changes in the structure transparent to the Equations.

However, under certain circumstances, it may be useful to use Macro NO_SEARCH to disable the default searching behavior, so LSD does only look for the specified label in the current or the expressly defined Object, and an error is produced when the Variable or Parameter is not found in this location. This may be useful to isolate some kinds of bugs, and to force the full specification of every element location, for maximum performance (the default scheme may add some overhead to the simulation processing). Macro USE_SEARCH restores the default search mechanism.

NO_ZERO_INSTANCE
USE_ZERO_INSTANCE
By default, LSD ensures that at least one instance of any Object is preserved and cannot be deleted (see DELETE(…)). This is done so all Macros dealing with Object instances produce adequate results. However, some Macros, but not all, may deal with Objects which have all instances deleted during simulation execution, when meaningful (e.g., when summing or counting instances). To enable this behavior, the Macro USE_ZERO_INSTANCE can be used by the modeler before any command which may require at least one Object instance.

The modeler must handle the results coming from those situations, usually NAN or NULL values, to avoid potentially serious errors. These restrictions apply to all Macros returning values or pointers computed from Objects, Variables or Parameters For instance, Macros returning values, when an argument Variable or Parameter is contained in an Object which has zero instances (all copies were deleted during the simulation run), will return the special value NAN (not a number). So, if the modeler knows this situation is expected, she must always check the value returned by SUM(…) for NANs (using the function is_nan(…)) and handle the Equation computation accordingly (please refer to the USE_NAN Macro to allow NANs to be used as legal Variable values). Similarly, Macros returning pointers, when receiving zero-instanced Objects as arguments, return a NULL (invalid) pointer. NULL pointers can be directly tested using regular conditional expressions.

To re-enable the default no-zero-instanced Object policy, the Macro NO_ZERO_INSTANCE is also available.

PARAMETER
This Macro within an Equation transforms the corresponding Variable in a Parameter, so that the Equation is never computed again during the simulation run. Normally, this is used in Equations meant to initialize a model at the very first time step, and does not need to be computed during a simulation run.

PARENT, PARENTS( obj )
GRANDPARENT, GRANDPARENTS( obj )
PARENT Macro returns a pointer to the Object instance immediately above (parent) of the current (THIS) Object in the model structure, if any, or NULL otherwise. PARENTS(…) operates similarly but returning the Object above (parent) the Object pointer obj, if any, or NULL.

Macro GRANDPARENT provides a pointer to the Object two levels above (grandparent) the current (THIS) one, if any, or NULL otherwise. GRANDPARENTS(…) operates similarly for the Object pointed by obj.

See the details about Object’s organization for more information.

Differently from the predefined C++ pointer up, the Macros PARENTS(…) and GRANDPARENTS(…) do check their arguments for NULL and invalid Object instances (if NO_POINTER_CHECK is not defined) and so are preferred to directly using the predefined pointers.

Warning: The parent or grandparent Object instances are not guaranteed to exist, as this depends on the position of the Object where the Variable/Function under computation is located in the model structure. It is the modeler responsibility not using those Macros at Objects at the model root (PARENT, PARENTS(…), GRANDPARENT and GRANDPARENTS(…)) or the first level (GRANDPARENT and GRANDPARENTS(…) only) of the model.

RECALC( "Lab" ), RECALCS( obj, "Lab" )
Mark the Variable named Lab contained in the Object instance currently under computation as not calculated in the current time, forcing recalculation if it has been already calculated. If the Variable has not been calculated yet, or if Lab is a Parameter, RECALC(…) does nothing. If Object pointer obj is supplied, Variable labelled Lab is supposed to be located in it. The search process for Lab is similar to V(…). This macro returns the current value of Variable Lab, without recalculating it if not already done.

For already computed Variables, RECALC(…) has to “slide back” the past values, if any. However, the oldest lagged value cannot be recovered except if the Variable Lab is marked to be saved. Lost lagged values are replaced by a NAN (not a number) value.

If Variable has a non-default updating scheme defined, RECALC(…) also sets the Variable to be recomputed in the current time step irrespective of the configured updating options. Subsequent updates are handled as initially configured.

Warning: In some circumstances, if Variable is marked for saving and it was already computed when calling RECALC(…), the saved value is not updated. In this case, it is necessary to subsequently call the UPDATE[…] Macro on the Object containing Variable, to ensure the latest computed value is properly saved. This issue does not affect the model computation, just the saved values.

RND
Macro RND produces a pseudorandom number uniformly distributed between 0 and 1 using the generator algorithm selected by RND_GENERATOR(…).

RND_GENERATOR( type )
The RND_GENERATOR(…) Macro selects the random number generator (hardware or algorithm) to be used when extract draws or producing pseudorandom numbers. The following type options are available:

0. True (hardware-based) random number generator in (0,1)
1. Linear congruential (32-bit) in (0,1)
2. Mersenne-Twister (32-bit) in (0,1) (default option)
3. Linear congruential (32-bit) in [0,1)
4. Mersenne-Twister (32-bit) in [0,1)
5. Mersenne-Twister (64-bit) in [0,1)
6. Lagged Fibonacci (24-bit) in [0,1)
7. Lagged Fibonacci (48-bit) in [0,1)

Advanced Users: the macro also returns a C++ void pointer to the selected generator engine. This allow the simulation to share the same entropy source as the internal LSD functions. Before usage, the returned pointer must be casted to the appropriate generator type (0:random_device, 1/3:minstd_rand, 2/4:mt19937, 5:mt19937_64, 6:ranlux24, 7:ranlux48) from the C++11 standard library.

RND_SEED
RND_SETSEED( seed )
Macros RND_SETSEED(…) changes the integer seed number used by the pseudorandom number generator to seed, an unsigned integer value. RND_SEED reads the currently set unsigned integer seed number in double precision format.

ROOT
ROOT Macro points to the model root Object (as the predefined C++ pointer root).

See the details about Object’s organization for more information.

SLEEP( msecs )
The Macro SLEEP(…) pauses the simulation execution for the number msecs of milliseconds (one millesimal or 1/1,000 of a second). For instance, SLEEP( 1000 ); pauses the simulation for one second. After this period elapses, the simulation continues normally. SLEEP(…) uses an efficient operating system method to pause LSD, preventing CPU usage during this period, and so is preferred to traditional do-nothing cycles/loops.

This Macro is normally used when LSD execution has to be slowed down to allow the user realize the changes are going on, for example, when a lattice is being updated.

T
LAST_T
RUN
LAST_RUN
The Macro T indicates the current time step of the simulation. Macro RUN provides the current simulation run number for multi-run executions, or 1 for single-run ones.

The Macro LAST_T provides the configured number of time steps before the simulation is stopped (or the last step). Macro LAST_RUN informs the current number of simulation runs configured (1 for single-run executions). Both Macros produce constant values throughout the simulation run(s).

The four Macros return positive integer numbers properly formatted as double precision values.

For example, the code below calculates the next time step to be run in the simulation, also considering the possibility of the current time step be the last time step in the simulation:

…
if( T < LAST_T )
  v[9] = T + 1;
…

THIS
THIS Macro points to the Object in which the current computation is going on (as the predefined C++ pointer p).

See the details about Object’s organization for more information.

TSEARCH( "obj_Lab", pos ), TSEARCHS( obj, "obj_Lab", pos )
INIT_TSEARCH( "obj_Lab" ), INIT_TSEARCHS( obj, "obj_Lab" )
INIT_TSEARCHT( "obj_Lab", num ), INIT_TSEARCHTS( obj, "obj_Lab", num )
Turbo searches Object instances contained in the level immediately below the current (THIS) or the chosen (obj) Object instance. The search process for Lab is similar to SEARCH(…). Instead of searching for a name (like SEARCH(…)) or a value (like SEARCH_CND(…)), TSEARCH(…) family looks for the Object instance position (pos) in the brotherhood chain of Object instances named obj_Lab. pos can be set from 1 (first instance in the chain) up to the total number of instances in the brotherhood.

A pointer to the instance in the pos position of the chain is returned by TSEARCH(…) Macros. If pos is larger than the number of existing Object instances, the last one will be returned.

Before the first usage of TSEARCH(…), a special search data structure must be initialized by the modeler using one of the forms of the INIT_TSEARCH(…) Macro. If TSEARCH(…)is used before initialization, an error is produced and the simulation stops. If the suffix “T” initialization Macro version is selected, the total number (num) of Object instances in the chain has to be informed. This speed up the initialization search procedure a little bit but has no effect on the subsequent TSEARCH(…) performance.

INIT_TSEARCH(…) Macros return the number of Object instances found for initialization.

The main benefit of the TSEARCH(…) family when compared to other search alternatives is its speed, usually several times faster for longer chains of Object instances. On the other hand, this family of Macros requires that the object chain be previously sorted (so the Object position is meaningful for the modeler) and the search algorithm be initialized (so the search data structure exists and can be used). Initialization must be repeated any time Object instances are added or deleted to/from a previously initialized Object brotherhood, before using TSEARCH(…) again (or unpredictable results are produced).

TSEARCH_CND( "Lab", value ), TSEARCH_CNDS( obj, "Lab", value )
INIT_TSEARCH_CND( "Lab" ), INIT_TSEARCH_CNDS( obj, "Lab" )
Turbo searches in and below the current (THIS) or the chosen (obj) Object instance for the first one which contains the Variable named Lab with the specific numeric value value. The search process for Lab is similar to V(…).

It performs similarly to SEARCH_CND(…) Macros, but using a much faster method to retrieve the desired instance, in particular when searching among many of them. The Variable or Parameter values are supposed to be unique for each Object instance.

The Macros return the pointer to the first found instance of Object obj_Lab or NULL if no instance matching value is found. If non-unique value instances are present, only the first one appearing in the brotherhood chain will be returned by TSEARCH_CND(…).

However, differently from SEARCH_CND(…), the TSEARCH_CND(…) Macros require the initialization of the search engine data structure using one of the forms of the INIT_TSEARCH_CND(…) Macro. If TSEARCH_CND(…)is used before initialization, an error is produced and the simulation stops. Initialization must be repeated any time the values stored in the Variable or Parameter Lab have changed, in any Object instance, or when Object instances are added or deleted to/from the set of instances. Performing TSEARCH_CND(…) under such conditions will produce undefined behavior.

INIT_TSEARCH_CND(…) Macros return the number of Object instances found for initialization.

Each set of Object instances under a common parent (a brotherhood) can have a single Variable or Parameter initialized for conditional turbo searching. Every time INIT_TSEARCH_CND(…) is called, it resets any existing initialization for the brotherhood containing the chosen Variable or Parameter Lab. However, multiple active initializations can be maintained in different brotherhoods.

UPDATE, UPDATES( obj )
UPDATE_REC, UPDATE_RECS( obj )
The first form of this Macro forces the immediate calculation of all Variables contained in the current (THIS) or the chosen (obj) Object, only if not already computed. Variables contained in descending Objects are not affected. Variables are calculated even if the Object is marked not to be computed in the model configuration. However, the defined Variable updating scheme is honored.

The second form (_REC suffix) triggers the calculation of all Variables in the current (THIS) or the chosen (obj) Object plus all Variables contained in all the descending Objects, recursively. Variables contained in descending Objects marked not to be computed are not computed in this case. The defined Variable updating scheme is always honored.

A typical application of these Macros is the updating of aggregate Variables before the deletion of Object instances containing Variables used in the aggregates computation.

V_CHEAT( "Lab", fake_caller ), V_CHEATS ( obj, "Lab", fake_caller )
V_CHEATL ( "Lab", lag, fake_caller ), V_CHEATLS ( obj, "Lab", lag, fake_caller )
When the standard V(…) family requests the value of Variable Lab, the executed Equation can access the Object instance containing the caller Variable (Object pointer CALLER). However, in some cases, mostly for reasons of efficiency, it may be useful to "cheat" the requested Variable in believing that the instance is something else. With V_CHEAT(…) the modeler must specify another Object (fake_caller) that will appear in the called Variable as if it were the caller. The search process for Lab is similar to V(…).

Please note that if Lab is a Function and fake_caller is NULL, the Function’s associated Equation will not be (re) computed and the last calculated value, if any, is returned.



Extended Object data
Data extensions allow adding to LSD Objects unrestricted data storage capabilities, beyond the usual sets of Variables and Parameters. Indeed, data extensions are conceptually closer to Parameters, as LSD does not try to update its values along the simulation. The modeler is entirely responsible for the initialization and the update of data in the data extension. However, data in the extension is readily available for use in any LSD Variable Equation that has access to the hosting Object using the Macro P_EXT(…) to acquire a pointer to the data structure.

In particular, if the data extension is a C++ class or struct, the modeler can use some additional Macros, like V_EXT(…), WRITE_EXT(…), and DO_EXT(…) to directly read from and write to the extensions’ internal variables and objects (if appropriate). Also, the extensions’ internal objects methods can be directly executed using EXEC_EXT(…).

ADDEXT( class ), ADDEXTS( obj, class )
ADDEXT_INIT( class, initializer ), ADDEXT_INITS( obj, class, initializer )
These Macros adds an arbitrary C++ data extension of type class to the current LSD Object under computation, allocating memory space for it. If an Object pointer obj is supplied, the data extension is added to this specific Object. The data extension can be from a simple scalar variable to a fully defined C++ object like a class or a struct. Data extension allocation is automatically handled by LSD, being destroyed when the host Object is deleted (C++ object constructor and destructor are called accordingly). Each LSD Object instance can have a single data extension; adding an extension (of the same class) when one already exists automatically destroys the previous one.

The ADDEXT_INIT(…) variations allow for defining the C++ class initialization parameter(s). If multiple parameters are required, simply separate them with commas (“,”) after the class name.

In the example below, firstly we define the required C++ extension types before the MODELBEGIN line:

#include "fun_head_fast.h"

#include <vector> // include the vector container in STL

class ext_firm    // this is simply a declaration, nothing is allocated here
{
  public:
  int num_params;
  vector < int > tech_params;   // define an initially zero-length vector
};

MODELBEGIN
…

In this case, our data extension is a C++ class named ext_fin which contains two elements, an integer variable (num_params) and a C++ dynamic integer vector named tech_params (defined in C++ standard library <vector>). Next, we add this extension to all Firm LSD Object instances by means of a LSD startup-type Variable and reset the variable holding the size of the vector, run just once:

EQUATION( "Init" )
/*
Initialization code, run just once in the beginning of the simulation
*/

CYCLE( cur, "Firm" )
{
  ADDEXTS( cur, ext_firm );  // adds the extension to each Firm in the CYCLE
  WRITE_EXTS( cur, ext_firm, num_params, 0 );  // set each Firm vector size to 0
}

PARAMETER;  // turns the variable "Init" into a parameter (run only once)

RESULT( 1 )

Then, in an Equation of a Firm-level Variable, it is possible to add/remove elements to the integer vector and read/change their values. In the example below, we make sure the tech_params vector is correctly set before using it to compute Firm’s productivity using an external library function (not shown below). If format is not ok, rebuild the vector, by clearing it and initializing the vector and the integer variable, before calling the function:

…
// refill the Firm-specific vector if it has less than 3 parameters
if ( V_EXT( ext_firm, num_params ) < 3 )  // get firm’s num_params and compare
{
  EXEC_EXT( ext_firm, tech_params, clear );    // clear the vector
  EXEC_EXT( ext_firm, tech_params, push_back, T ); // writes current time to 1st position

  // write 2 random integers between 1 and 100 in the 2nd and the 3rd positions
  EXEC_EXT( ext_firm, tech_params, push_back, ( int ) uniform_int( 1, 100 ) );
  EXEC_EXT( ext_firm, tech_params, push_back, ( int ) uniform_int( 1, 100 ) );

  WRITE_EXT( ext_firm, num_params, 3 );       // set Firm’s num_params to 3
}

// call an external library function requiring a pointer to a integer vector
v[1] = eval_prod( P_EXT( ext_firm ) );    // evaluate Firm productivity based on vector
…

Warning: when adding an extension class of a different type of the previously allocated one, DELETE_EXT(…) must be explicitly called before ADDEXT(…). Ignoring to do so will likely produce a crash in LSD.

CYCLE_EXT( iter, class, c_obj ), CYCLE_EXTS( obj, iter, class, c_obj )
Cycles (loops) over the elements of a C++ object named c_obj within an extension of type class (a class or a struct, in this case) hosted in the current Object THIS, or in the Object pointed by obj. Modeler has to supply iter , an existing iterator-type variable to the C++ object c_obj. The iterator iter is updated according to the cycle moves over the elements of object c_obj.

Iterators are created using the method iterator, as in the example below. An iterator method should be present in any valid C++ container type. All STL containers, like vectors and arrays, have it. If you create your own container class, you have to explicitly define an iterator method.

If the target LSD Object has no data extension set previously by ADDEXT(…), CYCLE_EXT(…) does nothing.

As an example, the code below builds on the example provided in ADDEXT(…) and copies the values in the Firm’s tech_params vector, defined in the Firm data extension of type ext_firm (a C++ class), to LSD temporary storage v[…]:

…
vector< int >::iterator iter;    // create iterator to integer vector type
i = 0;                      // index for v[…] array
CYCLE_EXT( iter, ext_firm, tech_params )  // loop over all vector elements
{
  v[i] = *iter;             // get current vector element
  i++;                      // update v[…] index (iter is updated by CYCLE_EXT)
}
…

DELETE_EXT( class ), DELETE_EXTS( obj, class )
The Macros delete the C++ data extension of type class in the LSD Object THIS under computation, or in the Object pointed by obj, if present, deallocating any allocated memory space. If the target LSD Object has no data extension set previously by ADDEXT(…), DELETE_EXT(…) does nothing.

DO_EXT( class, meth, … ), DO_EXTS( obj, class, meth, … )
DO_EXT(…) Macros execute method (function) meth defined by the top class of an extension of type class (a class or a struct, in this case) hosted in the current Object THIS, or in the Object pointed by obj. If method meth requires arguments, they should be also included in DO_EXT(…) call after the regular elements. If the method returns a value, the Macro also returns it.

Please note that DO_EXT(…) does not allow directly invoking a method belonging to an inner_class contained in class, please use the command EXEC_EXT(…) for this purpose.

If the target LSD Object has no data extension set previously by ADDEXT(…), DO_EXT(…) return value is undefined. If unsure if the extension exists, it is advisable to check it before executing DO_EXT(…), testing for a non-NULL P_EXT(…) return. If the extension is not a C++ class or struct, or the named meth does not exist inside it or have an incompatible type, a compiler error message is thrown.

See ADDEXT(…) for more details on C++ data extensions.

EXEC_EXT( class, c_obj, meth, … ), EXEC_EXTS( obj, class, c_obj, meth, … )
Executes method (function) meth defined by a C++ object named c_obj within an extension of type class (a class or a struct, in this case) hosted in the current Object THIS, or in the Object pointed by obj. If method meth requires arguments, they should be also included in EXEC_EXT(…) call after the regular elements. If the method returns a value, the Macro also returns it.

Please note that EXEC_EXT(…) does not allow directly invoking a method in class, please use the command DO_EXT(…) for this purpose.

If the target LSD Object has no data extension set previously by ADDEXT(…), EXEC_EXT(…) return value is undefined. If unsure if the extension exists, it is advisable to check it before executing EXEC_EXT(…), testing for a non-NULL P_EXT(…) return. If the extension is not a C++ class or struct, or the named c_obj and meth do not exist inside it or have an incompatible type, a compiler error message is thrown.

See ADDEXT(…) for more details on C++ data extensions and an example.

EXT( class ), EXTS( obj, class )
The Macros allow direct access to the C++ data extension of type class in the LSD Object THIS under computation, or in the Object pointed by obj, if present.

Warning: the data extension must be set before using this Macro. If the target LSD Object has no data extension set previously by ADDEXT(…), a LSD run time crash will happen.

See ADDEXT(…)for more details on C++ data extensions and here for an example.

P_EXT( class ), P_EXTS( obj, class )
The Macros return a pointer of type class to the C++ data extension in the LSD Object THIS under computation, or in the Object pointed by obj, if present. If the target LSD Object has no data extension set previously by ADDEXT(…), they return NULL.

See ADDEXT(…) for more details on C++ data extensions and an example.

V_EXT( class, var ), V_EXTS( obj, class, var )
Read the contents of a C++ variable named var contained within an extension of type class (a C++ class or struct) from the current Object THIS, or from the Object pointed by obj. The Macros return value can be assigned to any existing C++ variable of the same type.

If the target LSD Object has no data extension set previously by ADDEXT(…), V_EXT(…) return value is undefined. If unsure if the extension exists or not, it is advisable to check it before calling V_EXT(…), testing for a non-NULL P_EXT(…) return. If the extension is not a C++ class or struct, or the named var does not exist inside it nor has a type incompatible, a compiler error message is thrown.

See ADDEXT(…) for more details on C++ data extensions and an example.

WRITE_EXT( class, var, value ), WRITE_EXTS( obj, class, var, value )
WRITE_ARG_EXT( class, c_obj, value, … ), WRITE_ARG_EXTS( obj, class, c_obj, value, … )
WRITE_EXT(…) macros write value to the contents of a C++ variable named var contained within an extension of type class (a C++ class or struct) from the current Object THIS, or from the Object pointed by obj. value must have the same C++ type as the target var.

WRITE_ARG_EXT(…) writes value to C++ addressable object c_obj (like vectors and matrixes) passing the additional argument(s) to c_obj. Multiple arguments are separated by commas (“,”). Note that argument(s) must be passed after the value to be written.

If the target LSD Object has no data extension set previously by ADDEXT(…), WRITE_EXT(…) do nothing. If the extension is not a C++ class or struct, or the named var does not exist inside it nor has a type incompatible with value, a compiler error message is thrown.

See ADDEXT(…) for more details on C++ data extensions and an example.



Basic C++ coding
LSD modelers do not need to fully write programs in C++. They can just write basic code to compute the values of Variables. LSD Macros and functions are available to allow the reading of model’s values or to modify them. The elaboration of the Variables' Equation can use any legal C++ element or construction. Here we present some basic (and frequently used) commands. Remember that the code to compute a Variable's value is executed sequentially inside an EQUATION(…) block when the system requires that Variable to be updated. At the end of the Equation's computation the system assign the value informed in RESULT(…) to the Variable and then moves to compute another Variable.

The Equation code is composed by lines of commands that computer generally executes sequentially, moving to the next line when the previous one has been completed. Any line of code must respect the C++ syntax by terminating with a semi-colon '';'', unless the line is a LSD or C++ command operating on a block of code (like EQUATION(…)or if(…)).

Comments
The code can include comments, that is, text that is ignored by the compiler and serves to describe the code to readers. Comments in C++ come in two forms:

/*
 This is a multiline comment, continuing until
 a sequence "star slash" is encountered
*/

// this is a single line comment,
// terminating at the end of the line

Also, white space/blank characters (like spaces, tabs, new lines etc.) are ignored by the C++ compiler. The modeler is free to use them to organize the spacing and the organization of the code to its preference. Some general style rules, despite not require, are still recommended to keep the code more easily readable (at least by others).

Assignments, arithmetic operations and increments
If a is a (C++) variable, then the programmer can assign a value to it with the command “=”:

a = 4.3;

Any assignment (and most C++ commands) must be terminated with a semi-colon “;”.

It is also possible to assign values from other variables, and use the standard mathematical operations, using the parentheses to group relevant priorities:

a = b + 3 – d / (e + g) * (h + i);

Less obviously, it is also possible to use the same variable on both sides of the assignment:

a = a + 32;

The above line assigns a with its previous value increased of 32. Therefore, if a has the value of, say, 5, after the above line it is assigned the value of 37.

These commands incrementing the value of a variable are so common that they have also a short way to express them. For example, the command a = a + 32; can be expressed also with the command a += 32;, saving the expression of a in the left part of the assignment. This short expression can even be used for the other arithmetical operations:

·         a = a + 32; is equivalent to a += 32;

·         a = a / 32; is equivalent to a /= 32;

·         a = a * 32; is equivalent to a *= 32;

·         a = a - 32; is equivalent to a -= 32;

A C++ peculiar command (++) allows both to increase by 1 a variable and to use it as an assignment. That is, a++ (or ++a) is equivalent to a += 1, which in turn is equivalent to a = a + 1.

The command ++ (and its sister command --) may work differently depending on whether it is used after or before a variable symbol in some situations. If it is used after (a++) the command first assigns the current value, and then increases it by 1. Instead, if ++ is used before a variable (++a), it firstly increases its value, and then assigns the result. For example:

a = 3;
b = a++;
c = ++a;

At the end of the above list of commands, we will see that a equals 5 (3 and two increments in the second and third line), b equals 3 (because b = a++; firstly assigns the value of a to b, and then increments a), and c equals 5, because a is increased from 4 to 5 before being assigned to c.

Note the difference that exists between the equal sign “=” used as assignment (i.e., load the value on the left to the symbol on the right), from the logical condition of identity (are the right and left sides identical?). C++ uses two different symbols for the two operations. In the following paragraph, on the if(…) conditional statement, we will see the symbol used for the logical identity.

Condition: if ... then ... else ...
As said before, lines of code are executed sequentially, but the programmer can conditionally “jump” some blocks of code. A frequently used statement is the conditional execution of different blocks of commands depending on whether a condition is verified as true or not. The grammar of the conditional command in C++ is:

if ( condition )
{
  /*
   insert here any line you want to be executed
   if "condition" is true
  */
}
else
{
  /*
   insert here any line you want to be executed
   if "condition" is false
  */
}

The curly brackets “{” and “}” can be skipped if there is only one line of code (terminated by a “;”) to be executed conditionally.

The “if” condition is normally based on one of the following comparison operators, assuming a and b to be two numerical values or variables containing numerical values:

·         Equal ( a == b ): the condition is true of a equals b

·         Not equal ( a != b ): the condition is true if a is not equal to b

·         Larger than ( a > b ): the condition is true of a is larger than b

·         Larger or equal ( a >= b ): the condition is true of a is larger than or equal to b

·         Less than ( a < b ): the condition is true of a is smaller than b

·         Less or equal ( a <= b ): the condition is true of a is smaller than or equal to b

The condition can be composed with the logical operators connecting the comparisons:

·         Negation ( !( condition ) ): is true if condition is false, and vice versa.

·         Logical “and” ( && ): given two conditions cond1 and cond2, the composed condition ( cond1 && cond2 ) is true only if both conditions are true.

·         Logical “or” ( || ): given two conditions cond1 and cond2, the composed condition ( cond1 || cond2 ) is true if one or both conditions are true.

For example, the following code distinguishes three possible situations:

1.    a equals b and c does not equal d

2.    a equals b and c equals d

3.    a does not equal b (irrespective of c and d)

if (a == b && !(c == d))
{
  /*
   "a" equals "b" and "c" is different from "d"
  */
}
else
{ // if the process is here, than one of the two conditions above is not true
  if (a == b)
  {
     /*
      "c" equals "d"
     */
  }
  else
  {
     /*
      "a" is not equal to "b", but "c" may or may not equal "d"
     */
  }
}

Conditions in C++ are evaluated to integer numbers, where “0” (zero) means false, and any non-zero integer means true.

Cycles: for(…)
A frequently used command allows the repetition of a block of code again and again, until a specific condition is satisfied. The grammar for the cycle keyword for(…) is:

for ( INIT ; CONDITION ; ENDCYCLE )
{
  // Block of code of the cycle
  // here there can be any command
}

followed by the block of code to be repeated contained between curly brackets “{” and “}”. As usual, the curly brackets can be skipped if there is only one line of code (terminated by a “;”) to be repeatedly executed.

When the execution of the code reaches a line with a for(…) instruction, the following operations are executed:

1.    Execute any command contained in INIT; more than one command can be placed here separated by commas ","

2.    Control that CONDITION is true

3.    If CONDITION is not true exit the cycle and continue with the first line after the block of code following the for(…) command

4.    If the CONDITION is true, execute once the block of code

5.    Execute any command contained in ENDCYCLE; more than one command can be placed here separated by commas ","

6.    Restart from operation 2 above.

Example: a cycle for(…) to repeat a block of code 5 times, setting an array variable from -2 to 2:

for ( i = -2; i < 3; i++ )
{
  /*
   in the execution of these lines i
   assumes values -2, -1, 0, 1 and 2
  */
  v[i] = i;
}
// here i equals 3, because it was the condition to exit the cycle