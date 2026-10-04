## MPRemoteCommandCenter

MPRemoteCommandCenter — это объект, который реагирует на события удалённого управления, отправляемые внешними аксессуарами и системными элементами управления. Используйте метод sharedCommandCenter, чтобы получить его синглтон, а не создавайте экземпляр самостоятельно. Он предоставляет набор свойств типа MPRemoteCommand для обработки различных событий удалённого управления, которые можно настроить под свои нужды.

### Способ использования

#### 1. Включение команд

```objectivec
MPRemoteCommandCenter *commandCenter = [MPRemoteCommandCenter sharedCommandCenter];
// Команды воспроизведения, паузы, предыдущего и следующего трека по умолчанию включены, то есть enabled по умолчанию равен YES
commandCenter.playCommand.enabled = enable;
commandCenter.pauseCommand.enabled = enable;
commandCenter.previousTrackCommand.enabled = enable;
commandCenter.nextTrackCommand.enabled = enable;
// Включить команду воспроизведения/паузы для наушников (команда, которую вызывает кнопка воспроизведения на наушниках)
commandCenter.togglePlayPauseCommand.enabled = enable;
// Режим воспроизведения
commandCenter.changeRepeatModeCommand.enabled = enable;
// Скорость воспроизведения
commandCenter.changePlaybackRateCommand.enabled = enable;
// Перемотка по прогрессу
if (@available(iOS 9.1, *)) {
    commandCenter.changePlaybackPositionCommand.enabled = enable;
}
...
```

#### 2. Подписка на события удалённого управления

```objectivec
[commandCenter.playCommand addTarget:self action:@selector(play)];
[commandCenter.pauseCommand addTarget:self action:@selector(pause)];
if (@available(iOS 9.1, *)) {
    [commandCenter.changePlaybackPositionCommand addTarget:self action:@selector(changePlaybackPosition:)];
}

// Также можно использовать метод addTargetWithHandler:
[commandCenter.playCommand addTargetWithHandler:^MPRemoteCommandHandlerStatus(MPRemoteCommandEvent * _Nonnull event) {
    if (self.player.rate == 0.0) {
        [self.player play];
        return MPRemoteCommandHandlerStatusSuccess;
    }
    return MPRemoteCommandHandlerStatusCommandFailed;
}];
```

#### 3. Обработка событий

```objectivec
- (MPRemoteCommandHandlerStatus)play {
    // play audio
    return MPRemoteCommandHandlerStatusSuccess;
}

- (MPRemoteCommandHandlerStatus)pauseAction {
    // pause audio
    return MPRemoteCommandHandlerStatusSuccess;
}

- (MPRemoteCommandHandlerStatus)changePlaybackPositionCommand:(MPChangePlaybackPositionCommandEvent *)event {
    // change playback position
    // event.positionTime: позиция прогресса после перемотки, в секундах
    return MPRemoteCommandHandlerStatusSuccess;
}
```

#### 4. Удаление подписки на события удалённого управления

```objectivec
[commandCenter.playCommand removeTarget:self];
[commandCenter.pauseCommand removeTarget:self];
if (@available(iOS 9.1, *)) {
    [commandCenter.changePlaybackPositionCommand removeTarget:self];
}
```

### MPRemoteCommand

Объект, который реагирует на события удалённых команд.

Если вы явно не хотите включать определённую команду, получите объект команды и установите его свойство enable в NO. Отключение удалённой команды сообщает системе, что, когда ваше приложение является приложением «Сейчас играет», система не должна отображать для этой команды никакой связанной UI.
Фреймворк MediaPlayer определяет множество подклассов MPRemoteCommand для обработки команд определённых типов. Иногда эти подклассы позволяют указать дополнительную информацию, связанную с командой. Например, команда обратной связи MPFeedbackCommand позволяет задать локализованную строку, описывающую смысл обратной связи. Поддерживая конкретную команду, обязательно изучите соответствующий класс, который обрабатывает такие события.

```objectivec
typedef NS_ENUM(NSInteger, MPRemoteCommandHandlerStatus) {
    /// Команда выполнена успешно
    MPRemoteCommandHandlerStatusSuccess = 0,
    /// Команду нельзя выполнить, потому что запрошенный контент не существует в текущем состоянии приложения
    MPRemoteCommandHandlerStatusNoSuchContent = 100,
    /// Команду нельзя выполнить, потому что сейчас нет доступного элемента воспроизведения, необходимого для этой команды. Например, если приложение получает команду «включить языковую опцию», но в данный момент ничего не воспроизводится, будет возвращён этот код ошибки
    MPRemoteCommandHandlerStatusNoActionableNowPlayingItem MP_API(ios(9.1), macos(10.12.2)) = 110,
    /// Команду нельзя выполнить, потому что требуемое устройство недоступно. Например, если нужно надеть наушники или приложение для часов понимает, что для выполнения запроса ему нужен компаньон
    MPRemoteCommandHandlerStatusDeviceNotFound MP_API(ios(11.0), macos(10.13)) = 120,
    /// Команда не может быть выполнена по другой причине
    MPRemoteCommandHandlerStatusCommandFailed = 200
};

@interface MPRemoteCommand : NSObject

@property (nonatomic, assign, getter = isEnabled) BOOL enabled;

// Добавление обработки событий через Target-action style: action принимает экземпляр MPRemoteCommandEvent в качестве первого параметра. addTarget:action: не удерживает (retain) target, поэтому при освобождении target следует вызвать removeTarget:, чтобы его удалить

// selector должен возвращать значение MPRemoteCommandHandlerStatus, чтобы система могла корректно реагировать на команды, которые могут оказаться невыполнимыми в зависимости от текущего состояния приложения.
- (void)addTarget:(id)target action:(SEL)action;
- (void)removeTarget:(id)target action:(nullable SEL)action;
- (void)removeTarget:(nullable id)target;

/// Returns an opaque object to act as the target.
- (id)addTargetWithHandler:(MPRemoteCommandHandlerStatus(^)(MPRemoteCommandEvent *event))handler;

@end
```

Пример использования:

```objectivec
// Get the shared command center.
MPRemoteCommandCenter *commandCenter = [MPRemoteCommandCenter sharedCommandCenter];

// Add a handler for the play command.
[commandCenter.playCommand addTargetWithHandler:^MPRemoteCommandHandlerStatus(MPRemoteCommandEvent * _Nonnull event) {
    if (self.player.rate == 0.0) {
        [self.player play];
        return MPRemoteCommandHandlerStatusSuccess;
    }
    return MPRemoteCommandHandlerStatusCommandFailed;
}];
```

### Связанные команды

```objectivec
// Playback Commands
@property (nonatomic, readonly) MPRemoteCommand *pauseCommand; // Приостановить воспроизведение
@property (nonatomic, readonly) MPRemoteCommand *playCommand;  // Начать воспроизведение
@property (nonatomic, readonly) MPRemoteCommand *stopCommand;  // Остановить воспроизведение
@property (nonatomic, readonly) MPRemoteCommand *togglePlayPauseCommand; // Объект команды для переключения между воспроизведением и паузой текущего элемента, используется для наушников и т. п.
@property (nonatomic, readonly) MPRemoteCommand *enableLanguageOptionCommand MP_API(ios(9.0), macos(10.12.2)); // Включить языковую опцию
@property (nonatomic, readonly) MPRemoteCommand *disableLanguageOptionCommand MP_API(ios(9.0), macos(10.12.2)); // Отключить языковую опцию
@property (nonatomic, readonly) MPChangePlaybackRateCommand *changePlaybackRateCommand; // Скорость воспроизведения
@property (nonatomic, readonly) MPChangeRepeatModeCommand *changeRepeatModeCommand;     // Режим повтора (повтор одного трека / повтор по порядку)
@property (nonatomic, readonly) MPChangeShuffleModeCommand *changeShuffleModeCommand;   // Режим случайного воспроизведения

// Previous/Next Track Commands
@property (nonatomic, readonly) MPRemoteCommand *nextTrackCommand; 		 // Следующий трек
@property (nonatomic, readonly) MPRemoteCommand *previousTrackCommand; // Предыдущий трек

// Skip Interval Commands
@property (nonatomic, readonly) MPSkipIntervalCommand *skipForwardCommand;  // Перемотка вперёд
@property (nonatomic, readonly) MPSkipIntervalCommand *skipBackwardCommand; // Перемотка назад

// Seek Commands
@property (nonatomic, readonly) MPRemoteCommand *seekForwardCommand;
@property (nonatomic, readonly) MPRemoteCommand *seekBackwardCommand;
@property (nonatomic, readonly) MPChangePlaybackPositionCommand *changePlaybackPositionCommand MP_API(ios(9.1), macos(10.12.2)); // Перемотка по прогрессу

// Rating Command
@property (nonatomic, readonly) MPRatingCommand *ratingCommand; // Оценка

// Feedback Commands
// These are generalized to three distinct actions. Your application can provide
// additional context about these actions with the localizedTitle property in
// MPFeedbackCommand.
@property (nonatomic, readonly) MPFeedbackCommand *likeCommand;     // Нравится
@property (nonatomic, readonly) MPFeedbackCommand *dislikeCommand;  // Не нравится
@property (nonatomic, readonly) MPFeedbackCommand *bookmarkCommand; // Добавить закладку
```

### Справочные материалы

* [Apple Developer Documentation｜MPRemoteCommandCenter](https://developer.apple.com/documentation/mediaplayer/mpremotecommandcenter?language=objc)
