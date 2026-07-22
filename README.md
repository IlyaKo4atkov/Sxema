```mermaid
flowchart TD
    Start([Начало]) --> InitADC[adc_init: настройка АЦП (ADMUX, ADCSRA)]
    InitADC --> InitLCD[lcd_init: пины PORTD, команды 0x30×3, 0x28, 0x0C, 0x06, 0x01]
    InitLCD --> MainLoop{while(1)}
    MainLoop --> ReadADC[adc_read(0): выбор канала, запуск ADSC, ожидание, возврат ADC]
    ReadADC --> CalcTemp[Расчёт температуры: temp_x10 = raw * 500 / 1024]
    CalcTemp --> RoundTemp[Округление до целого градуса]
    RoundTemp --> StrConv[int_to_str_temp: разбор на сотни/десятки/единицы, ASCII, буфер]
    StrConv --> SetCursor[lcd_set_cursor(0,0): адрес 0x80 + col]
    SetCursor --> PrintLabel[lcd_print_str("Temp: ")]
    PrintLabel --> PrintVal[lcd_print_str(temp_buf)]
    PrintVal --> PrintUnit[lcd_data(' '); lcd_data('C')]
    PrintUnit --> Delay[_delay_ms(1000)]
    Delay --> MainLoop
    subgraph LcdWriteDetail [lcd_write_4bits]
        SetRS[Установить RS: LCD_DATA → 1, иначе 0]
        SendHigh[Передать старшие 4 бита: маска 0xF0, D4–D7, lcd_pulse_e()]
        SendLow[Передать младшие 4 бита: сдвиг на 4, маска 0xF0, D4–D7, lcd_pulse_e()]
    end
    LcdWriteDetail --> SetRS
    SetRS --> SendHigh
    SendHigh --> SendLow
    subgraph PulseDetail [lcd_pulse_e]
        SetE1[PORTD: E = 1]
        WaitUs[_delay_us(1)]
        SetE0[PORTD: E = 0]
    end
    PulseDetail --> SetE1
    SetE1 --> WaitUs
    WaitUs --> SetE0
    subgraph AdcReadDetail [adc_read]
        SelChan[ADMUX: выбор канала, сохранение старших битов]
        StartConv[ADCSRA: запуск ADSC]
        WaitDone[Ожидание: пока ADSC = 1]
        RetADC[Возврат значения ADC]
    end
    AdcReadDetail --> SelChan
    SelChan --> StartConv
    StartConv --> WaitDone
    WaitDone --> RetADC
    ReadADC -.-> AdcReadDetail
