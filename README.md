
# Парсинг с помощью Python

## Установка pip( для версии < 3.4 )

> Идём на https://pip.pypa.io/en/stable/installing/ и скачиваем get-pip.py с помощью ПКМ "сохранить как..."
> Вводим в консоли ```python get-pip.py```

## Установка библиотек

>* Requests https://2.python-requests.org//en/master/ ( отвечает за направление  )
>* Beautifulsoup: https://www.crummy.com/software/BeautifulSoup/bs4/doc/
>* lxml: https://lxml.de/parsing.html

``` pip install requests beautifulsoup4 lxml ``` или ```python -m pip install requests beautifulsoup4 lxml ``` если первая инструкция не работает
### Решение проблемы с pip:
*http://qaru.site/questions/54228/fatal-error-in-launcher-unable-to-create-process-using-cprogram-files-x86python33pythonexe-cprogram-files-x86python33pipexe*

### получение содержимого тега h1 с wordpress.org
```python

import requests
from bs4 import BeautifulSoup


def get_html(url):
    r = requests.get(url)
    return r.text


def get_data(html):
    soup = BeautifulSoup(html, 'lxml')
    h1 = soup.find('div', id='home-welcome').find('header').find('h1').text
    return h1



def main():
    url = 'https://wordpress.org/'
    print(get_data(get_html(url)))



if __name__ == '__main__':
    main()

```


### получение содержимого атрибута ```href``` ссылки ```a``` с wordpress.org и запись в exel файл
```python
import requests
from bs4 import BeautifulSoup
import csv



def get_html(url):
    r = requests.get(url)
    return r.text

# функция для обработки строки ,получаемой в переменную rating( '1,470 total ratings')
def refined(s):
        r = s.split(' ')[0]
        # убираем запятую из числа            
        return r.replace(',' , '')    

# функция для обработки данных, полученных  в  словарь data (функции get_data) для -> csv
def write_csv(data):
        # with - контекстный менеджер
        # если plugins.csv есть - open откроет его, если нет - создаст
        # флаг "а" позволит добавлять данные, не стирая старые
        # в f попадает открытый для записи файловый объект для записи plugins.csv
        with open('plugins.csv', 'a') as f:
                writer = csv.writer(f)

                # writerow принимает только один аргумент -> чтобы впихнуть все данные создадим кортеж
                writer.writerow((data['name'],
                                 data['url'],
                                 data['reviews']))
       



def get_data(html):
    soup = BeautifulSoup(html, 'lxml')
    popular = soup.find_all('section')[1]
    plugins = popular.find_all('article')
    
    for plugin in plugins:
        name = plugin.find('h2').text
        url = plugin.find('h2').find('a').get('href')

        r = plugin.find('span', class_='rating-count').find('a').text
        rating = refined(r)
        print(rating)

        # объединяем полученные данные для работы с csv
        # создать словарь data
        data = {'name': name, 'url': url, 'reviews': rating}
        
        # print(data)
        write_csv(data)


def main():
    url = 'https://wordpress.org/plugins/'
    print(get_data(get_html(url)))



if __name__ == '__main__':
    main()

```


### получение данных с coinmarketcap.com и запись в exel файл
```python
import requests
from bs4 import BeautifulSoup
import csv


def get_html(url):
    r = requests.get(url)
    return r.text

def write_csv(data):
    with open('cmc.csv', 'a') as f:
        writer = csv.writer(f)

# в аргументы или кортеж или список. Выбираю список с всеми колонками, которые мне нужны с csv файле
        writer.writerow([
            data['name'],
            data['symbol'],
            data['url'],
            data['price'],
        ])

def get_page_data(html):
    soup = BeautifulSoup(html, 'lxml')
    # trs - список всех строк элемента tr
    trs = soup.find('table', id='currencies').find('tbody').find_all('tr')
    for tr in trs:
        tds = tr.find_all('td')
        name = tds[1].find('a', class_='currency-name-container').text
        symbol = tds[1].find('a').text # получение биржевых тиккеров
        url = 'https://coinmarketcap.com' + tds[1].find('a').get('href')# ссылки на внутренние страницы
        price = tds[3].find('a').get('data-usd') # в data-usd цена в долларах

        # упаковка данных в словарь
        data = {
            'name': name,
            'symbol': symbol,
            'url': url,
            'price': price
        }
        # передаём данный словарь в функцию
        write_csv(data)
        print(price)
    

def main():
    url = 'https://coinmarketcap.com/'
    get_page_data(get_html(url))

if __name__ == "__main__":
    main()
````

### получение данных с учётом пагинации путём цикла for
```python
# -*- coding: utf-8 -*-
import requests
from bs4 import BeautifulSoup
import csv




def get_html(url):
    r = requests.get(url)
    # 
    # r.ok - сервер вернул код 200
    if r.ok:
        return r.text
    print(r.status_code)# вернёт код ошибки

# функция для работы с полученными ценами
def refine_price(txt):
    # 25.02грн/шт. - полученные данные на входе, на выходе будут 25.02
    return txt.split('г')[0]


def write_scv(data):
    with open('odessa.csv', 'a') as f:
        writer = csv.writer(f)
        writer.writerow(( # кортеж
            data['name'],
            data['url'],
            data['price']
        ))


def get_page_data(html):
    soup = BeautifulSoup(html, 'lxml')

    lists = soup.find_all('div', class_='card-wrapper')
    print(len(lists))

    for div in lists:
        # если какого то данного нет, то выбьет ошибку, для этого try cash
        try:
            name = div.find('b', class_="nc").text

        except:
            name = 'empty string'
        try:
            url = 'https://27.ua' + div.find('a', class_='card__photo').get('href')
        except:
            url = ''
        try:
            p = div.find('span', class_='card__price-sum').text.strip() # -> 20.4грн
            price = refine_price(p)  # -> 20.4 но почему то не все данные адекватны - есть пустоты, нужно разобраться
            # метод strip() убирает /n /t ...
            
        except:
            price = ''
        data = {
            'name': name,
            'url' : url,
            'price': price
        }
        write_scv(data)
# заметим, что первая страница открывается как с https:...bolty/ так и /bolty/?PAGEN_1=1, вторая с /bolty/?PAGEN_1=2
def main():
    pattern = 'https://27.ua/shop/odessa/bolty/?PAGEN_1={}'
    for i in range(1, 3):
        url = pattern.format(str(i)) # str приводит число к строке
        print(url) 
        # https://27.ua/shop/odessa/bolty/?PAGEN_1=1
        # https://27.ua/shop/odessa/bolty/?PAGEN_1=2
        get_page_data(get_html(url))


if __name__ == "__main__":
    main()
```
### получение данных с учётом пагинации с использованием while

```python
import requests
from bs4 import BeautifulSoup
import csv
import re


def get_html(url):
    r = requests.get(url)
    if r.ok:
        return r.text
    print(r.status_code)

def write_csv(data):
    with open('coin_pages.csv', 'a') as f:
        writer = csv.writer(f)
        writer.writerow((
            data['name'],
            data['url'],
            data['price']
        ))

def get_page_data(html):
    soup = BeautifulSoup(html, 'lxml')

    tr_list = soup.find('table', id='currencies').find('tbody').find_all('tr')
    for tr in tr_list:
        td_list = tr.find_all('td')

        try:
            name = td_list[1].find('a', class_='currency-name-container').text.strip()
        except:
            name = ''
        try:
            url = 'https://coinmarketcap.com' + td_list[1].find('a', class_='currency-name-container').get('href')
        except:
            url = ''
        try:
            price = td_list[3].find('a').get('data-usd').strip()
        except:
            price = ''
        
        data = {
            'name' : name,
            'url'  : url,
            'price': price
        }
        write_csv(data)



# <a href="2">Next 100</a> так выглядит ссылка пагинации
# НО на следующей странице ссылка для перехода будет уже не на том месте.
# Поэтому мы зацепимся за слово NEXT и испол. регулярные выражения
def main():
    url = 'https://coinmarketcap.com/' # эту url переопределим в блоке try

    while True:
        get_page_data(get_html(url))

        soup = BeautifulSoup(get_html(url), 'lxml')

        try:
            pattern = 'Next'
            url = 'https://coinmarketcap.com/' + soup.find('ul', class_='pagination').find('a', text=re.compile(pattern)).get('href')
        except:
            break # когда не будет ссылки - пагинация закончится, то выход из цикла

if __name__ == "__main__":
    main()
```
### Два способа записи в файл

```python
import csv
# write_csv2 и write_csv - примеры функций записи данных в файл
# 
def write_csv2(data):
    # with - контекстный менеджер
    # as file - сохранение открытого файлового объекта в переменную file
    with open('names.csv', 'a') as file:
        # delimiter укажет знак разграничивающий столбцы в csv файле
        # по умолчанию delimiter=',' поэтому ниже второй аргумент можно не писать
        writer = csv.writer(file, delimiter=',')
        writer.writerow((
            data['name'],
            data['surname'],
            data['age']
        ))
# в write_csv(data) мы указвали какие ключи словаря в каком порядке хочу записать - data['name'], data['surname'],data['age']
# в write_csv2 использую список, а не словарь - список сохр. порядок
def write_csv2(data):
    with open('names_2.csv', 'a') as file:
        # order будет определять последовательность записи в словарь
        order = ['name', 'surname', 'age']
        writer = csv.DictWriter(file, fieldnames=order)
        writer.writerow(data)

def main():
    d = {
        'name': 'Yura',
        'surname': 'Beloff',
        'age': 44
    }
    d1 = {
        'name': 'Marry',
        'surname': 'Henson',
        'age': 23
    }
    d2 = {
        'name': 'Nick',
        'surname': 'Jorck',
        'age': 40
    }
    # упакую словари в список для удобной работы в цикле
    l = [ d, d1, d2]

    for i in l:
        write_csv2(i)

if __name__ == "__main__":
    main()

```
### Чтение из файла

```
import csv
# ПРИМЕР ЧТЕНИЯ ИЗ ФАЙЛА

def main():
    
    with open('cmc.csv') as file:
        # укажу нужные имена ключей ,иначе названия формируются кто знает как
        keyNames = ['name', 'url', 'price']
        reader = csv.DictReader(file, fieldnames=keyNames)
        # в cmc.csv список строк - нужно каждую строку отдельно прочитать
        for row in reader:
            print(row)

if __name__ == "__main__":
    main()

```
