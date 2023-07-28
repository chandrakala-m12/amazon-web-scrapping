# amazon-web-scrapping
This code is used to create web scrapping using amazon and data can be stored ad csv file
from bs4 import BeautifulSoup
import requests
import pandas as pd
import numpy as np
def get_title(soup):
    try:
        title = soup.find("span", attrs={"id":'productTitle'})
        title_value = title.text
        title_string = title_value.strip()
    except AttributeError:
        title_string = ""
    return title_string
def get_price(soup):
    try:
        price = soup.find("span", attrs={'id':'priceblock_ourprice'}).string.strip()
    except AttributeError:
        try:
            price = soup.find("span", attrs={'id':'priceblock_dealprice'}).string.strip()
        except:
            price = ""
    return price
def get_rating(soup):
    try:
        rating = soup.find("i", attrs={'class':'a-icon a-icon-star a-star-4-5'}).string.strip()
    except AttributeError:
        try:
            rating = soup.find("span", attrs={'class':'a-icon-alt'}).string.strip()
        except:
            rating = ""
    return rating
def get_review_count(soup):
    try:
        review_count = soup.find("span", attrs={'id':'acrCustomerReviewText'}).string.strip()
    except AttributeError:
        review_count = ""
    return review_count
def get_availability(soup):
    try:
        available = soup.find("div", attrs={'id':'availability'})
        available = available.find("span").string.strip()
    except AttributeError:
        available = "Not Available"
    return available
URL = "https://www.amazon.in/s?k=playstation&crid=1ARQJGBDHQ0LV&sprefix=playstatio%2Caps%2C536&ref=nb_sb_noss_2"
if __name__ == '__main__':
    HEADERS = ({'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36', 'Accept-Language': 'en-US, en;q=0.5'})
    webpage = requests.get(URL, headers=HEADERS)
    webpage
    soup = BeautifulSoup(webpage.content, "html.parser")
    links = soup.find_all("a", attrs={'class':'a-link-normal s-no-outline'})
    links_list = []
    for link in links:
        links_list.append(link.get('href'))
        d = {"title":[], "price":[], "rating":[], "reviews":[],"availability":[]}
    for link in links_list:
        new_webpage = requests.get("https://www.amazon.in/" + link, headers=HEADERS)
        new_soup = BeautifulSoup(new_webpage.content, "html.parser")
        d['title'].append(get_title(new_soup))
        d['price'].append(get_price(new_soup))
        d['rating'].append(get_rating(new_soup))
        d['reviews'].append(get_review_count(new_soup))
        d['availability'].append(get_availability(new_soup))
         amazon_df = pd.DataFrame.from_dict(d)
         amazon_df['title'].replace('', np.nan, inplace=True)
         amazon_df = amazon_df.dropna(subset=['title'])
         amazon_df.to_csv("amazon_data.csv", header=True, index=False)
         amazon_df
