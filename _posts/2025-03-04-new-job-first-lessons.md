---
layout: post
title:  "new job, new challenges"
date:   2025-03-07
author: Shani
---


**Update**: Started a new job.  

I got a huge code base with no documentation.  

The first step was to understand which files are responsible for each process.  

The second step was to dig deeper into each part. Test each part. Add comments as I understand more. When something was unclear, I added a comment as `TODO:` with a question.  
I kept checking to see if I could answer those questions—and if so, document it as well.  

After I felt more comfortable with the code, I allowed myself to refactor.  

The code base handles WhatsApp chat processing. Four formats were covered: combinations of platform (Android/iPhone) and language (English/Hebrew).  
The original coder wrote parallel methods for each format. Lots of duplicated code. After I recognized a bug and solved it in one format, I found myself repeating the process with the three other formats.  

I refactored each batch of four methods into a single method to adhere to the DRY principle. The unique parts were mostly regular expressions. So I created multiple constants in this format:  

```python
text_patterns = {
    ("iphone", "hebrew"): <some regexp>,
    ("iphone", "english"): <another regexp>,
    ("android", "hebrew"): <also regexp>,
    ("android", "english"): <a puppy! jk. a regexp>,
}
```  

And in the methods themselves, I could do something like this:  

```python
pattern = text_patterns.get((self._platform, self._language))
```  

Much clearer and easier to read and debug.  

---  

I was overwhelmed at the start. Anxious that I wouldn't be able to decipher it all. But I did. So I'm proud of myself.  