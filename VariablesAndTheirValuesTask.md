1. При запуске бота CancellationTokenSource создаётся в самом начале метода. 
   Перенесём его непосредственно перед запуском асинхронной операции регистрации команд:
   ```
   stopRegistrationProcessCts = new CancellationTokenSource();
   
   _stopBotCts = stopRegistrationProcessCts;
   
   await RegisterCommandsAsync(stopRegistrationProcessCts.Token);
   
   stopRegistrationProcessCts.Dispose();
   _stopBotCts = null;
   ```
   
2. В коде инициализации команд бота из примера 1 также можно упростить процесс диспоуза токена используя конструкцию using вместо "ручного" вызова Dispose:
   ```
   using stopRegistrationProcessCts = new CancellationTokenSource();
   
   _stopBotCts = stopRegistrationProcessCts;
   
   await RegisterCommandsAsync(stopRegistrationProcessCts.Token);
   
   _stopBotCts = null;
   ```

3. В классе SaltedHashTable используется статическая соль которая объявлена в области видимости класса как `private byte[] _salt = { 10, 9, 4, 93};`.
   Поскольку соль не меняется, то имеет смысл объявить поле как readonly: `private readonly byte[] _salt = { 10, 9, 4, 93};`

4. В методе проверки наличия обновлений загружаемых ресурсов есть переменная хранящая результат, которая объявлена в самом начале метода, 
   но присваивается лишь в конце метода после получения результата запроса CheckForUpdates - checkUpdatesResult. Объявим её непосредственно перед использованием.
   ```
   ... // Логика конфигурации запроса и создания токена.
   
   CheckUpdatesResult checkUpdatesResult = default;
      
   try
   {
   	// Узнаём у категории есть ли доступные обновления, передавая ключи бандлов
   	checkUpdatesResult = await targetCategory.CheckForUpdates(resourceKeys, cancelRequestCts.Token);
   }
   catch (Exception exception)
   {
   	  // Логируем исключение
   }
   
   // Логика отправки события
      
   return checkUpdatesResult;
   ```
   
5. Есть набор ошибок которые могут произойти во время сохранения загруженных ресурсов на диск, все такие ошибки логируются с одним и тем же коротким сообщением с описанием ошибки для упрощения фильтрации в логах в Elastic.
   Сообщение представлено переменной в области видимости метода обработки ошибки `string resourceLoadErrorShortMessage = "Disk write error, resource was not saved!";`. 
   Добавим const в выражение и строка больше не сможет быть случайно изменена: `const string resourceLoadErrorShortMessage = "Disk write error, resource was not saved!";`

6. При инициализации фасада работы с ресурсами в конструктор фасада могут прислать вместо конфигурации null. В таком случае используем утилитарный метод который логирует ошибку и выбрасывает исключение:
   `resourcesConfiguration.AssertNotNull("Resource facade configuration can't be null!");`

7. Есть объект, который управляет иерархией окон UI. У объекта есть метод Open, он принимает презентер окна, которое должно быть отображено в иерархии окон.
   В методе есть проверка есть ли уже в иерархии такое окно, значение сохраняется в переменную, но переменная объявляется далеко от места её использования, перенесём её непосредственно перед использованием.
   Было:
   ```
   public void Open([NotNull] WindowPresenter windowPresenter)
   {
       ... // Валидация и поиск активного окна
         
      	bool containWindowPresenter = m_registeredWindows.Contains(windowPresenter);
         
      	if (!containWindowPresenter && windowPresenter.IsOpened)
      	{
      		throw new InvalidOperationException(
      			$"\"{windowPresenter.GetType().Name}\" already opened but is not in hierarchy!");
      	}
         
      	... // Отправка события и сокрытие окна
         
      	if (containWindowPresenter)
      	{
      		m_registeredWindows.Remove(windowPresenter);
      	}
         
      	... // Активация требуемого окна
   }
   ```
   Стало:
   ```
   public void Open([NotNull] WindowPresenter windowPresenter)
   {
       ... // Валидация и поиск активного окна
         
      	bool containWindowPresenter = m_registeredWindows.Contains(windowPresenter);
         
      	if (!containWindowPresenter && windowPresenter.IsOpened)
      	{
      		throw new InvalidOperationException(
      			$"\"{windowPresenter.GetType().Name}\" already opened but is not in hierarchy!");
      	}
   
        if (containWindowPresenter)
      	{
      		m_registeredWindows.Remove(windowPresenter);
      	}
   
        containWindowPresenter = false;
         
      	... // Отправка события и сокрытие окна
         
      	... // Активация требуемого окна
   }
   ```
   
8. Есть утилитарный метод рейкаста в игровом пространстве. В методе создаются переменные `ray` и `hit`, но не обнуляются после использования. Добавим обнуление этих переменных после использования.
   Было:
   ```
   public static void RaycastDownOnSurface(float x, float z, BattleMap battleMap, out SurfaceRaycastHit surfaceHit)
   {
   	    Ray ray = new Ray(new Vector3(x, 1000f, z), Vector3.down);
   	    RaycastHit hit;
      
        Physics.Raycast(ray, out hit, Mathf.Infinity, LayerManager.LayerSurfaceMask)

        ... // Заполнение surfaceHit
   
        ... // Корректировка surfaceHit согласно параметрам логического игрового поля
    }
   ```
   Стало:
   ```
   public static void RaycastDownOnSurface(float x, float z, BattleMap battleMap, out SurfaceRaycastHit surfaceHit)
   {
   	    Ray ray = new Ray(new Vector3(x, 1000f, z), Vector3.down);
      
        Physics.Raycast(ray, out RaycastHit hit, Mathf.Infinity, LayerManager.LayerSurfaceMask)

        ... // Заполнение surfaceHit
   
        ray = null;
        hit = default;
   
        ... // Корректировка surfaceHit согласно параметрам логического игрового поля
    }
   ```
   
9. Есть метод получения игрового объекта по координатам экрана. Существует путь исполнения (ltajknysq rtqc) после которого `hit` может быть использован невалидно. Принудительно сбросим его значение:
   ```
   public static bool TryGetGameObjectByScreenPoint(
			Vector2 screenPoint,
			int layerMask,
			out BattlefieldRaycastHit battlefieldHit)
		{
			Ray ray = ClientBattleContext.Screen.BattlefieldCamera.CachedCamera.ScreenPointToRay(screenPoint);

			if (Physics.Raycast(ray, out RaycastHit hit, Mathf.Infinity, layerMask))
			{
				GameObject go = hit.collider.gameObject;

				switch (go.layer)
				{
					case LayerManager.LayerTerrain:
					case LayerManager.LayerWater:
						...
						return true;
					case LayerManager.LayerUnit:
					case LayerManager.LayerBuilding:
						..
						return true;
					default:
						Debug.Fail("Unexpected layer (" + go.layer + ")"); // Тут может пройти дальше и hit будет не валиден
						break;
				}
			}
   
            hit = default; // Сбрасываем значение
			
            ... // Заполнение значений в случае если не попали лучом
   
			return false;
		}
   ```
   
10. В методе конвертации угла из логической системы координат в игровую(мировую). Вынесем магическое число 360 в константу:
    Было:
    ```
		public static short LogicAngleToWorldAngle(short logicAngle)
		{
			if (logicAngle == 0)
			{
				return 0;
			}
    
			return (short)(360 - logicAngle);
		}
    ```    

    Стало: 
    ```
		public static short LogicAngleToWorldAngle(short logicAngle)
		{
            const short FULL_CIRCLE_DEG = 360;
    
			return logicAngle == 0 ? 0 : (short)(FULL_CIRCLE_DEG - logicAngle);
		}
    ```
    
11. В методе поиска координат коллайдера на экране добавим обнуление переменной :
    Было:
    ```
    public static Recti GetColliderScreenBounds([NotNull] Collider collider)
		{
			Camera battlefieldCamera = ClientBattleContext.Screen.BattlefieldCamera.CachedCamera;
			Bounds colliderBounds = collider.bounds;
			Vector3 center = colliderBounds.center;
			Vector3 extents = colliderBounds.extents;
    
            // Заполнение объекта типа Recti с экранными координатами
		}
    ```
    
    Стало:
    ```
    public static Recti GetColliderScreenBounds([NotNull] Collider collider)
		{
			Camera battlefieldCamera = ClientBattleContext.Screen.BattlefieldCamera.CachedCamera;
			Bounds colliderBounds = collider.bounds;
			Vector3 center = colliderBounds.center;
			Vector3 extents = colliderBounds.extents;
    
            colliderBounds = default; // Сбрасываем значение colliderBounds
    
            // Заполнение объекта типа Recti с экранными координатами
		}
    ```

12. Есть класс отвечающий за селект юнитов на поле боя. Есть метод, который ставит селект на отдельного юнита. 
    Добавим ассерт для проверки того что при селекте отдельного юнита сбрасывается селект с остальных выделённых юнитов:
    ```
    public void SelectUnit(Unit unit))
	{
        if (_selectionType == UnitSelectionType.AddToSelection)
		{
			ClearSelection();
		}

		GGDebug.Assert(_selectedUnits.Capacity == 0, "Selection must be empty before selecting new unit");
        
        ... // Логика селекта юнита
    }
    ```
    
13. Есть метод загрузки расписания для бота из файла на диске. Добавим обнуление ссылки на `scheduleFile` т.к. после получения пакета он больше не нужен:
    ```
    private IList<ScheduleEntry> LoadFromFile(string fullPath)
    {
        FileInfo scheduleFile = new FileInfo(fullPath);
        using var scheduleExcelPachage = new ExcelPackage(scheduleFile);
    
        scheduleFile = null;

        ... // Загрузка и парсинг расписания
                
        return scheduleEntries;
    }
    ```
    
14. Есть метод регистрации уведомления о начале соревнования. Мы не можем добавлять 2 уведомления на 1 соревнование - при попытке такой операции метод не будет добавлять уведомление, хотя такое поведение ошибочно.
    Добавим отправку уведомления в лог о невалидном действии.
    ```
    public void Register(int competitionId, DateTime matchingTime, string battleName)
		{
			if (!IsInitialized)
			{
				return;
			}

			if (Exist(competitionId))
			{
				Debug.Error($"Competition notification with id={id} already exist!"); // Добавили явное логирование
                return;
			}

			... // Логика добавления уведомления.
		}
    ```

15. Есть метод обработки события выбора юнита. Добавляем ассерт, что юнит селектится только у игрока чьей стороне принадлежит юнит:
    ```
    public void OnSelected()
		{
			GGDebug.Assert(BattleEntityUtils.IsOwnSide(_unitBattleSide));
    
			_unitState |= ClientUnitStates.Selected;
			ChangeIndicators();
		}
    ```

