# Lab Report 5

## Part 1

### Original Post

**What environment are you using (computer, operating system, web browser, terminal/editor, and so on)?**

I am using a 2020 Macbook Pro with an M1 Chip, running MacOs Monterey. Terminal/editor-wise, I am running VsCode, and am using the default terminal on MacOs.

**Detail the symptom you're seeing. Be specific; include both what you're seeing and what you expected to see instead. Screenshots are great, copy-pasted terminal output is also great. Avoid saying “it doesn't work”.**

<img width="1437" alt="Screen Shot 2023-06-05 at 4 25 24 PM" src="https://github.com/cjcanaday/labreport5/assets/40177716/d36ee165-0224-4872-bc4b-4ff0682e2121">

<img width="1011" alt="Screen Shot 2023-06-05 at 4 34 49 PM" src="https://github.com/cjcanaday/labreport5/assets/40177716/128a3d06-3cfa-4723-9bb4-51ebf8b2ebb0">


Regardless of the url that I pass to my Grade Server grader, it gives the same errors, and JUnit score output. For instance, when passing the repository [here] (https://github.com/ucsd-cse15l-f22/list-methods-corrected), in which all code is implemented correctly and should pass all tests, I still get the output of 2/4 tests passed. Similarly, when passing the repository [here](https://github.com/ucsd-cse15l-f22/list-examples-subtle), which has many errors, I still get the exact same output of 2/4 tests passed.



**Detail the failure-inducing input and context. That might mean any or all of the command you're running, a test case, command-line arguments, working directory, even the last few commands you ran. Do your best to provide as much context as you can.**

Failure inducing input and context is any link passed to the grading script, as it provides the same output regardless of what is passed to it.

The commands that I have passed that should have completely different output, but have been the same are as follows:

`./grade.sh https://github.com/ucsd-cse15l-f22/list-methods-corrected`

`./grade.sh https://github.com/ucsd-cse15l-f22/list-examples-subtle.git`

My current working directory is:

`/Users/colincanaday/Documents/CSE/CSE15L/list-examples-grader`

### TA Reponse

Hi Colin, thanks so much for providing details related to this bug that you're encountering. The fact that you are receiving the same output regardless of the repository that you pass to the grading script, indicates that there is likley an issue with grading script itself. Would you be able to share the output of running `ls` within your current working directory? This should give some inuitions as to how far your grading script is getting before encountering an error.

### Student Response 

Thank you for your help! Included is the screenshot from running `ls`:

<img width="971" alt="Screen Shot 2023-06-05 at 4 46 11 PM" src="https://github.com/cjcanaday/labreport5/assets/40177716/67fefa23-6554-4dc3-a3f2-cc0339f95710">

Looking at this result, I was able to realize that I was missing the directory `student-submission`, which should have been added when passing the repository link to the grading script. As I know that these are valid repos and links, I knew that the error must be related to this line:

<img width="541" alt="Screen Shot 2023-06-05 at 4 50 48 PM" src="https://github.com/cjcanaday/labreport5/assets/40177716/bea58736-6295-4028-af1c-eef20b5bb619">

Upon reading up on Bash and how it handles passing command line arguments, I realized that `$0` actually holds the name of shell script, rather than the first command line argument. Realizing this, I was able to change the line to the following, which actually reads the passed repo:

<img width="286" alt="Screen Shot 2023-06-05 at 4 50 56 PM" src="https://github.com/cjcanaday/labreport5/assets/40177716/44857b9c-6ba1-4490-9d63-1a5f8e9a4b44">

Changing this resulted in the grader sucessfully running on the correct implementation and giving it full marks:

<img width="1020" alt="Screen Shot 2023-06-05 at 5 11 25 PM" src="https://github.com/cjcanaday/labreport5/assets/40177716/4b96dbfd-dceb-455a-94cb-3ab7e8fd1746">

The bug that had to do with each output showing 2/4 tests passed comes from the fact that I already had a `ListExamples.java` in my directory, regardless of whether I was downloading one from the passed repo. Therefore, when the `git clone` command failed, I was instead running the tests on my local `ListExamples` file, rather than the one that should have been downloaded.

### Additional Info

**File Directory:**

<img width="230" alt="Screen Shot 2023-06-05 at 4 56 36 PM" src="https://github.com/cjcanaday/labreport5/assets/40177716/669c0fa0-82eb-417e-8163-7e963d28bfc9">

**Content of grade.sh:**

> Code used and fixed for this example can be found [here](https://github.com/ucsd-cse15l-f22/list-examples-grader)


```bash
CPATH='.:lib/hamcrest-core-1.3.jar:lib/junit-4.13.2.jar'

rm -rf student-submission
git clone $0 student-submission
echo 'Finished cloning'

if [[ -f student-submission/ListExamples.java ]]
then
  echo 'ListExamples.java found'
else
  echo 'ListExamples.java not found'
  echo 'Score: 0/4'
fi

cp student-submission/ListExamples.java ./

javac -cp "$CPATH" *.java

java -cp $CPATH org.junit.runner.JUnitCore TestListExamples > junit-output.txt

# The strategy used here relies on the last few lines of JUnit output, which
# looks like:

# FAILURES!!!
# Tests run: 4,  Failures: 2

# We check for "FAILURES!!!" and then do a bit of parsing of the last line to
# get the count
FAILURES=`grep -c FAILURES!!! junit-output.txt`

if [[ $FAILURES -eq 0 ]]
then
  echo 'All tests passed'
  echo '4/4'
else
  # The ${VAR:N:M} syntax gets a substring of length M starting at index N
  # Note that since this is a precise character count into the "Tests run:..."
  # string, we'd need to update it if, say, we had a double-digit number of
  # tests. But it's nice and simple for the purposes of this script.

  # See, for example:
  # https://stackoverflow.com/questions/16484972/how-to-extract-a-substring-in-bash
  # https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#Shell-Parameter-Expansion

  RESULT_LINE=`grep "Tests run:" junit-output.txt`
  COUNT=${RESULT_LINE:25:1}

  echo "JUnit output was:"
  cat junit-output.txt
  echo ""
  echo "--------------"
  echo "| Score: $COUNT/4 |"
  echo "--------------"w
  echo ""
fi

```
**Contents of local ListExamples.java:**

```java
import java.util.ArrayList;
import java.util.List;

interface StringChecker { boolean checkString(String s); }

class ListExamples {

  // Returns a new list that has all the elements of the input list for which
  // the StringChecker returns true, and not the elements that return false, in
  // the same order they appeared in the input list;
  static List<String> filter(List<String> list, StringChecker sc) {
    List<String> result = new ArrayList<>();
    for(String s: list) {
      if(sc.checkString(s)) {
        result.add(s);
      }
    }
    return result;
  }


  // Takes two sorted list of strings (so "a" appears before "b" and so on),
  // and return a new list that has all the strings in both list in sorted order.
  static List<String> merge(List<String> list1, List<String> list2) {
    List<String> result = new ArrayList<>();
    int index1 = 0, index2 = 0;
    while(index1 < list1.size() && index2 < list2.size()) {
      if(list1.get(index1).compareTo(list2.get(index2)) < 0) {
        result.add(list1.get(index1));
        index1 += 1;
      }
      else {
        result.add(list2.get(index2));
        index2 += 1;
      }
    }
    while(index1 < list1.size()) {
      result.add(list1.get(index1));
      index1 += 1;
    }
    while(index2 < list2.size()) {
      result.add(list2.get(index2));
      index2 += 1;
    }
    return result;
  }


}
```
**Commands used to trigger bug:**

`./grade.sh https://github.com/ucsd-cse15l-f22/list-methods-corrected`

`./grade.sh https://github.com/ucsd-cse15l-f22/list-examples-subtle.git`

However, passing any link to grade.sh would trigger the same error.

**Fix:**

This bug can be fixed by changing "$0" in line 4 of grade.sh to "$1".

That is, changing the line

`git clone $0 student-submission`

to 

`git clone $1 student-submission`

## Part 2

One interesting thing that I learned through the second half of this class in lab is how to properly set up ssh keys. Being one of the people who early on in the quarter struggled with reliably getting in to the ieng6 server, being able to generate keys such that I don't need to worry about logging in is a very exciting idea. Moreover, the randomart image that is associated with this process is very cool to me, as cryptography is something that I would like to learn more about.





