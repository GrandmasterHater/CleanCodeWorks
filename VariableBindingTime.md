1. Cвязывание времени компиляции. - Константа `private const string BOT_API_TOKEN = "...";` используется при создании и запуске объекта телеграм бота. 
   Так как бот создавался как простой учебный и не требовал особого уровня защиты решено было выделить апи токен в константу.
   В более сложном случае можно было бы использовать файл конфигурации `appsettings.json`.

   ```
   private const string BOT_API_TOKEN = "...";
        
   private async static Task Main(string[] args)
   {
       ... // Инициализация объектов перед созданием бота 
    
       SсheduleBot bot = new SсheduleBot(BOT_API_TOKEN, scheduleDispatcher, Logger);      
   
       ...
   }
   ```

2. Связывание времени выполнения. - Есть метод, который занимается загрузкой ассетов из облака. Есть переменная в которую записывается url текущего CDN `URL cdnUrl = _cdnUrlsConfig.MainUrl;`. 
   Конфигурация может быть обновлена по команде от сервера, так сделано чтобы была возможность сменить основной и резервный CDN без перевыпуска клиента. 
   
   ```
   public  AsyncOperationHandle<TObject> LoadAssetAsync<TObject>(object key)
   {
      ... // Валидация ключа
   
      URL cdnUrl = _cdnUrlsConfig.MainUrl;
   
      ... // Попытка загрузки по URL и доп попытка по резервному
   }
   
   ```

3. Связывание при написании. - К примеру при инициализации контейнера уведомлений список для хранения уведомлений присваивается явно сразу переменной `_activityNotifications`. 
   Для хранения списка переменных не требуется сложной конфигурации поэтому можно сразу проинициализировать переменную нужным значением:
   
   ```
   private readonly List<ActivityNotification> _activityNotifications;
   
   public ActivityNotificationContainer(
			[NotNull] INotificationsLoader notificationsLoader,
			[NotNull] CompetitionConfiguration competitionConfiguration,
			[NotNull] ILocalizationStringsLoader localizationStringsLoader)
		{
			... // присвоение переменных
			_activityNotifications = new List<ActivityNotification>(DEFAULT_LIST_CAPACITY);
		}
   ```