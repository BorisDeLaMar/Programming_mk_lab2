STM32CubeMX: Чип STM32L152RBT6, PB7 - LD3, PB6 - LD4, PA0 - кнопка USER. В clock config HSI 16 (16 МГц). 
В project manager: Toolchain/IDE - MDK-ARM, чтобы через Keil uVision прогать, а не STMCube IDE. 
Далее generate code Keil uVision5: Flash - Configure flash tools - Debug. Здесь, при подключенном контроллере, где Use:ST-Link Debugger, + зайди в Settings. В SWDIO дожно быть Device name контроллера + во вкладке Flash Downloads галочку у Reset and Run. 
Если компилится с ошибкой, то чек Flash - Configure flash tools - Target - ARM Compiler (если Missing, то Use default compiler version 6 выбери, если есть, конечно). Компиляция - F7, загрузка на мк - F8.
NB Иногда при регенерации проекта через STM (с добавлением новых подключенных у-в/пинов, например) возникают библиотечные ошибки в uVision. Тогда пересоздай еще раз или помашни еще как-нибудь.
Вообще перед generate code в STM лучше просто uVision закрыть.

Добавление таймера.
STM: на PB7 ставим TM4_CH2. Далее в categories слева Timers->TIM4 и настраиваем ШИМ через таймер. Channel2 - PWM Generation CH2, все остальное Disable оставь. 
Во Configuration этого TIM4: Prescaler = 16000 (частота 16 МГц слишком большая, счетчик слишком быстро досчитает до предела), 
Counter Period = 1000 (досчитал до 1000=1с). Т.е. было 16 М раз в секунду, теперь 1000 раз (16М/16000/1000). Pulse = 500 (сколько длится сама инкремента). 
Так ШИМ и формируем: counter period - сколько длится "ноль" (маленькое напряжение, почти нулевое), pulse - "единица" (большое напряжение, лампочка горит).
NB Теперь LD3 горит без команд бесконечного цикла, менять ШИМ можно как в STMCubeMX и generate проекта каждый раз, так и в uVision в функции MX_TIM4_Init, которая ниже main в main.c объявлена.   

Обработчик прерываний:
STM: для кнопки ставим GPIO_EXITI0. Далее в system core->GPIO->NVIC->галочку у Enabled для EXITI line0 interrupt. В NVIC system core можно проверить, что это появилось.
stm32I1xx_it.c - файл со всеми обработчиками прерываний. Находим наш, пишем обработчик.
Так мы включаем LD4 по кнопке с УЧЕТОМ ДРЕБЕЗГА. Дребезг возникает при нажатии на кнопку, она не сразу включается полноценно. Поэтому, как только она в "первый раз" включилась(дребезг пошел), мы сразу через обработчик включаем диод. Не нужно ждать пока пройдет дребезжание и диод согласован с кнопкой. Включил - включился

Таймер без привязки к пину:
STM: TIM2->clock source->internal clock. Далее настравиваем prescaler, period. NVIC settings->Enabled. Тоже чекай по sys core->NVIC

https://www.st.com/en/evaluation-tools/32l152cdiscovery.html#cad-resources
