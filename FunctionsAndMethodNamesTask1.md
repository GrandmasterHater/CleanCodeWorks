Add - AddEntry - метод используется в ScheduleFacade для добавления записи в расписание, принимает тип ScheduleEntry.

Get - GetEntry - метод используется в ScheduleFacade для получения информации о записи в расписании и возвращает объект типа ScheduleEntry.

Remove - RemoveEntry - метод используется в ScheduleFacade для удаления информации о записи в расписании.

Update - UpdateEntry - метод используется в ScheduleFacade для обновления записи в расписании.

StartBot - Start - метод запуска бота, принадлежит объекту бота.

StopBot - Stop - метод остановки бота, принадлежит объекту бота.

NoCacheString - AddNoCacheParamToUrl - функция для модификации URL добавлением параметра NoCache.

GetParams - GetParamsFromUrl - функция для извлечения списка параметров из URL.

IsValid - IsChecksumEqual - метод проверяет совпадение контрольной суммы кэшированного и загружаемого ресурса. 
Является частью объекта валидатора ResourceMetadata.

ProcessBullets - UpdateBulletsStates - пересчитывает состояние всех пуль на поле боя. Метод находится в контексте калькулятора состояния боя.

InitSnapshot - InitFromSnapshot - метод корневого объекта боя, который инициализирует объект боя данными из снапшота. 
Вызывается когда игрок входит в бой из которого вылетел, например при переподключении после восстановления соединения.

UpdateEnergyRegen - StartRechargeCycle - метод находится в модели игрового ресурса и запускает цикл регенерации энергии (игровой ресурс для покупки героев).
