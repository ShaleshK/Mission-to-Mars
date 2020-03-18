#### Created a webpage on which to display Mars News scraped from NASA, a featured image scraped from JPL, Mars Weather from marswxreport's twitter feed, a Mars Facts table from Space-Facts.com, and images of Mars Hemisphers from Astreology.USFS.gov.  Use Splinter to navigate the sites when needed and BeautifulSoup to help find and parse out the necessary data.  Used Pymongo for CRUD applications. Used Flask to display the scraped information to an Index.HTML page

#### To run, open cmd.exe, and run 
`cd C:\Program Files\MongoDB\Server\4.2\bin\mongod` 
#### (or double-clicking on the mongod icon)   
`python app.py` in Anaconda Prompt

![MarsNews]("URL.JPG")
![MarsFacts]("MarsFacts.JPG")
![Hemispheres]("Hemispheres.JPG")

#### Scrape All function
```python
from bs4 import BeautifulSoup as bs
import pandas as pd
from splinter import Browser
import requests

def scrape_all():
    executable_path = {'executable_path': 'chromedriver.exe'}
    browser = Browser('chrome', **executable_path, headless=True)
    news_title, news_p = mars_news(browser)
    data = {
        "news_title": news_title, 
        "news_p": news_p,
        "featured_image": featured_image(browser),
        "hemispheres": hemispheres(browser),
        "mars_weather": mars_weather(browser),
        "mars_facts": mars_facts(browser)
    }
    browser.quit()
    return data
```

#### Function to scrape JPL's featured image
```python
def featured_image(browser):
    url = 'https://www.jpl.nasa.gov/spaceimages/?search=&category=Mars'
    browser.visit(url)
    full_image_a = browser.find_by_id('full_image')
    full_image_a.click()
    browser.is_element_present_by_text('more info',wait_time=1)
    more_info_a = browser.find_link_by_partial_text('more info')
    more_info_a.click()
    html = browser.html
    soup = bs(html, 'html.parser')
    # featured_image_url = soup.select_one("article.carousel_item").get("style")
    # featured_image_url.split('(')[1].split("'")[1]
    featured_image_url = soup.select_one('figure.lede a img').get('src')
    featured_image_url
    image_url = f'https://jpl.nasa.gov{featured_image_url}'
    return image_url
```

#### Function to scrape Mars News
```python
def mars_news(browser):
    url = 'https://mars.nasa.gov/news/'
    browser.visit(url)
    html = browser.html
    soup = bs(html, 'html.parser')
    try: 
        slide = soup.select_one('ul.item_list')
        news_title = slide.find('div', class_="content_title").get_text()
        news_p = slide.find('div', class_="article_teaser_body").get_text()
    except AttributeError:
        return None, None

    return news_title, news_p
```

#### Function to scrape Mars Weather from Marswxreport's Twitter feed
```python
def mars_weather(browser):
    url = 'https://twitter.com/marswxreport?lang=en'
    response = requests.get(url)
    soup = bs(response.text, 'html.parser')
    #tweet = soup.find('div',attrs={"class":"tweet","data-name":"Mars Weather"})
    # mars_weather = tweet.find('p','tweet-text').text
    # mars_weather
    tweet = soup.find('div',class_='js-tweet-text-container').text
    mars_weather = tweet.translate({ord(c): None for c in '\n'})
    return mars_weather
```

#### Function to scrape Mars Facts from Space-Facts.com
```python
def mars_facts(browser):
    url = 'https://space-facts.com/mars/'  
    browser.visit(url)
    html = browser.html
    soup = bs(html, 'html.parser') 
    table = soup.find('table', attrs={"class":"tablepress tablepress-id-p-mars"})
    table_rows = table.find_all('tr')
    l = []
    for tr in table_rows:
        td = tr.find_all('td')
        row = [tr.text for tr in td]
        l.append(row)
    table_df = pd.DataFrame(l, columns=["description", "value"])
    table_df.set_index("description",inplace=True)
    table_df
    return table_df.to_html(header=False, classes="table table-striped")
```

#### Function to scrape images of Mars Hemispheres from Astreology.USGS.gov
```python
def hemispheres(browser):
    url = 'https://astrogeology.usgs.gov/search/results?q=hemisphere+enhanced&k1=target&v1=Mars' 
    browser.visit(url)
    hemispheres = ['Cerberus Hemisphere', 'Valles Marineris Hemisphere', 'Syrtis Major Hemisphere', 'Schiaparelli Hemisphere']
    images = ['cerberus_enhanced', "valles_marineris_enhanced", 'syrtis_major_enhanced', 'schiaparelli_enhanced']
    hemisphere_image_urls = []
    for h, i in zip(hemispheres, images):
        hemisphere_a = browser.find_link_by_partial_text(h)
        hemisphere_a.click()
        html = browser.html
        soup = bs(html, 'html.parser')
        img = soup.find('img',class_='wide-image').get('src')
        img_url = f'https://astrogeology.usgs.gov{img}'
        title = soup.find("h2",class_="title").get_text()
        hemisphere_image_urls.append( {"title":title, "img_url": img_url})
    return hemisphere_image_urls
```

#### App.py file with Flask connection to Index.HTML
```python
from flask import Flask, render_template, redirect
from flask_pymongo import PyMongo
import mission_to_mars

app = Flask(__name__)
app.config["MONGO_URI"] = 'mongodb://localhost:27017/mars_app'
mongo = PyMongo(app)

@app.route("/")     
def home():
    mars_data = mongo.db.mars.find_one()
    return render_template("index.html", mars=mars_data)

@app.route("/scrape")
def scrape():
    mars = mongo.db.mars
    mars_data = mission_to_mars.scrape_all()
    mars.update({}, mars_data, upsert = True)
    return redirect('/', code=302)

if __name__ == "__main__":
    app.run(debug=True)
```

