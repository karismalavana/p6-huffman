# Project 6: Huffman Coding/Compression

See the [details document](docs/details.md) for information on using Git, starting the project, and more details about the project including information about the classes and concepts that are outlined briefly below. You'll absolutely need to read the information in the details document to understand how the classes in this project work independently and together. The details document also contains project-specific details. This current document provides a high-level overview of the assignment.

You are STRONGLY encouraged to work with a partner on 6! (as you were on P5 and will be on P7). See the [details document](docs/details.md) for information on using Git with a partner and how the workflow can proceed. If you'd like to be paired (somewhat randomly, but you can write about yourself or a partner) then fill out [this form](https://forms.office.com/r/qpLBLV6ZPs) to request a pairing.


## Outline 

- [Project Introduction](#project-introduction)
- [Part 0: Understanding and Running Starter Code](#part-0-understanding-and-running-starter-code)
- [Part 1: Implementing `HuffProcessor.decompress`](#part-1-implementing-huffprocessordecompress)
- [Part 2: Implementing `HuffProcessor.compress`](#part-2-implementing-huffprocessorcompress)
- [Analysis](#analysis)
- [Submitting and Grading](#submitting-and-grading)

## Project Introduction

There are many techniques used to compress digital data (that is, to represent it using less memory). This assignment covers Huffman Coding, which is used everywhere from zipping a folder to jpeg and mp3 encodings. See the [details document](docs/details.md) for background on how the compression algorith was developed.


<details>
<summary>Pre-reading self-assessment questions</summary>

1. Why are two passes over the input file to be compressed required when creating a compressed version of the input file?
2. What aspects of creating the Huffman tree from counts account for that process being a greedy algorithm?
3. At a high-level, how is the tree used to create 8-bit char/chunk encodings?
4. What is written first, after the magic number, in the compressed file?
5. Why are the bits written at the end of the compressed file representing PSEUDO_EOF required?
6. After reading the magic number and tree, how are the bits representing compressed data read when decompressing, e.g., how many bits are read each time the compressed data is accessed?

</details>

When you've read the description of the algorithm and data structures used you'll be ready to implement both decompression (a.k.a. uncompressing) and compression  using Huffman Coding. You'll be using input and output or I/O classes that read and write 1 to many bits at a time, i.e., a single zero or one to several zeros and ones. This will make debugging your program a challenge.


## Part 0: Understanding and Running Starter Code

Once you understand the Huffman coding algorithm, you should review this section to understand the organization of the starter code. For details on the `BitInputStream` and `BitOutputStream` classes, see [the details document](docs/details.md).


### Running Starter Code (with incomplete `HuffProcessor.decompress`)

**Run `HuffMainDecompress`**. This prompts for a file to decompress, then calls `HuffProcessor.decompress`. You're given a stub version of that method; it initially ***simply copies the first file to another file***, it doesn’t actually decompress it. To make sure you know how to use this program, we recommend you run the program as you cloned it and as described below.

Choose `mystery.tif.hf` from the data folder to decompress (the `.hf` suffix indicates this has been compressed by a working `HuffProcessor.compress`). When prompted with a name for the file to save, use a `UHF prefix`, i.e., save with the name `UFHmystery.tif.uhf (that suffix is the default).  

Then run `diff` on the command line/terminal (see [the details doc](docs/details.md) for information on using `diff`). Use diff to compare two files: the original, `mystery.tif.hf` and the uncompressed version: `UHFmystery.tif.uhf`. The `diff` program should _say_ these files are the same. This is because the code you first get from git simply copies the first file to another file, it doesn't actually decompress it. **You will use `diff` to check whether your implementation is working correctly locally, there are no JUnit tests for this project.**

The main takeaways here in running before implementing `HuffProcess.decompress` are to 
- Understand what to run when decompressing.
- Understand how to use `diff` on the command line to compare files. 



## Part 1: Implementing `HuffProcessor.decompress`

You should begin programming by implementing `decompress` first before moving on to `compress`. You'll remove the code you're given intially in `HuffProcessor.decompress` and implement code to actually **decompress** as described in this section. You **must remember to close the output file** before `decompress` returns. The call `out.close` is in the code you're given, be sure it's in the code you write as well.

There are four conceptual steps in decompressing a file that has been compressed using Huffman coding:
1. Read the 32-bit "magic" number as a check on whether the file is Huffman-coded (see lines 150-153 below)
2. Read the tree used to decompress, this is the same tree that was used to compress, i.e., was written during compression (helper method call on line 154 below).
3. Read the bits from the compressed file and use them to traverse root-to-leaf paths, writing leaf values to the output file. Stop when finding `PSEUDO_EOF` (hidden while loop on lines 156-174 below).
4. Close the output file (line 175 below).

We recommend using the code found in the [details document](docs/details.md) as a base/basis for your decompress method because it illustrates how to use the bit-stream classes and how to throw appropriate exceptions. However, you will not be penalized if you use an alternative method to code it.


## Part 2: Implementing `HuffProcessor.compress`

There are five conceptual steps to compress a file using Huffman coding. You do not need to use helper methods for these steps, but for some steps helper methods are **extremely useful and will facilitate debugging**.

1. Determine the frequency of every eight-bit character/chunk in the file being compressed (see line 78 below).
2. From the frequencies, create the Huffman trie/tree used to create encodings greedily (see line 79 below).
3. From the trie/tree, create the encodings for each eight-bit character chunk (see lines 83-84 below).
4. Write the magic number and the tree to the beginning/header of the compressed file (see lines 81-82 below).
5. Read the file again and write the encoding for each eight-bit chunk, followed by the encoding for PSEUDO_EOF, then close the file being written (not shown).

You won't need to throw exceptions for the steps outlined. A brief description of each step follows. More details can be found in the explanation of the Huffman algorithm in the [Huffman coding writeup](https://www.cs.duke.edu/csed/poop/huff/info/).

<div align="center">
  <img width="600" height="180" src="p6-figures/newcompress.png">
</div>

### Determining Frequencies (private int[] getCounts)

Create an integer array that can store 256 values (use `ALPH_SIZE`). You'll read 8-bit characters/chunks, (using `BITS_PER_WORD` rather than 8), and use the read/8-bit value as an index into the array, incrementing the frequency. Conceptually this is a map from 8-bit chunk to an `int` frequency for that chunk, it's easy to use an array for this, mapping the index to the number of times the index occurs, e.g., `counts['a']` is the number of times 'a' occurs in the input file being compressed. The code you start with in compress (and decompress) illustrates how to read until the sentinel -1 is read to indicate there are no more bits in the input stream. 

### Making Huffman Trie/Tree (private HuffNode makeTree)

You'll use a greedy algorithm and a `PriorityQueue` of `HuffNode` objects to create the trie. Since `HuffNode` implements `Comparable` (using weight), the code you write will remove the minimal-weight nodes when `pq.remove()` is called as shown in the pseudocode included in the section below.

#### makeTree pseudocode

``` java
PriorityQueue<HuffNode> pq = new PriorityQueue<>();
for(every index such that freq[index] > 0) {
    pq.add(new HuffNode(index,freq[index],null,null));
}
pq.add(new HuffNode(PSEUDO_EOF,1,null,null)); // account for PSEUDO_EOF having a single occurrence

while (pq.size() > 1) {
   HuffNode left = pq.remove();
   HuffNode right = pq.remove();
   // create new HuffNode t with weight from
   // left.weight+right.weight and left, right subtrees
   pq.add(t);
}
HuffNode root = pq.remove();
return root;
```

You'll need to ***be sure that `PSEUDO_EOF` is represented in the tree. *** As shown above, you should only add nodes to the priority queue for indexes/8-bit values that occur, i.e., that have non-zero weights.


### Make Codings from Trie/Tree (private makeEncodings)

For this, you'll essentially implement a recursive helper method, similar to code you've seen in discussion for the [LeafTrails APT problem](https://www2.cs.duke.edu/csed/newapt/leaftrails.html). As shown in the example of compress above, this method populates an array of Strings such that `encodings[val]` is the encoding of the 8-bit chunk val. See the debugging runs at the end of this write-up for details. As with the LeafTrails APT, the recursive helper method will have the array of encodings as one parameter, a node that's the root of a subtree as another parameter, and a string that's the path to that node as a string of zeros and ones. The first call of the helper method might be as shown, e.g., in the helper method `makeEncodings`.
``` java
    String[] encodings = new String[ALPH_SIZE + 1];
    makeEncodings(root,"",encodings);
```

In this method, if the `HuffNode` parameter is a leaf (recall that a leaf node has no left or right child), an encoding for the value stored in the leaf is added to the array, e.g.,
``` java
   if (isLeaf(root)) {
        encodings[root.value] = path;
        return;
   }
```
If the root is not a leaf, you'll need to make recursive calls adding "0" to the end of the path when making a recursive call on the left subtree and adding "1" to the end of the path when making a recursive call on the right subtree. Every node in a Huffman tree has two children. ***Be sure that you only add a single "0" for left-call and a single "1" for right-call. Each recursive call has a String path that's one more character than the parameter passed, e.g., path + "0" and path + "1".***

### Writing the Tree (private void writeTree)

Writing the tree is similar to the code you wrote to read the tree when decompressing. If a node is an internal node, i.e., not a leaf, write a single bit of zero. Else, if the node is a leaf, write a single bit of one, followed by _nine bits_ of the value stored in the leaf.  This is a pre-order traversal: write one bit for the node, then make two recursive calls if the node is an internal node. No recursion is used for leaf nodes. You'll need to write 9 bits, or `BITS_PER_WORD + 1`, because there are 257 possible values including `PSEUDO_EOF`.

### Writing Compressed Bits

After writing the tree, you'll need to read the file being compressed one more time. As shown above, the ***`BitInputStream` is reset, then read again to compress it***. The first reading was to determine frequencies of every 8-bit chunk. The encoding for each 8-bit chunk read is stored in the array created when encodings were made from the tree. These encodings are stored as strings of zeros and ones, e..g., "010101110101". To convert such a string to a bit-sequence you can use `Integer.parseInt` specifying a radix, or base of two. For example, to write the encoding for the 8-bit chunk representing 'A', which has an ASCII value of 65, you'd use:
``` java
    String code = encoding['A']
    out.writeBits(code.length(), Integer.parseInt(code,2));
```
You'll use code like this for every 8-bit chunk read from the file being compressed. You must also write the bits that encode `PSEUDO_EOF`, i.e.,
``` java
    String code = encoding[PSEUDO_EOF]
    out.writeBits(code.length(), Integer.parseInt(code,2));
```
You'll write these bits _after_ writing the bits for every 8-bit chunk. The encoding for `PSEUDO_EOF` is used when decompressing, ***so you'll need to write the encoding bits before the output file is closed.***

**See the [appendix section in the details.md](docs/details.md) file for important informatoon on understanding this compression algorithm.

## Analysis

You'll submit the analysis as a PDF separate from the code in Gradescope. If you are working with a partner, you and your partner should submit a single analysis document.

For the analysis questions, we will let $`N`$ be the number of total characters in a file to encode, and let $`M`$ be the total number of *unique* characters in the file. Note that both refer to the *non-compressed file*. Note that $`M \leq N`$. Define the *compression ratio* of a file to be the number of bits in the original file divided by the number of bits in the compressed file.

Note that running the `HuffMainCompress` and `HuffMainDecompress` programs will print information to the terminal about the number of bits and the runtime of the compress and decompress algorithms.

**Question 1.** Suppose you want to compress two different files: `fileA` and `fileB`. Both have $`N`$ total characters and $`M`$ unique characters. The characters in `fileA` follow a uniform distribution, meaning each of the unique characters appears $`N/M`$ times. In `fileB`, the $`i`$'th unique character appears $`2^i`$ times (and the numbers add up to $`N`$), so some characters are much more common than others. Which file should achieve a higher compression ratio? Explain your answer. **The compression ratio is the size of the original file divided by the size of the compressed file.**

**Question 2.** Typically the number of total characters  $`N`$ is much greater than the number of unique characters, $`M`$ which is at most 256 when reading 8-bit chunks as with Huffman coding. Files are read twice when compressing: once to count/determine encodings, and once to actually write the encodings. Run your compression code on two images from the `data` folder: `oak-ridge.jpg` and `mtblanc.jpg`. In your analysis, include the time it takes to compress each of these files, what the compression ratio is: size of original/size of compressed. Do the same for the text file `hawthorne.txt` Can you make any conclusions about compression of images compared to text files? About the time to run compression on files? Discuss and justify your answers based on empirical/runtime data. 

**Question 3.** When running `decompress`, each character that is decompressed requires traversing nodes in the Huffman coding tree, and there are $`N`$ such characters. Run your `decompress` code on the images you compressed in the previous question. Make some conclusions about the runtimes of compressed images compared to compressed text files when decompressing? Justify your answers by runtimes as you can.

## Submitting and Grading

Push your code to Git. Do this often. To submit:

1. Submit your code on gradescope to the autograder. If you are working with a partner, refer to [this document](https://docs.google.com/document/d/e/2PACX-1vREK5ajnfEAk3FKjkoKR1wFtVAAEN3hGYwNipZbcbBCnWodkY2UI1lp856fz0ZFbxQ3yLPkotZ0U1U1/pub) for submitting to Gradescope with a partner. 
2. Submit a PDF to Gradescope in the separate Analysis assignment. Be sure to mark pages for the questions as explained in the [gradescope documentation here](https://help.gradescope.com/article/ccbpppziu9-student-submit-work#submitting_a_pdf). If you are working with a partner, you should submit a single document and [add your partner to your group on gradescope](https://help.gradescope.com/article/m5qz2xsnjy-student-add-group-members).

Points are awarded equally for compression and decompression. You'll get points for decompressing and compressing text and image files. These are 10 points each, for a total of 40 points possible on the code. The analysis is scored separately by TAs for a total of 12 points (4 per question).

