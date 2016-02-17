Using Regular Expressions for finding DNA sequences of interest in Python
Now that we have learned how to use regular expressions, let apply them using Python to find some DNA sequences of interest. In the first example we will look for key sequences that are part of the Crispr bacterial immune system. In the second example we will search for transcription factor binding sites.
Crispr - a bacterial immune system
The CRISPR/Cas system is a bacterial immune system which gives bacteria resistance to plasmids and phages. Crispr spacers (see diagram below) from the Bacterium's genome recognize and help cut sequences complementary to the spacers in the plasmids and phages. Repeats from the bacterium's genome flank the spacers.
alt text
Loading fasta files via Screed
http://screed.readthedocs.org/en/latest/screed.html
In [1]:
import screed
In [2]:
# assign filename to variable
Genomefastafile = "data/Acidithiobacillus_ferrooxidans.fasta"

# Loop through all the records in fasta file and assign to 
# variables (assuming only one genome is in the file)
for record in screed.open(Genomefastafile):
    Genome_seqname = record.name
    Genome_sequence = record.sequence
    
print(Genome_seqname)
print(Genome_sequence[1:100])
gi|218665024|ref|NC_011761.1| Acidithiobacillus ferrooxidans ATCC 23270, complete genome
AGTTAAAAGAAAAAATATAAATTATTTTTATAAAGAGACGCCATCGATCCCTTTCCAGTCCTGGCATTCTAGGAGCACATCCCGATGAAAATCACCATA
Load Crispr repeat sequences
In [3]:
# assign filename to variable
Crisprfastafile = "data/Acidithiobacillus_ferrooxidans_Crispr.fasta"

# Create empty lists to append sequences to
Crispr_seqname = list()
Crispr_sequence = list()

# Loop through all the records in fasta file and add them the the lists
for record in screed.open(Crisprfastafile):
    Crispr_seqname.append(record.name)
    Crispr_sequence.append(record.sequence) 

# zip the two lists together to make a dictionary
Crispr_repeats = dict(zip(Crispr_seqname, Crispr_sequence))
 
# print the dictionary to view the seqeunces and their names    
print(Crispr_repeats)
{'NC_011761_1': 'CTTCTCAGCCGCGCGTGTGGCGGCATACCGC', 'NC_011761_5': 'GGGCGATTCGTTTCACCTCCTCCGC', 'NC_011761_3': 'GTATGCCGCCAGGTGCGCGGCTGAGAAC', 'NC_011761_6': 'CCTGGTCAGTACAACAACGGCTACGG', 'NC_011761_2': 'CTTCTCAGCCGCGCGTGTGGCGGCATACCGC', 'NC_011761_4': 'GTATGCCGCCATATGCGCAGCTTGTAAT'}
Find Crispr repeats
Access the sequence of a single repeat
In [4]:
# select the sequence from the dictionary by subsetting it's key
Crispr_repeats['NC_011761_1']
Out[4]:
'CTTCTCAGCCGCGCGTGTGGCGGCATACCGC'
Import regular expression library
In [5]:
import re
Find a single repeat in genome
In [6]:
# use re.search to find sequence in genome, inputs are 1) pattern and 2) string to search for pattern
# '(' and ')' surround the string that you want to capture
first_repeat = re.search('(' + Crispr_repeats['NC_011761_1'] + ')', Genome_sequence)

# re.search object has has a method called group() for what it matched (one group per capture group)
print('matching sequence for a NC_011761_1 Crispr repeat is:', first_repeat.group(1))
matching sequence for a NC_011761_1 Crispr repeat is: CTTCTCAGCCGCGCGTGTGGCGGCATACCGC
In [7]:
# re.search object also has position information!
print('matching sequence postition for NC_011761_1 Crispr is:', first_repeat.span())
matching sequence postition for NC_011761_1 Crispr is: (934846, 934877)
Compile a pattern ahead of time for speed and readability
In [8]:
# specify and compile pattern ahead of time
NC_011761_1pattern = re.compile('(' + Crispr_repeats['NC_011761_1'] + ')')

# then use re.search 
first_repeat = NC_011761_1pattern.search(Genome_sequence)
print('matching sequence for NC_011761_1 Crispr is:', first_repeat.group(1))
matching sequence for NC_011761_1 Crispr is: CTTCTCAGCCGCGCGTGTGGCGGCATACCGC
Find unknown sequence between Crispr repeats
In [9]:
# put capture group between sequences
NC_011761_1spacerspattern = re.compile(Crispr_repeats['NC_011761_1'] + '([ATCG]+)' + Crispr_repeats['NC_011761_1'])

# search for all occurrences of pattern
first_repeat_spacers = NC_011761_1spacerspattern.search(Genome_sequence)
print('The spacer sequence is:',  first_repeat_spacers.group(1))
The spacer sequence is: TGATCATCTTGGTAGACCGCGACATATCA
Find Crispr repeats and spacer
In [10]:
### Use re.findall to find multiple repeats and the spacer sequences!
NC_011761_1pattern = re.compile('(' + Crispr_repeats['NC_011761_1'] + ')' + '([ATCG]+)' + '(' + Crispr_repeats['NC_011761_1'] + ')' )

# search for all occurrences of pattern
first_repeat_matches = NC_011761_1pattern.search(Genome_sequence)
print('The repeat sequence is:', first_repeat_matches.group(1))
print('The spacer sequence is:', first_repeat_matches.group(2))
The repeat sequence is: CTTCTCAGCCGCGCGTGTGGCGGCATACCGC
The spacer sequence is: TGATCATCTTGGTAGACCGCGACATATCA
Use re.findall to find multiple repeats
In [11]:
# specify and compile pattern ahead of time
NC_011761_1pattern = re.compile('(' + Crispr_repeats['NC_011761_1'] + ')')

# search for all occurrences of pattern
first_repeat_matches = NC_011761_1pattern.findall(Genome_sequence)
print('All the matches are:', first_repeat_matches)
All the matches are: ['CTTCTCAGCCGCGCGTGTGGCGGCATACCGC', 'CTTCTCAGCCGCGCGTGTGGCGGCATACCGC']
Finding positions of multiple matches
re.findall() is very useful for finding multiple sequences, but what if we want to find the positions of multiple sequences? For example, multiple repeats? To do this we need to use another function, re.finditer():
In [12]:
# specify and compile pattern ahead of time
NC_011761_1pattern = re.compile('(' + Crispr_repeats['NC_011761_1'] + ')')

# search for all occurrences AND positions of pattern
first_repeat_matches = NC_011761_1pattern.finditer(Genome_sequence)

# loop over finditer object and print start position, end position and sequence (capture group)
for match in first_repeat_matches:
    print(match.start(), match.end(), match.group(1))
934846 934877 CTTCTCAGCCGCGCGTGTGGCGGCATACCGC
934906 934937 CTTCTCAGCCGCGCGTGTGGCGGCATACCGC
Challenge
Find and print the spacer for the 2nd Crispr repeats (NC_011761_2) of A. ferrooxidans.
Find the positions for the repeats and the spacer.
What about the reverse strand? How might we address this (we won't implement, just discuss).
Finding transcription factor binding site
Transcription factors are key molecules for regulating gene expression. Finding new genes that have transcription factor binding sites using experimentally derived consensus sequences is a common method for searching for new genes whose expression might be regulated by the transcription factor that binds to that consensus sequence.
Let's take the example of a C. elegans transcription factor, ELF-1. The preferred nucleotides that this transcription factor binds to is illustrated below:
alt text
http://motifmap.ics.uci.edu/
Making a test for our Regular Expression
Before we start searching for our regular expresssion, we should develop some test for it to help us build a good pattern to use. When we develop the test we should think about the edge cases we might encounter, such as no input sequence is provided, a protein sequence is provided instead of a dna sequence, or incorrect nucleotides are included at a position where they shouldn't be for that consensus sequence. We should also add some examples that we expect to work.
In [13]:
def test_elf1_consensus():
    assert elf1_consensus('') == None, 'should return none if given an empty string'
    assert elf1_consensus('SRTYRTRAEMQPLM') == None, 'should not match protien sequences'
    assert elf1_consensus('GCTGGTTT') == None, 'should not match this sequence'
    assert elf1_consensus('ATTGGTTT') == None, 'should not match this sequence'
    assert elf1_consensus('ACTGGTTT') == ['ACTGGTTT'], 'should match this sequence'
When we run the test the first time, it should fail, since we have yet to write the function, elf1_consensus() which will perform our regular expression search and matching.
In [14]:
test_elf1_consensus()
---------------------------------------------------------------------------
NameError                                 Traceback (most recent call last)
<ipython-input-14-3b04608d86ae> in <module>()
----> 1 test_elf1_consensus()

<ipython-input-13-f74e069989b4> in test_elf1_consensus()
      1 def test_elf1_consensus():
----> 2     assert elf1_consensus('') == None, 'should return none if given an empty string'
      3     assert elf1_consensus('SRTYRTRAEMQPLM') == None, 'should not match protien sequences'
      4     assert elf1_consensus('GCTGGTTT') == None, 'should not match this sequence'
      5     assert elf1_consensus('ATTGGTTT') == None, 'should not match this sequence'

NameError: name 'elf1_consensus' is not defined
Let's now start to write the actual elf1_consensus() function, for the case that we get no input:
In [15]:
def elf1_consensus(sequence_to_search):
    if sequence_to_search == '':
        return None
Now let's run our test code:
In [16]:
test_elf1_consensus()
---------------------------------------------------------------------------
AssertionError                            Traceback (most recent call last)
<ipython-input-16-3b04608d86ae> in <module>()
----> 1 test_elf1_consensus()

<ipython-input-13-f74e069989b4> in test_elf1_consensus()
      4     assert elf1_consensus('GCTGGTTT') == None, 'should not match this sequence'
      5     assert elf1_consensus('ATTGGTTT') == None, 'should not match this sequence'
----> 6     assert elf1_consensus('ACTGGTTT') == ['ACTGGTTT'], 'should match this sequence'

AssertionError: should match this sequence
We passed the first few tests, but not others, let's now try to address these by adding a regular expression pattern to our function:
In [17]:
def elf1_consensus(sequence_to_search):
    if sequence_to_search == '':
        return None
    
    first_elf1site = re.findall('([AT]C[TCA]GGTT[TCG])', sequence_to_search)
    
    if first_elf1site:
        return first_elf1site
    
    return None
Let's test out the function again:
In [18]:
test_elf1_consensus()
This looks great, but we forgot a test case... what if the sequence is provided in lower case? There are two options in Python:
We can provide the lower case options in our regular expression. For example if the nucleotide could be G or A, we could write that as [GgAa].
Or we can use the re.compile() function with the re.IGNORECASE argument.
Option #2 is easier to read, and so we can try that. Let's first add the case to our test, and then test our function.
In [19]:
def test_elf1_consensus():
    assert elf1_consensus('') == None, 'should return none if given an empty string'
    assert elf1_consensus('SRTYRTRAEMQPLM') == None, 'should not match protien sequences'
    assert elf1_consensus('GCTGGTTT') == None, 'should not match this sequence'
    assert elf1_consensus('ATTGGTTT') == None, 'should not match this sequence'
    assert elf1_consensus('ACTGGTTT') == ['ACTGGTTT'], 'should match this sequence'
    assert elf1_consensus('actggttt') == ['actggttt'], 'should match this sequence'
Now let's run the test.
In [20]:
test_elf1_consensus()
---------------------------------------------------------------------------
AssertionError                            Traceback (most recent call last)
<ipython-input-20-3b04608d86ae> in <module>()
----> 1 test_elf1_consensus()

<ipython-input-19-073187196004> in test_elf1_consensus()
      5     assert elf1_consensus('ATTGGTTT') == None, 'should not match this sequence'
      6     assert elf1_consensus('ACTGGTTT') == ['ACTGGTTT'], 'should match this sequence'
----> 7     assert elf1_consensus('actggttt') == ['actggttt'], 'should match this sequence'

AssertionError: should match this sequence
As expected, our new test fails, so let's implement the case-insensitivity feature into the function.
In [21]:
def elf1_consensus(sequence_to_search):
    if sequence_to_search == '':
        return None
    
    elf1_pattern = re.compile('([AT]C[TCA]GGTT[TCG])', re.IGNORECASE)
    
    first_elf1site = elf1_pattern.findall(sequence_to_search)
    
    if first_elf1site:
        return first_elf1site
    
    return None
Now let's run the test.
In [22]:
test_elf1_consensus()
Whoohoo! It passed! Now we have confidence that our regular expression will find what we want it to find, and only what we want it to find when given a big sequence. Let's now use it on some real sequences to look for ELF-1 binding sites!
Load the sequence data
Again we'll use screed to load the sequence data from the fasta file. We are loading a large region of chromosome II which contains the following genes:
alt text
In [23]:
# assign filename to variable
F25B4fastafile = "data/F25B4Sequence.fasta"

# Loop through all the records in fasta file and assign to 
# variables (assuming only one genome is in the file)
for record in screed.open(F25B4fastafile):
    F25B4_sequence = record.sequence
    
print(F25B4_sequence[1:100])
gagaaataacaaacactactgctgcttccccaggagacaccgggagagactcatacaataaggaggtgaatcacgcattgctaatctaatctttgcgat
Now let's see if there are any transcriptor factor binding sites in this region of the C. elegans genome.
In [24]:
F25B4_elf1_sites = elf1_consensus(F25B4_sequence)

print('There are ', len(F25B4_elf1_sites), 'in this region of the genome. They are:', F25B4_elf1_sites)
There are  8 in this region of the genome. They are: ['tccggttt', 'acaggttt', 'tctggttt', 'tctggttt', 'accggttt', 'tctggttc', 'actggttg', 'tctggttt']
Challenge
Can you modify the test_elf1_consensus() and elf1_consensus() functions to find the position of the transcription factor binding site(s)? From these results, what genes from the above list might be regulated by ELF-1?
Make a function and a test function so that you could find matching sequences to this preferred sequence for the transcription factor TRA-1: alt text
