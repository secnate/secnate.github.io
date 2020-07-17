---
title: "Coding"
excerpt: ""

collection: resources
author_profile: true

# Table of contents setting
toc: true
toc_label: Table of Contents
toc_icon: file-alt
toc_sticky: true

---

While many resources exist for learning programming, I recommend that beginners use books with extensive and challenging end-of-chapter exercises. 

Why? I find that online walkthroughs do not teach concepts systematically, leaving gaps in conceptual knowledge. Furthermore, they typically lack technical depth and extensive, well-designed exercises for cementing skills. Concepts aren't mastered by typing a pre-prepared demonstration program with ease, but by independently creating a program that uses them. In addition to providing a solid foundation, these books can serve later as solid references!
{: style="text-align: justify;"}

One can't run before learning to walk. Focus on the fundamentals. Do the exercises. 
{: style="text-align: justify;"}

This list _will not_ cover all the languages and tools one might use in InfoSec. That is impossible. Instead, it will build a foundation that allows one to easily pivot to new libraries, frameworks, or languages needed for their InfoSec goals. I explained why I consider the following to be important in [my post](https://secnate.github.io/advice/learning-to-code-for-infosec/) explaining how to get started with coding for InfoSec.
{: style="text-align: justify;"}

## Python

- [Python Programming: An Introduction to Computer Science, 3rd Edition](https://www.amazon.com/Python-Programming-Introduction-Computer-Science/dp/1590282752/ref=sr_1_1?dchild=1&keywords=python+programming+introduction+to+computer+science+by+john+zelle&qid=1594573184&sr=8-1)
- [Intro to Python for Computer Science and Data: Learning to Program with AI, Big Data, and The Cloud](https://www.amazon.com/Intro-Python-Computer-Science-Data/dp/0135404673/ref=sr_1_3?dchild=1&keywords=deitel+python+data+science&qid=1594574309&sr=8-3): It helps students master Python while preparing them for a future where scalability is a core development consideration.

These can be supplemented with online lecture videos, if one prefers to learn theory that way. There are plenty out there. I'm especially fond of [MIT OpenCourseware 6.0001.](https://www.youtube.com/watch?v=nykOeWgQcHM&list=PLUl4u3cNGP63WbdFxL8giv4yhgdMGaZNA)

There are also security-specific books. I listed them separately; while beginners can complete them, they are best-suited for those with prior Python knowledge. I _do not_ recommend starting with them as they do not have extensive exercises for practice. However, they provide fantastic examples of Python's applications:
{: style="text-align: justify;"}

- [Violent Python: A Cookbook for Hackers, Forensic Analysts, Penetration Testers, and Security Engineers](https://www.amazon.com/Violent-Python-Cookbook-Penetration-Engineers/dp/1597499579)
- [Black Hat Python: Python Programming for Hackers and Pentesters, 1st Edition](https://www.amazon.com/Black-Hat-Python-Programming-Pentesters/dp/1593275900/ref=sr_1_1?dchild=1&keywords=black+hat+python&qid=1594574660&sr=8-1)
- [Cracking Codes with Python: An Introduction to Building and Breaking Ciphers](https://www.amazon.com/Cracking-Codes-Python-Introduction-Building/dp/1593278225/ref=sr_1_33?dchild=1&keywords=no+starch+python&qid=1594583862&sr=8-33)
- [Mining Social Media: Finding Stories in Internet Data](https://www.amazon.com/Mining-Social-Media-Finding-Internet/dp/1593279167/ref=sr_1_11?dchild=1&keywords=no+starch+python&qid=1594583862&sr=8-11)
- [Web Scraping with Python: Collecting More Data from the Modern Web](https://www.amazon.com/Web-Scraping-Python-Collecting-Modern/dp/1491985577/ref=sr_1_5?dchild=1&keywords=web+scraping&qid=1594591699&sr=8-5)

## C++

C++ was first released in 1998 as an object-oriented extension of C and has [undergone revisions](https://www.geeksforgeeks.org/history-of-c/) since. The next major update was formalized in February 2020 and is due to be published soon.
{: style="text-align: justify;"} 

Such changes have their effects. For starters, learning resources may not cover the latest version's capabilities. And even if updated book editions are published shortly after the latest version's release, they are often _ridiculously overpriced._
{: style="text-align: justify;"} 

Beginners should not worry about language versions for two main reasons:

1. C++ is backwards-compatible. A C++14 compiler will process C++98 code without any issues.
2. The newely added features cover advanced functionality and syntax cases. Unless you are a highly-experienced C++ programmer, you will most likely never see nor need them.

Hence, learning an earlier version is fine. I recommend that one learn _at least_ C++11 in order to learn its traditional functionality and to get an understanding of the language's trajectory. If resources are available for later versions, even better!
{: style="text-align: justify;"}

This is why I recommend
- [C++ How to Program, Edition 9](https://www.addall.com/SuperRare/UsedRare.cgi?title=C%2B%2B+How+to+Program+Edition+9&author=&keyword=&isbn=&exclude=&binding=Any+Binding&min=&max=&dispCurr=USD&order=PRICE&ordering=ASC&match=Y&timeout=15&store=ABAA&store=Alibris&store=Abebooks&store=AbebooksAU&store=AbebooksDE&store=AbebooksFR&store=AbebooksUK&store=Amazon&store=AmazonCA&store=AmazonUK&store=AmazonDE&store=AmazonFR&store=Antiqbook&store=Biblio&store=BiblioUK&store=Bibliophile&store=Bibliopoly&store=Booksandcollectibles&store=ILAB&store=Powells&store=ZVAB&via=new): Later editions exist but are often overpriced.
- [C++ Primer, 5th Edition](https://www.amazon.com/C-Programming-Language-4th/dp/0321563840/ref=sr_1_2?crid=2QYO38TMUACJT&dchild=1&keywords=programming+principles+and+practice+using+c%2B%2B&qid=1594601499&s=books&sprefix=programming%3A+princ%2Cstripbooks%2C216&sr=1-2)
- [The C++ Programming Language, 4th Edition](https://www.amazon.com/C-Programming-Language-4th/dp/0321563840/ref=sr_1_2?crid=2QYO38TMUACJT&dchild=1&keywords=programming+principles+and+practice+using+c%2B%2B&qid=1594601499&s=books&sprefix=programming%3A+princ%2Cstripbooks%2C216&sr=1-2)

These can be supplemented with online lecture videos, if one prefers to learn theory that way. While there are many, I find [Stanford's CS106B](https://www.youtube.com/playlist?list=PL24126B3A47B69CB5) to be the best.
{: style="text-align: justify;"}

## Assembly

The goal is to gain a rudimentary understanding of how a computer processor works and how to read and write program code. The specific type one learns is less important than might be expected; different architectures have many similarities and share fundamental underlying concepts. Learn one well and you will be well-prepared.
{: style="text-align: justify;"}

My suggestions:
- [Computer Organization and Design MIPS Edition: The Hardware/Software Interface](https://www.amazon.com/David-Patterson/dp/0124077269/ref=sr_1_4?dchild=1&keywords=computer+architecture&qid=1594602970&sr=8-4): The [MARS simulator](http://courses.missouristate.edu/kenvollmar/mars/) can be used for writing small MIPS programs
- [Programming from the Ground Up (x86 Architecture)](https://www.amazon.com/Programming-Ground-Up-Jonathan-Bartlett/dp/1616100648/ref=sr_1_1?dchild=1&keywords=programming+from+the+ground+up&qid=1594682302&sr=8-1) 
- [ARM Assembly Language: Fundamentals and Techniques, 2nd Edition](https://www.amazon.com/ARM-Assembly-Language-Fundamentals-Techniques-ebook/dp/B00OGLE6CW/ref=sr_1_1?dchild=1&keywords=ARM+Assembly+Language%3A+Fundamentals+and+Techniques+2nd+Edition&qid=1594604793&s=books&sr=1-1)

## Web Design

Web applications can be extremely complex, with advanced frontend and backend functionality. First learn how the frontend works. Backends are built with database and API tools and frameworks that can be learned after developing a basic programming skillset.
{: style="text-align: justify;"}

HTML lists a website's bare bones, CSS formats it, and JavaScript makes its elements interactive. Learn the basics and you'll be well-positioned.
{: style="text-align: justify;"}

- [Learning Web Design: A Beginner's Guide to HTML, CSS, JavaScript, and Web Graphics, 4th edition](https://www.amazon.com/gp/product/1449319270/ref=as_li_qf_sp_asin_il_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=1449319270&linkCode=as2&tag=sixrevi-20)
- [Web Design with HTML, CSS, JavaScript and jQuery Set, First Edition](https://www.amazon.com/Web-Design-HTML-JavaScript-jQuery/dp/1118907442/ref=sr_1_3?dchild=1&keywords=html+and+css+book&qid=1594687556&sr=8-3)

I am also a big fan of the [w3schools](https://www.w3schools.com/) web development page. Unlike many online programming tutorials I have seen, it is so well-laid out that it can serve as stand-alone documentation.
{: style="text-align: justify;"}

***

This should be enough to get you started. I wish you luck on your programming journey.
{: style="text-align: justify;"}