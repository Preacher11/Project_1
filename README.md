
# Цель проекта:  
Интегрировать модель Luma в игровой движок Unity. Результатом этого должна стать возможность генерировать и импортировать в сцену готовые 3D модели.  

# Практическая часть:  
Для интеграции связи с luma  в игровой движок Unity будем использовать Luma AI Genie Unofficial C# Library. Luma AI Genie — это библиотека C# для взаимодействия с API Luma Labs.  
Она позволяет пользователям создавать 3D-модели с помощью текстовых подсказок.  
Эта библиотека позволит нам работать с Luma в unity, хоть и не напрямую.  
Также, понадобится пакет Unity Web Browser. Unity Web Browser (UWB) — это пакет Unity, который позволяет просматривать веб-страницы и взаимодействовать с ними из Unity.  
Он будет необходим для авторизации и получения токена Luma. Вместе с тем, будет нужен пакет Runtime OBJ Importer.  
Этот пакет позволяет импортировать 3D модели для последующего их форматирования в поддерживаемый формат. Также, chatGPT, который на основе названия объекта будет задавать физические параметры объектов в JSON.  


# Интеграция В Unity  
Основные классы, которые отвечают за связь с Luma, ChatGPT, генерацию и описание параметров объекта.  
Описание класса GenieClient:  

## **🔥 Назначение класса**  

GenieClient — это клиент для взаимодействия с API LumaGenie (Luma Labs), который управляет созданием, проверкой статуса, конвертацией и загрузкой 3D моделей.  
Этот класс инкапсулирует работу с REST API LumaGenie, включая:  
Отправку запросов на генерацию модели  
Проверку статуса генерации  
Запрос на конвертацию  
Загрузку и распаковку ZIP-файлов с моделями  

## **📂 Основные поля**  

private readonly HttpClient _httpClient;  
private string _token;  
_httpClient — для отправки HTTP-запросов.  
_token — токен авторизации для API.  

## **🌐 API URL’ы**  

private const string API_URL = "https://webapp.engineeringlumalabs.com/api/v3/";  
CREATIONS_URL — для создания моделей.  
CONVERT_URL — для конвертации созданных моделей в нужный формат.  
STATUS_URL — для получения статуса генерации по UUID.  

## **✅ Основные методы**  

## 🔑 UpdateToken(string token)  
Позволяет обновить токен авторизации без пересоздания клиента.  

## *🖌 Создание модели*  

public async Task<CreationResponse> CreationRequestAsync(CreationRequest creationRequest)  

Формирует JSON-запрос с настройками генерации.  
Добавляет случайное зерно (seed), если оно не задано — это влияет на уникальность генерации.  
Отправляет POST-запрос с авторизацией.  
Возвращает CreationResponse с UUID созданной модели.  

## 🔄 Запрос на конвертацию  

public async Task<ConvertResponse?> ConvertRequestAsync(ConvertRequest convertRequest)  

Отправляет POST-запрос на конвертацию модели (например, в формат .obj).  
Возвращает ConvertResponse, который содержит ссылку на архив с результатами.  

## 📈 Проверка статуса генерации


public async Task<StatusResponse?> GetStatusRequestAsync(string uuid)  
GET-запрос по UUID.  
Позволяет узнать, завершена ли генерация модели.  

## 📦 Загрузка и распаковка архива   

public async Task<List<(string Name, Stream Content)>> DownloadAndExtractFileAsync(string fileUrl)  

Загружает ZIP-архив по URL.  
Распаковывает его на лету.  
Возвращает список файлов в виде пар (имя файла, поток содержимого).  
Очень полезный метод для интеграции с Unity, где сразу можно взять поток и передать его в загрузчик OBJ или текстур.  


## **🎲 Генерация случайного seed**


public static string GenerateRandomSeed()  
Выдает случайное число в виде строки — используется для генерации уникальных результатов.  

## **⚙ Пример использования цепочки методов**

var genieClient = new GenieClient("your_token");  
var creationResponse = await genieClient.CreationRequestAsync(new CreationRequest("a fantasy castle"));  
var statusResponse = await genieClient.GetStatusRequestAsync(creationResponse.Response[0]);  
var convertResponse = await genieClient.ConvertRequestAsync(new ConvertRequest(creationResponse.Response[0], ExportFormat.Obj));  
var files = await genieClient.DownloadAndExtractFileAsync(convertResponse.Response.UploadedFiles[0].FileUrl);  

#**✅ Вывод — что делает этот класс?**

GenieClient полностью автоматизирует цикл генерации 3D контента:  
Отправка задания на генерацию модели  
Ожидание завершения генерации  
Конвертация результата в нужный формат  
Загрузка и извлечение итоговых файлов  


# GPT

# **Описание кода для интеграции ChatGPT**

## **📚 Назначение класса**

GptClient — это клиент для работы с OpenAI API (GPT-4o-mini). Он предназначен для отправки запросов на генерацию ответов от GPT по предоставленным инструкциям и пользовательскому промпту.

## **🗝 Основные поля**


private const string API_URL = "https://api.openai.com/v1/chat/completions";  
private string _apiKey;  

API_URL — базовый адрес для работы с моделью OpenAI через endpoint /chat/completions.  
_apiKey — API-ключ для авторизации в OpenAI.  

## **✅ Конструктор и вспомогательные методы**

🔑 GptClient(string apiKey)  
Создает клиента и сохраняет переданный API-ключ.  
✏ UpdateKey(string key)  
Позволяет обновить API-ключ во время работы без пересоздания объекта.  

## **🤖 Основной метод генерации**


public async Task<ApiResponse> Completions(string instructions, string prompt)

## *8📥 Входные параметры:*8  

instructions — системные инструкции для модели (как ей себя вести).  
prompt — сообщение пользователя, на которое модель отвечает.

## **📤 Как работает:**

Формирует тело запроса с моделью gpt-4o-mini и структурой сообщений в формате Chat API.  
Устанавливает параметры генерации:  
temperature = 1 — повышенная креативность  
max_tokens = 256 — ограничение на длину ответа  
другие параметры — стандартные  
Добавляет заголовок Authorization с API-ключом.  
Отправляет POST-запрос к OpenAI.  
При успехе — возвращает ApiResponse (распарсенный JSON-ответ).  
В случае ошибки — выводит ошибку в консоль Unity.  

## **🌐 Пример тела JSON-запроса, которое формируется:**
json

{  
  "model": "gpt-4o-mini",  
  "messages": [   
    { "role": "system", "content": [{ "type": "text", "text": "You are a helpful assistant." }] },  
    { "role": "user", "content": [{ "type": "text", "text": "Tell me a joke." }] }  
  ],  
  "temperature": 1,  
  "max_tokens": 256  
}  


## **🧪 Проверка валидности API-ключа**  


public static async Task<bool> VerifyApiKey(string apiKey)  

Отправляет пустой запрос к API.  
Если возвращается BadRequest (400), значит ключ валиден (API его распознал).  
Используется для проверки правильности введенного API-ключа.  


## **⚙ Пример использования**

GptClient gpt = new GptClient("your_openai_api_key");  
var response = await gpt.Completions("Ты помощник", "Придумай название магического предмета.");  
if (response != null) {  
    Debug.Log(response.Choices[0].Message.Content);  
}  


# **🟢 Итог — что делает этот класс**

✅ Отправляет текстовые запросы к OpenAI GPT-4o-mini  
 ✅ Принимает структурированные ответы в виде JSON  
 ✅ Работает асинхронно и готов к использованию в Unity  
 ✅ Позволяет проверять API-ключ на валидность  
 
## **Используемые библиотеки:**

System.Net.Http — для HTTP-запросов  
System.Text — для работы с кодировками  
System.Threading.Tasks — для асинхронности  
Newtonsoft.Json — для сериализации и десериализации JSON  
UnityEngine — для работы с Unity, включая Debug.LogError()  
System.Net — для проверки статуса ответа (HttpStatusCode)  




# **GENERATOR**

# **Описание кода для генерации моделей**

## **Основная функциональность**

ModelGenerator выполняет две главные задачи:  
Генерация 3D модели по текстовому запросу с помощью LumaGenie.  
Создание предмета и его физических характеристик через генерацию JSON-ответа от GPT (OpenAI).  

## **📂 Поля класса**

_material — материал, применяемый к загруженной 3D модели.  
_gptInstructions — инструкции для GPT для генерации JSON-ответа с описанием предмета и его физических свойств.  
OnModelLoaded — событие, вызываемое при загрузке модели.  
IsGenerating — флаг, показывающий, идет ли сейчас процесс генерации.  
_genieClient — клиент для работы с API LumaGenie.  
_gptClient — клиент для работы с OpenAI API.  
_instanceMaterial — экземпляр материала для конкретной модели.  

## **🏗 Инициализация**

В методе Awake():  
Инициализируются клиенты для LumaGenie и GPT на основе токенов из SettingsManager.  
Подписка на событие изменения настроек.  
В OnDestroy() снимается подписка.  
При изменении настроек (OnSettingsChanged()):  
Токены обновляются у обоих клиентов.  

## **✨ Генерация 3D модели**

TryGenerate(string prompt)  
Асинхронная генерация модели:  
Отправка запроса на создание модели (CreationRequestAsync).  
Ожидание завершения генерации (статус Completed).  
Получение и распаковка OBJ-модели и текстур.  
Применение материала и извлечение файлов (OnFileExtracted()).  
Генерация завершается с флагом IsGenerating.  

## **🤖 Генерация описания предмета через GPT**

CreatePromptAsync(string prompt)  
Отправка инструкции и запроса GPT.  
Получение JSON-ответа в формате:  
json  

{  
  "name": "<item name>",  
  "soundMaterial": <material index>,  
  "bounciness": <float>,  
  "dynamicFriction": <float>,  
  "staticFriction": <float>  
}  

Десериализация ответа в объект ItemResponse.  

## **📄 Загрузка и обработка файлов модели**

OnFileExtracted(string fullName, Stream stream)  
В зависимости от расширения:  
**.obj:**  
Загружается 3D модель через OBJLoader.  
Применяется материал.  
Вызывается событие OnModelLoaded.  
**.jpg:**  
Загружаются текстуры.  
Назначаются соответствующие карты материала:  
_BaseMap  
_RoughnessMap  
_MetallicMap  

## **📈 Интеграция с Unity**

Класс ModelGenerator привязан к Unity-объекту:  
Использует SerializeField для настройки через инспектор.  
Работает с MeshRenderer, Material, Texture2D.  
Использует Debug.LogException для отладки.  

# **✅ Вывод — что делает этот класс?**

Этот генератор:

Получает текстовый запрос от пользователя.   
Отправляет его на LumaGenie, получает 3D модель и текстуры.  
Может получить описание физического поведения предмета от GPT в формате JSON.  
Загружает модель в сцену Unity с применением текстур и материалов.  
Вызывает событие OnModelLoaded, чтобы другие компоненты могли использовать сгенерированную модель.  
Системные библиотеки .NET — базовые операции, асинхронность, работа с файлами;  
UnityEngine — работа с 3D-объектами, материалами и сценой;  
Dummiesman — парсинг и загрузка .obj файлов;  

## **Используемые библиотеки**

LumaGenie — API для генерации 3D-моделей;  
OpenAI (через GptClient) — генерация описания предметов с помощью GPT;  
Newtonsoft.Json — парсинг JSON-ответов;  
Проектные менеджеры и модули — SettingsManager для управления API-ключами и токенами.  
