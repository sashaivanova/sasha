---
title: Обзор урока
actions: ['Проверить', 'Подсказать']
skipCheckAnswer: true
material:
  saveZombie: false
  zombieResult:
    hideNameField: true
    ignoreZombieCache: true
    answer: 1
---

В Уроке 1 вы выстроили «Фабрику Зомби» с целью создать зомби-армию. 

* Фабрика будет содержать данные всех зомби в армии
* У фабрики будет функция создания новых зомби
* У каждого зомби будет случайный уникальный внешний вид

В следующих уроках мы добавим больше функционала, например, сделаем так, чтобы зомби мог нападать на людей и других зомби! Но пока мы будем добираться до этого момента, напишем базовый функционал создания новых зомби. 

## Как устроена ДНК зомби 

Внешний вид зомби обусловлен его ДНК. ДНК зомби — простое целое число из 16 цифр, например: 

```
8356281049284737
```

Как и в настоящей ДНК, различные части этого числа будут отражать специфические черты. Первые две цифры определяют внешний вид головы зомби, следующие две — разрез глаз и так далее. 

> Обрати внимание: это сильно упрощенный урок, поэтому у зомби возможно только 7 разных типов голов (хотя из 2 цифр можно получить 100 возможных вариантов). Если мы захотим увеличить число вариантов зомби, . бы хотели увеличить количество вариаций зомби.для Note: For this tutorial, we've kept things simple, and our zombies can have only 7 different types of heads (even though 2 digits allow 100 possible options). Later on we could add more head types if we wanted to increase the number of zombie variations.

Например, первые 2 цифры зомби-ДНК, приведенной выше, равны `83`. Чтобы определить тип головы зомби, мы выполняем операцию `83 % 7 + 1` = 7. Этот зомби получит седьмой тип головы.  For example, the first 2 digits of our example DNA above are `83`. To map that to the zombie's head type, we do `83 % 7 + 1` = 7. So this Zombie would have the 7th zombie head type. 

Чтобы увидеть, какой черте соответствует `83`, в правой панели сдвинь слайдер `head gene` (ген головы) до типа 7 (шапка Санта Клауса)до  In the panel to the right, go ahead and move the `head gene` slider to the 7th head (the Santa hat) to see what trait the `83` would correspond to.

# Попробуй!

1. Поиграй со слайдером в праой части страницы и увидишь, как различные цифровые комбинации соответсвуют разным аспектам внешнего вида зомби. Play with the sliders on the right side of the page. Experiment to see how the different numerical values correspond to different aspects of the zombie's appearance.

Ладно, поигрались и хватит. Когда готов, нажми на кнопку «Следующая глава» внизу страницы, и погрузись в изучение Solidity целиком! Ok, enough playing around. When you're ready to continue, hit "Next Chapter" below, and let's dive into learning Solidity!
