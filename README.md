```mermaid
flowchart TD
    Start([Начало]) --> InitADC[adc_init():<br/>настройка АЦП (ADMUX, ADCSRA)]
    InitADC --> InitLCD[lcd_init():<br/>настройка пинов PORTD,<br/>последовательность команд 0x30×3,<br/>0x28, 0x0C, 0x06, 0x01]
    InitLCD --> MainLoop{while(1)}

    MainLoop --> ReadADC[adc_read(0):<br/>выбор канала в ADMUX,<br/>запуск ADSC,<br/>ожидание завершения,<br/>возврат значения ADC]
    ReadADC --> CalcTemp[Расчёт температуры:<br/>temp_x10 = raw * 500 / 1024]
    CalcTemp --> RoundTemp[Округление до целого градуса]
    RoundTemp --> StrConv[int_to_str_temp():<br/>разбор на сотни/десятки/единицы,<br/>перевод в ASCII,<br/>запись в буфер]
    StrConv --> SetCursor[lcd_set_cursor(0,0):<br/>вычисление адреса (0x80 + col)]
    SetCursor --> PrintLabel[lcd_print_str("Temp: ")]
    PrintLabel --> PrintVal[lcd_print_str(temp_buf)]
    PrintVal --> PrintUnit[lcd_data(' '); lcd_data('C')]
    PrintUnit --> Delay[_delay_ms(1000)]
    Delay --> MainLoop

    subgraph LcdWriteDetail [lcd_write_4bits(value, mode)]
        SetRS[Установить RS:<br/>LCD_DATA → 1, иначе 0]
        SendHigh[Передать старшие 4 бита:<br/>маска 0xF0,<br/>установка D4–D7,<br/>lcd_pulse_e()]
        SendLow[Передать младшие 4 бита:<br/>сдвиг на 4,<br/>маска 0xF0,<br/>установка D4–D7,<br/>lcd_pulse_e()]
    end
    LcdWriteDetail --> SetRS
    SetRS --> SendHigh
    SendHigh --> SendLow

    subgraph PulseDetail [lcd_pulse_e()]
        SetE1[PORTD: E = 1]
        WaitUs[_delay_us(1)]
        SetE0[PORTD: E = 0]
    end
    PulseDetail --> SetE1
    SetE1 --> WaitUs
    WaitUs --> SetE0

    subgraph AdcReadDetail [adc_read(channel)]
        SelChan[ADMUX: выбор канала,<br/>сохранение старших битов]
        StartConv[ADCSRA: запуск ADSC]
        WaitDone[Ожидание: пока ADSC = 1]
        RetADC[Возврат значения из регистра ADC]
    end
    AdcReadDetail --> SelChan
    SelChan --> StartConv
    StartConv --> WaitDone
    WaitDone --> RetADC
    ReadADC -.-> AdcReadDetail
