### 3.1

1. Метод получения описания способности юнита-героя. 
   ```
   private AbilityDescription GetDescription(UpgradeLevel level, UnitType unitType, ILocalizationStringsLoader localizationStringsLoader, int abilityIndex)
		{
            // Проверяем что level содержит апгрейд способности, а не самого юнита и что он принадлежит именно герою
			if (level.Owner is not UpgradeUnit upgradeUnit || upgradeUnit.Owner == null || !upgradeUnit.Owner.Hero)
			{
				return null;
			}

			... // Поиск описания
		}
   ```
   
2. В методе наведения орудий при расчёте угла наведения необходимо делать коррекцию угла через проекцию вектора направления орудия на плоскость.
   ```
   // Если ось direction будет в направлении ствола юнита, то проекция цели на плоскость, образованную direction, orientation и M3
   // не всегда будет представлять реальное положение цели. Поэтому, для корректного вычисления угла поворота, нужно повернуть сам direction
   // по оси orientation в направлении цели с сохранением положения. 
   direction = AdjustDirectionToTargetOnAxis(direction, Weapon.Target.TargetPosition, orientation);
   ```
   
3. Есть сложное условие проверки может ли юнит быстро реагировать, к нему стоит добавить комментарий:
   ```
   // Разрешаем быструю реакцию, если юнит находится между клетками, не в прыжке, отправлен не в текущую клетку и находится недалеко от клетки на которой находился юнит (уже отошел от действия предыдущей быстрой реакции)
   if (unit.CurrentStep != 0 && !unit.hasFlag(UnitFlags.FLAG_JUMPING) && unit.Distance > unit.Undergo && (x2 != unit.X || y2 != unit.Y) && (unit.Dx < BattleCell.Size && unit.Dx > -BattleCell.Size && unit.Dy < BattleCell.Size && unit.Dy > -BattleCell.Size)) {
   	   unit.FastResponse();
   }
   ```
   
4. В методе расчёта маршрута юнита есть проверка на выбор направления обхода:
   ```
   // В зависимости от того, с какой стороны от направления на центр лежит направление на цель, выбираем ближайший обход
   if (MathUtils.isOrientBetween(targetOrient, startToCenterOrient, (startToCenterOrient + tangentOffset) % 360)) 
   {
      ... // Выбираем обход в ту же сторону куда двигаемся
   } 
   else 
   {
      ... // Выбираем обход в противоположную сторону
   }
   ```
   
5. Уточняем комментарием причину установки нулевого таймаута для платформы:
   ```
   // Eсли это IOS сборка запущенная на MacOS, то отключаем определение таймаута при покупке, т.к. ухода в паузу в момент покупки на MacOS не происходит
   responceTimeout = NativeUtils.IsiOSAppOnMac() ? 0 : ClientConfiguration.BillingResponseTimeout;

   billingService.SetBillingResponseTimeout(responceTimeout);
   ```
   
6. В методе обработки апгрейда зданий есть проверка каторая проверяет что завершён именно апгрейд здания. Дополним её комментарием:
   ```
   // Проверяем, принадлежит ли улучшение зданию, и находится ли оно в специальном слоте для повышения уровня здания.
   // Если это не так, дальнейшая обработка не требуется.
   if (upgradeNotification.Upgrade.Owner is not BuildingUpgrade { Slot: UpgradesHelper.BUILDING_LEVEL_UPGRADE_SLOT_ID } upgrade)
   {
      return;
   }
   ```
   
7. В телеграм боте при считывании строк таблицы пропускаем первую строку т.к. там заголовок. Комментарий хорошо это прояснит:
   ```
   // Пропускаем заголовок (первая строка), начинаем со второй строки
   for (int i = worksheet.Dimension.Start.Row + 1; i <= rows; i++)
   ```
   
### 3.2

Комментариев в основном не писал поэтому попробую отрефакторить приведённые примеры из п. 3.1

1. В случе с телеграм ботом из п.7 можно вынести расчёт стартовой строки в переменную и прояснить код без комментария:
   ```
   firstRowAfterTitle = worksheet.Dimension.Start.Row + 1;
   
   for (int i = firstRowAfterTitle; i <= rows; i++)
   ```
   
2. В случае с проверкой апгрейда здания из п.6. Идея избавиться от условных проверок и комментариев, применяя типизацию как инструмент выразительности:
   Добавим новый тип `BuildingSlotUpgrade`, инвертируем проверку и добавим метод обработки:
   ```
   if (upgradeNotification.Upgrade is BuildingSlotUpgrade buildingSlotUpgrade)
   {
      HandleBuildingSlotUpgrade(buildingSlotUpgrade);
   }
   ```
   
3. В примере из п.4 вынесем проверку в переменную с говорящим названием:
   ```
   bool isTargetOnSameSide = 
    MathUtils.isOrientBetween(targetOrient, startToCenterOrient, (startToCenterOrient + tangentOffset) % 360);

   if (isTargetOnSameSide)
   {
      // Обход в ту же сторону, куда двигаемся
   }
   else
   {
      // Обход в противоположную сторону
   }
   ```
   
4. В примере из п.3 разобьём сложное условие на несколько переменных каждая из которых будет ответственная за отдельную часть проверки.
   Результирующую проверку также запишем в переменную, тогда вся проверка будет говорить сама за себя:
   ```
   bool isBetweenCells = unit.CurrentStep != 0;
   bool isNotJumping = !unit.hasFlag(UnitFlags.FLAG_JUMPING);
   bool hasMovedSinceLastResponse = unit.Distance > unit.Undergo;
   bool isTargetingDifferentCell = x2 != unit.X || y2 != unit.Y;
   bool isNearPreviousPosition = unit.Dx < BattleCell.Size && unit.Dx > -BattleCell.Size 
                               && unit.Dy < BattleCell.Size && unit.Dy > -BattleCell.Size;
   
   bool canFastRespond = isBetweenCells && isNotJumping && hasMovedSinceLastResponse && isTargetingDifferentCell && isNearPreviousPosition;
   
   if (canFastRespond)
   {
      unit.FastResponse();
   }
   ```

5. В примере из п.1 применим тот же подход, что и в предыдущим, выделим новый тип `HeroAbilityUpgradeLevel`:
   ```
   if (upgradeNotification.Upgrade is not HeroAbilityUpgradeLevel heroAbilityUpgradeLevel)
   {
      return null;
   }
   ```

   

