### Материал по реализации REST между 1C-Bitrix и Java (Android)

В тот или иной период времени разработчик упирается в веб-возможности, когда для реализации интернет магазина клиенту приходиться интегрировать мобильное приложение.

Руководствовался решением из битриксовского marketplace [REST API от Дениса Артамонова](http://marketplace.1c-bitrix.ru/solutions/artamonov.api/) 

На выходе данное решение выплевывает `JSON` Object - Array.

Теперь остается реализовать get запрос на стороне мобильного приложения (java), в результате чего мы будем обрабатывать полученные данные. 

У нас есть некий web-сервер с сайтом (https://example.com/), это будет сайт на 1С Битрикс с установленным интернет-магазином. Нам нужно получить некие данные с этого сайта и как-то вывести их у себя в приложении.

#### Json

Имеется ответ от сервера

```sh {
  "status":200,
  "result":
  {
    "DATE":"2018-09-04 09:06:32",
    "REQUEST_METHOD":"GET",
    "IP_ADDRESS":"168.0.0.1",
    "COUNTRY_CODE":"RU",
    "CONTROLLER":"example",
    "ACTION":"check",
    "PARAMETERS":
    {
      "uid":"0001",
    },
    "SECRET":
    	[
          {
            "USER":"1"
          },
          {
            "NAME":"Rishat7c"
          }
        ],
    "OPERATING_METHOD":"OBJECT_ORIENTED"
  }
}
```

У нас появился JSON объект с различными ключами:значениями, и один массив `SECRET`

Для создания геттеров и сеттеров, я использую www.jsonschema2pojo.org/. 
Все просто копируем json ответ от сервера и вставляем его на сайте, выбираем нужные параметры и сайт нам создаст нужную иерархию классов. Быстро и качественно.

#### Java

Для POST/GET запросов использую библиотеку okHTTP3 (https://github.com/square/okhttp). Чтобы добавить библиотеку в наш проект, заходим в build.gradle и добавляем в секцию «dependencies» строку:
> implementation 'com.squareup.okhttp3:okhttp:3.10.0'

и сразу добавим библиотеку GSON

> implementation 'com.google.code.gson:gson:2.8.5'

Выполняем синхронизацию Gradle.

Переходим в класс, в котором будем делать запрос к серверу, для простоты мы будем делать это в MainActivity метод onCreate.

```sh
Request request = new Request.Builder()
                .url("http://example.com/api-mobile/mobile/check/?uid=" + userid)
                .build();
client.newCall(request).enqueue(new Callback() {
                @Override
            public void onFailure(Call call, IOException e) {
                        Log.e("myAPP", e.getMessage()); 
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                if (response.isSuccessful()){ 
                    try {
                        auth = gson.fromJson(response.body().charStream(), Auth.class);
                         Log.i("GSON", "ID: " + auth.result.sECRET.get(0).getUSER()); // Получим '1'
                         Log.i("GSON", "LOGIN: " + auth.result.sECRET.get(1).getNAME()); // Получим 'Rishat7c'
                    } catch (Exception e) {
                        e.printStackTrace();
                    }

                    runOnUiThread(new Runnable() {
                        @Override
                        public void run() {
                            updateUI();
                        }
                    });
                }
            }

        });
```

Маппим ответ от сервера в класс экземпляр класса `Auth`

Вообще okhttp позволяет выполнить запрос двумя методами: execute() и enqueue(). 
Использование enqueue() проще, т.к. по настоятельным рекомендациям Google все внешние запросы должны отделять от UI потока. И тут у нас два пути развития: 
 - использовать execute(), но его нужно будет отделять от UI потока путем AsyncTask'a
 - использовать enqueue() c callbak'ом

### Development
Code and description - [Rishat7c](https://github.com/rishat7c)
