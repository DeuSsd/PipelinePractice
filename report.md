Выполняем настройку тестирования в VSCode и производим заупск тестов 

![Screenshot_1.png](screenshots\Screenshot_1.png)

Все тесты пройдены успешно, посмотрим на вывод веб сервера в консоли и видим что были обращения в тестах к веб серверу

![Screenshot_2.png](screenshots\Screenshot_2.png)

Теперь сломаем обработчик запросов у 3х функций изменив url c ```@app.route("/multiply/<int:a>&<int:b>")``` на ```@app.route("/multiply/<a>&<b>")``` 

![Screenshot_3.png](screenshots\Screenshot_3.png)

Перезапустим тестирование и 3 теста должны упасть

![Screenshot_4.png](screenshots\Screenshot_4.png)

В логах сервера видим, что появилась ошибка 500 и 404

![Screenshot_5.png](screenshots\Screenshot_5.png)

В IDE VSCode видим описание упавших тестов

![Screenshot_6.png](screenshots\Screenshot_6.png)

Такой конфиг будет использоваться при создании Пайплайнов

```yaml
# Python package
# Create and test a Python package on multiple Python versions.
# Add steps that analyze code, save the dist with the build record, publish to a PyPI-compatible index, and more:
# https://docs.microsoft.com/azure/devops/pipelines/languages/python

trigger: # в данном блоке определяется, при каком событии запускать пайплайн
- master # запускаем, как только пришел новый коммит в ветку master

pool: # здесь определяем образ докера, в котором запускается приложение и версию интерпритатора
  vmImage: ubuntu-latest # выбираем ubuntu
strategy: # здесь выбираем стек программирования
  matrix: # matrix позволяет запускать параллельные конвейеры, если требуются разные версии
#    Python36: # пока отключаем запуск на 3.6
#      python.version: '3.6'
    Python37: # запускаем на 3.7
      python.version: '3.7'

steps: # здесь указываются шаги конвейера
- task: UsePythonVersion@0 # используем версию питона
  inputs:
    versionSpec: '$(python.version)'
  displayName: 'Use Python $(python.version)'

- script: | # запускаем апдейт питона, устанавливаем зависимости (в нашем случае flask)
    python -m pip install --upgrade pip
    pip install -r requirements.txt
  displayName: 'Install dependencies' # здесь отображается название текущей задачи

- script: | # запускаем юнит тесты
    pip install pytest
    pytest tests/unit_tests && pip install pycmd && py.cleanup tests/
  displayName: 'pytest'

- task: ArchiveFiles@2 # архивируем наш проект чтобы опубликовать артефакт. Артефакт это по сути то, что отдает клиенту (например архив с программой)
  displayName: 'Archive application'
  inputs:
    rootFolderOrFile: '$(System.DefaultWorkingDirectory)/'
    includeRootFolder: false
    archiveFile: '$(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip'

- task: PublishBuildArtifacts@1 # публикуем артефакт как результат нашего пайплайна
  displayName: 'Publish Artifact: drop'
```
На портале *Azure DevOps* выполним подключение гитхаб-репозитория

![Screenshot_7.png](screenshots\Screenshot_7.png)

Наблюдаем созданный пайплайн

![Screenshot_8.png](screenshots\Screenshot_8.png)

Убуеждаемся в корректной ининциализации пайплайна

![Screenshot_9.png](screenshots\Screenshot_9.png)

Заглянув в *artifacts* увидим ресурсы проекта

![Screenshot_10.png](screenshots\Screenshot_10.png)

Теперь вернувшись в пайплайны видим, что он успешно инициализирован

![Screenshot_11.png](screenshots\Screenshot_11.png)

Выполним создание группы ресурсов

![Screenshot_12.png](screenshots\Screenshot_12.png)

Создадим веб приложение

![Screenshot_13.png](screenshots\Screenshot_13.png)
![Screenshot_14.png](screenshots\Screenshot_14.png)

Наблюдаем, что разворачивание приложения выполнено успешно

![Screenshot_15.png](screenshots\Screenshot_15.png)

Выполним создание *Stage* для деплоя приложения

![Screenshot_16.png](screenshots\Screenshot_16.png)

Настроим deploy

![Screenshot_17.png](screenshots\Screenshot_17.png)

Выполним создание релиза и видим, что всё пошло успешно

![Screenshot_18.png](screenshots\Screenshot_18.png)
![Screenshot_19.png](screenshots\Screenshot_19.png)

Перейдём на запущенный сайт ```http://web-service-calc.azurewebsites.net``` и наблюдаем, что всё работает

![Screenshot_20.png](screenshots\Screenshot_20.png)
![Screenshot_21.png](screenshots\Screenshot_21.png)

Создадим ещё один *Stage* для тестирования и произведём её настройку

```shell
pip install selenium pytest
echo $(ChromeWebDriver) 
pytest Agent.HomeDirectory/tests/functional_tests --url http://web-service-calc.azurewebsites.net  --junitxml=TestResults/test-results.xml

```

![Screenshot_22.png](screenshots\Screenshot_22.png)

Перед запуском наблюдаем что не в правильном порядке, поэтому идём изменять порядок запуска стейджев

![Screenshot_23.png](screenshots\Screenshot_23.png)
![Screenshot_24.png](screenshots\Screenshot_24.png)

Теперь производим запуск нового релиза

![Screenshot_25.png](screenshots\Screenshot_25.png)

Видим после запуска что тесты пройдены успешно

![Screenshot_26.png](screenshots\Screenshot_26.png)
![Screenshot_27.png](screenshots\Screenshot_27.png)
![Screenshot_28.png](screenshots\Screenshot_28.png)

Переёдя на вкладку тестирования видим более подробную информацию

![Screenshot_29.png](screenshots\Screenshot_29.png)
![Screenshot_30.png](screenshots\Screenshot_30.png)



По итогу работы с пайплайнами появляется возможность добавить в свой проект данный status badge, позволяющий увидеть состояние пайплайна

[![Build Status](https://dev.azure.com/alekseevap/ognev/_apis/build/status/DeuSsd.PipelinePractice?branchName=master)](https://dev.azure.com/alekseevap/ognev/_build/latest?definitionId=15&branchName=master)

![Screenshot_31.png](screenshots\Screenshot_31.png)
