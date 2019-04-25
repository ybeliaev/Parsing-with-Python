
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
