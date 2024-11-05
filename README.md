﻿# lifeproject1
import requests
from bs4 import BeautifulSoup
import pandas as pd
import sqlite3
import json


baseurl = 'https://www.wholefoodsmarket.com/'

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/129.0.0.0 Safari/537.36'
}

r = requests.get('https://www.wholefoodsmarket.com/search?text=baby+foods&category=baby-child')
soup = BeautifulSoup(r.content, 'lxml')

productlist = soup.find_all('div', class_='w-pie--product-tile')

productlinks = []
for item in productlist:  
    for link in item.find_all('a', href=True):
        productlinks.append(baseurl + link['href'])



# Scrape product data
products_data = []
for link in productlinks:
    r = requests.get(link, headers=headers)
    soup = BeautifulSoup(r.content, 'lxml')

    try:
        # Extracting product name and ingredients
        brand = soup.find('a', class_='w-link').text.strip()
        name = soup.find('h1', class_='w-cms--font-headline__serif').text.strip()
        ingredients = soup.find('div', class_='w-cms--font-body__sans').text.strip()

        # Extracting and formatting nutrition facts
        try:
            nutritions = soup.find('section', class_='w-pie--nutrition-facts').text.strip()
       
        except AttributeError:
            nutritions = {}

    except Exception as e:
        break

    # Store the product details in a dictionary
    babyfood = {
        'brand' : brand,
        'name': name,
        'ingredients': ingredients,
        'nutritions': nutritions
    }

    # Append the baby food data to the product list
    products_data.append(babyfood)


# Create a pandas DataFrame
df = pd.DataFrame(products_data)
#path = r'C:\Users\Mky91\Downloads\test'  
#df.to_csv(path+'.csv')
print(df)   
#conn = sqlite3.connect('babyitems.db')
#delete_sql = "DROP TABLE babyfood"
#create_sql = "CREATE TABLE IF NOT EXISTS babyfoods(name TEXT, ingredients TEXT, nutrients TEXT)"
#c = conn.cursor()
#for babyfood in products_data:
    # Convert the nutrients dictionary to a JSON string
#    nutrients_str = json.dumps(babyfood['nutritions'])  # Convert to string
#   c.execute('INSERT INTO babyfoods (name, ingredients, nutritions) VALUES (?, ?, ?)', 
#              (babyfood['name'], babyfood['ingredients'], nutrients_str))


#c.execute('INSERT INTO babyfoods values (?,?,?)',[babyfood['name'], babyfood['ingredients'], babyfood['nutritions']])
#cursor.execute(create_sql)
##df.to_sql(name="babyfoods", con=conn, if_exists='replace', index=False)
#conn.commit()
#conn.close()

#print(products_data)
#df.to_csv('baby_foods_nutrition.csv', index=False)
#print("\nData saved to 'baby_foods_nutrition.csv'")
