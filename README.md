```mermaid
flowchart TD
    Start([НАЧАЛО]) --> Init[Инициализация:<br/>- настройка портов I/O ATmega16<br/>- настройка таймеров<br/>- все светодиоды ВЫКЛ]
    Init --> OpConnect[ОПЕРАТОР:<br/>подключить плату к разъёму «ПЛАТА»<br/>установить напряжение на ИП2: 6.3/10/16/20/25 В]
    OpConnect --> OpPower[ОПЕРАТОР:<br/>включить тумблер «питание стенда»]
    OpPower --> CheckPower{Питание ВКЛ?}
    CheckPower -- НЕТ --> ErrPower[ВКЛ светодиод «системная ошибка»<br/>ожидание устранения]
    ErrPower --> CheckPower
    CheckPower -- ДА --> LedPower[МК: ВКЛ светодиод питания]
    LedPower --> OpVoltSel[ОПЕРАТОР:<br/>выбрать напряжение переключателем «выбор напряжения заряда»]
    OpVoltSel --> CheckVolt{Допустимое значение?}
    CheckVolt -- НЕТ --> ErrVolt[ВКЛ светодиод «системная ошибка»<br/>ожидание исправления]
    ErrVolt --> OpVoltSel
    CheckVolt -- ДА --> OpChgEn[ОПЕРАТОР:<br/>включить тумблер «напряжение заряда»]
    OpChgEn --> BlinkProc[МК:<br/>запустить таймер 1 с<br/>3 мигания светодиода «заряд/разряд»<br/>(подтверждение совместимости)]
    BlinkProc --> WaitStart[Ожидание нажатия кнопки «ПУСК»]
    WaitStart --> CheckStart{Кнопка «ПУСК» нажата?}
    CheckStart -- НЕТ --> WaitStart
    CheckStart -- ДА --> RunProc[МК:<br/>ВКЛ светодиод «заряд/разряд» постоянно<br/>активировать силовые ключи/реле<br/>запуск цикла заряда/разряда]
    RunProc --> Monitor{Процесс завершён?<br/>(по времени / по параметрам)}
    Monitor -- НЕТ --> RunProc
    Monitor -- ДА --> FinishProc[МК:<br/>ВЫКЛ светодиод «заряд/разряд»<br/>ВКЛ светодиод «завершение работы»<br/>отключить силовые цепи]
    FinishProc --> WaitStop[Ожидание нажатия кнопки «СТОП»]
    WaitStop --> CheckStop{Кнопка «СТОП» нажата?}
    CheckStop -- НЕТ --> WaitStop
    CheckStop -- ДА --> LedDoneOff[МК: ВЫКЛ светодиод «завершение работы»]
    LedDoneOff --> OpReset[ОПЕРАТОР:<br/>выключить тумблер «напряжение заряда»<br/>перевести «выбор напряжения» в ОТКЛ]
    OpReset --> CheckReset{Тумблеры в ОТКЛ?}
    CheckReset -- НЕТ --> OpReset
    CheckReset -- ДА --> ResetSys[МК: сброс системы<br/>все светодиоды ВЫКЛ<br/>силовые цепи обесточены]
    ResetSys --> Repeat{Повторить проверку?}
    Repeat -- ДА --> OpConnect
    Repeat -- НЕТ --> End([КОНЕЦ])

