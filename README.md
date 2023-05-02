Download Link: https://assignmentchef.com/product/solved-cop3502-assignment-2-lonely-party-array
<br>
In this programming assignment, you will implement lonely party arrays (arrays that are broken into fragments that get allocated and deallocated on an as-needed basis, rather than being allocated all at once as one big array, in order to eliminate unnecessary memory bloat). This is an immensely powerful and awesome data structure, and it will ameliorate several problems we often encounter with arrays in C (see Section 1 of this PDF).

By completing this assignment, you will gain advanced experience working with dynamic memory management, pointers, and structs in C. You will also learn how to use <em>valgrind</em> to test your programs for memory leaks, and you will gain additional experience managing programs that use custom header files and multiple source files. In the end, you will have an awesome and useful data structure that you can use to solve all sorts of interesting problems.

For a lot of programming tasks that use arrays, it’s not uncommon to allocate an array that is large enough to handle any worst-case scenario you might throw at your program, but which has a lot of unused, wasted memory in most cases.

For example, suppose we’re writing a program that needs to store the frequency distribution of scores on some exam. If we know the minimum possible exam score is zero and the maximum possible exam score is 109 (because there are a few bonus questions), and all possible scores are integers, then we might create an array of length 110 (with indices 0 through 109) to meet our needs.

Suppose, then, that we’re storing data for 25 students who took that exam. If 3 of them earned 109%, 6 students earned 98%, 8 students earned 95%, 5 students earned 92%, 2 students earned 83%, and 1 student earned a 34%, the frequency array for storing their scores would look like this:

0         1..33        34       35..82       83       84..91       92        93        94        95    96          97          98      99..108     109

<strong>↑        ↑                   ↑                    ↑                  ↑      ↑   ↑        ↑                   ↑</strong>

wasted    wastedwasted wasted spacewasted space           wasted space

space                                                                 spacespace

Notice that with only 6 distinct scores in the class, we only use 6 cells to record the frequencies of scores earned for all 25 students. The other 104 cells in the array are just wasted space.

<strong>1.1        Lonely Party Arrays to the Rescue!</strong>

In this assignment, you will implement a new array-like data structure, called a lonely party array (or “LPA”),<a href="#_ftn1" name="_ftnref1"><sup>[1]</sup></a> that will solve the problem described above. In this data structure, instead of allocating one large array, we will allocate smaller array <em>fragments</em> an as-needed basis.

For example, in the application described above, we could split our array into 11 fragments, each of length 10. The first fragment would be used to store the frequencies of scores 0 through 9, the second fragment would store data for scores 10 through 19, and so on, up until the eleventh fragment, which would store data for scores 100 through 109.

The twist here is that we will only create array fragments on an as-needed basis. So, in the example above, the only fragments we would allocate would be the fourth (for scores 30 through 39), ninth (for scores 80 through 89), tenth (for scores 90 through 99), and eleventh (for scores 100 through 109). Each fragment would use 40 bytes (since each one has 10 integers, and an integer in C is typically 4 bytes), meaning we’d be using a total of 160 bytes for those 4 fragments. Compare this to the 4 * 110 = 440 bytes occupied by the original array of length 110, and you can see how this new data structure allows us to save memory. In this case, the LPA would reduce our memory footprint by over 63%.<a href="#_ftn2" name="_ftnref2"><sup>[2]</sup></a>There are a few other twists with this <em>LonelyPartyArray</em> data structure. If we have a bunch of small arrays (called “fragments” in this assignment), we need to keep track of their addresses. So, we’ll create an array of integer pointers to store all the base addresses of those arrays. If a fragment hasn’t been allocated, we’ll just store a NULL pointer in place of the address for that fragment.

We’ll also store how many cells in each fragment are occupied. In the example above, fragment 3 only has one occupied cell (since 34% is the only score that anyone earned in the range 30 through 39). We’ll keep track of that so that if we delete values from the LPA and get to a point where there are zero occupied cells in a particular fragment, we can free all the memory associated with that fragment.

The following diagram shows a complete <em>LonelyPartyArray</em> struct, with all its constituent members, for the example described above. There are 11 fragments possible, each of length 10, with 4 currently active. Additional details about the members of this struct are included in the pages that follow.

<table width="600">

 <tbody>

  <tr>

   <td width="209"><strong>LonelyPartyArray</strong> *party</td>

   <td width="239">0x5298</td>

   <td width="151">(party-&gt;fragments[3])</td>

  </tr>

 </tbody>

</table>

0x64282

<table width="606">

 <tbody>

  <tr>

   <td width="228">0x64282         (*party)</td>

   <td width="53">0</td>

   <td width="38">1</td>

   <td width="38">2</td>

   <td width="38">3</td>

   <td width="38">4</td>

   <td width="32">5</td>

   <td width="44">6</td>

   <td width="38">7</td>

   <td width="38">8</td>

   <td width="18">9</td>

  </tr>

 </tbody>

</table>

<table width="173">

 <tbody>

  <tr>

   <td width="173"><strong>int </strong>size

    <table width="149">

     <tbody>

      <tr>

       <td width="149">6</td>

      </tr>

     </tbody>

    </table><strong>int </strong>num_fragments

    <table width="149">

     <tbody>

      <tr>

       <td width="149">11</td>

      </tr>

     </tbody>

    </table><strong>int </strong>fragment_length

    <table width="149">

     <tbody>

      <tr>

       <td width="149">10</td>

      </tr>

     </tbody>

    </table><strong>int </strong>num_active_fragments

    <table width="149">

     <tbody>

      <tr>

       <td width="149">4</td>

      </tr>

     </tbody>

    </table><strong>int </strong>**fragments

    <table width="149">

     <tbody>

      <tr>

       <td width="149">0x88402</td>

      </tr>

     </tbody>

    </table><strong>int </strong>*fragment_sizes

    <table width="149">

     <tbody>

      <tr>

       <td width="149">0x48664</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

 </tbody>

</table>

0x5432             (party-&gt;fragments[8])

0          1         2          3         4                                                                                     5          6         7          8         9

0x5664             (party-&gt;fragments[9])

0          1         2          3         4                                                                                     5          6         7          8         9

0x5708           (party-&gt;fragments[10])

0          1         2          3         4                                                                                     5          6         7          8         9

^

<strong>Note:</strong> To access this cell, you would use party-&gt;fragments[10][0].

<table width="626">

 <tbody>

  <tr>

   <td width="461">0x88402</td>

   <td width="166">(party-&gt;fragments)</td>

  </tr>

 </tbody>

</table>

0               1               2               3               4               5               6               7            8               9              10

0x48664                                                                                                      (party-&gt;fragment_sizes)

0               1               2               3               4               5               6               7            8               9              10

As you can see from the diagram above, there are a few other things to keep track of, but this section gives the basic idea behind the lonely party array. All the juicy details about everything else you need to keep track of are given below in Section 3, “Function Requirements.”

<h2>1.2        Advantages of Lonely Party Arrays</h2>

As with normal arrays in C, we will have fast, direct access to any index of a lonely party array at any given time. This data structure has three main advantages over C’s traditional arrays:

<ol>

 <li>As we saw above, normal arrays often have wasted space. We will only allocate fragments of our lonely party arrays on an as-needed basis, and deallocate them when they’re no longer in use, which will reduce the amount of unused memory being wasted.</li>

 <li>We will use <em>get()</em>, <em>set()</em>, and <em>delete()</em> functions to access and modify individual elements of the lonely party array, and these functions will help us avoid segfaults by first checking that we aren’t accessing array indices that are out of bounds. (Recall that C doesn’t check whether an array index is out of bounds before accessing it during program execution. That can lead to all kinds of wacky trouble!)</li>

 <li>In C, if we have to pass an array to a function, we also typically find ourselves passing its length to that function as a second parameter. With lonely party arrays, all the information you need about the data structure will get passed automatically with the array fragments themselves, as everything will be packaged together in a struct.</li>

</ol>

<h2>1.3       Overview of What You’ll Submit</h2>

The lonely party arrays you implement for this assignment will be designed to hold integers. The precise number of fragments – and the lengths of those fragments – will be allowed to vary from one LPA to the next. A complete list of the functions you must implement, including their functional prototypes, is given below in Section 3, “Function Requirements.”

You will submit a single source file, named <em>LonelyPartyArray.c</em>, that contains all required function definitions, as well as any auxiliary functions you deem necessary. In <em>LonelyPartyArray.c</em>, you should #include any header files necessary for your functions to work, including <em>LonelyPartyArray.h</em> (see Section 2, “LonelyPartyArray.h”).

<strong>Note that you will <u>not</u> write a main() function in the source file you submit!</strong> Rather, we will compile your source file with our own <em>main()</em> function(s) in order to test your code. We have attached example source files that have main() functions, which you can use to test your code. You should also write your own <em>main()</em> functions for testing purposes, but your code must not have a <em>main()</em> function when you submit it. We realize this is still fairly new territory for most of you, so don’t panic. We’ve included instructions on compiling multiple source files into a single executable (e.g., mixing your <em>LonelyPartyArray.c</em> with our <em>LonelyPartyArray.h</em> and <em>testcase01.c</em> files) in Section 5 (pg. 16).

Although we have included sample main() functions to get you started with testing the functionality of your code, we encourage you to develop your own test cases, as well. Ours are by no means comprehensive. We will use much more elaborate test cases when grading your submission.

<em>Start early. Work hard. Good luck!</em>

<h1>2. LonelyPartyArray.h</h1>

This header file contains the struct definition and functional prototypes for the lonely party array functions you will be implementing. You <strong><u>must</u></strong> #include this file from <em>LonelyPartyArray.c</em>, as follows. Recall that the “quotes” (as opposed to &lt;brackets&gt;) indicate to the compiler that this header file is found in the same directory as your source, not a system directory:




#include “LonelyPartyArray.h”

Please do not modify <em>LonelyPartyArray.h</em> in any way, and do not send <em>LonelyPartyArray.h</em> when you submit your assignment. We will use our own copy of <em>LonelyPartyArray.h</em> when compiling your program.

If you write auxiliary functions (“helper functions”) in <em>LonelyPartyArray.c</em> (which is strongly encouraged!), you should <strong><u>not</u></strong> add those functional prototypes to <em>LonelyPartyArray.h</em>. Our test case programs will not call your helper functions directly, so they do not need any awareness of the fact that your helper functions even exist. (We only list functional prototypes in a header file if we want multiple source files to be able to call those functions.) So, just put the functional prototypes for any helper functions you write at the top of your <em>LonelyPartyArray.c</em> file.

<strong>Think of <em>LonelyPartyArray.h</em> as a bridge between source files.</strong> It contains struct definitions and functional prototypes for functions that might be defined in one source file (such as your <em>LonelyPartyArray.c</em> file) and called from a different source file (such as <em>testcase01.c</em>).

The basic struct you will use for this data structure (defined in the header file) is as follows:







typedef struct LonelyPartyArray

{

int size;                  // number of occupied cells across all fragments    int num_fragments;         // number of fragments (arrays) in this struct    int fragment_length;       // number of cells per fragment    int num_active_fragments;  // number of allocated (non-NULL) fragments    int **fragments;           // array of pointers to individual fragments    int *fragment_sizes;       // stores number of used cells in each fragment } LonelyPartyArray;







The <em>LonelyPartyArray</em> struct contains an <em>int**</em> pointer that can be used to set up a 2D <em>int</em> array (which is just an array of <em>int</em> arrays). That <em>fragments</em> array will have to be allocated dynamically whenever you create a new LPA struct. The <em>size</em> member of this struct tells us how many elements currently reside in the lonely party array (i.e., how many <em>int</em> cells are actually being used across all the fragments and are not just wasted space). The <em>fragment_length</em> member tells us how many integer cells there should be in each individual fragment that we allocate. (All the non-NULL fragments within a given LPA struct will always be the same length.) <em>num_active_fragments</em> tells us how many non-NULL fragments this lonely party array is using. The <em>fragment_sizes</em> array is used to keep track of how many cells are actually being used in each fragment. This count allows us to determine very quickly whether we can deallocate a fragment any time we delete one of the elements it contains.

This header file also contains definitions for UNUSED, LPA_SUCCESS and LPA_FAILURE, which you will use in some of the required functions described below.

<h1>3. Function Requirements</h1>

In the source file you submit, <em>LonelyPartyArray.c</em>, you must implement the following functions. You may implement any auxiliary functions you need to make these work, as well. Please be sure the spelling, capitalization, and return types of your functions match these prototypes exactly. In this section, I often refer to <em>malloc()</em>, but you’re welcome to use <em>calloc()</em> or <em>realloc()</em> instead, if you’re familiar with those functions.

LonelyPartyArray *createLonelyPartyArray(int num_fragments, int fragment_length);

<strong>Description:</strong> This function will create a <em>LonelyPartyArray</em> (LPA) that can accommodate up to <em>num_fragments</em> different array fragments, each of length <em>fragment_length</em>. So, the maximum number of integers this LPA can hold is <em>num_fragments </em>x<em> fragment_length</em>.

Start by dynamically allocating space for a new <em>LonelyPartyArray</em> struct. Initialize the struct’s <em>num_fragments</em> and <em>fragment_length</em> members using the arguments passed to this function. Initialize <em>num_active_fragments</em> and <em>size</em> to the appropriate values. Dynamically allocate the <em>fragments</em> array to be an array of <em>num_fragments</em> integer pointers, and initialize all those pointers to NULL. (Eventually, that array will store pointers to any individual array fragments you allocate when adding elements to this data structure.) Dynamically allocate the <em>fragment_sizes</em> array, and initialize all values to zero (since none of the fragments currently hold any elements).

If one or both of the parameters passed to this function are less than or equal to zero (i.e., if they are not both positive integers), you should immediately return NULL. If any of your calls to <em>malloc()</em> fail, you should free any memory you have dynamically allocated in this function call up until that point (to avoid memory leaks), and then return NULL.

<strong>Output:</strong> If the function successfully creates a new LPA, print the following: “-&gt; A new

LonelyPartyArray has emerged from the void. (capacity: &lt;M&gt;, fragments: &lt;N&gt;)”

Do not print the quotes, and do not print &lt;brackets&gt; around the variables. Terminate the line with a newline character, ‘
’. &lt;M&gt; is the maximum number of integers this LPA can contain (<em>num_fragments </em>x<em> fragment_length</em>). &lt;N&gt; is simply <em>num_fragments</em>. Note that in testing, we will never create a lonely party array whose capacity would be too large to store in a 32-bit int.

<strong>Returns:</strong> A pointer to the new LonelyPartyArray, or NULL if any calls to <em>malloc()</em> failed.

LonelyPartyArray *destroyLonelyPartyArray(LonelyPartyArray *party);

<strong>Description:</strong> Free all dynamically allocated memory associated with this LonelyPartyArray struct, and return NULL. Be careful to avoid segfaulting in the event that <em>party</em> is NULL.

<strong>Output:</strong> “-&gt; The LonelyPartyArray has returned to the void.” (Output should not include the quotes. Terminate the line with a newline character, ‘
’.) If <em>party</em> is NULL, this function should not print anything to the screen.

<strong>Returns:</strong> This function should always return NULL.

int set(LonelyPartyArray *party, int index, int key);

<strong>Description:</strong> Insert <em>key</em> into the appropriate index of the lonely party array. Based on the <em>index</em> parameter passed to this function, as well as the number of fragments in the LPA and the length of each fragment, you will have to determine which fragment <em>index</em> maps to, and precisely which cell in that array fragment is being sought. For example, if <em>index</em> is 14 and the LPA has <em>num_fragments = 3</em> and <em>fragment_length = 12</em>, then <em>key</em> would go into the second array fragment (<em>fragments[1]</em>) at index 2 (<em>fragments[1][2]</em>), since the first fragment (<em>fragments[0]</em>) would be used for indices 0 through 11, the second fragment would be used for indices 12 through 23, and the third fragment would be used for indices 24 through 35, for a total of 36 cells across the 3 fragments. For additional indexing examples, please refer to the test cases included with this assignment.

If the fragment where we need to insert key is NULL, then you should dynamically allocate space for that fragment (an array of <em>fragment_length</em> integers), initialize each cell in that new fragment to UNUSED (which is #defined in <em>LonelyPartyArray.h</em>), update the struct’s <em>num_active_fragments</em> member, and store key at the appropriate index in the new allocated array.

If the index being modified was previously empty (i.e., UNUSED), and/or if the fragment was not yet allocated, then after inserting key into the lonely party array, be sure to increment the struct’s <em>size</em> member so that it always has an accurate count of the number of cells in the LPA that are currently occupied, and be sure to increment the appropriate value in the struct’s <em>fragment_size</em> array so that it always has an accurate count of the number of cells currently being used in each particular fragment.

If index is invalid (see note below), or if a NULL party pointer is passed to this function, simply return LPA_FAILURE without attempting to modify the data structure in any way and without causing any segmentation faults.

<strong>Note on “invalid index” values:</strong> In our <em>set()</em>, <em>get()</em>, and <em>delete()</em> functions, <em>index</em> is considered valid if it falls in the range 0 through (<em>num_fragments</em> x <em>fragment_length</em> – 1), even if <em>index</em> refers to a cell marked as UNUSED or a fragment that has not been allocated yet. An “invalid index” is one that falls outside that specified range.

<strong>Output:</strong> There are three cases that should generate output for this function. (Do not print the quotes, and do not print &lt;brackets&gt; around the variables. Terminate any output with ‘
’.) •       If calling this function results in the allocation of a new fragment in memory, print:

“-&gt; Spawned fragment &lt;P&gt;. (capacity: &lt;Q&gt;, indices: &lt;R&gt;..&lt;S&gt;)”  &lt;P&gt; is the index where the newly allocated fragment’s address is stored in the struct’s fragments array, and &lt;Q&gt; is the length of the new array (i.e., the number of integers it can hold), and &lt;R&gt; and &lt;S&gt; correspond to the lowest and highest <em>index</em> values that would map to this particular fragment.

<ul>

 <li>If <em>party</em> is non-NULL and <em>index</em> is invalid (see definition of “invalid index” above), print: “-&gt; Bloop! Invalid access in set(). (index: &lt;L&gt;, fragment: &lt;M&gt;,</li>

</ul>

offset: &lt;N&gt;)”  &lt;L&gt; is value of <em>index</em> passed to this function; &lt;M&gt; is the invalid <em>fragments</em> array index that index would have mapped to, had that fragment been within bounds for this lonely party array; and &lt;N&gt; is the precise index within that fragment that <em>index</em> would have mapped to. For example, if <em>num_fragments = 2</em>, <em>fragment_length = 10</em>, and <em>index = 21</em>, then the valid indices for this LPA would be 0 through 19, and <em>index</em>, being invalid, would produce this error message with &lt;M&gt; = 2 and &lt;N&gt; = 1. See  <em>testcase16.c</em> and <em>output16.txt</em> for examples of how to handle output for negative <em>index</em> values. For additional examples, please refer to the test cases included with this assignment.

<ul>

 <li>If <em>party</em> is NULL, print: “-&gt; Bloop! NULL pointer detected in set().” Output should not contain any quotes or angled brackets. Terminate the line with a newline character, ‘
’.</li>

</ul>

<strong>Returns:</strong> If this operation is successful, return LPA_SUCCESS. If the operation is unsuccessful (either because <em>index</em> is invalid or the function receives a NULL <em>party</em> pointer, or because any calls to <em>malloc()</em> fail), return LPA_FAILURE. LPA_SUCCESS and LPA_FAILURE are #defined in <em>LonelyPartyArray.h</em>.

int get(LonelyPartyArray *party, int index);

<strong>Description:</strong> Retrieve the value stored at the corresponding index of the lonely party array. As with the <em>set()</em> function, based on the <em>index</em> parameter passed to this function, as well as the number of fragments in the LPA and the length of each fragment, you will have to determine which fragment <em>index</em> maps to, and precisely which cell in that array fragment is being sought.

Keep in mind that <em>index</em> could try taking you to a fragment that has not yet been allocated. It’s up to you to avoid going out of bounds in an array and/or causing any segmentation faults.

<strong>Output:</strong> There are two cases that should generate output for this function. (Do not print the quotes, and do not print &lt;brackets&gt; around the variables. Terminate any output with ‘
’.)

<ul>

 <li>If <em>index</em> is invalid (see definition of “invalid index” in the <em>set()</em> function description) and <em>party</em> is non-NULL, print: “-&gt; Bloop! Invalid access in get(). (index: &lt;L&gt;, fragment: &lt;M&gt;, offset: &lt;N&gt;)” For explanations of &lt;L&gt;, &lt;M&gt;, and &lt;N&gt;, see the <em>set()</em> function description above. Note that if a cell is marked as UNUSED, but <em>index</em> is within range, you should not generate this error message.</li>

 <li>If <em>party</em> is NULL, print: “-&gt; Bloop! NULL pointer detected in get().”</li>

</ul>

<strong>Returns:</strong> If <em>index</em> is valid (see definition of “invalid index” in the <em>set()</em> function description), return the value stored at the appropriate index in the lonely party array (even if it is marked as UNUSED). If <em>index</em> is technically valid (according to the definition in the <em>set()</em> function description) but refers to a cell in an unallocated fragment, return UNUSED. If the operation is unsuccessful (either because <em>index</em> is invalid or because the function receives a NULL <em>party</em> pointer), return LPA_FAILURE.

<em>Continued on the following page…</em>

int delete(LonelyPartyArray *party, int index);

<strong>Description:</strong> Set the value stored at the corresponding index of the lonely party array to UNUSED. As with the <em>set()</em> and <em>get()</em> functions, based on the index parameter passed to this function, as well as the number of fragments in the LPA and the length of each fragment, you will have to determine which fragment <em>index</em> maps to, and precisely which cell in that array fragment is being sought.

If the cell being sought is <em>not</em> already set to UNUSED, then after writing UNUSED to that cell, decrement the struct’s <em>size</em> member so that it always has an accurate count of the total number of cells that are currently being used, and decrement the appropriate value in the struct’s <em>fragment_size</em> array so that it always has an accurate count of the number of cells currently being used in each fragment.

If deleting the value at this index causes the fragment containing that value to become empty (i.e., all of its cells are now UNUSED), deallocate that array, set the appropriate pointer in the struct’s <em>fragments</em> array to NULL, and update the struct’s <em>num_active_fragments</em> member. You should never loop through a fragment to see if all of its cells are unused. Instead, you should rely on the <em>fragment_sizes</em> array to keep track of whether or not a fragment is ready for deallocation.

Keep in mind that <em>index</em> could try taking you to a fragment that has not yet been allocated. It’s up to you to avoid going out of bounds in an array and/or causing any segmentation faults.

<strong>Output:</strong> There are three cases that should generate output for this function. (Do not print the quotes, and do not print &lt;brackets&gt; around the variables. Terminate any output with ‘
’.) •       If calling this function results in the deallocation of a fragment in memory, print:

“-&gt; Deallocated fragment &lt;P&gt;. (capacity: &lt;Q&gt;, indices: &lt;R&gt;..&lt;S&gt;)”  For explanations of &lt;P&gt;, &lt;Q&gt;, &lt;R&gt;, and &lt;S&gt;, see the <em>set()</em> function description above.

<ul>

 <li>If <em>index</em> is invalid (see definition of “invalid index” in the set() function description) and <em>party</em> is non-NULL, print: “-&gt; Bloop! Invalid access in delete(). (index:</li>

</ul>

&lt;L&gt;, fragment: &lt;M&gt;, offset: &lt;N&gt;)”  For explanations of &lt;L&gt;, &lt;M&gt;, and &lt;N&gt;, see the <em>set()</em> function description above. Note that if a cell is marked as UNUSED, but <em>index</em> is within range, you should not generate this error message.

<ul>

 <li>If <em>party</em> is NULL, print: “-&gt; Bloop! NULL pointer detected in delete().”</li>

</ul>

<strong>Returns:</strong> Return LPA_FAILURE if this operation refers to an invalid index (as defined in the <em>set()</em> function description), if this function receives a NULL <em>party</em> pointer, if this operation refers to a cell that lies within an unallocated fragment, or if this operation refers to a cell whose value was already set to UNUSED when the function was called. Otherwise, return LPA_SUCCESS.

int containsKey(LonelyPartyArray *party, int key);

<strong>Description:</strong> Perform a linear search through the entire lonely party array to determine whether it contains <em>key</em> (i.e., whether <em>key</em> is stored in any of the cells in the LPA). Be careful to avoid segmentation faults.

<strong>Output:</strong> This function should not print anything to the screen.

<strong>Returns:</strong> Return 1 if the LPA contains <em>key</em>. Otherwise, return 0. If <em>party</em> is NULL, return 0.

int isSet(LonelyPartyArray *party, int index);

<strong>Description:</strong> Determine whether there is a value (other than UNUSED) being stored at the corresponding index of the lonely party array. Be careful to avoid segmentation faults.

<strong>Output:</strong> This function should not print anything to the screen.

<strong>Returns:</strong> If <em>index</em> is invalid (according to the definition of “invalid index” given in the <em>set()</em> function description), or if it refers to a cell marked as UNUSED or a fragment that has not been allocated yet, or if <em>party</em> is NULL, return 0. Otherwise, return 1.

int printIfValid(LonelyPartyArray *party, int index);

<strong>Description:</strong> Print the value stored at the corresponding index of the lonely party array. As with the <em>set()</em>, <em>get()</em>, and <em>delete()</em> functions, based on the <em>index</em> parameter passed to this function, as well as the number of fragments in the LPA and the length of each fragment, you will have to determine which fragment <em>index</em> maps to, and precisely which cell in that array fragment is being sought.

<strong>Output:</strong> Simply print the appropriate integer to the screen, followed by a newline character, ‘
’. This function should not print anything if <em>index</em> is invalid (as defined in the <em>set()</em> function description), if <em>index</em> refers to a cell whose value is set to UNUSED, or if <em>party</em> is NULL.

<strong>Returns:</strong> Return LPA_SUCCESS if this function prints a value to the screen. Otherwise, return LPA_FAILURE.

LonelyPartyArray *resetLonelyPartyArray(LonelyPartyArray *party);

<strong>Description:</strong> Reset the lonely party array to the state it was in just after it was created with <em>createLonelyPartyArray()</em>. Be sure to avoid any memory leaks or segmentation faults.

You will need to deallocate any array fragments that are currently active within <em>party</em>. You will also need to reset all the values in the struct’s <em>fragments</em> and <em>fragment_sizes</em> arrays. However, you should not re-allocate the <em>fragments</em> or <em>fragment_sizes</em> arrays; simply reset the values contained in those already-existing arrays.

You will also need to reset the struct’s <em>size</em> and <em>num_active_fragments</em> members. You should not, however, change the values of <em>num_fragments</em> or <em>fragment_length</em>.

<strong>Output:</strong> There are two cases that should generate output for this function. (Do not print the

quotes, and do not print &lt;brackets&gt; around the variables. Terminate any output with ‘
’.)

<ul>

 <li>If <em>party</em> is non-NULL, print: “-&gt; The LonelyPartyArray has returned to its nascent state. (capacity: &lt;M&gt;, fragments: &lt;N&gt;)” Here, &lt;M&gt; is the maximum number of integers the lonely party array can hold, and &lt;N&gt; is the value of the struct’s <em>num_fragments</em></li>

 <li>If <em>party</em> is NULL, be sure to avoid segmentation faults, and simply return from the function after printing the following: “-&gt; Bloop! NULL pointer detected in resetLonelyPartyArray().”</li>

</ul>

<strong>Returns:</strong> This function should always return <em>party</em>.

int getSize(LonelyPartyArray *party);

<strong>Description:</strong> This function simply returns the number of elements currently in the LPA (<strong><u>not</u></strong> including any elements marked as UNUSED). This should be a near-instantaneous function call; it should not loop through the cells in the LPA at all.

We provide this function to discourage other programmers from ever accessing <em>party→size</em> directly if they try to compile code that uses our fancy LPA data structure. That way, if we release this data structure to the public but then end up updating it a few months later to rename the <em>size</em> member of the <em>LonelyPartyArray</em> struct to something else, the programmers who have been using our code and end up downloading the latest version can get it working right out of the box; they don’t have to go through their own code and change all instances of <em>party→size</em> to something else, as long as we still provide them with a <em>getSize()</em> function that works as intended.<a href="#_ftn3" name="_ftnref3"><sup>[3]</sup></a>

<strong>Output:</strong> This function should not print anything to the screen.

<strong>Returns:</strong> Return the number of elements currently in the LPA, or -1 if <em>party</em> is NULL.

int getCapacity(LonelyPartyArray *party);

<strong>Description:</strong> This function should simply return the maximum number of elements that party can hold, based on the values of its <em>num_fragments</em> and <em>fragment_length</em> members. For example, for the lonely party array shown on pg. 4 of this PDF, <em>getCapacity()</em> would return 110, because when all of its array fragments are allocated, it will be able to hold up to 110 integer elements.

Note that in testing, we will never create a lonely party array whose capacity would be too large to store in a 32-bit <em>int</em>, so you don’t need to worry about type casting when calculating this return value.

<strong>Output:</strong> This function should not print anything to the screen.

<strong>Returns:</strong> Return the capacity of the LPA (as just described), or -1 if <em>party</em> is NULL.

int getAllocatedCellCount(LonelyPartyArray *party);

<strong>Description:</strong> This function should return the maximum number of elements that <em>party</em> can hold without allocating any new array fragments. For example, for the lonely party array shown on pg. 4 of this PDF, <em>getAllocatedCellCount()</em> would return 40, because the non-NULL fragments are able to hold up to 40 integer elements in total.

<strong>Output:</strong> This function should not print anything to the screen.

<strong>Returns:</strong> Return the number of allocated integer cells (as just described), or -1 if the <em>party</em> pointer is NULL.

long long unsigned int getArraySizeInBytes(LonelyPartyArray *party);

<strong>Description:</strong> This function should return the number of bytes that would be used if we were using a standard array rather than a <em>LonelyPartyArray</em> struct. For example, for the LPA struct shown on pg. 4 of this PDF, a traditional array representation would have 110 integer cells, which would occupy 110 x<em> sizeof(int)</em> = 440 bytes, and so this function should return 440 for that struct. For additional examples, please refer to the test cases included with this assignment.

<strong>Note:</strong> This number could get quite large, and so as you perform the arithmetic here, you should cast to <em>long long unsigned int</em>. (For details, see Appendix B on pg. 23.)

<strong>Note:</strong> You should use <em>sizeof()</em> in this function (rather than hard-coding the size of an integer as 4 bytes), and cast the <em>sizeof()</em> values to <em>long long unsigned int</em> as appropriate.

<strong>Note:</strong> If your system does not use 32-bit integers for some reason, using <em>sizeof()</em> in this function could cause output mismatches on some test cases. For this function to work, you will need to ensure that you’re testing on a system that uses 32-bit integers (which you almost certainly are). To check that, you can compile and run <em>SanityCheck.c</em> (included with this assignment), or simply run the <em>test-all.sh</em> script.

<strong>Output:</strong> This function should not print anything to the screen.

<strong>Returns:</strong> The number of bytes (as just described), or 0 if the <em>party</em> pointer is NULL.

long long unsigned int getCurrentSizeInBytes(LonelyPartyArray *party);

<strong>Description:</strong> This function should return the number of bytes currently taken up in memory by the LPA. You will need to account for all of the following:

<ul>

 <li>The number of bytes taken up by the LPA pointer itself: <em>sizeof(LPA*)</em></li>

 <li>The number of bytes taken up by the LPA struct (which is just the number of bytes taken up by the four integers and two pointers within the struct): <em>sizeof(LPA)</em></li>

 <li>The number of bytes taken up by the <em>fragments</em> array (i.e., the number of bytes taken up by the pointers in that array).</li>

 <li>The number of bytes taken up by the <em>fragment_sizes</em> array (i.e., the number of bytes taken up by the integers in that array).</li>

 <li>The number of bytes taken up by the active fragments (i.e., the number of bytes taken up by all the integer cells in the individual array fragments).</li>

</ul>

The <em>getArraySizeInBytes()</em> and <em>getCurrentSizeInBytes()</em> functions will let you see, concretely, the amount of memory saved by using a lonely party array instead of a traditional C-style array.<a href="#_ftn4" name="_ftnref4"><sup>[4]</sup></a>

<strong>Note:</strong> This number could get quite large, and so as you perform the arithmetic here, you should cast to <em>long long unsigned int</em>. (For details, see Appendix B on pg. 23.)

<strong>Note:</strong> You should use <em>sizeof()</em> in this function (rather than hard-coding the sizes of any data types), and cast the <em>sizeof()</em> values to <em>long long unsigned int</em> as appropriate.

<strong>Output:</strong> This function should not print anything to the screen.

<strong>Returns:</strong> The number of bytes (as just described), or 0 if the <em>party</em> pointer is NULL.

double difficultyRating(void);

<strong>Note:</strong> You do not need to call this function anywhere in your code. I will call this function myself to gather data on how difficult students found this assignment.

<strong>Output:</strong> This function should not print anything to the screen.

<strong>Returns:</strong> A double indicating how difficult you found this assignment on a scale of 1.0 (ridiculously easy) through 5.0 (insanely difficult).

double hoursSpent(void);

<strong>Note:</strong> You do not need to call this function anywhere in your code. I will call this function myself to gather data on how much time students estimate spending on this assignment.

<strong>Output:</strong> This function should not print anything to the screen.

<strong>Returns:</strong> A reasonable estimate (greater than zero) of the number of hours you spent on this assignment.

<h2>3.1      Bonus Functions</h2>

This function is optional. Any credit awarded for this function will be given as bonus points.

LonelyPartyArray *cloneLonelyPartyArray(LonelyPartyArray *party);

<strong>Description:</strong> Dynamically allocate a new <em>LonelyPartyArray</em> struct and set it up to be a clone of <em>party</em>. The clone should have entirely new, separate copies of all the data contained within <em>party</em>. (For example, the clone should not simply refer to <em>party</em>’s fragments. Instead, it should have entirely new copies of those fragments.)

If any calls to <em>malloc()</em> fail, free any memory that this function dynamically allocated up until that point, and then return NULL.

<strong>Output:</strong> If <em>party</em> is non-NULL, print the following: “-&gt; Successfully cloned the LonelyPartyArray. (capacity: &lt;M&gt;, fragments: &lt;N&gt;)”  This output should not include the quotes or angled brackets. Terminate the line with a newline character, ‘
’. &lt;M&gt; is the maximum number of integers this LPA can contain (<em>num_fragments </em>x<em> fragment_length</em>). &lt;N&gt; is simply <em>num_fragments</em>. Note that in testing, we will never create a lonely party array whose capacity would be too large to store in a 32-bit <em>int</em>. If <em>party</em> is NULL, or if any calls to <em>malloc()</em> fail, this function should not print anything to the screen.

<strong>Returns:</strong> If <em>party</em> is NULL, or if any calls to <em>malloc()</em> fail, simply return NULL. Otherwise, return a pointer to the newly allocated lonely party array.

<h1>4. Running All Test Cases on Eustis (<em>test-all.sh</em>)</h1>

The test cases included with this assignment are designed to show you some ways in which we might test your code and to shed light on the expected functionality of your code. We’ve also included a script, <em>test-all.sh</em>, that will compile and run all test cases for you.

<table width="646">

 <tbody>

  <tr>

   <td colspan="2" width="646"><strong>Super Important:</strong> Using the <em>test-all.sh</em> script to test your code on Eustis is the safest, most sure-fire</td>

  </tr>

  <tr>

   <td width="431">way to make sure your code is working properly before submitting.</td>

   <td width="215"></td>

  </tr>

 </tbody>

</table>

You can run the script on Eustis by placing it in a directory with <em>LonelyPartyArray.c</em>,

<em>LonelyPartyArray.h</em>, the <em>sample_output</em> directory, and all the test case files, and then typing:

bash test-all.sh

Transferring all your files to Eustis with MobaXTerm isn’t too hard, but if you want to transfer them from a Linux or Mac command line, here’s how you do it:

<ol>

 <li>At your command line, use <em>cd</em> to go to the folder that contains all your files for this project (<em>c</em>, <em>LonelyPartyArray.h</em>, the test case files, and the <em>sample_output</em> folder).</li>

 <li>Type the following command (replacing “YOUR_NID” with your actual NID):</li>

</ol>

scp -r . <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="2a73657f787564636e6a4f5f595e4359044f4f4959045f494c044f4e5f">[email protected]</a>:~

<strong><em>Warning! </em></strong>Note that the dot (“.”) refers to your current directory when you’re at the command line in Linux or Mac OS. This command transfers the <em><u>entire contents</u></em> of your current directory to Eustis. That will include any subdirectories, so for the love of all that is good, please don’t run that command from your desktop folder if you have a ton of files on your desktop!

<strong><em>Important Note:</em></strong> When grading your programs, we will use different test cases from the ones we’ve release with this assignment, to ensure that no one can game the system and earn credit by simply hard-coding the expected output for the test cases we’ve released to you. You should create additional test cases of your own in order to thoroughly test your code. In creating your own test cases, you should always ask yourself, “How could these functions be called in ways that don’t violate the function descriptions, but which haven’t already been covered in the test cases included with the assignment?”

<h1>5. Running the Provided Test Cases Individually</h1>

If the <em>test-all.sh</em> script tells you that one of your test cases is failing, you’ll want to compile and run that test case individually to examine its output. Here’s how to do that:

To compile your source file with one of our test cases (such as <em>testcase01.c</em>) at the command line:

gcc LonelyPartyArray.c testcase01.c

By default, this will produce an executable file called <em>a.out</em>, which you can run by typing:

./a.out

If you want to name the executable file something else, use:

gcc LonelyPartyArray.c testcase01.c -o LPA.exe

…and then run the program using:

./LPA.exe

Running the program could potentially dump a lot of output to the screen. If you want to redirect your output to a text file in Linux, it’s easy. Just run the program using the following command, which will create a file called <em>whatever.txt</em> that contains the output from your program:

./LPA.exe &gt; whatever.txt

Linux has a helpful command called <em>diff</em> for comparing the contents of two files, which is really helpful here since we’ve provided several sample output files. You can see whether your output matches ours exactly by typing, e.g.:

diff whatever.txt sample_output/output01.txt

If the files differ, <em>diff</em> will spit out some information about the lines that aren’t the same. For example:

<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="710214101f020b31140402051802">[email protected]</a>:~$ diff whatever.txt output01.txt

1c1

&lt; fail whale &#x1f641;

—

&gt; Hooray! <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="483b2d29263b32082d3d3b3c213b">[email protected]</a>:~$ _

If the contents of <em>whatever.txt</em> and <em>output01.txt</em> are exactly the same, <em>diff</em> won’t have any output:

<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="4635232728353c06233335322f35">[email protected]</a>:~$ diff whatever.txt output01.txt <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="e59680848b969fa5809096918c96">[email protected]</a>:~$ _

<h1>6. Testing for Memory Leaks with Valgrind</h1>

Part of the credit for this assignment will be awarded based on your ability to implement the program without any memory leaks. To test for memory leaks, you can use a program called <em>valgrind</em>, which is installed on Eustis.

Valgrind will <strong><u>not</u></strong> guarantee that your code is completely free of memory leaks. It will only detect whether any memory leaks occur when you run your program. So, if you have a function called <em>foo()</em> that has a nasty memory leak, but you run your program in such a way that <em>foo()</em> never gets called, Valgrind won’t be able to find that potential memory leak.

The <em>test-all.sh</em> script will automatically run your program through all test cases and use <em>valgrind</em> to check whether any of them result in memory leaks. If you want to run <em>valgrind</em> manually, simply compile your program with the <em>-g</em> flag, and then run it through <em>valgrind</em>, like so:

gcc LonelyPartyArray.c testcase01.c -g valgrind –leak-check=yes ./a.out

In the output of <em>valgrind</em>, the magic phrase you’re looking for to indicate that no memory leaks were detected is:

All heap blocks were freed – no leaks are possible

For more information about <em>valgrind</em>’s output, see: <a href="http://valgrind.org/docs/manual/quick-start.html">http://valgrind.org/docs/manual/quick-start.html</a>

<h1>7. Style Restrictions (<em>Super Important!</em>)</h1>

Please conform as closely as possible to the style I use while coding in class. To encourage everyone to develop a commitment to writing consistent and readable code, the following restrictions will be strictly enforced:

 Any time you open a curly brace, that curly brace should start on a new line.

 Any time you open a new code block, indent all the code within that code block one level deeper than you were already indenting.

 Please avoid block-style comments: /*<em> comment </em>*/

 Instead, please use inline-style comments: //<em> comment</em>

 Always include a space after the “//” in your comments: “// <em>comment</em>” instead of “//<em>comment</em>”

 The header comments introducing your source file should always be placed above your #include statements.

 Comments longer than three words should always be placed <em><u>above</u></em> the lines of code to which they refer. Furthermore, such comments should be indented to properly align with the code to which they refer. For example, if line 16 of your code is indented with two tabs, and line 15 contains a comment referring to line 16, then line 15 should also be intended with two tabs.

 Any libraries you <em>#include</em> should be listed <em>after</em> the header comment at the top of your file that includes your name, course number, semester, NID, and so on.

 Please do not write excessively long lines of code. (Prefer fewer than 100 characters wide.)

 Avoid excessive consecutive blank lines. In general, you should never have more than one or two consecutive blank lines.

 When defining a function that doesn’t take any arguments, always put <em>void</em> in its parentheses. For example, define a function using <em>int do_something(void)</em> instead of <em>int do_something()</em>.

 Please leave a space on both sides of any binary operators you use in your code (i.e., operators that take two operands). For example, use <em>(a + b) – c</em> instead of <em>(a+b)-c</em>.

 When defining or calling a function, do not leave a space before its opening parenthesis. For example: use <em>int main(void)</em> instead of <em>int main (void)</em>. Similarly, use <em>printf(“…”)</em> instead of <em>printf (“…”)</em>.

 Do leave a space before the opening parenthesis in an <em>if</em> statement or a loop. For example, use use <em>for (i = 0; i &lt; n; i++)</em> instead of <em>for(i = 0; i &lt; n; i++)</em>, and use <em>if (condition)</em> instead of <em>if(condition)</em> or <em>if( condition )</em>.

 Use meaningful variable names. It’s fine to use single-letter variable names for short functions (e.g., a simple <em>max</em> function, such as <em>int max(int a, int b)</em>, where it would be silly to try to come up with more meaningful variable names for those two input parameters), for control variables in your <em>for</em> loops (where <em>i</em>, <em>j</em>, and <em>k</em> are common variable name choices), or for sizes and lengths of certain inputs (e.g., using <em>n</em> for the length of an array). Otherwise, please try to use variable names that convey the intended use of your variables. Names like <em>cheeseburger</em> and <em>pizza</em> are not good choices for this particular program.

<a href="#_ftnref1" name="_ftn1">[1]</a> “It’s called a lonely party array because sometimes you invite 3000 people to a party, but only 3 show up.” – CS1 TA

<a href="#_ftnref2" name="_ftn2">[2]</a> We’ll see later that we also have to store quite a few pointers in our lonely party arrays, so although we will still save memory, the savings won’t end up being quite so substantial in this particular example once we’ve fleshed it out.

<a href="#_ftnref3" name="_ftn3">[3]</a> Note, by the way, that it is common to use the term “size” to refer to the number of elements that have been inserted into a data structure, while “length” often refers to the number of cells in an array, whether those cells have been used or not. (I.e., “length” refers to the maximum capacity of an array, in terms of the number of elements it can hold.)

<a href="#_ftnref4" name="_ftn4">[4]</a> Note that lonely party arrays are best used to replace large, sparsely populated arrays. Because of all the pointers in the LPA struct, a densely populated lonely party array could actually take up <em>more</em> space than a traditional C-style array.