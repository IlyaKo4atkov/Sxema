```mermaid
flowchart TD
    Start([Старт]) --> Init[Инициализация]
    Init --> AdcInit[adc_init:<br/>ADMUX=AVCC, ADCSRA=АЦП+делитель 128]
    AdcInit --> LcdInit[lcd_init:<br/>DDRD=выходы, задержка 50 мс,<br/>0x30 x3, 0x28, 0x0C, 0x06, 0x01]
    LcdInit --> Loop{while(1)}
    Loop --> Read[Чтение ADC0:<br/>выбор канала, запуск, ожидание, возврат ADC]
    Read --> Calc[Расчёт температуры:<br/>temp_x10 = raw*500/1024,<br/>округление до целого]
    Calc --> ToStr[int_to_str_temp:<br/>разбор на разряды,<br/>перевод в ASCII,<br/>запись в буфер]
    ToStr --> SetPos[lcd_set_cursor(0,0):<br/>адрес 0x80+col]
    SetPos --> PrintHead[lcd_print_str("Temp: ")]
    PrintHead --> PrintVal[lcd_print_str(temp_buf)]
    PrintVal --> PrintUnit[lcd_data(' '); lcd_data('C')]
    PrintUnit --> Delay[_delay_ms(1000)]
    Delay --> Loop
    subgraph LcdFuncs [Вспомогательные функции LCD]
        Cmd[lcd_cmd(cmd):<br/>lcd_write_4bits(cmd, LCD_CMD),<br/>_delay_ms(2)]
        Data[lcd_data(data):<br/>lcd_write_4bits(data, LCD_DATA),<br/>_delay_ms(1)]
        Write4[lcd_write_4bits:<br/>установка RS,<br/>старшие 4 бита + pulse_e,<br/>младшие 4 бита + pulse_e]
        Pulse[lcd_pulse_e:<br/>E=1 → задержка 1 мкс → E=0]
    end
    Cmd -.-> Write4
    Data -.-> Write4
    Write4 -.-> Pulse
    subgraph AdcFuncs [Вспомогательные функции АЦП]
        AdcRead[adc_read(channel):<br/>выбор канала в ADMUX,<br/>запуск ADSC,<br/>ожидание ADSC=0,<br/>возврат ADC]
    end
    Read -.-> AdcRead
