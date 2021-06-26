---
title: "String similarity functions, An overview"
layout: post
tags: 
- review 
- similarity
date:   2021-06-12 23:43:22 +0530
---


String similarity functions have widespread use cases ranging from search to record linkage. This post contains a list of most frequently used string similarity functions, How they work internally and try to assess the scenarios/conditions that they might be best suited for. Following are the metrics that will be covered in this post. 

- Levenstein Distance
- Jaro-Winkler
- Q-gram
- Jaccard Index
- TF-IDF


### Levenstein Distance: 
Levenstein distance is a measure of how many operations (addition, deletion or replacement of a character) are required to convert a string A to B. It could be viewed as a more sophisticated version of [Hamming Distance](https://en.wikipedia.org/wiki/Hamming_distance) which is defined for equal length strings only. 
{% katexmm %}
Mathematically, it's defined as: 

$$
lev(a, b) = \begin{cases}
	|a| &\text{if } |b| = 0 \\
	|b| &\text{if } |a| = 0 \\
	lev(tail(a), tail(b)) &\text{if } a[0] = b[0] \\
	1 + min \begin{cases}
		lev(tail(a), b) \\
		lev(a, tail(b)) &\text{otherwise } \\
		lev(tail(a), tail(b))
		\end{cases}
	\end{cases}
$$

The tail function here refers to all the other characters following the character that we're looking at. 
{% endkatexmm %}

A quick recursive implementation for the same would be: 
```python
def lev(a, b):
	if len(a) == 0 or len(b) == 0:
		return max(len(a), len(b))
	# If the characters are same, look into the remaining strings. 
	if a[0] == b[0]:
		return lev(a[1:], b[1:])
	# If characters are different, incur a penalty and check for 
	# the minimum distance between remaining strings. 
	else:
		return 1 + min(
			# Addition of a character
			lev(a, b[1:]), 
			# Subtraction of a character
			lev(a[1:], b), 
			# Substitution of a character
			lev(a[1:], b[1:])
		)
```
- First two cases are pretty straightforward. 
- In the third case, since the characters we're looking at are same, the levenshtein distance is essentially the distance between the remaining texts.  
- Lastly, if the characters do differ, we incur a penalty one and check recursively for the remaining string considering addition, deletion or substitution. 

This metric tends to work well in cases where both the string lengths are similar and we are more or less looking for small-ish spelling mistakes. e.g. Comparing card numbers, phone numbers etc. 

### Jaro-Winkler: 
Jaro-Winkler similarity is a modified version of jaro similarity criterion. It gives more weightage to the matches in the begining of the string as compared to matches that occur at the end. In practise, it's very commonly employed to compute similarity between names (human and business). 

**Jaro Similarity**
{% katexmm %}

The jaro similarity of two strings $s_1$ and $s_2$ is defined as: 
$$
sim_j = \begin{cases} 
	0 &\text{if } m = 0 \\
	\frac{1}{3} (\frac{m}{|s_1|} + \frac{m}{|s_2|} + \frac{m-t}{m}) &\text{otherwise }
	\end{cases}
$$
In this case, 
- $|s_i|$ is the length of string i. 
- $m$ is the number of matching characters. 
- $t$ is the number of transpositions. 

Two characters are considered "matching characters" if they are no more than $\lfloor\frac{max(|s_1|, |s_2|)}{2}\rfloor - 1$ apart. This is to account for slight spelling mistakes/differences that might come up while typing. 

On the other hand, transpositions are defined as the matching characters that are in different sequence order divided by 2. e.g. l<u>am</u>e and l<u>ma</u>e will have t = 1 and m = 2. 

**Jaro-Winkler similarity**
Jaro-winkler similarity builds on top of this and adds some weightage for the matches in the beginning of the strings. 

$$ sim_w = sim_j + lp(1 - sim_j)$$

where, 
- $sim_j$ is the jaro similarity score. 
- $l$ is the length of prefix at the start of the string or 4, whichever is smaller. 
- $p$ is a scaling factor to keep the score between 0 and 1. It's value is generally taken to be 0.1. 


Implementation of the above in python: [[source](https://www.geeksforgeeks.org/jaro-and-jaro-winkler-similarity/)]
```python

# Function to calculate the Jaro Similarity of two strings
def jaro(s1, s2) :
    # If the strings are equal
    if (s1 == s2) :
        return 1.0;
    # Length of two strings
    len1 = len(s1);
    len2 = len(s2);
    if (len1 == 0 or len2 == 0) :
        return 0.0;
    # Max distance to consider a match
    max_dist = (max(len(s1), len(s2)) // 2 ) - 1
    # Count of matches
    match = 0;
    # Hash for matches
    hash_s1 = [0] * len(s1) ;
    hash_s2 = [0] * len(s2) ;
    # Traverse through the first string
    for i in range(len1) :
        # Check if there is any matches
        for j in range( max(0, i - max_dist), 
            min(len2, i + max_dist + 1)):
            # If there is a match
            if (s1[i] == s2[j] and hash_s2[j] == 0):
                hash_s1[i] = 1
                hash_s2[j] = 1
                match += 1;
                break
                
    # If there is no match
    if (match == 0) :
        return 0.0;

    # Number of transpositions
    t = 0
    point = 0

    # Count number of occurrences where two characters match but
    # there is a third matched character in between the indices
    for i in range(len1) :
        if (hash_s1[i]) :
            # Find the next matched character
            # in second string
            while (hash_s2[point] == 0) :
                point += 1
            if (s1[i] != s2[point]) :
                point += 1
                t += 1
            else :
                point += 1
        t /= 2;
    # Return the Jaro Similarity
    return ((match / len1 + match / len2 +
            (match - t) / match ) / 3.0)
 
# Jaro Winkler Similarity
def jaro_Winkler(s1, s2) :
    jaro_dist = jaro(s1, s2);
    # If the jaro Similarity is above a threshold
    if (jaro_dist > 0.7) :
        # Find the length of common prefix
        prefix = 0;
        for i in range(min(len(s1), len(s2))) :
            # If the characters match
            if (s1[i] == s2[i]) :
                prefix += 1
            # Else break
            else :
                break
        # Maximum of 4 characters are allowed in prefix
        prefix = min(4, prefix);
        # Calculate jaro winkler Similarity
        jaro_dist += 0.1 * prefix * (1 - jaro_dist)
    return jaro_dist
```


{% endkatexmm %}
### Q-gram
Q-gram distance is defined as the sum of absolute difference between the n-gram vector representations of both strings. It serves as a lower bound for the Levenshtein distance and is significantly faster in calculation ( O(m+n) ) compared to Levenshtein ( O(m.n)). 

e.g. For two strings Ramesh and Rakesh. The corresponding trigrams would be: 
```
Ramesh: {ram, ame, mes, esh}
Rakesh: {rak, ake, kes, esh}
```

In this case, the qgram distance would be 6. 

A simple python implementation of this metric is as follows: 
```python
def generate_ngrams(s, n):
    ngrams = dict()
    for i in range(0, len(s) - n):
        key = s[i:i+n]
        key in ngrams ? ngrams[key] += 1 : ngrams[key] = 1
    return ngrams

def qgram(a, b):
    a_profile = generate_ngrams(a)
    b_profile = generate_ngrams(b)
    diff = 0
    union_set = set(a_profile.keys + b_profile.keys)
    for gram in set: 
        val_a = gram in a_profile ? a_profile[gram] : 0
        val_b = gram in b_profile ? b_profile[gram] : 0
        diff += abs(val_a - val_b)
    return diff
```

### Jaccard Index
Jaccard index was primarily created to find the similarity between two sets. This can be extended and easily used for text as well. We can generate sets from text by breaking it into tokens of n-grams. These generated sets can then be used for calculation of Jaccard index, which is defined as: 

{% katexmm %}

$$
J(A, B) = \frac{|A \cap B|}{|A \cup B|}
$$

{% endkatexmm %}

An implementation for the same in python would be: 
```python
def jaccard(a, b):
	# Tokenise the strings and generate sets. 
	s1 = set(a.strip().split())
	s2 = set(a.strip().split())

	# Calculate the union and intersection. 
	union = s1.union(s2)
	intersection = s1.intersection(s2)

	# Return the division
	return len(intersection)/len(union)
```

### TF-IDF
TF-IDF is short for Term Frequency - Inverse document frequency. Given a bunch of documents, a word that's frequent in a document but infrequent outside would be much more representative of that particular document. This is the central idea behind the tf-idf metric. 

So, while comparing two documents for similarity, or finding the most appropriate document for a query. We penalise a common word pair if it's commonly found in other documents and give more score in case it's very frequent in the document(s) that we're intereseted in. 

We can generate the tf-idf score using the following method: 

{% katexmm %}
- Let $t_n$ be the number of times the term t occurs in the current doc. 
- And $D_n$ be the number of docs the term t occurs in. 
- N is the total number of documents in the corpus. 

Then we define tf-idf score as: 
$$
tf-idf = t_n * log(\frac{N}{D_n})
$$
Here, the second term $log(\frac{N}{D_n})$ is known as the inverse document frequency. 

{% endkatexmm %}





### References
1. [A Comparison of String Distance Metrics for Name-Matching Tasks.](http://www.cs.cmu.edu/~wcohen/postscript/ijcai-ws-2003.pdf)
2. [Creating a Q-gram algorithm to determine the similarityof two character strings.](https://support.sas.com/resources/papers/proceedings16/2080-2016.pdf)
