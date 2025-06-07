1. Есть метод расчёта состояний аур юнитов. Поскольку ауры редко используются с обращением через индекс тут отлично подойдёт связный список.
   Было:
   ```
   public void UpdateBattleAuras(Battle battle)
   {
       List<Aura> activeAuras = battle.Auras;

       for (int index = 0; index < activeAuras.Count; index++)
       {
           ... // Рассчёт состояний и удаление аур, завершивших своё действие
       }
   }
   ```
   Стало:
   ```
   public void UpdateBattleAuras(Battle battle)
   {
       LinkedList<Aura> activeAuras = battle.Auras; // При условии что Auras - LinkedList

       for (var node = activeAuras.First; node != null; node = node.Next)
       {
           ... // Рассчёт состояний и удаление аур, завершивших своё действие
       }
   }
   ```
   
2. Есть метод отправки нативных уведомлений на устройство. Избавимся от цикла преобразовав операции в Linq выражение:
   Было:
   ```
   private void SendNotificationsPackage(List<InGameActivityNotification> notifications)
   {
   	   List<NativeNotification> nativeNotifications = new List<NativeNotification>();
         
   	   foreach (InGameActivityNotification inGameNotification in notifications)
   	   {
   	   	   foreach (ActivityNotificationInfo notification in inGameNotification.NotificationsSchedule)
   	   	   {
   	   	   	   GGNotification localNotificationItem = GetLocalNotificationItem(notification);
   	   	   	   nativeNotifications.AddIfNotNull(localNotificationItem);
   	   	   }
   	   }
         
   	   LocalNotificationsHelper.SendNotificationsPackage(nativeNotifications);
   }
   ```
   Стало:
   ```
   private void SendNotificationsPackage(List<InGameActivityNotification> notifications)
   {
   	   List<NativeNotification> nativeNotifications = notifications
           .SelectMany(n => n.NotificationsSchedule)
           .Select(GetLocalNotificationItem)
           .Where(item => item != null)
           .ToList();

       LocalNotificationsHelper.SendNotificationsPackage(nativeNotifications);
   }
   ```
   
3. Есть метод, который удаляет все специальные символы из строки, заменая их пустыми строками. Можно упростить этот метод используя регулярные выражения.
   Было:
   ```
   private string RemoveSpecSymbols([NotNull] string message)
   {
        StringBuilder messageBuilder = new StringBuilder(message.Length);
        bool startSkipSymbols = false;

		for (int index = 0; index < message.Length; index++)
		{
			char symbol = message[index];

			if (symbol == '<')
			{
				startSkipSymbols = true;
			}
			else if (symbol == '>')
			{
				startSkipSymbols = false;
				continue;
			}

			if (startSkipSymbols == false)
			{
				messageBuilder.Append(symbol);
			}
		}

		return messageBuilder.ToString();
   }
   ```
   Стало:
   ```
   private string RemoveSpecSymbols([NotNull] string message)
   {   
       return Regex.Replace(message, "<.*?>", string.Empty);
   }
   ```

4. Есть метод отмены квеста. Он реализован через обычный цикл for. В данном случае можно использовать foreach, чтобы избежать работы с индексами:
   Было:
   ```
   public void CancelQuest()
   {
       for (int index = 0; index < activeQuests.Count; index++)
       {
           ... // Логика отмены квеста
       }
   }
   ```
   Стало:
   ```
   // Удаление квеста из списка не предполагается ни в одном из вариантов. Тут выполняются проверки и отправка команд на удаление.
   public void CancelQuest()
   {
       foreach (ActiveQuest activeQuest in activeQuests)
       {
           ... // Логика отмены квеста
       }
   }
   ```
   
5. Есть метод обработки команд UI интерфейса, который аккумулирует команды в массив команд. За один кадр исполняется 1 команда.
   Данные считываются в порядке получения. Для такой задачи лучше всего подойдёт очередь, поскольку такая структура данных не будет вызывать лишних 
   сдвигов при удалении и рассчитана на последовательную обработку. Сменим `UiCommand[]` на `Queue<UiCommand>`:
   Было:
   ```
   public void AddCommand(UiCommand command)
   {
       ... // Валидация команды
   
       _commands.Add(command);
   }
   
   public void ExecuteNextCommand()
   {
       UiCommand command = _commands.FirstOrDefault()
   
       ... // Логика исполнения команды
   }
   ```
   Стало:
   ```
   public void AddCommand(UiCommand command)
   {
       ... // Валидация команды
   
       _commands.Enqueue(command);
   }
   
   public void ExecuteNextCommand()
   {
       UiCommand command = _commands.Dequeue();
   
       ... // Логика исполнения команды
   }
   ```