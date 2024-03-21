# Details for P6 Huffman Coding

## Starter Code and Using Git
**_You should have installed all software (Java, Git, VS Code) before completing this project._** You can find the [directions for installation here](https://coursework.cs.duke.edu/201fall23/resources-201/-/blob/main/installingSoftware.md) (including workarounds for submitting without Git if needed).

We'll be using Git and the installation of GitLab at [coursework.cs.duke.edu](https://coursework.cs.duke.edu). All code for classwork will be kept here. Git is software used for version control, and GitLab is an online repository to store code in the cloud using Git.

**[This document details the workflow](https://coursework.cs.duke.edu/201fall23/resources-201/-/blob/main/projectWorkflow.md) for downloading the starter code for the project, updating your code on coursework using Git, and ultimately submitting to Gradescope for autograding.** We recommend that you read and follow the directions carefully this first time working on a project! While coding, we recommend that you periodically (perhaps when completing a method or small section) push your changes as explained in below.

## Details on Git with a Partner

You may find it helpful to begin by reading the Working Together section of the [Git tutorial](https://gitlab.oit.duke.edu/academic-technology/cct/-/tree/master/git) from the Duke Colab. For more, see the [Git tutoraial by Gitlab](https://docs.gitlab.com/ee/tutorials/make_your_first_git_commit.html) including the link to an [extensive video tutorial](https://www.youtube.com/watch?v=4lxvVj7wlZw) if you prefer that.

One person should fork the starter code and then add their partner as a collaborator on the project. Choose Settings>Members>Invite Members. Then use the autocomplete feature to invite your partner to the project as a *maintainer*. Both of you can now clone and push to this project. See the [gitlab documentation here](https://docs.gitlab.com/ee/user/project/members/).

Now you should be ready to clone the code to your local machines.

1. Both students should clone the same repository and import it into VS Code just like previous projects.  
2. After both students have cloned and imported, one person should make a change (you could just write a comment in the code, for example). Commit and push this change. 
3. The other partner will then issue a git pull request. Simply use the command-line (in the same project directory where you cloned the starter code for the project) and type:
```bash
git pull
```
4. If the other partner now opens the project in VS Code again, they should see the modified code with the edit created by the first partner. 
5. You can continue this workflow: Whenever one person finishes work on the project, they commit and push. Whenever anyone starts work on the project, they begin by downloading the current version from the shared online repository using a git pull command.

This process works as long as only one person is editing at a time, and **you always pull before editing** and remember to **commit/push when finished**. If you forget to pull before editing your local code, you might end up working from an old version of the code different than what is in the shared online gitlab repository. If that happens, you may experience an error when you attempt to push your code back to the shared online repository. 

There are many ways to resolve these conflicts, essentially you just need to pick which of the different versions of the code you want to go with. See the [working together Git tutorial](https://gitlab.oit.duke.edu/academic-technology/cct/-/blob/master/git/working_together.md) and the [branching and merging Git tutorial](https://gitlab.oit.duke.edu/academic-technology/cct/-/blob/master/git/branching_merging.md) from the Duke Colab for more information. You can also refer to our [Git troubleshooting document](https://coursework.cs.duke.edu/201-public-documentation/resources-201/-/blob/main/troubleshooting.md#git-faq). 

## Background on Huffman Coding

(optional, but strong encouragec)

Huffman coding was invented by David Huffman while he was a graduate student at MIT in 1950 when given the option of a term paper or a final exam. For details, see [this 1991 Scientific American Article][Huffman Article]. In an autobiography Huffman had this to say about the epiphany that led to his invention of the coding method that bears his name:

"*-- but a week before the end of the term I seemed to have nothing to show for weeks of effort. I knew I'd better get busy fast, do the standard final, and forget the research problem. I remember, after breakfast that morning, throwing my research notes in the wastebasket. And at that very moment, I had a sense of sudden release, so that I could see a simple pattern in what I had been doing, that I hadn't been able to see at all until then. The result is the thing for which I'm probably best known: the Huffman Coding Procedure. I've had many breakthroughs since then, but never again all at once, like that. It was very exciting.*"

[The Wikipedia reference][Huffman Wikipedia] is extensive as is [this online material][Duke Huffman ] developed as one of the original [Nifty Assignments][Nifty Assignments]. Both jpeg and mp3 encodings use Huffman Coding as part of their compression algorithms. In this assignment you'll implement a complete program to compress and uncompress data using Huffman coding.
**You should read about the Huffman algorithm, and work to understand it.** You won't be able to complete the project easily without the overview you'll get from this reading: [https://www.cs.duke.edu/csed/poop/huff/info/](https://www.cs.duke.edu/csed/poop/huff/info/).

When you've read the description of the algorithm and data structures used you'll be ready to implement both decompression (aka uncompressing) and compression  using Huffman Coding. You'll be using input and output or I/O classes that read and write 1 to many bits at a time, i.e., a single zero or one to several zeros and ones. This will make debugging your program a challenge.

### Acknowledgments and History

We first gave a Huffman coding assignment at Duke in Spring of 1994. Over the years many people have worked on creating the infrastructure for the bit-reading and -writing code (as we changed from C to C++ to Java at Duke) and the GUI that drives the Huffman assignment. It was one of the original so-called "nifty assignments" (see http://nifty.stanford.edu) in 1999. In Fall of 2018 we moved away from the GUI and using a simple main and the command-line. This was done for pragmatic and philosophical reasons.

## `BitInputStream` and `BitOutputStream` Classes

These two classes are provided to help in reading/writing bits in (un)compressed files. They extend the Java [InputStream](https://docs.oracle.com/javase/9/docs/api/java/io/InputStream.html) and [OutputStream](https://docs.oracle.com/javase/9/docs/api/java/io/OutputStream.html) classes, respectively. They function just like Scanner, except instead of reading in / writing out arbitrarily delimited “tokens”, they read/write a specified number of bits. Note that two consecutive calls to the `readBits` method will likely return different results since InputStream classes maintain an internal "cursor" or "pointer" to a spot in the stream from which to read -- and once read the bits won't be read again (unless the stream is reset).

The only methods you will need to interact with are the following:
1. *`int BitInputStream.readBits(int numBits)`*: This method reads from the source the specified number of bits and returns an integer. Since integers are 32-bit in Java, the parameter `numBits` must be between 1 and 32, inclusive. **It will return -1 if there are no more bits to read.**
2. `void BitInputStream.reset()`: This method repositions the “cursor” to the beginning of the input file.
3. `void BitOutputStream.writeBits(int numBits, int value)`: This method writes the least-significant `numBits` bits of the value to the output file.

## Details on diff via the command line interface (CLI)

There is a mac/unix command `diff` you can run in a terminal/bash shell on Mac/Windows (including the built-in terminal in VS Code). This command-line (CLI) `diff`  compares two files and indicates if they are the same bit-for-bit or not. First you must navigate to the directory/folder containing the files you would like to compare. You can use the following commands to navigate directories in your terminal:

- `pwd` stands for "print working directory." Entering it will print the path to your current directory (the folder you are in).
- `ls` stands for "list subdirectories." Entering it will print all files and subfolders in your current folder. 
- `cd` stands for "change direcotory. Entering it, followed by a subfolder name, will navigate to that subfolder. For example, `cd Documents`. Entering `cd` with nothing after will navigate to your home directory. Entering `cd ..` will navigate to the enclosing directory (that is, will "back up one level" in the folder hierarchy).

Once you are in the same folder as the files you would like to compare, you can type (terminal/shell): `diff foo.txt bar.txt` to compare `foo.txt` and `bar.txt`.

If the files are the same _nothing is printed_. If the files are different there's an indication of where they are different for text files, and just `different` if the files are binary/compressed/image files. For your purposes in P5 it isn't especially crucial that you understand the output printed by `diff` - generally you will just want to check if a decompressed file is exactly same as the original file before any compression/decompression.

## Outline of decompress method

<div align="center">
  <img width="837" height="254" src="p6-figures/decompress.png">
</div>

To understand this in more detail, please review the explanation in the [Huffman coding writeup](https://www2.cs.duke.edu/csed/poop/huff/info/) -- in particular you'll need to know how the tree was written to write code that reads the tree back.


As you can see in the example above, a `HuffException` is thrown if the file of compressed bits does not start with the 32-bit value `HUFF_TREE`. Your code should also throw a `HuffException` if reading bits ever fails, i.e., the `readBits` method returns -1. That could happen in the helper methods when reading the tree and when reading the compressed bits.

### Reading the Tree (private HuffNode readTree)

Reading the tree using a helper method is essentially required since reading the tree, stored using a pre-order traversal, is much simpler with recursion. You don't have to use the names or parameters described above, though you can.
In the _201 Huffman Compression Protocol_, interior tree nodes are indicated by the single bit 0 and leaf nodes are indicated by the single bit 1. No bits/values are written after the `0` for internal nodes and a 9-bit value is written after the `1` for a leaf node. 

This leads to the following pseudocode to read the tree:

``` java
private HuffNode readTree(BitInputStream in) {
        bit = in.readBits(1);
        if (bit == -1) throw exception
        if (bit == 0) {
                left = readTree(...)
                right = readTree(...)
                 return new HuffNode(0,0,left,right);
        }
        else {
            value = read BITS_PER_WORD+1 bits from input
            return new HuffNode(value,0,null,null);
        }
  }
``` 

For example, the tree below corresponds to the bit sequence `0001X1Y01Z1W01A1B`, with each letter representing a 9-bit sequence to be stored in a leaf as shown in the tree to the right. You'll read these 9-bit chunks with an appropriate call of `readBits`. Rather than use 9, you should use `BITS_PER_WORD + 1`, the +1 is needed since one leaf stores `PSEUDO_EOF` so all leaf nodes store a 9-bit value. See the [appendix](#appendix-how-the-tree-in-decompress-was-generated) for a detailed explanation on how this tree was constructed.

#### Example Tree

<div align="center">
  <img width="291" height="213" src="p6-figures/huffheadtreeNODES.png">
</div>


### Reading Compressed Bits (while (true))

Once you've read the bit sequence representing the tree, you'll read the remaining bits from the `BitInputStream` representing the compressed file one bit at a time, traversing the tree from the root and going left or right depending on whether you read a zero or a one. This is represented in the pseudocode for `decompress` by the hidden while loop.

The pseudocode from the [Huffman coding writeup](https://www.cs.duke.edu/csed/poop/huff/info/)  is reproduced in the section below, this is the same code shown in that document -- it's not perfect Java, hence pseudocode. (Note: you break when reaching `PSEUDO_EOF`, and then no bits are written to the output file. Otherwise, when writing a value stored in a leaf to the output stream, you write 8, or `BITS_PER_WORD` bits).

#### pseudocode example to decompress with tree

``` java
  HuffNode current = root; 
  while (true) {
      int bits = input.readBits(1);
      if (bits == -1) {
          throw new HuffException("bad input, no PSEUDO_EOF");
      }
      else { 
          if (bits == 0) current = current.left;
          else current = current.right;

          if (current is a leaf node) {
              if (current.value == PSEUDO_EOF) 
                  break;   // out of loop
              else {
                  write BITS_PER_WORD bits to output for current.value;
                  current = root; // start back after leaf
              }
          }
      }
  }
  close output file
```

# Appendix: How the Tree in `decompress` was generated

A 9-bit sequence represents a "character"/chunk stored in each leaf. This character/chunk, actually a value between 0-255, will be written to the output stream when uncompressing. One leaf stores PSEUDO_EOF, this won't be printed during uncompression, but will stop the process of reading bits.

<div align="center">
  <img width="291" height="213" src="p6-figures/huffheadtreeNODES.png">
</div>

When you read the first 0, you know it's an internal node (it's a 0), you'll create an internall `HuffNode` node, and recursively read the left and right subtrees. The left subtree call will read the bits 001X1Y01Z1W as the left subtree of the root and the right subtree recursive call will read  01A1B as the right subtree. Note that in the bit-sequence representing the tree, a single bit of 0 and 1 differentiates INTERNAL nodes from LEAF nodes, not the left/right branch taken in uncompressing -- that comes later. The internal node that's the root of the left subtree of the overall root has its own left subtree of 01X1Y and a right subtree of 01Z1W. When you read the single 1-bit, your code will need to read 9-bits to obtain the value stored in the leaf.

</details>

