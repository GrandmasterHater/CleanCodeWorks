1. Замена:
   ```
      if (channelPost.Type != MessageType.Text && channelPost.Type != MessageType.Document)
      {
          return;
      }
   ```
   на:
   ```
      bool isTextOrDocument = channelPost.Type == MessageType.Text || channelPost.Type == MessageType.Document
      
      if (!isTextOrDocument)
      {
          return;
      }
   ```
   Используется при обработке сообщения в канале с телеграм ботом. Требуется обрабатывать только документы или текст, все остальные типы сообщений игнорировать. 
   Вынос условия в переменную лучше читается, а также его можно будет дополнять другими типами сообщений.

2. В коде, запрашивающем отображение обнаруженного юнита на миникарте:
   ```
      if (visibilityStatePrev == UnitVisibilityState.Hidden
         && _currentVisibilityState == UnitVisibilityeState.Detected
         && BattleEntityUtils.IsOwnSide(_unitSide))
      {
         Minimap.ShowUnitAsDetected(this);
      }
   
      if (BattleEntityUtils.IsOwnSide(_unitSide))
      {
          SoundEffects.PlayUnitDetected();
      }
   ```

   разделим проверки на переход в видимое состояние и проверку стороны юнита и выделим в переменные.

   ```
      bool isUnitBecameVisible = visibilityStatePrev == UnitVisibilityState.Hidden && _currentVisibilityState == UnitVisibilityeState.Detected;
      bool isOwnSideUnit = BattleEntityUtils.IsOwnSide(_unitSide);
   
      if (isUnitBecameVisible && isOwnSideUnit)
      {
         Minimap.ShowUnitAsDetected(this);
      }
   
      if (isOwnSideUnit)
      {
          SoundEffects.PlayUnitDetected();
      }
   ```
   
3. Замена: `await botClient.SendTextMessageAsync(channelPost.Chat.Id, "Произошла ошибка при чтении файла. Расписание не было обновлено.");`
   на: 
   ```
      string errorMessage = Localization.GetString(FileReadErrorId);
      
      await botClient.SendTextMessageAsync(channelPost.Chat.Id, errorMessage);
   ```
   
   Используется при обработке ошибки чтения файла с расписанием, который передали боту в чате.
   Задействуем локализацию сразу чтобы работа со строками была универсальна и не требовалось много усилий для расширения локализации.

4. К: `float interpolationFactor = LogicThread.LogicStepTimeMs / durationNext;`
   добавляем проверку:
   ```
   if (nextFrameDurationMs <= 0.0f)
   {
      throw new ArgumentOutOfRangeException("Duration must be greater than zero");
   }
   
   float interpolationFactor = LogicThread.LogicStepTimeMs / nextFrameDurationMs;
   
   interpolationFactor = Mathf.Clamp01(interpolationFactor);
   ```
   
   Используется при расчёте позиции пули на поле боя. Переименована переменная durationNext в nextFrameDurationMs. Добавлена проверка на то, что nextFrameDuration должен быть больше определённого минимального значения.
   Также добавлена проверка ограничивающая диапазон интерполяции от 0 до 1.

5. Есть код регистрации и обработки строковых команд ботом в чате:
   ```
      var commands = new List<BotCommand>
      {
          new BotCommand { Command = "/schedule_bot_run", Description = "Запуск бота на канале" },
          new BotCommand { Command = "/schedule_bot_stop", Description = "Остановка бота на канале" },
          new BotCommand { Command = "/schedule_bot_update_xlsx", Description = "Обновление файла с расписанием. Файл должен быть прикреплён к команде!" }
      };
   
      ...
   
      switch (commandString)
      {
          case "/schedule_bot_run":
              HandleRunCommand(channelPost);
              break;
          
          case "/schedule_bot_stop" :
              HandleStopCommand(channelPost);
              break;
          
          case "/schedule_bot_update_xlsx" :
              await HandleUpdateCommand(botClient, channelPost, _stopBotCts!.Token);
              break;
          
          default:
              await HandleUnknownCommand(botClient, channelPost, message);
              break;
      }
   ```       
   
   Выделим класс для представления команды для бота:

   ```
   public static class ScheduleBotCommands
   {
      public const string RUN_COMMAND = "/schedule_bot_run";
      public const string STOP_COMMAND = "/schedule_bot_stop";
      public const string UPDATEXLSX_COMMAND = "/schedule_bot_update_xlsx";
   }
   ```
   
   И получим:

   ```
   var commands = new List<BotCommand>
   {
       new BotCommand { Command = ScheduleBotCommands.RUN_COMMAND, Description = "Запуск бота на канале" },
       new BotCommand { Command = ScheduleBotCommands.STOP_COMMAND, Description = "Остановка бота на канале" },
       new BotCommand { Command = ScheduleBotCommands.UPDATEXLSX_COMMAND, Description = "Обновление файла с расписанием. Файл должен быть прикреплён к команде!" }
   };
   
   ...
   
   switch (message)
        {
            case ScheduleBotCommands.RUN_COMMAND:
                HandleRunCommand(channelPost);
                break;
            
            case ScheduleBotCommands.STOP_COMMAND:
                HandleStopCommand(channelPost);
                break;
            
            case ScheduleBotCommands.UPDATEXLSX_COMMAND:
                await HandleUpdateCommand(botClient, channelPost, _stopBotCts!.Token);
                break;
            
            default:
                await HandleUnknownCommand(botClient, channelPost, message);
                break;
        }
   ```
   
   В итоге работа с командами не будет зависеть от написания команды, сама команда может быть легко изменена.

6. В коде из примера 5 добавим работу с локализацией. Есть набор команд бота:

   ```
   var commands = new List<BotCommand>
   {
       new BotCommand { Command = ScheduleBotCommands.RUN_COMMAND, Description = "Запуск бота на канале" },
       new BotCommand { Command = ScheduleBotCommands.STOP_COMMAND, Description = "Остановка бота на канале" },
       new BotCommand { Command = ScheduleBotCommands.UPDATEXLSX_COMMAND, Description = "Обновление файла с расписанием. Файл должен быть прикреплён к команде!" }
   };
   ```
   
   Локализуем описание команды:

   ```
   string startCommandDescription = Localization.GetString(StartCommandDescriptionId);
   string stopCommandDescription = Localization.GetString(StopCommandDescriptionId);
   string updateCommandDescription = Localization.GetString(UpdateCommandDescriptionId);
   
   var commands = new List<BotCommand>
   {
       new BotCommand { Command = ScheduleBotCommands.RUN_COMMAND, Description = startCommandDescription },
       new BotCommand { Command = ScheduleBotCommands.STOP_COMMAND, Description = stopCommandDescription },
       new BotCommand { Command = ScheduleBotCommands.UPDATEXLSX_COMMAND, Description = updateCommandDescription }
   };
   ```
   
   Теперь сообщения могут быть представлены на нескольких языках.

7. При определении доступной позиции застройки сравниваются радиусы застройки зданий. Выполняем сравнение через дельту:

   ```
   const float RADIUS_EPSILON = 0.001f;
   
   if (Mathf.Abs(nearestBuildingRadiusCells - constructingBuildingRadiusCells) <= RADIUS_EPSILON)
   {
      ShowUnavailableCell(constructingBuilding);
   }
   ``` 
   
8. Есть свойство представляющее нормализованное значение юнита (0 до 1):
   ```
    public float NormalizedHealth
    {
        get
       	{
       		float normalizedHealth = _health / (float)_maxHealthForCurrentType;
       		return normalizedHealth;
       	}
    }
   ```
   Для того чтобы отбросить погрешности вычислений ограничиваем выходное значение 0 до 1. Остальные проверки не требуются т.к. изменения здоровья и валидация конфигурации происходят в соответствующих методах.
   ```
   public float NormalizedHealth
   {
      get
      {
   if (_maxHealthForCurrentType <= 0)
        {
            // Предотвращаем деление на 0 и нелогичные значения
            return 0f;
        }
   
         float normalizedHealth = _health / (float)_maxHealthForCurrentType;
         normalizedHealth = Mathf.Clamp01(normalizedHealth);
         return normalizedHealth;
      }
   }
   ```
   
9. Метод делает рейкаст на поверхность и находит координаты на поверхности:

   ```
   public static bool RaycastDownOnSurfaceFast(float x, float z, out SurfaceRaycastHit surfaceHit)
    {
    	x = (float)Math.Round(x, 3);
    	z = (float)Math.Round(z, 3);
    	return GetSurfaceRaycaster(x, z).Raycast(x, z, out surfaceHit);
    }
   ```
   Удаляем лишние преобразования типов при округлении значения после точки задействовав класс MathF вместо Math.
   ```
   public static bool RaycastDownOnSurfaceFast(float x, float z, out SurfaceRaycastHit surfaceHit)
    {
    	x = MathF.Round(x, 3);
    	z = MathF.Round(z, 3);
    	return GetSurfaceRaycaster(x, z).Raycast(x, z, out surfaceHit);
    }
   ```
   
10. Утилитарный метод, рассчитывает относительную дельту во времени на основе тиков. Используется при расчёте поворотов юнитов и движения камеры.
   ```
   public static float CalcDeltaTime(int currentTick, int startTick, int finishTick) =>
			(currentTick - startTick) / (float)(finishTick - startTick);
   ```
   Переименован метод для обозначения того, что это коэфициент интерполяции. Также добавлена проверка входных значений.
   ```
   public static float CalcTimeFactor(int currentTick, int startTick, int finishTick)
   {
      if (currentTick < startTick || currentTick > finishTick)
      {
          throw new ArgumentOutOfRangeException("Current tick must be between start and finish ticks");
      }
   
      if (startTick >= finishTick)
      {
          throw new ArgumentOutOfRangeException("Start tick must be less than finish tick");
      }

      return (currentTick - startTick) / (float)(finishTick - startTick);
   }
   ```
 
11. Условие проверяет может ли пользователь получать уведомления.
    `if (person.IsActive && !person.IsBanned && person.LastLoginDate > DateTime.UtcNow.FindMonthAgo())`

   Вынесем условие в переменную для улучшения читаемости кода.
   ```
   bool isEligibleForSendMessage = person.IsActive && !person.IsBanned && person.LastLoginDate > DateTime.UtcNow.FindMonthAgo();

   if (isEligibleForSendMessage)
   {
      ...
   }
   ```
   
12. Проверка можно ли пользователю показать админ панель.
   ```
   if (user != null && user.IsAdmin && !user.IsBanned && user.LastActivity > DateTime.UtcNow.FindWeekAgo())
   {
       ShowAdminPanel(user);
   }
   ```

   Условие составное и достаточно сложное, вынесем его отдельные части в переменные и скомбинируем их. 

   ```
   bool isAdminPanelAvailable = user != null && user.IsAdmin && !user.IsBanned && user.LastActivity > DateTime.UtcNow.FindWeekAgo();
   
   if (isAdminPanelAvailable)
   {
       ShowAdminPanel(user);
   }
   ```
