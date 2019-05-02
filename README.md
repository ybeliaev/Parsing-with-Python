
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
