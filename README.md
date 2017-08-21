# NOTE:
Apstudynotes has since changed their site and this script does not work anymore 😔. Please reach out to me on email.

# apstudynotes-Hacker
All the apstudynotes.org/essays essays for free!

Disclaimer: The legality of using this program is a grey area. I've only provided the code and program.

To use the program...
 - Click "Download ZIP"
 - Extract the files from "apstudynotes-Hacker.zip"
 - Run "AP Study Notes Hacker.py" #NOTE: You must have Python installed to be able to run a .py
 - Wait for the program to create all the files for you to enjoy!

# The Buildup
When I was a high school senior applying to colleges, I found the website www.apstudynotes.org/essays/ extremely helpful.
I took immense inspiration from reading the essays, and it significantly helped me shape my essays when application season started.

However, most of the essays are barred behind a pay wall. At the time, I was able to bypass this by going into inspect element
and just deleting the paywall and changing the 'blur' effect. They've since changed the site and it's become a little more difficult
to actually get through to the essays.

I've been learning Python and I finally reached the web scraping chapter. A few pages in, I decided to stop where I
was at and start writing a program to dowload all 141 essays on the apstudynotes site. Let the hacking begin!

# The Code
First, let's go over the imports.
```python
import os, requests, bs4
```
We import os, requests, and bs4. We use os (Operating System) to check for and create directories - where the files will be located, requests to get the website html, and bs4 (Beautiful Soup) to parse it.

====================================================================

The next part of the code is self explanatory.
```
# Create a txt, html, and doc folder if they don't already exist
if not os.path.exists("./txt"):
    os.makedirs("./txt")
if not os.path.exists("./html"):
    os.makedirs("./html")
if not os.path.exists("./doc"):
    os.makedirs("./doc")
```

====================================================================

Now the next step we *could* take is to open up every single url and copy it into a .txt document. Then we'd read every single url from the .txt document... But why do that when we have Python?
```
# Get the URLs to apstudynotes.org's 141 essays.
res = requests.get('https://www.apstudynotes.org/essays/')
soup = bs4.BeautifulSoup(res.text, 'html.parser')
elems = soup.select('h3 > a')
urlList = []
for link in elems:
    urlList.append(str(link.attrs))
for i in range(len(urlList)):
    urlList[i] = urlList[i].replace("{'href': '", "")
    urlList[i] = urlList[i].replace("'}", "")
    urlList[i] = 'https://www.apstudynotes.org' + urlList[i]
    print(urlList[i])
```
Instead, we requests.get('https://www.apstudynotes.org/essays/') and parse the html for all links in header 3. Each link is added to urlList and edited a little to get the full url to each different essay on the website.

====================================================================

To create a name for each file, we edit urlList.
```
# Get the titles of each essay, so we can name the files accurately
titleList = []
for i in range(len(elems)):
    tempString = urlList[i]
    tempString = tempString[len(str(r"https://www.apstudynotes.org/")):(len(tempString)-1)]
    for j in range(len(tempString)):
        if tempString[j] == '/':
            tempString = tempString[:j] + '-' + tempString[j+1:]
    titleList.append(tempString)
```
We remove the beginning of the url and the very last slash, so we're left something like 'uc-berkeley/describe-the-world-you-come-from'. Then we go through the string and change every instance of '/' to '-'. We then add that to titleList.

====================================================================

I love for loops. We go through each url in urlList and requests.get() it. Beautiful Soup is then used to parse each page for only the paragraph tags. We also create all the files here - a .txt, .html, and .doc - and name them according to number and name in titleList.
```
# Now we have all the pages - perfect!
# Time to get all the text from each page and put it into a .txt file
for i in range(len(urlList)):
    print('Getting contents: ' + urlList[i] + '...')
    res = requests.get(urlList[i])
    soup = bs4.BeautifulSoup(res.text, 'html.parser')

    elems = soup.select('p')
    
    print('\tCreating .txt, .html, and .doc files')
    filetxt = open(str('txt/' + str(i+1) + ' ' + titleList[i] + '.txt'), 'w', encoding='utf-8')
    filehtml = open(str('html/' + str(i+1) + ' ' + titleList[i] + '.html'), 'w', encoding='utf-8')
    filedoc = open(str('doc/' + str(i+1) + ' ' + titleList[i] + '.doc'), 'w', encoding='utf-8')
```

====================================================================

I love for loops within for loops. Each paragraph tag's text is added to to the three files. The .txt and .doc files are appended with just text, but the .html file is appended with the actual html. We then close out of each file, and the for loop runs again for each url in urlList.
```
    for i in range(len(elems)):
        filetxt.write(str('\n' + elems[i].getText() + '\n\n')) #txt file
        filehtml.write(str(elems[i])) #html file
        filedoc.write('\n' + elems[i].getText() + '\n\n') #doc file
        
    filetxt.close()
    filehtml.close()
    filedoc.close()
```
