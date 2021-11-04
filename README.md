# Моё портфолио

## Первый проект
Задача:
  * Создать ```.csv``` файл, который будет содержать вакансии по запросу "python" с сайта https://www.superjob.ru/
  * Из результата поиска запарсить название вакансии, адрес, требования, зарплату и ссылку на эту вакансию
  * Результат должен содержать все вакансии с первой по последнюю страницу
 
Решение:
  * Сначала импортирую все нужные библиотеки
  * Создаю список который будет содержать ссылки на каждую конкретную вакансию
  ```python
  import requests
  from bs4 import BeautifulSoup
  import pandas as pd

  list_of_urls = []

  for j in range(1,6):
      url = f'https://www.superjob.ru/vacancy/search/?keywords=python&geo%5Bt%5D%5B0%5D=4&page={j}'
      response = requests.get(url).text
      soup = BeautifulSoup(response, 'html.parser')

      for i in range(len(soup.find_all(class_='f-test-search-result-item'))):
          try:
              vacancy_url = soup.find_all(class_='f-test-search-result-item')[i].find('a', href=True)['href']
              if('/vakansii/' in vacancy_url):
                  list_of_urls.append('https://www.superjob.ru' + vacancy_url)
          except:
              continue
  ```
  * Теперь парсим нужную нам инфу переходя по каждой ссылке из ``` list_of_urls ```
  * Инфу с каждой конкретной вакансии помещаем в словарь, а словарь помещаем в список ```final_result```
  
  ```python
  final_result = []
  for vacancy_info_url in list_of_urls:
      final_vacancy_data = {}
      response = requests.get(vacancy_info_url).text
      soup = BeautifulSoup(response, 'html.parser')
      final_vacancy_data['Название'] = soup.find(class_='f-test-vacancy-base-info').find(class_='rFbjy').text # title
      final_vacancy_data['Адрес'] = soup.find(class_='f-test-vacancy-base-info').find(class_='f-test-address').find(class_='_6-z9f').text # location
      final_vacancy_data['Требования'] = str(BeautifulSoup([i.text for i in soup.find(class_='f-test-vacancy-base-info').find(class_='_3MVeX')][-2], 'html.parser')) # requirements

      final_vacancy_data['Зарплата'] = str(BeautifulSoup(soup.find(class_='f-test-vacancy-base-info').find(class_='_1OuF_ ZON4b').find(class_='_2Wp8I').text, 'html.parser')) # salary per month

      final_vacancy_data['Ссылка'] = vacancy_info_url
      final_result.append(final_vacancy_data)
  
  ```
  * Создаем ```DataFrame``` из списка ```final_result```, а затем и ```.csv``` файл
  
  ```python
   data = pd.DataFrame(data=final_result)
   data.to_csv("superjob final result.csv", index=False)
  ```
  
## Второй проект
Задача:
  * Запарсить данные из топ 100 фильмов всех времен по версии сайта https://www.rottentomatoes.com/ в формат таблицы
  * В таблице должны быть столбцы содержащие рейтинг, название, год выпуска и количество рецензий каждого фильма из топа 
  * Фильмы должны быть отсортированы по количеству рецензий в порядке от большего к меньшему 
  * Результатом должен быть файл в формате ```.csv```
Решение:
  * Импорт всех нужных библиотек 
  * Действуем по той же схеме что и в предыдущем проекте - помещаем все нужные данные в словарь, а словарь помещаем в список ```list_of_data```
```python  
  import requests
  import pandas as pd
  from bs4 import BeautifulSoup

  url = 'https://www.rottentomatoes.com/top/bestofrt/'
  response = requests.get(url).text
  soup = BeautifulSoup(response, 'html.parser')

  list_of_data = []
  clean_data = lambda x: str(BeautifulSoup(x, 'html.parser')).strip()

  for i in range(1,len(soup.find(class_='table').find_all('tr'))):
      score = soup.find(class_='table').find_all('tr')[i].find(class_='tMeterScore').text # score
      title = soup.find(class_='table').find_all('tr')[i].find(class_='unstyled articleLink').text # title
      num_of_views = soup.find(class_='table').find_all('tr')[i].find(class_='right hidden-xs').text # number of views
      list_of_data.append({"Score" : clean_data(score),"Title" : clean_data(title),"No. of Reviews" : clean_data(num_of_views)})
```
* Создаем ```DataFrame``` из списка ```list_of_data```
* Год выпуска фильма содержится в строке с названием фильма, значит, нужно извлечь год выпуска каждого фильма и поместить его в отдельный столбец
* После этого сортируем таблицу по количеству рецензий
```python
  data = pd.DataFrame(data=list_of_data)
  data['Release Year'] = None
  for i in range(len(data['Title'])):
      data['Title'][i] = data['Title'][i].split(' ')
      data['Release Year'][i] = data['Title'][i][-1]
      data['Release Year'][i] = data['Release Year'][i].replace(')', '')
      data['Release Year'][i] = data['Release Year'][i].replace('(', '')
      data['Title'][i] = ' '.join(data['Title'][i][:-1])
  data = data[['Score', 'Title','Release Year', 'No. of Reviews']]

  data['Release Year'] = data['Release Year'].astype(int)
  data['No. of Reviews'] = data['No. of Reviews'].astype(int)
  data.sort_values(by='No. of Reviews', ascending=False)
  data = data.sort_values(by='No. of Reviews', ascending=False)
```
* Создаем ```.csv``` файл со всеми данными
```python
  data.to_csv('rt final result.csv', index=False)
```
  
