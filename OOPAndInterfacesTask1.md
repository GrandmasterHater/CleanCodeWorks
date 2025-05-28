### 3.1 

ScheduleEntry.ForDateTime(GUID id, DateTime dateTime) - создаёт объект ScheduleEntry для указанных даты и времени.
ScheduleEntry.ForDateTimeRecurring(GUID id, DateTime dateTime, RecurringTypesEnum recurringTypes) - создаёт объект ScheduleEntry для указанных даты, времени и указанным периодом повторения.

ScheduleBot.WithLogger(ILogger logger) - создаёт бота с указанным логгером.

PlayerProfile.WithPremium(PremiumSettings premiumSettings) - создаёт профиль игрока с указанными настройками премиума.

### 3.2

IScheduleFacade - ScheduleFacade и реализация ScheduleFacadeImpl
// Фасад для работы с расписанием. Предоставляет единую точку входа для взаимодействия с расписанием.

IWaitingWindowDispatcher - WaitingWindowDispatcher и реализация WaitingWindowDispatcherImpl
// Интерфейс, который предоставляет единую точку входа для взаимодействия с окном ожидания.

IBullet - Bullet и реализация BallisticBullet
// Интерфейс, который определяет базовое представление всех видов пуль (снарядов) в игре.