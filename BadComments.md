1. По пункту 7 - избыточный комментарий. Можно удалять обязательный комментарий т.к. из названия и сигнатуры метода понятно его назначение.
   ```
   /// <summary>
   /// Попытка получить текущую позицию цели по BulletMagnet
   /// </summary>
   private bool TryGetBulletMagnetPosition(out Vector3 position)
   {
      ... // Реализация
   }
   ```
   
2. По пункту 4 - шум. В классе с данными юнита есть поле содержащее список орудий. 
   Расположение поля и именование списка говорят сами за себя, комментарий тут лишний, можно удалить.
   ```
   public class Unit : BattleEntity 
   {
        ... // Поля класса

		private List<Weapon> _weapons; // список орудий, которые есть у юнита
        
        ...
   } 
   ```
   
3. По пункту 5 - позиционный маркер. Из класса в п.2 тип Unit также содержит поля отмеченные заголовком `//------------- Информация только для самолетов ---------------------//` 
   только для летающих юнитов. Как минимум тут можно выделить регион, но мне видится лучшим решением выделить тип FlightUnit, унаследовать его от 
   Unit и определить эти поля там:
   Было:
   ```
   //------------- Информация только для самолетов ---------------------//
		private FlightMap _flightMap; // карта полета
		private Building _airport; // домашний аэродром
		private AirportSlot _airportSlot; // место на аэродроме
		private int _bombDx, _bombDy; // интервал в пикселях между бомбами при сбросе
   ```
   Стало:
   ```
   public class FlightUnit : Unit 
		private FlightMap _flightMap; // карта полета
		private Building _airport; // домашний аэродром
		private AirportSlot _airportSlot; // место на аэродроме
		private int _bombDx, _bombDy; // интервал в пикселях между бомбами при сбросе
   ```
   
4. По пункту 4 - шум. Пример из пункта 3 также можно доработать - поля `_flightMap`, `_airport` и `_airportSlot`
   не нуждаются в комментировании:
   ```
		private FlightMap _flightMap; 
		private Building _airport; 
		private AirportSlot _airportSlot; 
		private int _bombDx, _bombDy; // интервал в пикселях между бомбами при сбросе
   ```
   
5. По пункту 7 - избыточный комментарий. В диспетчере ресурсов есть метод проверки доступности контента. 
   Именование и сигнатура метода говорят сами за себя, комментарий избыточен.
   ```
   /// <summary>
   /// Проверить доступность контента конкретной категории (bool)
   /// </summary>
   /// <param name="categoryId">Категория ресурсов</param>
   /// <returns>bool</returns>
   public bool IsContentAvailable(CategoryId categoryId)
   ```
   
6. По пункту 4 - шум. В телеграм боте есть инициализация модуля работы с БД. Комментарий там не несёт никакой пользы и может быть удалён.
   ```
   // Проверяем, инициализирована ли база
   if (!IsInitialized)
       throw new InvalidOperationException(NOT_INITIALIZED_EXCEPTION_MESSAGE);
   ```
   
7. По пункту 11 - закомментированный код. В классе работы с файлом настроек расписания есть метод загрузки файла с расписанием. 
   Там есть закомментированный дебаг, его следует удалить:
   ```
   private List<ScheduleEntryDto>? LoadFromFile(string path)
   {
       // _logger.Invoke(LogLevel.Debug, "Starting to load schedule from file");
   
       ... // Логика загрузки
   }
   ```
   
8. По пункту 2 - бормотание. Для обновления всей бд временно на начальном этапе выделил отдельный метод который из потока извлекал все данные и перезаписывал их в бд. 
   Но были планы переехать с EXCEL формата хранения на SQL. В одном из предыдущих заданий предложил исправление на todo комментарий `//todo Отрефакторить в версии бота 1.2.0`,
   однако его стоит уточнить.
   Было:
   ```
   //todo Отрефакторить в версии бота 1.2.0
    public async ValueTask<bool> TryUpdateDatabaseAsync(Stream stream, CancellationToken cancellationToken)
    {
        ... // Логика апдейта информации о расписании в БД
    }
   ```
   Стало:
   ```
   // TODO: Временный метод для полной перезаписи БД из потока. Необходимо отрефакторить этот метод к версии бота 1.2.0, 
   // заменив логику работы с Excel на работу с базой данных.
    public async ValueTask<bool> TryUpdateDatabaseAsync(Stream stream, CancellationToken cancellationToken)
    {
        ... // Логика апдейта информации о расписании в БД
    }
   ```
   
9. По пункту 12 - можно выделить метод. Есть метод обновления кэша всех категорий ресурсов. в нём есть комментарий описывающий прицип работы части метода. 
   Сам комментарий расположен рядом с переменной `generalTotalBytes`, но его контекст шире чем переменная, да и напрямую в ней не относится. Предлагается вынести 
   логику описанную в комментарии в отдельный метод с говорящим названием, тогда необходимость в комментарии отпадёт.
   Было:
   ```
    public async UniTask<UpdateOperationResult> UpdateAllCache(IProgress<DownloadProgress>? progress = null, CancellationToken cancellationToken)
		{
			... // Валидация входных параметров

			// Проходимся по списку категорий ресурсов и узнаём у каждой категории кэшируется ли она, формируем список кэшируемых категорий
			ulong generalTotalBytes = 0L;

			... // Формируем список тасок на получение размера обновлений
   
            ... // Обрабатываем результаты и формируем ответ
		}
   ```
   Стало:
   ```
   public async UniTask<UpdateOperationResult> UpdateAllCache(IProgress<DownloadProgress> progress, CancellationToken cancellationToken)
		{
			... // Валидация входных параметров


			ulong generalTotalBytes = 0L;
   
            List<ResourceCategory> cacheableCategories = GetCacheableCategories(m_categories);

			... // Формируем список тасок на получение размера обновлений
   
            ... // Обрабатываем результаты и формируем ответ
		}
   
   private List<ResourceCategory> GetCacheableCategories(List<ResourceCategory> allCategories)
        {
           ... 
        }
   ```
   
10. По пункту 4 - шум. В том же методе из примера в п.9 для случая когда ни для одной категории не нашлось объявлений комментарий описывает очевидную логику.
    Такой комментарий можно удалить.
    ```
    if (resourceCategories.Length == 0)
	{
		// если ошибок нет, считаем обновление успешным и неуспешным в ином случае
		bool isSuccess = exceptions.Count == 0;
		AggregateException exception = exceptions.Count > 0 ? new AggregateException(exceptions) : null;
		result = new UpdateOperationResult(isSuccess, exception);   
    
        return result;  
    }
    ```
    
11. По пункту 9 - нелокальная информация. В методе `GetUpdateEstimatedSize` есть комментарий описывающий намерения вынести работу с 
    Unity Addressables в прокси, чтобы проще было тестировать систему управления ресурсами. Однако эта информация шире чем отдельный метод.
    Лучше всего убрать комментарий и создать отдельную задачу на создание прокси объекта.
    ```
    private async UniTask<long> GetUpdateEstimatedSize(ResourceKeys lables, CancellationToken cancellationToken)
	{
        // TODO: Вынести всю работу с системой adrressables в прокси объект
		AsyncOperationHandle<long> downloadSizeHandle = Addressables.GetDownloadSizeAsync(lables.Keys);

        ... // Логика определения размера загрузки
    }
    ```
    
12. По пункту 3 - недостоверный комментарий. В методе исполнения операции получения ассета есть комментарий описывающий 
    предупреждение и некоторое намерение связанное с проблемой утечки памяти. Он не соответствует действительности - в 
    реальности утечка может быть только в случа если у CancellationTokenSource не был вызван Dispose. В остальных случаях 
    всё будет нормально и утечки не произойдёт. Максимум что тут можно сделать это оставить точный предупредительный комментарий:
    Было:
    ```
    public GetAssetOperation<T> ExecuteOperation<T>(IGetAssetOperationData<T> operationData)
		{
			... // Валидация и подготовка операции 
    
			if (operationData.CancellationToken != CancellationToken.None)
			{
				// TODO может возникнуть утечка памяти, если токен не был отменён
				operationData.CancellationToken.Register(() => { ReleaseAsset<T>(operationData.Key); });
			}

			... // Завершение создания операции	
		}
    ```
    Стало:
    ```
    public GetAssetOperation<T> ExecuteOperation<T>(IGetAssetOperationData<T> operationData)
		{
			... // Валидация и подготовка операции 
    
			if (operationData.CancellationToken != CancellationToken.None)
			{
				// Убедитесь, что у CancellationTokenSource которому принадлежит токен вызывается Dispose, иначе может возникнуть утечка памяти
				operationData.CancellationToken.Register(() => { ReleaseAsset<T>(operationData.Key); });
			}

			... // Завершение создания операции	
		}
    ```
13. По пункту 11 - закомментированный код. В методе восстановления видимости юнита есть закомментированный код. 
    Он не актуален поэтому его можно просто удалить.
    ```
    private void RestoreVisibility()
	{
		if (IsAnimationIgnoreInvisibilityEnabled)
		{
			UnityUtils.SetActiveWidgets(_hideWhenInvisibleObjects, true);
		}
		else
		{
			UnityUtils.SetActiveWidgets(_childGameObjects, true);
        }
        
        //	GameObjectCache.SetActive(true);
        //	foreach (GameObject visualizator in _visualizators) {
        //		visualizator.SetActive(true);
        //	}
        }
    ```

14. По пункту 5 - позиционные маркеры. В коде юнита на клиенте есть маркер разделяющий свойства и методы класса. 
    Можно заменить это конструкцией `#region ... #endregion` или вовсе убрать разделитель, он не несёт никакой полезной информации.
    ```
    public UnitVisibleType VisibleType => m_visibleType;

	//=====================================================
	//
	//=====================================================

	/// <summary>
	/// Перевести юнита в режим движения (перемещение или поворот).
	/// </summary>
	public void TranslateToMoveState(UnitMoveMode unitMoveMode)
    ```

15. По пункту 2 - бормотание. В методе перевода юнита в специальное состояние перед логикой есть комментарий с советом о том, 
    как можно оптимизировать код. В целом комментарий довольно дельный, но носит общий характер, непонятно нужно это делать или нет, если да то когда?
    Предлагается переформировать комментарий в todo формат и указать конкретную задачу в которой будет реализована эта оптимизация, 
    а саму оптимизацию уже описать в задаче.
    Было:
    ```
    public void TranslateToSpecState(int duration)
	{
         ... // Валидация состояний юнита

         // Optimize: Отключать те орудия, которые предназначены для обычного режима и включать те, что предназначены для специального режима

		 ... // Обработка перехода в специальное состояние
    }
    ```
    Стало:
    ```
    // todo: Оптимизировать переход в специальное состояние. Задача #TASK-473
    public void TranslateToSpecState(int duration)
    {
         ... // Валидация состояний юнита
    
		 ... // Обработка перехода в специальное состояние
    }
    ```
   