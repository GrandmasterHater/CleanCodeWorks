1. Есть код, который создает и настраивает кнопки интерфейса для отображения юнитов, еще не готовых к использованию. В нём переменные unitType и buildingType создаются гораздо раньше чем используются. 
   Исправим это, перенесём переменные к месту использования, рядом с вызовом `Initialize`:
   Было:
   ```
   for (int i = 0; i < notReadyUnitsSorted.Count; i++)
   {
       UnitType unitType = notReadyUnitsSorted[i];
       BuildingType buildingType = m_notReadyUnits[unitType];
       
       GUIBattleSpawnOrderButton constructionItem;
       
       if (constructionItemIdx < m_items.Count)
       {
           constructionItem = m_items[constructionItemIdx];
       }
       else
       {
           constructionItem = CreateButton();
           m_items.Add(constructionItem);
       }
       
       constructionItem.gameObject.SetActive(true);
       constructionItem.Initialize(unitType, buildingType);
       ++constructionItemIdx;
   }
   ```
   Стало:
   ```
   for (int i = 0; i < notReadyUnitsSorted.Count; i++)
   {
       UnitType unitType = notReadyUnitsSorted[i];
       BuildingType buildingType = m_notReadyUnits[unitType];
       
       GUIBattleSpawnOrderButton constructionItem;
       
       if (constructionItemIdx < m_items.Count)
       {
           constructionItem = m_items[constructionItemIdx];
       }
       else
       {
           constructionItem = CreateButton();
           m_items.Add(constructionItem);
       }
       
       constructionItem.gameObject.SetActive(true);
       constructionItem.Initialize(unitType, buildingType);
       ++constructionItemIdx;
   }
   ```
   
2. В коде из примера один также можно сузить область видимости переменной-счётчика `constructionItemIdx` и включить её инициализацию в цикл.
   Было:
   ```
   int constructionItemIdx = 0;
      
   for (int index = 0; index < m_listUnitBuildinPairs.Count; ++index)
   {
   	// код настройки кнопок интерфейса
    ++сonstructionItemIdx;
   }
   ```
   Стало:
   ```
    for (int index = 0, constructionItemIdx = 0; index < m_listUnitBuildinPairs.Count; ++index, ++constructionItemIdx)
   {
   	// код настройки кнопок интерфейса
   }
   ```
   
3. Есть объект, который управляет иерархией окон UI. У объекта есть метод Open, он принимает презентер окна, которое должно быть отображено в иерархии окон.
   В методе есть проверка есть ли уже в иерархии такое окно, значение сохраняется в переменную, но переменная объявляется далеко от места её использования, перенесём её непосредственно перед использованием.
   Было:
   ```
   public void Open([NotNull] WindowPresenter windowPresenter)
   {
       ... // Валидация и поиск активного окна
      	bool containWindowPresenter = m_registeredWindows.Contains(windowPresenter);

      	if (!containWindowPresenter && windowPresenter.IsOpened)
      	{
      		throw new InvalidOperationException("\"{windowPresenter.GetType().Name}\" already opened but is not in hierarchy!");
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
      		throw new InvalidOperationException("\"{windowPresenter.GetType().Name}\" already opened but is not in hierarchy!");
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
   
4. Есть код который выбирает эффект которым будет подсвечен выбранный юнит. Есть проверка, что юнит это трансформируемый герой. 
   Сузим область видимости переменной перенеся `isHeroTransformable` внутрь условия `if (_clientUnit.Prototype.Hero)`:
   Было:
   ```
   private SelectionPrefabs UnitSelection(ActionPrefabCollection actions, HeroSelectionPrefabCollection heroActions)
		{
            SelectionPrefabs selectionPrefabs = new SelectionPrefabs();
   
			bool isHeroTransformable = _clientUnit.Prototype.Type == UnitType.UNIT_TYPE_TRANSFORMABLE;

			if (_clientUnit.Prototype.Hero)
			{
				HeroSelectionItems selectionTap =
					isHeroTransformable ? heroActions.HeroSelectionTap : heroActions.HeroSiegeModeSelectionTap;
				HeroSelectionItems selection =
					isHeroTransformable ? heroActions.HeroSelection : heroActions.HeroSiegeModeSelection;

				... // Логика установки префаба
			}
			else
			{
				... // Логика установки префаба
			}

			return selectionPrefabs;
		}
   ```
   Стало:
   ```
   private SelectionPrefabs UnitSelection(ActionPrefabCollection actions, HeroSelectionPrefabCollection heroActions)
		{
            SelectionPrefabs selectionPrefabs = new SelectionPrefabs();

			if (_clientUnit.Prototype.Hero)
			{
                bool isHeroTransformable = _clientUnit.Prototype.Type == UnitType.UNIT_TYPE_TRANSFORMABLE;
   
				HeroSelectionItems selectionTap =
					isHeroTransformable ? heroActions.HeroSelectionTap : heroActions.HeroSiegeModeSelectionTap;
				HeroSelectionItems selection =
					isHeroTransformable ? heroActions.HeroSelection : heroActions.HeroSiegeModeSelection;

				... // Логика установки префаба
			}
			else
			{
				... // Логика установки префаба
			}

			return selectionPrefabs;
}

5. Есть класс, который отвечает за отображение иконок на юнитах. Есть метод AddEffect, через который добавляются эффекты.
   Стало:
   ```
   public void AddEffect(BuffType effect)
		{
            bool isAllySide = BattleEntityUtils.IsAllySide(_side);
   
			if (effect == null)
			{
				return;
			}
   
            int effectType = effect.Type;

			switch (effectType) 
			{
				case BUFF when isAllySide:
					SafeAddEffect(effect, _buffEffects);
					break;
				case DEBUFF:
					SafeAddEffect(effect, _debuffEffects);
					break;
			}

			if (_contextMenu != null)
			{
				_contextMenu.UpdateBuffsGrisPosition();
			}
   
            OnNewBuffAdded.Invoke(effect);
		}
   ```
   Стало:
   ```
   public void AddEffect(BuffType effect)
		{
            bool isAllySide = BattleEntityUtils.IsAllySide(_side);
   
			if (effect == null)
			{
				return;
			}

			switch (effect.Type) // Используем без лишней переменной
			{
				case BUFF when isAllySide:
					SafeAddEffect(effect, _buffEffects);
					break;
				case DEBUFF:
					SafeAddEffect(effect, _debuffEffects);
					break;
			}

			if (_contextMenu != null)
			{
				_contextMenu.UpdateBuffsGrisPosition();
			}
   
            OnNewBuffAdded.Invoke(effect);
		}
   ```

6. В примере выше описано добавление эффектов. Еще улучшим код - вынесем проверку isAllySide в свойство т.к. она используется в нескольких местах:
   ```
   public void AddEffect(BuffType effect)
		{
            bool isAllySide = BattleEntityUtils.IsAllySide(_side);
   
			if (effect == null)
			{
				return;
			}

			switch (effect.Type)
			{
				case BUFF when isAllySide:
					SafeAddEffect(effect, m_buffEffects);
					break;
				case DEBUFF:
					SafeAddEffect(effect, m_debuffEffects);
					break;
			}

			if (_contextMenu != null)
			{
				_contextMenu.UpdateBuffsGrisPosition();
			}
   
            OnNewBuffAdded.Invoke(effect);
		}
   ```
   Стало:
   ```
   private bool IsAllySide => BattleEntityUtils.IsAllySide(_side);
   
   public void AddEffect(BuffType effect)
		{
			if (effect == null)
			{
				return;
			}

			switch (effect.Type)
			{
				case BUFF when IsAllySide:
					SafeAddEffect(effect, m_buffEffects);
					break;
				case DEBUFF:
					SafeAddEffect(effect, m_debuffEffects);
					break;
			}

			if (_contextMenu != null)
			{
				_contextMenu.UpdateBuffsGrisPosition();
			}
   
            OnNewBuffAdded.Invoke(effect);
		}
   ```

7. Есть метод предрасчёта маршрута бомбардировщика по координатам. В следующих пунктах постараемся его улучшить:
   ```
   public virtual void PrecalculateRoute(UnitStateType unitState, int bonusPercent, int numberOfBombers, int startX, int startY, int targetX, int targetY)
   {
        const int ROUTE_LENGTH = 2;
        const int FULL_CIRCLE_DEGREES = 360;
        const int QUARTER_TURN_DEGREES = 90;
        const int PERCENTAGE_FACTOR = 100;
        
        routeLength = ROUTE_LENGTH;
        
        WeaponType mainWeapon = unitState.Weapons[0];
        int numberOfBombs = mainWeapon.ShotCount;
        _bombCount = numberOfBombs;
        
        int flightSpeed = unitState.Flight * (PERCENTAGE_FACTOR + bonusPercent) / PERCENTAGE_FACTOR;
        
        int singleBombInterval = (mainWeapon.ShotTick[1] - mainWeapon.ShotTick[0]) * flightSpeed;
        int fullBombSequenceLength = (mainWeapon.ShotTick[numberOfBombs - 1] - mainWeapon.ShotTick[0]) * flightSpeed;
        
        int deltaX = targetX - startX;
        int deltaY = targetY - startY;
        
        int orientation = CheckAndCalc.calcOrient(battle.MiscData, deltaX, deltaY);
        Coordinate bombStepVector = CheckAndCalc.calcVector(battle.MiscData, singleBombInterval, orientation);
        bombDistance = bombStepVector;
        
        int orientation90 = (orientation + QUARTER_TURN_DEGREES) % FULL_CIRCLE_DEGREES;
        int orientation180 = (orientation90 + QUARTER_TURN_DEGREES) % FULL_CIRCLE_DEGREES;
        int orientation270 = (orientation180 + QUARTER_TURN_DEGREES) % FULL_CIRCLE_DEGREES;
        
        Coordinate centerOffsetVector = CheckAndCalc.calcVector(battle.MiscData, fullBombSequenceLength / 2, orientation180);
        Coordinate lateralOffsetVector = CheckAndCalc.calcVector(battle.MiscData, singleBombInterval * (numberOfBombers - 1) / 2, orientation90);
        
        int firstBomberX = startX + lateralOffsetVector.X;
        int firstBomberY = startY + lateralOffsetVector.Y;
        
        int lastBomberX = targetX + lateralOffsetVector.X;
        int lastBomberY = targetY + lateralOffsetVector.Y;
        
        _baseRoute[0] = new Coordinate(firstBomberX, firstBomberY);
        _baseRoute[1] = new Coordinate(lastBomberX, lastBomberY);
        
        int centerBombX = (firstBomberX + lastBomberX) / 2 + centerOffsetVector.X;
        int centerBombY = (firstBomberY + lastBomberY) / 2 + centerOffsetVector.Y;
        
        baseBombPositions[0] = new Coordinate(centerBombX, centerBombY);
        
        for (int i = 1; i < numberOfBombs; i++)
        {
            baseBombPositions[i] = new Coordinate(
                baseBombPositions[i - 1].X + bombStepVector.X,
                baseBombPositions[i - 1].Y + bombStepVector.Y
            );
        }

        routeShift = CheckAndCalc.calcVector(battle.MiscData, singleBombInterval, orientation270);
   }
   ```
   
   Переменная `numberOfBombs` присваивается переменной класса `_bombCount` и позже используется лишь раз при расчёте `fullBombSequenceLength`. Также помести присвоение непосредственно перед использованием в методе.
   Её можно убрать, в итоге получим:
   ```
   public virtual void PrecalculateRoute(UnitStateType unitState, int bonusPercent, int numberOfBombers, int startX, int startY, int targetX, int targetY)
   {
       ... 
       _bombCount = mainWeapon.ShotCount;
       int fullBombSequenceLength = (mainWeapon.ShotTick[numberOfBombs - 1] - mainWeapon.ShotTick[0]) * flightSpeed;
       ...
   }
   ```
   
8. Из метода в примере 7. Для работы с ориентациями добавим структуру `Orientation`, в которую инкапсулируем всю логику расчёта ориентаций, таким образом заменим 4 переменных одной:
   
   ```
   private struct Orientation
   {
        private const int FULL_CIRCLE_DEGREES = 360;
        private const int QUARTER_TURN_DEGREES = 90;
        
        public int ZeroDegrees { get; }      
        public int 90Degrees { get; }        
        public int 180Degrees { get; }       
        public int 270Degrees { get; }       
        
        public OrientationSet(Battle battle, int startX, int startY, int targetX, int targetY)
        {
            int deltaX = targetX - startX;
            int deltaY = targetY - startY;
   
            ZeroDegrees = CheckAndCalc.calcOrient(battle.MiscData, deltaX, deltaY); 
            90Degrees = (ZeroDegrees + QUARTER_TURN_DEGREES) % FULL_CIRCLE_DEGREES;
            180Degrees = (90Degrees + QUARTER_TURN_DEGREES) % FULL_CIRCLE_DEGREES;
            270Degrees = (180Degrees + QUARTER_TURN_DEGREES) % FULL_CIRCLE_DEGREES;
        }
   }
   
   public virtual void PrecalculateRoute(UnitStateType unitState, int bonusPercent, int numberOfBombers, int startX, int startY, int targetX, int targetY)
   {
        ...
        
        int orientation = new Orientation(battle.MiscData, deltaX, deltaY);
        
        ... // тут используется orientation.ZeroDegrees
        
        Coordinate centerOffsetVector = CheckAndCalc.calcVector(battle.MiscData, fullBombSequenceLength / 2, orientation.180Degrees);
        Coordinate lateralOffsetVector = CheckAndCalc.calcVector(battle.MiscData, singleBombInterval * (numberOfBombers - 1) / 2, orientation.90Degrees);
        
        ...

        routeShift = CheckAndCalc.calcVector(battle.MiscData, singleBombInterval, orientation.270Degrees);
   }
   ```
9. Из метода в примере 7. Еще немного упростим код вынеся расчёт _baseRoute[0] и _baseRoute[1] в отдельный метод, который принимает объект Battle и координаты, а возвращает объект Coordinate:

   ```
   private Coordinate CalculateBaseRoute(int x, int y, Coordinate lateralOffsetVector)
   {
      return new Coordinate(x + lateralOffsetVector.X, y + lateralOffsetVector.Y);
   }      
   
   public virtual void PrecalculateRoute(UnitStateType unitState, int bonusPercent, int numberOfBombers, int startX, int startY, int targetX, int targetY)
   {
        ...
        
        _baseRoute[0] = CalculateBaseRoute(startX, startY, lateralOffsetVector);
        _baseRoute[1] = CalculateBaseRoute(targetX, targetY, lateralOffsetVector);
        
        ...
   }
   ```
   
10. Из метода в примере 7. Выделим расчёт _baseBombPositions[0] в отдельный метод:
    ```
    private Coordinate[] CalculateBaseBombPositions(Coordinate firstBomberPosition, Coordinate lastBomberPosition, Coordinate centerOffsetVector, Coordinate bombStepVector, int bombCount)
    {
    
        int centerBombX = (firstBomberPosition.X + lastBomberPosition.X) / 2 + centerOffsetVector.X;
        int centerBombY = (firstBomberPosition.Y + lastBomberPosition.Y) / 2 + centerOffsetVector.Y;
        Coordinate centerBombPosition = new Coordinate(centerBombX, centerBombY);
        
        // Заполняем массив бомбовых позиций
        Coordinate[] positions = new Coordinate[bombCount];
        positions[0] = centerBombPosition;
        
        for (int i = 1; i < bombCount; i++)
        {
            positions[i] = new Coordinate(
                positions[i - 1].X + bombStepVector.X,
                positions[i - 1].Y + bombStepVector.Y
            );
        }
        
        return positions;
    }       
    
    
    public virtual void PrecalculateRoute(UnitStateType unitState, int bonusPercent, int numberOfBombers, int startX, int startY, int targetX, int targetY)
    {
         ...
    
         _baseBombPositions[0] = CalculateBaseBombPositions(_baseRoute[0], _baseRoute[1], centerOffsetVector, bombStepVector, _bombCount); 
         ...
    }
    ```
    
11. Из метода в примере 7. Избавимся от `bombStepVector` т.к. есть глобальная переменная хранящая то же значение:
    ```
    public virtual void PrecalculateRoute(UnitStateType unitState, int bonusPercent, int numberOfBombers, int startX, int startY, int targetX, int targetY)
    {
         ...
    
          _bombDistance = CheckAndCalc.calcVector(battle.MiscData, singleBombInterval, orientation.ZeroDegrees);
    
         ...

         _baseBombPositions[0] = CalculateBaseBombPositions(_baseRoute[0], _baseRoute[1], centerOffsetVector, _bombDistance, _bombCount);
         
         ...
    }
    ```
    
12. Из метода в примере 7. Переменную `fullBombSequenceLength` перенесем ближе к месту расчёта `centerOffsetVector` т.к. она используется только там. Также обе переменные перенесём ближе к месту где они требуются, 
    а именно расчёту _baseBombPositions:
    ```
    public virtual void PrecalculateRoute(UnitStateType unitState, int bonusPercent, int numberOfBombers, int startX, int startY, int targetX, int targetY)
    {
         ...
    
         int fullBombSequenceLength = (mainWeapon.ShotTick[_bombCount - 1] - mainWeapon.ShotTick[0]) * flightSpeed;
         Coordinate centerOffsetVector = CheckAndCalc.calcVector(battle.MiscData, fullBombSequenceLength / 2, orientation.Degrees180);
    
         baseBombPositions = CalculateBaseBombPositions(_baseRoute[0], _baseRoute[1], centerOffsetVector, _bombDistance, _bombCount);
         
         ...
    }
    ```
    
13. Из метода в примере 7. Рассчёты `flightSpeed` и `singleBombInterval` перенесм ближе к первому месту использования, а именно расчёту `_bombDistance`:
    ```
    public virtual void PrecalculateRoute(UnitStateType unitState, int bonusPercent, int numberOfBombers, int startX, int startY, int targetX, int targetY)
    {
         ...
    
         int flightSpeed = unitState.Flight * (PERCENTAGE_FACTOR + bonusPercent) / PERCENTAGE_FACTOR;
         int singleBombInterval = (mainWeapon.ShotTick[1] - mainWeapon.ShotTick[0]) * flightSpeed;
         OrientationSet orientation = new Orientation(battle, deltaX, deltaY);
         
         _bombDistance = CheckAndCalc.calcVector(battle.MiscData, singleBombInterval, orientation.ZeroDegrees);
         
         ...
    }
    ```
    
    В итоге полная версия метода будет выглядеть следующим образом:
    ```
    public void PrecalculateRoute(UnitStateType unitState, int bonusPercent, int numberOfBombers, int startX, int startY, int targetX, int targetY)
    {
         WeaponType mainWeapon = unitState.Weapons[0];

         _bombCount = mainWeapon.ShotCount;
         
         int flightSpeed = unitState.Flight * (PERCENTAGE_FACTOR + bonusPercent) / PERCENTAGE_FACTOR;
         int singleBombInterval = (mainWeapon.ShotTick[1] - mainWeapon.ShotTick[0]) * flightSpeed;
         Orientation orientation = new Orientation(battle, targetX, targetY, startX, startY);
         
         _bombDistance = CheckAndCalc.calcVector(battle.MiscData, singleBombInterval, orientation.ZeroDegrees);
         
         Coordinate lateralOffsetVector = CheckAndCalc.calcVector(battle.MiscData, singleBombInterval * (numberOfBombers - 1) / 2, orientation.Degrees90);
         
         _baseRoute[0] = CalculateBaseRoute(startX, startY, lateralOffsetVector);
         _baseRoute[1] = CalculateBaseRoute(targetX, targetY, lateralOffsetVector);
         
         int fullBombSequenceLength = (mainWeapon.ShotTick[_bombCount - 1] - mainWeapon.ShotTick[0]) * flightSpeed;
         Coordinate centerOffsetVector = CheckAndCalc.calcVector(battle.MiscData, fullBombSequenceLength / 2, orientation.Degrees180);
         
         _baseBombPositions = CalculateBaseBombPositions(_baseRoute[0], _baseRoute[1], centerOffsetVector, _bombDistance, _bombCount);
         
         _routeShift = CheckAndCalc.calcVector(battle.MiscData, singleBombInterval, orientation.Degrees270);
    }
    ```
    
    Сокращение количества локальных переменных, выделение методов и структуры улучшают выразительность кода и упрощают его чтение.

14. Есть метод перерасчёта карты полёта. Снизим область видимости переменной `currentStep` до уровня цикла:
    Было:
    ```
    public static void RecalculateFlightMap(Battle battle, Unit unit)
    {
        if (unit.Side.PlaneNoEnergyFlightDecrPerc == 0)
           return;
    
        FlightMap flightMap = unit.FlightMap;
        FlightSegment flightSegment;
        int currentStep = flightMap.StepCurr;
    
        for (int segmentIndex = flightMap.SegCurr; segmentIndex < flightMap.Segments.Count; segmentIndex++)
        {
            flightSegment = flightMap.Segments[segmentIndex];
            flightSegment.recalcOnVelocityDecr(battle, unit, currentStep);
            currentStep = 0;
        }
    }
    ```
    Стало:
    ```
    public static void RecalculateFlightMap(Battle battle, Unit unit)
    {
        if (unit.Side.PlaneNoEnergyFlightDecrPerc == 0)
           return;
    
        FlightMap flightMap = unit.FlightMap;
        FlightSegment flightSegment;
    
        for (int segmentIndex = flightMap.SegCurr, currentStep = flightMap.StepCurr; segmentIndex < flightMap.Segments.Count; segmentIndex++, currentStep = 0)
        {
            flightSegment = flightMap.Segments[segmentIndex];
            flightSegment.recalcOnVelocityDecr(battle, unit, currentStep);
        }
    }
    ```
15. Доработаем метод перерасчёта карты полёта из примера 14. Сузим область видимости переменной `flightSegment` до уровня цикла:
    ```
    public static void RecalculateFlightMap(Battle battle, Unit unit)
    {
        if (unit.Side.PlaneNoEnergyFlightDecrPerc == 0)
           return;
    
        FlightMap flightMap = unit.FlightMap;
    
        for (int segmentIndex = flightMap.SegCurr, currentStep = flightMap.StepCurr; segmentIndex < flightMap.Segments.Count; segmentIndex++, currentStep = 0)
        {
            FlightSegment flightSegment = flightMap.Segments[segmentIndex];
            flightSegment.recalcOnVelocityDecr(battle, unit, currentStep);
        }
    }
    ```