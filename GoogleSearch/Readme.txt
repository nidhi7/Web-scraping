import urllib 
import mechanize 
from bs4 import BeautifulSoup
from urlparse import urlparse
import hashlib

def searchPic(term):
    img_list = getPic(term)
    print "done..."     

def getPic (search):
    search = search.replace(" ","%20")
    browser = mechanize.Browser()
    browser.set_handle_robots(False)
    browser.addheaders = [('User-agent','Chrome')]

    #the following link is used to scrape from Google images, did not know how to get more images other than what is displayed initially
    #htmltext = browser.open("https://www.google.com/search?site=imghp&tbm=isch&source=hp&biw=1414&bih=709&q="+search+"&oq="+search).read()

    #loop to get the first 10 pages google search results
    for ind in range(1,10):
        #the following link used to scrape from Google Search results
        htmltext = browser.open("http://www.google.com/search?q="+search+"&tbm=nws&start="+str(ind*10)).read()
        
        links = []
        img_urls = []
        formatted_images = []
        soup = BeautifulSoup(htmltext)
        results = soup.findAll('a')
        for r in results:
            hre = r.get('href')
            start = hre.find("/url?q=")+7
            end = hre.find("&",start)
            links.append(hre[start:end])
            #print hre[start:end]
        #for link in links:
        counter = 0
        print counter
        lenghtLinks = len(links)
        print lenghtLinks
        for index in range(0,lenghtLinks): #crawl each link in the search result
            try:  #exception handling, added this later
                print ind
                print counter
                counter = counter + 1
                if links[index].startswith('http')== False and links[index].startswith('www') == False:
                    continue
                #print links[index]
                linkhtml=browser.open(links[index]).read()  
                baseurlend = find_nth(links[index], "/", 3) #get the baseurl to append in case the img is relative path
                baseurl= links[index][0:baseurlend]
                soup = BeautifulSoup(linkhtml)
                images = soup.findAll('img')  
                print "images lenth" + str(len(images))
                sourcetextH1 = None
                if soup.h1 is not None: #get html header <h1>
                    sourcetextH1 = soup.h1.string
                f = open("googleText\\text2.txt" ,'a+b') #open file in append mode
                sourcetextPara = None 
                for i in images:  #get the text of the first para <p> in the same <div> as the img
                    for parent in i.parents:
                        if parent.name == 'div' and parent.p is not None:
                            if parent.p.string is not None:
                                sourcetextPara = parent.p.string
                                sourcetextPara = sourcetextPara.encode('utf-8')
                            break   
                    sourceimage = i.get('src')
                    #print sourceimage
                    if sourceimage is not None:  #to avoid exceptions
                        if sourceimage.endswith('.jpg') or sourceimage.endswith('.png') or sourceimage.endswith('.bmp') or sourceimage.endswith('.jpeg') or sourceimage.endswith('gif')  or sourceimage.endswith('.webp'):
                            if sourceimage.startswith('http')== False and sourceimage.startswith('www') == False:
                                if sourceimage.startswith('/'):
                                    sourceimage = baseurl+sourceimage
                                else:
                                    sourceimage = baseurl+"/"+sourceimage
                            #print sourceimage
                            print counter
                            #counter = counter + 1
                            #sourcetextH1 = i.get('h1')
                            sourcetextTitle = i.get('title')  #get the title 
                            #print sourcetextTitle
                            sourcetextAlt = i.get('alt')    #alternate text (alt) has useful information 
                            #print 
                            sourcetextTitle = str(sourcetextTitle.encode('utf-8') if sourcetextTitle else sourcetextTitle)
                            #print sourcetextTitle
                            #print "sourcetextTitle"
                            #print sourcetextTitle
                            #print "sourcetextAlt"
                            #print sourcetextAlt
                            sourcetextAlt = str(sourcetextAlt.encode('utf-8') if sourcetextAlt else sourcetextAlt)
                            sourceText = ', '.join([str(sourcetextTitle),str(sourcetextAlt)]) #concatenate all data collected
                            if sourcetextPara is not None:
                                sourceText = ';\n'.join([str(sourceText),str(sourcetextPara)])
                            if sourcetextH1 is not None:
                                sourceText = ';\n'.join([str(sourcetextH1),str(sourceText)])
                            hs = hashlib.sha224(sourceimage).hexdigest()    #get hash based on the img url 
                            #print hs
                            file_extension = sourceimage.split(".")[-1]     #get extension like jpg, png
                            imguri = "C:\\Python27\\googleImages2\\"
                            dest = imguri+hs+"."+file_extension
                            #print dest
                            urllib.urlretrieve(sourceimage,dest)    #retrienve the image from web
                            #print sourceText
                            #textWithUnicode = unicode(hs+": "+sourcetext+"\n", errors='ignore')
                            f.write(hs+": "+sourceText+"\n\n")  #write to file
            except:
                pass
            
#utility function to get the nth occurence of a string
def find_nth(haystack, needle, n):
    start = haystack.find(needle)
    while start >= 0 and n > 1:
        start = haystack.find(needle, start+len(needle))
        n -= 1
    return start
        
searchPic("world cup 2014 protests")   
