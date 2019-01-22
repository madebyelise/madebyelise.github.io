---
published: false
---
---
layout: post
title: Scraping thousands of diamonds' information
---

Web scraping is a critical skill for any analyst or growth hacker. It opens up opportunities to complement your business dataset and lets you capitalize on the most abundant data source out there: the web. 

Want to learn how web scraping works? I recommend you first take two actions:
1. Pick a website you are interested in. For starters, look for one that is easy to scrape. What does this mean? If you can extract the data by mining the HTML code of the page, it's easy. If you have to deal with JavaScript, it takes a lot more work.
2. Go to BeautifulSoup and look at their excellent [documentation](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) for Python.

In this blog post, I'll walk you through how I scraped Brilliant Earth, a diamond selling website using Python and BeautifulSoup. The goal is to scrape the info of as many diamonds as possible and put it into a neat dataframe. From there, we could do fun things like building a regression model to predict diamond pricing. 

## Scrape diamond info from one single page
Each diamond on Brilliant Earth has a dedicated page from which you can grab various diamond characteristics such as Price, Carat Weight, Shape and Cut. For example, check out this beautiful diamond.

What we want to do is send a request to the website, get a result page back, parse it through an HTML parser which produces a BeautifulSoup object, and then mine through HTML tags/divs to grab what you want. 

The nice thing about BeautifulSoup is it lets you do all of this in a few lines of codes. The function processDiamond below takes in a BeautifulSoup object and returns various characteristics of the diamond in a Python dictionary. 

```python

colNames = ['Stock Number','Gemstone','Origin','Price','Carat Weight','Shape','Cut','Color',
                               'Clarity','Measurements','Table','Depth','Symmetry','Polish', 'Girdle',
                              'Culet','Fluorescence']
data = []
nCols = len(colNames)
url = 'https://www.brilliantearth.com/loose-diamonds/view_detail/'

def processDiamond(soup):
    soupDL = soup.findAll('dl')
    #find where the data table starts in the soupDL list
    starting = [i for i in range(len(soupDL)) if "Stock Number" in str(soupDL[i])][0]
    #iterate through every position from starting position through end of soupDL list
    #check whether the field matches name of column in our data table
    diamondDict = {}
    for pos in range(starting, len(soupDL)):
        s = str(soupDL[pos])
        if ('Stock Number' in s) or ('Gemstone' in s):
            fieldName = s[s.find('<dt>')+4:s.find('</dt>')-1]
            val = s[s.find('<dd>')+4:s.find('</dd>')]
            diamondDict[fieldName] = val
        elif 'Origin' in s:
            fieldName = 'Origin'
            val = s[s.find('<dd>')+4:s.find('</dd>')]
            diamondDict[fieldName] = val
        elif ('Girdle' in s) or ('Polish' in s):
            fieldName = s[s.find('</a>')+4:s.find('</dt>')-10]
            val = s[s.find('<dd>')+4:s.find('</dd>')]
            diamondDict[fieldName] = val
        else:
            fieldName = s[s.find('</a>')+4:s.find('</dt>')-1]
            if fieldName in colNames:
                val = s[s.find('<dd>')+4:s.find('</dd>')]
                diamondDict[fieldName] = val
    return diamondDict

```

Notice that I hard-coded the column names. This is possible because Brilliant Earth took the diamond stats from two reputable international organizations which provide standard diamond statistics for every diamond. For this reason, diamonds provide a good exercise for those new to web scraping. If you are taking on something less structured, be prepared to spend a lot of time cleaning the data! 

## Explore multiple diamonds 
So you know how to extract data when you have one single page. But one diamond is not useful by itself. Ideally you want a dataset of diamonds with 10,000+ diamonds to do some analysis. That means we need a way to generate 60k+ links (the number of natural diamonds on Brillant Earth).

You could use a crawler to automatically explore the website. 

Or you could use a simple trick to save you time writing a crawler. Notice the link structure:

[https://www.brilliantearth.com/loose-diamonds/view_detail/7100145/](https://www.brilliantearth.com/loose-diamonds/view_detail/7100145/)

Notice how you only need to change the diamond ID number to get to a different diamond. If you put in a number that doesn't correspond to any diamond, for example 999, Brilliant Earth would send you a result page saying the diamond doesn't exist. 

After browsing their search tool, you'd notice you can do a reasonable guess of the diamond IDs simply by trying every number in between 6,500,000 and 7,500,000. Maybe this is not the most effective way out there to get diamond page links, it keeps the code super clean for this exercise.

The code below lets you try every single diamond ID between 6,400,000 and 6,650,000. I also put in some spacing between each request. It's nice and also sometimes necessary to avoid bombarding a website with requests. You don't want to be blocked from accessing a website. When that happens, you need to go to the next step, which is to use proxy IP addresses.

```python
def addDiamond(diamondId):
    num_retries = 0
    got_response = False
    while True:
        try:
            link = requests.get(f"{url}{diamondId}/", timeout=5)
            got_response = True
        except Exception as exception:
            if num_retries < 5:
                num_retries += 1
            time.sleep(4**num_retries)
        if got_response:
            break
    soup = BeautifulSoup(link.content,'html.parser')
    if 'error404' not in str(soup):        
        diamondDict = processDiamond(soup)
        data.append(diamondDict)
    time.sleep(0.3)
    
diamondList = range(6400000,6650000)
for i in tqdm(diamondList):
    addDiamond(i)
```

## The Data
Let's look at the data we obtained from this process. Convert the _data_ variable, which is a list, into a Pandas dataframe. This is what the data looks like. 

![Diamond data]({{site.baseurl}}/images/diamondData.PNG)

Quite neat, isn't it?
      
## Conclusion
Hopefully after reading my blog post, you have a good idea of how web scraping works in a simple example. There's a lot to learn about to web scraping. The thing about web scraping is it's not scalable because every website follows a different structure. However, that also means it's a valuable skill because it's difficult to automate and you'll always be able to find someone needing scraping help!

View the full source code [here](https://github.com/madebyelise/diamond) (view file diamond.ipynb). Feel free to contact me with any questions about the post. I'm curious to learn about different organizations' data needs and how my skills can help them obtain better intelligence and advantage in the marketplace.
